## Пример поиска минимального элемента

Давайте начнем с простого примера, напишем функцию возвращающую минимум от двух аргументов:

```python
def my_min(a: float, b: float) -> float:
    """ Function to get minimum of two arguments

    Returns:
    --------
    The smallest argument.

    Examples:
    ---------
    >>> my_min(0, 1)
    0
    """
    return a if a < b else b
```

!!! note "Докстроки"
    В тройных кавычках записывается строка документации (docstring), в которой описывается назначение функции (класса, модуля), список аргументов, возвращаемое значение, примеры использования функции и т.д. Примеры обычно записываются в виде документированных тестов (doctest), которые можно запустить с помощью модуля `doctest`. Есть различные соглашения по оформелнию строк документации, но единого правила нет. Существует несколько проектов предназначенных для автоматической генерации документации на основе докстрок, например, [Sphinx](http://www.sphinx-doc.org/en/master/).

!!! note "Аннотации функций"
    Аннотации типов (или подсказки типов) в основном предназначены для сообщения о том, какими типами обладают аргументы в функциях и методах. Есть несколько популярных «тайпчекеров», среди которых наиболее популярны [mypy](http://mypy-lang.org/) и [pyre](https://pyre-check.org/). За информацией по аннотированию типов и функций можно обратиться к [PEP-3107](https://www.python.org/dev/peps/pep-3107/) и [PEP-484](https://www.python.org/dev/peps/pep-0484/).

Список инструкций, которые будут выполнены при вызове функции, также достаточно простой, на стек помещаются два значения, выполняется операция сравнения и, в зависимости от результата, возвращается значение `a` или `b`:

```python
>>> dis.dis(my_min)
 13           0 LOAD_FAST                0 (a)
              2 LOAD_FAST                1 (b)
              4 COMPARE_OP               0 (<)
              6 POP_JUMP_IF_FALSE       12
              8 LOAD_FAST                0 (a)
             10 RETURN_VALUE
        >>   12 LOAD_FAST                1 (b)
             14 RETURN_VALUE
```

При вызове функции мы можем передать наши аргументы как позиционные, в таком случае важен порядок следования аргументов (хотя не для нашей функции `my_min`):

```python
>>> my_min(2, 1)
1
>>> my_min(1, 2)
1
>>> dis.dis("my_min(1, 2)")
  1           0 LOAD_NAME                0 (my_min)
              2 LOAD_CONST               0 (1)
              4 LOAD_CONST               1 (2)
              6 CALL_FUNCTION            2
              8 RETURN_VALUE
```

Также мы можем передать аргументы как именованные, в этом случае не имеет значения в каком порядке мы их указываем:

```python
>>> my_min(a=2, b=1)
1
>>> my_min(b=2, a=1)
1
>>> dis.dis("my_min(b=1, a=2)")
  1           0 LOAD_NAME                0 (my_min)
              2 LOAD_CONST               0 (1)
              4 LOAD_CONST               1 (2)
              6 LOAD_CONST               2 (('b', 'a'))
              8 CALL_FUNCTION_KW         2
             10 RETURN_VALUE
```

И, наконец, мы можем использовать смешанный вариант передачи аргументов:

```python
>>> my_min(2, b=1)
1
>>> dis.dis("my_min(2, b=1)")
  1           0 LOAD_NAME                0 (my_min)
              2 LOAD_CONST               0 (2)
              4 LOAD_CONST               1 (1)
              6 LOAD_CONST               2 (('b',))
              8 CALL_FUNCTION_KW         2
             10 RETURN_VALUE
```

Вне зависимости от того как мы осуществляем вызов функции, все аргументы являются обязательными для передачи в функцию. Давайте немного усложним пример добавив еще один аргумент, таким образом, мы будем искать минимальное среди трех аргументов:

```python
def my_min(a: float, b: float, c: float) -> float:
    if a <= b and a <= c:
        return a
    elif b <= a and b <= c:
        return b
    else:
        return c
```

Мы можем обобщить нашу функцию на список значений:

```python
from typing import List, Optional

def my_min(values: List[Optional[float]]) -> float:
    result = float('inf')
    for v in values:
        if v < result:
            result = v
    return result
```

или на произвольное число позиционных аргументов с помощью оператора `*` (для переменного числа именованных аргументов используется `**`):

```python
def my_min(*values: float) -> float:
    result = float('inf')
    for v in values:
        if v < result:
            result = v
    return result
```

Сейчас мы можем вызвать функцию `my_min` без аргументов (в таком случае результатом работы функции будет «плюс» бесконечность), если же мы хотим, чтобы в нее был передан хотя бы один обязательный аргумент, то мы должны это явно указать:

```python
def my_min(x: float, *values: float) -> float:
    result = x
    for v in values:
        if v < result:
            result = v
    return result
```

Давайте еще немного усложним нашу функцию, добавив нижнюю и верхнюю границы, которые задают диапазон для поиска минимального:

```python
def my_min(x: float, *values: float, lower: float=float('-inf'), upper: float=float('inf')) -> float:
    """ Function to get the smallest number.

    Parameters:
    -----------
    x: float
        Required numeric value.
    values: float, optional
        Variable length argument list of numeric values.
    lower: float, optional
        Lower bound. The default lower bound is negative infinity.
    upper: float, optional
        Upper bound. The default upper bound is positive infinity.

    Returns:
    --------
    The smallest value.

    Examples:
    ---------
    >>> my_min(-1, 0, 1, 2, 3)
    -1
    >>> my_min(-1, 0, 1, 2, 3, lower=0)
    0
    >>> my_min(-1, 0, 1, 2, 3, lower=4, upper=5)
    >>> my_min(-1, 0, 1, 2, 3, lower=3, upper=-1)
    Traceback (most recent call last):
    ...
    Exception: `lower` must be less or equal to `upper`
    """
    if lower > upper:
        raise Exception('`lower` must be less or equal to `upper`')
    
    result = x if lower <= x <= upper else None
    
    for v in values:
        if (lower <= v <= upper) and (result is None or v < result):
            result = v
    
    return result
```

Обратите внимание, что если у аргументов `lower` и `upper` не указать значения по умолчанию, то в Python 3, эти аргументы будут обязательными для передачи в функцию:

```python
>>> def my_min(x, *values, lower, upper):
...    # ...

>>> my_min(5, 1, 0, 3)
...
TypeError: my_min() missing 2 required keyword-only arguments: 'lower' and 'upper'
```

Мы можем обязать пользователя передавать только [именованные аргументы](https://www.python.org/dev/peps/pep-3102/), указав `*` перед списком аргументов (пример взят из [статьи](https://treyhunner.com/2018/04/keyword-arguments-in-python/) Trey Hunner’а):

```python
from random import choice, shuffle
import string

def random_password(*, upper: int, lower: int, digits: int, length: int) -> str:
    """
    >>> random_password(upper=1, lower=1, digits=1, length=8)
    'ooM2yCFc'
    >>> random_password(upper=1, lower=1, digits=1, length=8)
    'HeCr68ct'
    >>> random_password(1, 1, 1, 8)
    Traceback (most recent call last):
    ...
    TypeError: random_password() takes 0 positional arguments but 4 were given
    """
    chars = [
        *(choice(string.ascii_uppercase) for _ in range(upper)),
        *(choice(string.ascii_lowercase) for _ in range(lower)),
        *(choice(string.digits) for _ in range(digits)),
        *(choice(string.ascii_letters + string.digits) for _ in range(length-upper-lower-digits)),
    ]
    shuffle(chars)
    return "".join(chars)
```

## Атрибуты функций

Как мы уже говорили, все является объектом, включая функции. Функции представлены структурой [PyFunctionObject](https://github.com/python/cpython/blob/3.8/Include/funcobject.h#L21):
```c
typedef struct {
    PyObject_HEAD
    PyObject *func_code;        /* A code object, the __code__ attribute */
    PyObject *func_globals;     /* A dictionary (other mappings won't do) */
    PyObject *func_defaults;    /* NULL or a tuple */
    PyObject *func_kwdefaults;  /* NULL or a dict */
    PyObject *func_closure;     /* NULL or a tuple of cell objects */
    PyObject *func_doc;         /* The __doc__ attribute, can be anything */
    PyObject *func_name;        /* The __name__ attribute, a string object */
    PyObject *func_dict;        /* The __dict__ attribute, a dict or NULL */
    PyObject *func_weakreflist; /* List of weak references */
    PyObject *func_module;      /* The __module__ attribute, can be anything */
    PyObject *func_annotations; /* Annotations, a dict or NULL */
    PyObject *func_qualname;    /* The qualified name */
    vectorcallfunc vectorcall;

    /* Invariant:
     *     func_closure contains the bindings for func_code->co_freevars, so
     *     PyTuple_Size(func_closure) == PyCode_GetNumFree(func_code)
     *     (func_closure may be NULL if PyCode_GetNumFree(func_code) == 0).
     */
} PyFunctionObject;
```

Давайте разберемся с полями этой структуры.

Поле `func_code` (`__code__`) хранит ссылку на структуру `PyCodeObject` («объект кода»), которая в свою очередь содержит инструкции для виртуальной машины Python, число аргументов, сами аргументы и т.д. Более подробно мы остановимся на этой структуре в одной из следующих лекций.

```python
def square(x): return x**2

>>> dis.dis(square)
  1           0 LOAD_FAST                0 (x)
              2 LOAD_CONST               1 (2)
              4 BINARY_POWER
              6 RETURN_VALUE
>>> square.__code__.co_code
b'|\x00d\x01\x13\x00S\x00'
>>> list(square.__code__.co_code)
[124, 0, 100, 1, 19, 0, 83, 0]
```

Список опкодов можно найти в файле [Include/opcode.h](https://github.com/python/cpython/blob/3.8/Include/opcode.h):

```c
// ...
#define BINARY_POWER             19
// ...
#define RETURN_VALUE             83
// ...
#define LOAD_CONST              100
// ...
#define LOAD_FAST               124
```

### `func_globals`

https://punchagan.muse-amuse.in/blog/python-globals/

!!! quote
    Every function has an associated __globals__ dictionary, which is the same as the module’s __dict__ for the module where it was defined. This __globals__ dict is the name-space that is looked up when trying to fetch globals within a function.

```
>>> square.__globals__
{'__annotations__': {},
 '__builtins__': <module 'builtins' (built-in)>,
 '__doc__': None,
 '__loader__': <class '_frozen_importlib.BuiltinImporter'>,
 '__name__': '__main__',
 '__package__': None,
 '__spec__': None,
 'square': <function square at 0x1060a50d0>}
```

Поля `func_defaults` (`__defaults__`) и `func_kwdefaults` (`__kwdefaults__`) содержат значения по умолчанию для позиционных и ключевых аргументов, соответственно:

```python
>>> my_min.__kwdefaults__
{'lower': -inf, 'upper': inf}
```

Важно отметить, что поля `__defaults__` и `__kwdefaults__` являются изменяемыми и инициализируются один раз при создании функции. Рассмотрим два примера: 

```python
def buggy_append(value, L=[]):
    L.append(value)
    return L

>>> buggy_append.__defaults__
([],)
>>> buggy_append(1)
[1]
>>> buggy_append(2)
[1, 2]
>>> buggy_append.__defaults__
([1, 2],)
>>> buggy_append.__defaults__[0].append(3)
>>> buggy_append(4)
[1, 2, 3, 4]
```

```python
def square(*, x):
  return x * x

>>> square()
...
TypeError: foo() missing 1 required keyword-only argument: 'x'
>>> square.__kwdefaults__ = {'x': 5}
>>> square()
25
```

Поле `__closure__` содержит кортеж значений, а именно ячеек (cell objects), необходимых функции, но находящихся в объемлющем пространстве имен. Давайте рассмотрим следующий пример:

```python
def curry_pow(base=1):
    def power(x):
        return base**x
    return power

>>> pow2 = curry_pow2(2)
>>> pow2(3)
8
```

!!! note
    Приведенный пример иллюстрирует каррирование, то есть процесс, при котором функция от нескольких аргументов преобразуется в функцию (или набор функций) от одного аргумента.

```python
>>> pow2.__closure__
(<cell at ...: int object at ...>,)
>>> pow2.__closure__[0].cell_contents
2
```

```python
>>> dis.dis(curry_pow)
  2           0 LOAD_CLOSURE             0 (base)
              2 BUILD_TUPLE              1
              4 LOAD_CONST               1 (<code object power at 0x10a5e0810 ...>)
              6 LOAD_CONST               2 ('curry_pow.<locals>.power')
              8 MAKE_FUNCTION            8
             10 STORE_FAST               1 (power)

  4          12 LOAD_FAST                1 (power)
             14 RETURN_VALUE

Disassembly of <code object power at 0x10a5e0810 ...>:
  3           0 LOAD_DEREF               0 (base)
              2 LOAD_FAST                0 (x)
              4 BINARY_POWER
              6 RETURN_VALUE
```

Поле `func_doc` (`__doc__`), как вы уже могли догадаться, содержит строку документации:

```python
>>> my_min.__doc__
' Function to get the smallest number.\n...'
```

Поле `func_name` (`__name__`) является изменяемым и содержит имя функции. Значение этого атрибута обычно используется такими модулями как [pydoc](https://docs.python.org/3/library/pydoc.html) для генерирования документации:

```python
>>> square.__name__
'square'
>>> help(square)
Help on function square...
>>> square.__name__ = 'cube'
>>> help(square)
Help on function cube...
```

Необходимость в возможности изменения этого атрибута станет очевидной, когда мы будем говорить о декораторах, но мы всегда можем получить исходное имя функции через неизменяемый атрибут `co_name` у объекта кода:

```python
>>> square.__code__.co_name
'square'
```

Поле `func_dict` (`__dict__`) содержит ссылку на словарь с произвольными (пользовательскими) атрибутами (см. [PEP 232 - Function Attributes](https://www.python.org/dev/peps/pep-0232/)).

```python
>>> square.__dict__
{}
>>> sqaure.ru_doc = 'Функция возведения значния аргумента в квадрат'
>>> square.__dict__
{'ru_doc': 'Функция возведения значния аргумента в квадрат'}
```

==Поле `func_weakreflist` содержит список, так назыаемых, слабых ссылок, о которых мы будем говорить в одной из следующих лекций.==

Поле `func_module` (`__module__`) это имя модуля, в котором была определена функция:

```python
>>> globals()['__name__']
'__main__'
>>> sqaure.__module__
'__main__'
>>> globals()['__name__'] = '__secondary__'
>>> def cube(x): return x**3
>>> cube.__module__
'__secondary__'
```

Поле `func_annotations` (`__annotations__`) содержит аннотации и зачастую используется статистическими анализаторами кода, такими как [mypy](http://mypy-lang.org/) или [pyre](https://pyre-check.org/):

```python
>>> my_min.__annotations__
{
    'x': <class 'float'>,
    'values': <class 'float'>,
    'lower': <class 'float'>,
    'upper': <class 'float'>,
    'return': <class 'float'>
}
```

Поле `func_qualname` (`__qualname__`) содержит «расширенное» имя функции или класса и используется для [интроспекции](https://ru.wikipedia.org/wiki/%D0%98%D0%BD%D1%82%D1%80%D0%BE%D1%81%D0%BF%D0%B5%D0%BA%D1%86%D0%B8%D1%8F_%D0%B2_Python) (см. [PEP 3155](https://www.python.org/dev/peps/pep-3155/)):

```python
class A:
    class B:
        def d(self):
            pass

>>> A.B.d.__name__
'd'
>>> A.B.d.__qualname__
'A.B.d'
```

## Вызов функций

Концепция callable-объекта является фундаментальной в Python. Когда мы думаем о том, что может быть «вызвано» (called), то первое, что приходит на ум, это функции. Но кроме функций есть еще методы и классы, а также любой объект, в типе которого определен магический метод `__call__`:

```python
class Joke:
    def __call__(self):
        return 'That what she said'

>>> joke = Joke()
>>> joke()
'That what she said'
```

В этом примере мы «вызываем» класс `Joke` для инстанцирования нового объекта, а затем осуществляем «вызов» экземпляра класса как если бы это была обычная функция (о классах мы будем говорить в лекции [«ООП. Классы»](../lectures/classes.md)). 

==Можно сказать, что наличие `(arg1, arg2,...)` указывает на то, что происходит «вызов» и, в большинстве случаев, генерируется опкод `CALL_FUNCTION` (вызов callable-объекта с позиционными аргументами):==

```python
>>> import dis
>>> dis.dis("add(1,2)")
  1           0 LOAD_NAME                0 (add)
              2 LOAD_CONST               0 (1)
              4 LOAD_CONST               1 (2)
              6 CALL_FUNCTION            2
              8 RETURN_VALUE
```

Вот описание опкода из документации:

!!! quote "`CALL_FUNCTION(argc)`"
    Calls a callable object with positional arguments. `argc` indicates the number of positional arguments. The top of the stack contains positional arguments, with the right-most argument on top. Below the arguments is a callable object to call. `CALL_FUNCTION` pops all arguments and the callable object off the stack, calls the callable object with those arguments, and pushes the return value returned by the callable object.

Давайте кратко рассмотрим обработчик опкода [CALL_FUNCTION](https://github.com/python/cpython/blob/3.8/Python/ceval.c#L3496):

```c hl_lines="5"
case TARGET(CALL_FUNCTION): {
    PREDICTED(CALL_FUNCTION);
    PyObject **sp, *res;
    sp = stack_pointer;
    res = call_function(tstate, &sp, oparg, NULL);
    stack_pointer = sp;
    PUSH(res);
    if (res == NULL) {
        goto error;
    }
    DISPATCH();
}
```

Фактически происходит вызов функции [call_function](https://github.com/python/cpython/blob/3.8/Python/ceval.c#L3496), куда передается адрес вершины стека `sp` и число позиционных аргументов `oparg`:

```c
(gdb) p *(sp-1)
$1 = 2
(gdb) p *(sp-2)
$2 = 1
(gdb) p *(sp-3)
$3 = <function at remote 0x1014597d0>
(gdb) p ((PyFunctionObject*)(*(sp-3)))->func_name
$4 = 'add'
(gdb) p oparg
$5 = 2
```

!!! note
    https://www.ics.uci.edu/~pattis/common/handouts/macmingweclipse/allexperimental/mac-gdb-install.html 

`call_function` является общей для вызова функций, методов, классов и других объектов:

```c hl_lines="15"
Py_LOCAL_INLINE(PyObject *) _Py_HOT_FUNCTION
call_function(PyThreadState *tstate, PyObject ***pp_stack, Py_ssize_t oparg, PyObject *kwnames)
{
    PyObject **pfunc = (*pp_stack) - oparg - 1;
    PyObject *func = *pfunc;
    PyObject *x, *w;
    Py_ssize_t nkwargs = (kwnames == NULL) ? 0 : PyTuple_GET_SIZE(kwnames);
    Py_ssize_t nargs = oparg - nkwargs;
    PyObject **stack = (*pp_stack) - nargs - nkwargs;

    if (tstate->use_tracing) {
        x = trace_call_function(tstate, func, stack, nargs, kwnames);
    }
    else {
        x = _PyObject_Vectorcall(func, stack, nargs | PY_VECTORCALL_ARGUMENTS_OFFSET, kwnames);
    }

    // ...

    return x;
}
```

Происходит подготовка аргументов для вызова функции [`_PyObject_Vectorcall`](https://github.com/python/cpython/blob/3.8/Include/cpython/abstract.h#L114), где `nargs` и `nkwargs` указывает на число позиционных и именованных аргументов, соответственно, `nkwargs` представляет собой кортеж с именами ключевых аргументов (см. опкод `CALL_FUNCTION_KW`), `stack` указывает на первый аргумент функции, а `func` указывает на объект `PyFunctionObject` (нашу функцию `add`):

```c
(gdb) p nargs
$6 = 2
(gdb) p nkwargs
$7 = 0
(gdb) p kwnames
$8 = 0x0
(gdb) p *stack
$9 = 1
(gdb) p *(stack+1)
$10 = 2
(gdb) p *(stack-1)
$11 = <function at remote 0x1014597d0>
(gdb) p func
$12 = <function at remote 0x1014597d0>
(gdb) p ((PyFunctionObject*)0x1014597d0)->func_name
$13 = 'add'
```

```c hl_lines="8 13"
static inline PyObject *
_PyObject_Vectorcall(PyObject *callable, PyObject *const *args,
                     size_t nargsf, PyObject *kwnames)
{
    PyObject *res;
    vectorcallfunc func;
    // ...
    func = _PyVectorcall_Function(callable);
    if (func == NULL) {
        Py_ssize_t nargs = PyVectorcall_NARGS(nargsf);
        return _PyObject_MakeTpCall(callable, args, nargs, kwnames);
    }
    res = func(callable, args, nargsf, kwnames);
    return _Py_CheckFunctionResult(callable, res, NULL);
}
```

В функции `_PyObject_Vectorcall` проверяется поддерживает ли callable-объект новый протокол Vectorcall, который был введен в [PEP 590](https://www.python.org/dev/peps/pep-0590/) с целью оптимизации вызова callable-объектов:

!!! quote
    The poor performance is largely a result of having to create intermediate tuples, and possibly intermediate dicts, during the call. This is mitigated in CPython by including special-case code to speed up calls to Python and builtin functions. Unfortunately, this means that other callables such as classes and third party extension objects are called using the slower, more general `tp_call` calling convention.

    This PEP proposes that the calling convention used internally for Python and builtin functions is generalized and published so that all calls can benefit from better performance...

Отметим, что все пользовательские функции, начиная с Python 3.8, поддерживают протокол Vectorcall. Возможно, вы обратили внимание, что последнем полем структуры `PyFunctionObject` является `vectorcall` типа [`vectorcallfunc`](https://github.com/python/cpython/blob/3.8/Include/cpython/object.h#L58):

```c
typedef PyObject *(*vectorcallfunc)(PyObject *callable, PyObject *const *args,
                                    size_t nargsf, PyObject *kwnames);
```

Это поле инициализируется при [создании](https://github.com/python/cpython/blob/3.8/Objects/funcobject.c#L12) новой функции (см. опкод `MAKE_FUNCTION`):

```c hl_lines="8"
PyObject *
PyFunction_NewWithQualName(PyObject *code, PyObject *globals, PyObject *qualname)
{
    PyFunctionObject *op;
    // ...
    op = PyObject_GC_New(PyFunctionObject, &PyFunction_Type);
    // ...
    op->vectorcall = _PyFunction_Vectorcall;
    // ...
    return (PyObject *)op;
}
```

Таким образом, вызов:
```c
res = func(callable, args, nargsf, kwnames);
```

эквивалентен вызову:

```c
res = _PyFunction_Vectorcall(callable, args, nargsf, kwnames);
```

Наконец перейдем к [`_PyFunction_Vectorcall`](https://github.com/python/cpython/blob/3.8/Objects/call.c#L386):

```c
PyObject *
_PyFunction_Vectorcall(PyObject *func, PyObject* const* stack,
                       size_t nargsf, PyObject *kwnames)
{
    PyCodeObject *co = (PyCodeObject *)PyFunction_GET_CODE(func);
    
    // ...

    return _PyEval_EvalCodeWithName((PyObject*)co, globals, (PyObject *)NULL,
                                    stack, nargs,
                                    nkwargs ? _PyTuple_ITEMS(kwnames) : NULL,
                                    stack + nargs,
                                    nkwargs, 1,
                                    d, (int)nd, kwdefs,
                                    closure, name, qualname);
}
```

Отметим лишь два момента, первый, это получение объекта кода, о котором мы говорили ранее, и второе, создание и исполнение (evaluation) нового фрейма (о фреймах мы будем говорить в одной из следующих лекций) с телом нашей функции. 

## Декораторы

!!! note
    A decorator is any callable Python object that is used to modify a function, method or class definition. A decorator is passed the original object being defined and returns a modified object, which is then bound to the name in the definition.

```python
def logtime(func):
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        total_time = time.time() - start_time
        with open("timelog.txt", "a") as outfile:
            outfile.write(f"{time.time()}\t{func.__name}\t{total_time}\n")
        return result
    return wrapper
```

```python
def accepts(*types):
    def check_accepts(f):
        assert len(types) == f.func_code.co_argcount
        def new_f(*args, **kwds):
            for (a, t) in zip(args, types):
                assert isinstance(a, t), \
                       "arg %r does not match %s" % (a,t)
            return f(*args, **kwds)
        new_f.func_name = f.func_name
        return new_f
    return check_accepts
```

```python
def returns(rtype):
    def check_returns(f):
        def new_f(*args, **kwds):
            result = f(*args, **kwds)
            assert isinstance(result, rtype), \
                   "return value %r does not match %s" % (result,rtype)
            return result
        new_f.func_name = f.func_name
        return new_f
    return check_returns
```
