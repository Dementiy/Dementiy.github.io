Мы уже рассмотрели как происходит поиск символьного имени («имени переменной») в лекции посвященной пространству имен и областям видимости. В этой лекции мы поговорим о разрешении имен атрибутов и от том как можно «вмешаться» в поиск атрибута переопределив магические методы `__getattr__` и `__getattribute__`.

Давайте рассмотрим следующий пример, пусть у нас есть класс `Goofy` и мы обращаемся к атрибуту `x` у экзмепляра этого класса:

```python
source = '''
class Goofy:
    pass

g = Goofy()
g.x
'''
```

Для вас не должно стать сюрпризом, что выполнение этого кода завершится исключением `AttributeError`. Но как CPython определил, что атрибута `x` не существует и следует возбудить исключение `AttributeError`? Давайте начнем с того, что определим какая инструкция в байт-коде отвечает за поиск атрибута:

```
import dis
dis.dis(source)
  6          20 LOAD_NAME                1 (g)
             22 LOAD_ATTR                2 (x)
             24 POP_TOP
             26 LOAD_CONST               2 (None)
             28 RETURN_VALUE
```

Инструкция `LOAD_NAME` вам уже должна быть знакома, она помещает значение `g` на стек. Нас интересует инструкция `LOAD_ATTR`:

```c
case TARGET(LOAD_ATTR): {
    PyObject *name = GETITEM(names, oparg);
    PyObject *owner = TOP();
    PyObject *res = PyObject_GetAttr(owner, name);
    Py_DECREF(owner);
    SET_TOP(res);
    if (res == NULL)
        goto error;
    DISPATCH();
}
```

Макрос `GETITEM` возвращает указатель на искомый атрибут (`x`), а `TOP` возвращает указатель на объект, который находится на вершине стека (`g`), на нем и осуществляется поиск. Затем происходит вызов функции `PyObject_GetAttr`, которая определена в [Objects/objects.c](https://github.com/python/cpython/blob/3.8/Objects/object.c#L923):
```c
PyObject *
PyObject_GetAttr(PyObject *v, PyObject *name)
{
    PyTypeObject *tp = Py_TYPE(v);

    if (!PyUnicode_Check(name)) {
        PyErr_Format(PyExc_TypeError,
                     "attribute name must be string, not '%.200s'",
                     name->ob_type->tp_name);
        return NULL;
    }
    if (tp->tp_getattro != NULL)
        return (*tp->tp_getattro)(v, name);
    if (tp->tp_getattr != NULL) {
        const char *name_str = PyUnicode_AsUTF8(name);
        if (name_str == NULL)
            return NULL;
        return (*tp->tp_getattr)(v, (char *)name_str);
    }
    PyErr_Format(PyExc_AttributeError,
                 "'%.50s' object has no attribute '%U'",
                 tp->tp_name, name);
    return NULL;
}
```

Во-первых, мы получаем ссылку на тип объекта (в данном случае `Goofy`), который фактически определяет допустимые операции над объектом. Затем осуществляется проверка какой из методов доступен: `tp_getattro` или `tp_getattr`, последний является [устаревшим](https://docs.python.org/3.8/c-api/typeobj.html#c.PyTypeObject.tp_getattr) и не рекомендуется к использованию. Оба метода, или как их называют - слота, содержат указатель на одну из трех возможных функций: `PyObject_GenericGetAttr`, `slot_tp_getattro` или `slot_tp_getattr_hook`, одна из которых и будет вызвана для выполнения поиска атрибута. Остается выяснить какая именно функция.

Решение о том какую именно функцию следует установить в слот `tp_getattro` происходит во время создания типа (в нашем случае класса `Goofy`), а именно в вызове функции [`type_new`](https://github.com/python/cpython/blob/3.8/Objects/typeobject.c#L2318). В конце вызова `type_new` происходит вызов диспетчера слотов [`fixup_slot_dispatcher`](https://github.com/python/cpython/blob/3.8/Objects/typeobject.c#L2813), который отвечает за инициализацию слотов с учетом переопределенных пользователем магических методов, другими словами, [будет выбрана](https://github.com/python/cpython/blob/3.8/Objects/typeobject.c#L7193) или пользовательская реализация (specific) или реализация по умолчанию (generic). Итак, если пользователь переопределил `__getattr__` или `__getattribute__`, то будет установлен `slot_tp_getattr_hook`, иначе `PyObject_GenericGetAttr`.

!!! note
    Более подробно о создании типов и функции `tp_new` мы будем говорить в лекции посвященной метаклассам.

Давайте начнем с рассмотрения принципов работы функции [`PyObject_GenericGetAttr`](https://github.com/python/cpython/blob/3.8/Objects/object.c#L1326), которая чаще всего вызывается при осуществлении поиска атрибутов:

```c
PyObject *
PyObject_GenericGetAttr(PyObject *obj, PyObject *name)
{
    return _PyObject_GenericGetAttrWithDict(obj, name, NULL, 0);
}

PyObject *
_PyObject_GenericGetAttrWithDict(PyObject *obj, PyObject *name,
                                 PyObject *dict, int suppress)
{
    PyTypeObject *tp = Py_TYPE(obj);
    ...
    descr = _PyType_Lookup(tp, name);                       // (1)

    f = NULL;
    if (descr != NULL) {
        Py_INCREF(descr);
        f = descr->ob_type->tp_descr_get;
        if (f != NULL && PyDescr_IsData(descr)) {           // (2)
            res = f(descr, obj, (PyObject *)obj->ob_type);
            ...
            goto done;
        }
    }

    if (dict == NULL) {                                     // (3)
        /* Inline _PyObject_GetDictPtr */
        dictoffset = tp->tp_dictoffset;
        if (dictoffset != 0) {
            ...
            dictptr = (PyObject **) ((char *)obj + dictoffset);
            dict = *dictptr;
        }
    }
    if (dict != NULL) {                                     // (4)
        Py_INCREF(dict);
        res = PyDict_GetItemWithError(dict, name);
        if (res != NULL) {
            ...
            goto done;
        }
        ...
    }

    if (f != NULL) {                                        // (5)
        res = f(descr, obj, (PyObject *)Py_TYPE(obj));
        ...
        goto done;
    }

    if (descr != NULL) {                                    // (6)
        res = descr;
        descr = NULL;
        goto done;
    }

    if (!suppress) {
        PyErr_Format(PyExc_AttributeError,
                     "'%.50s' object has no attribute '%U'",
                     tp->tp_name, name);
    }
  done:
    ...
    return res;
}
```

1. Поиск атрибута осуществляется сначала в словаре класса, а затем в родительских классах в порядке определяемым [MRO](https://www.python.org/download/releases/2.3/mro/).
2. Если атрибут был найден, то проверяется является ли найденный атрибут дескриптором данных (data descriptior, дескриптор у которого определены методы `__get__` и `__set__`) и если является, то происходит вызов метода `__get__` у дескриптора.
3. Получение смещения для словаря экземпляра класса, а затем по смещению и самого словаря (`f.__dict__`).
4. Если атрибут не был найден в цепочке MRO или не является дескпритором данных, то поиск осуществляется в словаре экзмепляра класса.
5. Если в словаре экземпляра атрибут не найден, но ранее был найден в цепочке MRO, то проверяется является ли найденный атрибут дескриптором не данных (non-data descriptor, дескриптор у которого определен только метод `__get__`) и если является, то вызывается метод `__get__` у дескриптора.
6. Если атрибут не является дескриптором не данных, но был найден в цепочке MRO, то возвращается найденное значение.
7. Атрибут не найден, порождается исключение `AttributeError`.

!!! note
    MRO (Method Resolution Order, порядок разрешения методов) определяет порядок поиска атрибутов и методов в классах предках, если они не обнаружены непосредственно в классе-потомке. Более подробно про MRO мы будем говорить в лекции посвященной наследованию.

Теперь перейдем к функции `slot_tp_getattr_hook`:

```c
static PyObject *
slot_tp_getattro(PyObject *self, PyObject *name)
{
    PyObject *stack[1] = {name};
    return call_method(self, &PyId___getattribute__, stack, 1);
}

static PyObject *
slot_tp_getattr_hook(PyObject *self, PyObject *name)
{
    PyTypeObject *tp = Py_TYPE(self);
    ...
    getattr = _PyType_LookupId(tp, &PyId___getattr__);
    if (getattr == NULL) {
        tp->tp_getattro = slot_tp_getattro;
        return slot_tp_getattro(self, name);
    }
    ...
    getattribute = _PyType_LookupId(tp, &PyId___getattribute__);
    if (getattribute == NULL ||
        (Py_TYPE(getattribute) == &PyWrapperDescr_Type &&
         ((PyWrapperDescrObject *)getattribute)->d_wrapped ==
         (void *)PyObject_GenericGetAttr))
        res = PyObject_GenericGetAttr(self, name);
    else {
        Py_INCREF(getattribute);
        res = call_attribute(self, getattribute, name);
        Py_DECREF(getattribute);
    }
    if (res == NULL && PyErr_ExceptionMatches(PyExc_AttributeError)) {
        PyErr_Clear();
        res = call_attribute(self, getattr, name);
    }
    Py_DECREF(getattr);
    return res;
}
```

Задачей функции `slot_tp_getattr_hook` является определить какой из магических методов `__getattr__` или `__getattribute__` был переопределен пользователем. Сначала проверяется был ли переопределен метод `__getattr__`, если нет, то это означает, что переопределен метод `__getattribute__` и слот подменяется на функцию `slot_tp_getattro`, которая просто вызывает пользовательский метод. В противном случае интерпретатор проверяет был ли переопределен метод `__getattribute__`, так как вызов этого метода происходит [безусловно](https://docs.python.org/3/reference/datamodel.html#object.__getattribute__). Здесь следует иметь ввиду, что все пользовательские типы имеют своим предком [`PyBaseObject_Type`](https://github.com/python/cpython/blob/3.8/Objects/typeobject.c#L4783) (`object`), у которого в слот `tp_getattro` установлена функция `PyObject_GenericGetAttr`, которая и будет реализацией по умолчанию для метода `__getattribute__`. Если в результате вызова `__getattribute__` атрибут не был найден и было порождено исключение `AttributeError`, то будет вызван переопределенный метод `__getattr__`.
