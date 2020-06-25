Эта лекция является первой в серии посвященной объектно-ориентированной парадигме программирования.

## Процедурный подход

Представьте, что вы пишите веб-сервис и перед вами встала следующая задача: «Как представить пользователя системы в программе?». Для начала нам необходимо выделить ряд значимых характеристик нашего пользователя:

* имя (username)
* адрес электронной почты (email)
* пароль (password)

Можно представить пользователя в виде нескольких переменных:

```py
username = 'user'
email = 'user@example.com'
password = 'qwerty'
```

Мы пониманием, что переменные логически должны быть связаны между собой, но мы эту связь никак не показали. Другими словами, переменная `username`  может содержать имя одного пользователя, `email` относиться ко второму пользователю, а `password` к третьему.

Как показать, что существует логическая связь между этими переменными? Мы можем использовать любую подходящую структуру, например, словарь:

```py
user = {
    'username': 'bob',
    'email': 'bob@example.com',
    'password': 'qwerty'
}
```

А так мы теперь могли бы представить список пользователей:

```python
users = [
    {'username': 'bob', 'email': 'bob@example.com', 'password': 'qwerty'},
    {'username':'joe', 'email': 'joe@example.com', 'password': 'secret'},
]
```

Таким образом, объединив несколько значений (имя, адрес электронной почты и пароль) в контейнер, мы попытались показать, что существует логическая связь между этими значениями.

Итак, каждый контейнер (словарь) хранит состояние (характеристики, атрибуты) конкретного пользователя. Для управляемого доступа к состоянию объекта используют специальные функции, так называемые «геттеры» и «сеттеры», например:

```python
def get_email(user: Dict[str, str]) -> str:
    return user['email']

def set_email(user: Dict[str, str], email: str) -> None:
    match = re.match('^[_a-z0-9-]+(\.[_a-z0-9-]+)*@[a-z0-9-]+(\.[a-z0-9-]+)*(\.[a-z]{2,4})$', email)
    if not match:
        raise ValueError("Invalid email")
    user['email'] = email


>>> user = {}
>>> set_email(user, 'bob@example')
...
ValueError: Invalid email
>>> set_email(user, 'bob@example.com')
>>> get_email(user)
'bob@example.com'
```

Одним из недостатков такого подхода является то, что мы не показали логическую связь между состоянием пользователя и функциями, которые предоставляют доступ к этому состоянию. В решении этой проблемы нам может помочь инкапсуляция.

!!! info
    **Инкапсуляция** – это свойство системы, позволяющее объединить данные и методы, работающие с ними, в классе и скрыть детали реализации от пользователя.

## Создание простого класса

Начнем с создания простого класса:

```python
class User:
    pass
```

!!! info
    **Класс** – это способ описания сущности, определяющий состояние и поведение, зависящее от этого состояния, а также правила для взаимодействия с данной сущностью (контракт). 

Теперь создадим новый объект класса пользователь:

```python
>>> u = User()
```

!!! info
    **Объект** (экземпляр) – это отдельный представитель класса, имеющий конкретное состояние и поведение, полностью определяемое классом.

Атрибуты хранятся в специальном словаре (подробнее про модель данных в Python можно почитать [тут](https://docs.python.org/3.6/reference/datamodel.html)), к которому можно обратиться по имени `__dict__`:

```python
>>> u.__dict__
{}
```

Так как мы не создалили еще ни одного атрибута, то и словарь будет пустым. Давайте добавим несколько атрибутов (Python позволяет динамически привязывать новые атрибуты к объекту, в конце концов это просто словарь):

```python
>>> u.username = 'bob'
>>> u.password = 'bob@example.com'
>>> u.email = 'qwerty'

>>> u.__dict__
{'username': 'bob', 'password': 'bob@example.com', 'email': 'qwerty'}

# Следующие два выражения в нашем примере эквиваленты
>>> u.username
'bob'
>>> u.__dict__['username']
'bob'
```

Если мы обратимся к атрибуту, которого не существует, то будет возбуждено исключение `AttributeError` (более подробно о поиске атрибутов мы поговорим в лекции [«ООП. Разрешение имен атрибутов»](../lectures/attribute_lookup.md)):

```python
>>> u.created_at
...
AttributeError: 'User' object has no attribute 'created_at'
```

Работать с атрибутами можно с помощью следующих функций:

* `hasattr(obj, attr_name)` - проверить наличие атрибута `attr_name` в объекте `obj`. Если атрибут присутствует, то функция возвращает `True`, иначе `False`.
* `getattr(obj, attr_name[, default_value])` - получить значение атрибута `attr_name` в объекте `obj`. Если атрибут не был найден, то будет возбуждено исключение `AttributeError`. Можно указать значение по умолчанию `default_value`, которое будет возвращено, если атрибута не существует.
* `setattr(obj, attr_name, value)` - изменить значение атрибута `attr_name` на `value`. Если атрибут не существовал, то он будет создан.

```python
>>> hasattr(u, 'created_at')
False
>>> hasattr(u, 'username')
True

>>> getattr(u, 'created_at')
...
AttributeError: 'User' object has no attribute 'created_at'

>>> import datetime
>>> getattr(u, 'created_at', datetime.datetime.now())
datetime.datetime(2017, 4, 11, 16, 45, 36, 757869)

>>> setattr(u, 'created_at', datetime.datetime.now())
datetime.datetime(2017, 4, 11, 16, 45, 36, 757869)
```

Функция `setattr` может оказаться полезной, когда нам необходимо добавить в объект множество атрибутов, хранящихся в каком-нибудь контейнере:

```python
u = User()
attrs = {
    'username': 'bob',
    'email': 'bob@example.com',
    'password': 'qwerty'
}
for attr, value in attrs.items():
    setattr(u, attr, value)
```

Добавим теперь в ранее созданный объект «сеттер» и «геттер» для работы со свойством адреса электронной почты:

```python
def get_email(user: User) -> str:
    return user.email

def set_email(user: User, email: str) -> None:
    match = re.match('^[_a-z0-9-]+(\.[_a-z0-9-]+)*@[a-z0-9-]+(\.[a-z0-9-]+)*(\.[a-z]{2,4})$', email)
    if not match:
        raise ValueError("Invalid email")
    user.email = email

>>> u.get_email = get_email
>>> u.set_email = set_email
>>> u.__dict__
{
    'email': 'bob@example.com',
    'get_email': <function get_email at 0x10bfb79d8>,
    'password': 'qwerty',
    'set_email': <function set_email at 0x10bfb7268>,
    'username': 'bob'
}
>>> u.get_email(u)
'bob@example.com'
```

Обратите внимание, что мы обращаемся к функции `get_email` у объекта `u` и в качестве аргумента передаем сам объект `u`. Выглядит немного странно [^1]. Также отметим, что до этого момента мы добавляли атрибуты и функции в конкретный экземпляр класса `User`, если мы создадим еще один объект, то у него не будет этих свойств и нам придется добавлять их заново. Забегая немного вперед скажем, что класс это тоже объект и у него также есть специальный словарь куда мы можем добавлять свои атрибуты (говорят «атрибуты класса»):

```python
>>> User.get_email = get_email
>>> User.set_email = set_email
>>> User.__dict__
mappingproxy({'__dict__': <attribute '__dict__' of 'User' objects>,
              '__doc__': None,
              '__module__': '__main__',
              '__weakref__': <attribute '__weakref__' of 'User' objects>,
              'get_email': <function get_email at 0x10370e0d0>,
              'set_email': <function set_email at 0x1038a02f0>})
```

Добавив таким образом функции в класс, мы получим возможность вызывать их у всех экземпляров этого класса, при этом без необходимости передавать сам объект в качестве первого аргумента функции, так как он будет передан автоматически:

```python
>>> u = User()
>>> u.set_email('bob@example.com')
>>> u.get_email()
'bob@example.com'
```

Более подробно про то, как именно работает этот механизм, мы поговорим на лекциях [«ООП. Дескрипторы»](../lectures/descriptors.md), [«ООП. Разрешение имен атрибутов»](../lectures/attribute_lookup.md) и [«ООП. Порядок разрешения методов](../lectures/mro.md).

Последним шагом является добавление возможности создавать и инциалзировать одинаковый набор атрибутов для всех экземпляров класса. Для этих целей в Python используется «магический» метод `__init__` (все методы, имя которых начинается и заканчивается двумя нижними подчеркиваниями, называются «магическими методами», так как имеют определенное назначение), который автоматически вызывается в процессе инстанцирования нового экземпляра класса:

```python
import hashlib
import random
import re
import string


class User:

    def __init__(self, username: str, email: str, password: str) -> None:
        self._username = username
        self._email = None
        self._password = None
        self.set_email(email)
        self.set_password(password)

    def get_username(self) -> str:
        return self._username

    def set_username(self, username: str) -> None:
        self._username = username

    def get_email(self) -> str:
        return self._email

    def set_email(self, email: str) -> None:
        match = re.match('^[_a-z0-9-]+(\.[_a-z0-9-]+)*@[a-z0-9-]+(\.[a-z0-9-]+)*(\.[a-z]{2,4})$', email)
        if not match:
            raise ValueError("Invalid email")
        self._email = email
    
    def set_password(self, password: str, salt: str=None) -> None:
        if salt == None:
            salt = self._make_salt()
        self._password = hashlib.sha256(password.encode() + salt.encode()).hexdigest() + "," + salt

    def check_password(self, user_password: str) -> bool:
        # @see: http://pythoncentral.io/hashing-strings-with-python/
        # @see: https://docs.python.org/3.5/library/hashlib.html
        password, salt = self._password.split(',')
        return password == hashlib.sha256(user_password.encode() + salt.encode()).hexdigest()

    def _make_salt(self) -> str:
        return ''.join(random.choice(string.ascii_letters) for _ in range(5))
```

!!! note
    **Интерфейс** – это набор методов класса, доступных для использования другими классами.

## Создание экземпляра класса

Давайте немного остановимся на том как создаются новые экземпляры класса:

```python
dis.dis("class User: pass\nu = User()")
  2          14 LOAD_NAME                0 (User)
             16 CALL_FUNCTION            0
             18 STORE_NAME               1 (u)
             20 LOAD_CONST               2 (None)
             22 RETURN_VALUE
```

Итак, на стек помещается класс `User` и затем выполняется инструкция `CALL_FUNCTION`:

```c
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

```c
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

    assert((x != NULL) ^ (_PyErr_Occurred(tstate) != NULL));

    /* Clear the stack of the function object. */
    while ((*pp_stack) > pfunc) {
        w = EXT_POP(*pp_stack);
        Py_DECREF(w);
    }

    return x;
}
```

Произодйет вызов функции `_PyObject_Vectorcall` (см. [PEP 590](https://www.python.org/dev/peps/pep-0590/)):

```c hl_lines="10"
static inline PyObject *
_PyObject_Vectorcall(PyObject *callable, PyObject *const *args,
                     size_t nargsf, PyObject *kwnames)
{
    assert(kwnames == NULL || PyTuple_Check(kwnames));
    assert(args != NULL || PyVectorcall_NARGS(nargsf) == 0);
    vectorcallfunc func = _PyVectorcall_Function(callable);
    if (func == NULL) {
        Py_ssize_t nargs = PyVectorcall_NARGS(nargsf);
        return _PyObject_MakeTpCall(callable, args, nargs, kwnames);
    }
    PyObject *res = func(callable, args, nargsf, kwnames);
    return _Py_CheckFunctionResult(callable, res, NULL);
}
```

```c hl_lines="6 13"
PyObject *
_PyObject_MakeTpCall(PyObject *callable, PyObject *const *args, Py_ssize_t nargs, PyObject *keywords)
{
    /* Slow path: build a temporary tuple for positional arguments and a
     * temporary dictionary for keyword arguments (if any) */
    ternaryfunc call = Py_TYPE(callable)->tp_call;
    
    // ...

    PyObject *result = NULL;
    if (Py_EnterRecursiveCall(" while calling a Python object") == 0)
    {
        result = call(callable, argstuple, kwdict);
        Py_LeaveRecursiveCall();
    }

    // ...

    result = _Py_CheckFunctionResult(callable, result, NULL);
    return result;
}
```

Макрос `PY_TYPE` возвращает тип объекта. Мы уже упомянули, что классы также являются объектами, а следовательно и у них есть тип, другими словами, есть классы, которые порождают другие классы и называются они метаклассами (иногда встречается термин «метатип»):

```python
>>> type(User)
<class 'type'>
```

По умолчанию метаклассом для всех классов является класс [`type`](https://github.com/python/cpython/blob/3.8/Objects/typeobject.c#L3607), только если мы явно не указали другой метакласс. Также отметим, что все типы в виртуальной машине CPython представлены структурой [`PyTypeObject`](https://docs.python.org/3/c-api/typeobj.html), содержащей по большей части указатели на функции (слоты), которые определяют поведение объекта. Таким образом, в листинге выше происходит обращение к слоту `tp_call`, который по умолчанию содержит указатель на функцию `type_call`:

```c hl_lines="7 20"
static PyObject *
type_call(PyTypeObject *type, PyObject *args, PyObject *kwds)
{
    PyObject *obj;

    // ...
    obj = type->tp_new(type, args, kwds);
    obj = _Py_CheckFunctionResult((PyObject*)type, obj, NULL);
    if (obj == NULL)
        return NULL;

    // ...
    /* If the returned object is not an instance of type,
       it won't be initialized. */
    if (!PyType_IsSubtype(Py_TYPE(obj), type))
        return obj;

    type = Py_TYPE(obj);
    if (type->tp_init != NULL) {
        int res = type->tp_init(obj, args, kwds);
        // ...
    }
    return obj;
}
```

В качестве первого аргумента в функцию `type_call` передается тип (`User`), затем передаются позиционные и ключевые аругменты, которые были указаны при создании экземпляра класса. В функции `type_call` происходит вызов конструктора `tp_new` и инициализатора `tp_init`, которые соответствуют магическим методам [`__new__`](https://docs.python.org/3.8/reference/datamodel.html#object.__new__) и `__init__`. Мы не переопределяли для класса `User` ни один из этих магических методов, поэтому их реализация наследуется от базового класса. Базовым классом для всех объектов в Python является класс [`object`](https://github.com/python/cpython/blob/3.8/Objects/typeobject.c#L4783) (кроме него самого), который представлен структурой `PyBaseObject_Type`. В этой структуре слоты `tp_new` и `tp_init` инициализированы указателями на функции `object_new` и `object_init`, которые и будут вызваны.

Функция `object_init` для нас не представляет особого интереса. В функции `object_new` происходит выделение памяти под новый объект со следующей структурой:

Поле | Размер в байтах
------------ | -------------
`PyGC_Head` | 16 (24 до Python 3.8)
`PyObject_HEAD` | 16
`__dict__` | 8
`__weakref__` | 8
Всего | 48 (56 до Python 3.8)

Где [`PyGC_Head`](https://github.com/python/cpython/blob/3.8/Include/cpython/objimpl.h#L46) это ==элемент двойного связанного списка==, который используется сборщиком мусора для обнаружения циклических ссылок. `__weakref__` это ссылка на список, так называемых, слабых ссылок (weak reference) на данный объект. В одной из следующих лекций мы будем говорить про управление памятью в CPython, сейчас мы не будем на этом подбробно останавливаться.

И наконец заметим, что словарь `__dict__` для экземпляра класса не создается в процессе выделения памяти. При создании нового типа определяется значение `tp_dictoffset`, то есть смещение относительно адреса объекта, по которому находится указатель на словарь (`PyGC_Head` не учитывается в этом смещении):

```python
import ctypes

def magic_dict_ptr(o):
    dict_addr = id(o) + type(o).__dictoffset__
    dict_ptr  = ctypes.cast(dict_addr, ctypes.POINTER(ctypes.py_object))
    return dict_ptr

>>> u = User()
>>> d_ptr = magic_dict_ptr(u)
>>> dptr.contents
py_object(<NULL>)
>>> u.username = 'bob'
>>> dptr.contents
py_object({'username': 'bob'})
```

Таким образом, фактическое выделение памяти под `__dict__` происходит при первом обращении к нему, например, при добавлении нового атрибута.

Итак, упрощенно процесс создания и инициализации нового объекта можно описать следующими шагами:

1. Мы хотим инстанцировать новый объект некоторого класса: `u = User()`
2. Происходит вызов метода `__call__` у метакласса: `type(User).__call__(User)`.
3. Вызывается конструктор объекта `__new__`, который возвращает «пустой» объект.
4. Созданный объект передается в инициализатор `__init__` в качестве первого аргумента с именем `self` (такое имя не является обязательным, но используется по соглашению), за ним передаются все остальные аргументы указанные при инстанцировании класса.
5. У объекта создаются все требуемые атрибуты, например `self.username = username`. При первом обращении к `__dict__` под него выделяется память.
6. Инициализированный объект возвращается на место вызова класса, в примере переменная `u` связывается с созданным объектом.

!!! note
    Более подробно о создании новых типов мы будем говорить в лекции [«ООП. Метаклассы»](../lectures/metaclasses.md).

## «Приватные» поля класса

Вы обратили внимание, что в нашем классе `User` некоторые атрибуты начинаются с нижнего подчеркивания? Это одно из множества соглашений принятых в сообществе разработчиков на языке Python, согласно которому «приватные» атрибуты должны начинаться с одного символа нижнего подчеркивания. Давайте создадим нового пользователя:

```python
>>> u = User('bob', 'bob@example.com', 'qwerty')
>>> u.get_username()
'bob'
>>> u.get_email()
'bob@example.com'
>>> u.check_password('qwerty')
True
```

Допустим, что мы хотим изменить пароль (или адрес электронной почты) и делаем это через прямое обращение к атрибуту:
```python
>>> u._password = 'foobar'
>>> u.check_password('foobar')
False
```

Почему пароль не прошел проверку? Мы изменили значение атрибута напрямую, не используя функцию `set_password()`, таким образом, мы сохранили пароль в открытом виде. В свою очередь функция `check_password()` хеширует переданный ей пароль в качестве аргумента и затем сравнивает его с паролем, который хранился в атрибуте `_password`.

Очевидно, что нужно менять значение пароля или адреса электронной почты с помощью методов `set_password()` и `set_email()`, чтобы избежать подобного рода ошибок. А прямое обращение к полям `_password` и `_email` нужно ограничить. В языке Python сложно что-то запретить, в частности обращение к полям экземпляра класса, но есть соглашения. Как уже было сказано, если имя атрибута начинается с одного нижнего подчеркивания, то он считается приватным, другими словами, указывая нижнее подчеркивание перед именем атрибута мы сообщаем клиентам нашего класса «Не нужно обращаться к этому полю напрямую, иначе можно нарушить логику работы».

!!! note
    Больше про нижние подчеркивания можно узнать [тут](https://shahriar.svbtle.com/underscores-in-python).

## Магические методы

```python
class User:
    # ...

    def __eq__(self, other):
        if isinstance(other, User):
            return self._email == other._email
        raise NotImplemented

    def __repr__(self):
        return f"User(username={self._username}, email={self._email})"
```

```python
>>> bob = User('bob', 'bob@example.com', 'qwerty')
>>> bob
User(username=bob, email=bob@example.com)

>>> bobby = User('bob', 'bob@example.com', 'qwerty')
>>> bob == bobby
True
```

!!! note
    Все строки в формате [f-strings](https://cito.github.io/blog/f-strings/), который был введен в Python 3.6.
    
    Метод `__repr__` переопределен, чтобы выводить чуть больше полезной информации об объекте, чем просто его адрес в памяти. Узнать больше про магические методы можно прочитав статью на [Хабре](https://habrahabr.ru/post/186608/).

## Пример: создание простой ORM

Что такое ORM? Вот пояснение с сайта [Full Stack Python](https://www.fullstackpython.com/object-relational-mappers-orms.html):

!!! quote
    An object-relational mapper (ORM) is a code library that automates the transfer of data stored in relational databases tables into objects that are more commonly used in application code.

В этом примере (полностью основанном на [этом коде](https://codescience.wordpress.com/2011/02/06/python-mini-orm/)) мы рассмотрим пример создания примитивной ORM для SQLite базы данных, которая имеет [встроенную поддержку](https://docs.python.org/3.6/library/sqlite3.html) в Python.

Создадим БД с таблицей `Users` и добавим туда несколько записей:

```py
import sqlite3

# Создание нового соединения с БД
conn = sqlite3.connect('users_db.sqlite3')

# Курсор это объект, который позволяет выполнять запросы к БД
cursor = conn.cursor()

# Создание таблицы пользователей
cursor.execute('CREATE TABLE users (id, username, email, password)')

# Добавление новых записей
users = [
   (1, 'john', 'john@thebeatles.com', 'foobar'),
   (2, 'paul', 'paul@thebeatles.com', 'barfoo'),
   (3, 'ringo', 'ringo@thebeatles.com', 'foobaz'),
   (4, 'george', 'george@thebeatles.com', 'bazfoo')
]
cursor.executemany('INSERT INTO users VALUES (?,?,?,?)', users)
conn.commit()

# Вывод всех записей
for row in cursor.execute('SELECT * FROM users'):
   print(row)
```

В результате вы должны увидеть следующие записи:
```py
(1, 'john', 'john@thebeatles.com', 'foobar')
(2, 'paul', 'paul@thebeatles.com', 'barfoo')
(3, 'ringo', 'ringo@thebeatles.com', 'foobaz')
(4, 'george', 'george@thebeatles.com', 'bazfoo')
```

Теперь перейдем к ORM:
```py
import sqlite3


class DataBase:
    def __init__(self, db='db'):
        self.conn = sqlite3.connect(f"{db}.sqlite3")
        self.cursor = self.conn.cursor()

    def get_columns(self, tbl_name):
        self.sql_rows = f"SELECT * FROM {tbl_name}"
        columns = f"PRAGMA table_info({tbl_name})"
        self.cursor.execute(columns)
        return [row[1] for row in self.cursor.fetchall()]

    def Table(self, tbl_name):
        columns = self.get_columns(tbl_name)
        return Query(self.cursor, self.sql_rows, columns, tbl_name)


class Query:
    def __init__(self, cursor, rows, columns, tbl_name):
        self.cursor = cursor
        self.sql_rows = rows
        self.columns = columns
        self.tbl_name = tbl_name

    def filter(self, criteria):
        key_word = "AND" if "WHERE" in self.sql_rows else "WHERE"
        sql = f"{self.sql_rows} {key_word} {criteria}"
        return Query(self.cursor, sql, self.columns, self.tbl_name)

    def order_by(self, criteria):
        return Query(self.cursor, f"{self.sql_rows} ORDER BY {criteria}", self.columns, self.tbl_name)

    def group_by(self, criteria):
        return Query(self.cursor, f"{self.sql_rows} GROUP BY {criteria}", self.columns, self.tbl_name)

    @property
    def rows(self):
        print(self.sql_rows)
        self.cursor.execute(self.sql_rows)
        return [Row(zip(self.columns, fields), self.tbl_name) for fields in self.cursor.fetchall()]


class Row:
    def __init__(self, fields, table_name):
        self.__class__.__name__ = table_name + "_Row"

        for name, value in fields:
            setattr(self, name, value)

    def __repr__(self):
        attrs =  ', '.join([f"{attr}={value}" for attr, value in self.__dict__.items()])
        return f"{self.__class__.__name__}({attrs})"
```
Класс `DataBase` отвечает за создание соединения, класс `Query` за формирование запроса к БД, класс `Row` представляет одну запись в таблице.

Ниже приведен пример использования:

```py
>>> db = DataBase('users_db')

>>> db.get_columns('users')
['id', 'username', 'email', 'password']

>>> db.Table('users').rows:
SELECT * FROM users
[users_Row(id=1, username=john, email=john@thebeatles.com, password=foobar),
 users_Row(id=2, username=paul, email=paul@thebeatles.com, password=barfoo),
 users_Row(id=3, username=ringo, email=ringo@thebeatles.com, password=foobaz),
 users_Row(id=4, username=george, email=george@thebeatles.com, password=bazfoo)]

>>> db.Table('users').filter('id > 2').rows
SELECT * FROM users WHERE id > 2
[users_Row(id=3, username=ringo, email=ringo@thebeatles.com, password=foobaz),
 users_Row(id=4, username=george, email=george@thebeatles.com, password=bazfoo)]

>>> db.Table('users').order_by('username DESC').rows
SELECT * FROM users ORDER BY username DESC
[users_Row(id=3, username=ringo, email=ringo@thebeatles.com, password=foobaz),
 users_Row(id=2, username=paul, email=paul@thebeatles.com, password=barfoo),
 users_Row(id=1, username=john, email=john@thebeatles.com, password=foobar),
 users_Row(id=4, username=george, email=george@thebeatles.com, password=bazfoo)]

>>> user = db.Table('users').rows[0]
SELECT * FROM users
>>> user.id
1
>>> user.username
'john'
```

**Задания**:

- вы должны были заметить, что мы получаем объекты класса `users_Row`, а не класса `User`. Попробуйте внести изменения, чтобы мы получали объекты класса `User`:
```python
>>> user = db.Table('users').rows[0]
>>> type(user)
<class '__main__.User'>
>>> user.get_username()
'john'
```
- добавьте метод `limit(N)` в класс `DataBase`, который позволяет получить не больше N записей.
- добавьте метод `insert(obj)`, который создает в БД новую запись об объекте `obj`.

[^1]:
    Ради справедливости нужно отметить, что мы можем динамически создать метод у объекта, таким образом, чтобы не пришлось передавать объект в качестве аргумента:

    ```python
    >>> from types import MethodType
    >>> u.get_email = MethodType(get_email, u)
    >>> u.get_email()
    'bob@example.com'

    >>> u.__dict__
    {
        'email': 'bob@example.com',
        'get_email': <bound method get_email of <__main__.User object at 0x10e6e8f28>>,
        'password': 'qwerty',
        'username': 'bob'
    }
    ```