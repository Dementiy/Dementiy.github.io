Настало время поговорить об областях видимости и пространстве имен. Но прежде вспомним, что в Python переменных в том смысле, в каком они есть в таких языках программирования как C/C++/Java/и т.д., нет, но есть символьные имена, которые связаны с объектами (у которых уже есть тип, точнее ссылка на соответствующую структуру). Таким образом, запись вида `a = 1` означает связывание имени (name binding) `a` с объектом `1` (`PyLongObject`).

Рассмотрим следующий простой пример:

```python
>>> some_variable
# ...
NameError: name 'some_variable' is not defined
```

Мы получили ошибку `NameError`, но откуда интерпретатор знает, что такого имени не существует? Чтобы ответить на этот вопрос мы должны познакомиться с еще двумя [понятиями](https://softwareengineering.stackexchange.com/questions/273302/what-is-the-relationship-between-scope-and-namespaces-in-python): пространство имен и области видимости:

!!! quote
    A namespace is a dictionary, mapping names (as strings) to values. When you do an assignment, like `a = 1`, you’re mutating a namespace. When you make a reference, like `print(a)`, Python looks through a list of namespaces to try and find one with the name as a key.

    A scope defines which namespaces will be looked in and in what order. The scope of any reference always starts in the local namespace, and moves outwards until it reaches the module’s global namespace, before moving on to the builtins (the namespace that references Python’s predefined functions and constants, like range and getattr), which is the end of the line.

Итак, о пространстве имен можно думать как словаре, в котором ключами являются имена «переменных», а значениям связанные с именами объекты. В свою очередь область видимости опеределяет какие пространства имен мы должны просмотреть в поиске указанного имени. Перебор областей видимости происходит в следующем порядке: локальная (L - Local) область видимости, объемлющая (E - Enclosed), глоабльная (G - Global) и встроенная (B - Built-in).

Вернемся к примеру с нашей переменной some_variable:

```python
>>> dis.dis("some_variable")
  1           0 LOAD_NAME                0 (some_variable)
              2 RETURN_VALUE
```

Опкод [`LOAD_NAME`](https://github.com/python/cpython/blob/master/Python/ceval.c) отвечает за помещение значения `some_variable` на стек и при выполнении этой инструкции происходит обход областей видимости в указанном выше пордяке, и, если имя не было найдено, то будет порождено исключение `NameError`. Упрощенно этот процесс можно представить следующим [образом](https://tech.blog.aknin.name/2010/06/05/pythons-innards-naming/):

```python
def LOAD_NAME(name):
    try:
        value = current_stack_frame.locals[name]
    except KeyError:
        try:
            value = current_stack_frame.globals[name]
        except KeyError:
            try:
                value = current_stack_frame.builtins[name]
            except KeyError:
                raise NameError(f"name {name} is not defined")
    PUSH(value)

def STORE_NAME(name):
    value = POP()
    current_stack_frame.locals[name] = value
```

Кроме пары опкодов `LOAD_NAME/STORE_NAME` есть и другие опкоды `LOAD_*/STORE_*`, которые взаимодействуют с областью видимости. Давайте рассмотрим следующий пример очень простой функции:

```python
>>> def f():
...     local_var = 'local variable'
...     print(local_var)

>>> dis.dis(f)
  2           0 LOAD_CONST               1 ('local variable')
              2 STORE_FAST               0 (local_var)

  3           4 LOAD_GLOBAL              0 (print)
              6 LOAD_FAST                0 (local_var)
              8 CALL_FUNCTION            1
             10 POP_TOP
             12 LOAD_CONST               0 (None)
             14 RETURN_VALUE
```

Опкод `LOAD_CONST` отношения к области видимости не имеет, потому что константы не являются переменными. Тем не менее, мы можем задаться вопросом «Где эти константы искать?». Для этого нам придется ввести еще одно понятие - [code object](https://docs.python.org/3/reference/executionmodel.html):

!!! quote
    A Python program is constructed from code blocks. A block is a piece of Python program text that is executed as a unit. The following are blocks: a module, a function body, and a class definition. Each command typed interactively is a block. A script file (a file given as standard input to the interpreter or specified as a command line argument to the interpreter) is a code block. A script command (a command specified on the interpreter command line with the '-c' option) is a code block. The string argument passed to the built-in functions eval() and exec() is a code block.

    A code block is executed in an execution frame. A frame contains some administrative information (used for debugging) and determines where and how execution continues after the code block’s execution has completed.

Code object в первую очередь содержит исполняемый байт-код, а также набор атрибутов среди которых: число аргументов, список констант, список имен локальных перменных и т.д.

```python
>>> f.__code__.co_consts
(None, 'local variable')

>>> f.__code__.co_varnames
('local_var',)
```

Продолжая рассматривать пример мы обнаружим, что используется еще пара опкодов `LOAD_FAST/STORE_FAST`, которые используются в том случае, когда компилятор может вывести, что некоторые имена используются исключительно в локальной области видимости. Следует заметить, что в таком случае для хранения значений локальных переменных используется массив фиксированного размера (вместо словарей), что приводит к более быстрому поиску значения для соответствующего имени.

Теперь рассмотрим пример с использованием глобальных имен:

```python
>>> global_var = 'global variable'
>>> def f():
...     print(global_var)
... 
>>> dis.dis(f)
  2           0 LOAD_GLOBAL              0 (print)
              2 LOAD_GLOBAL              1 (global_var)
              4 CALL_FUNCTION            1
              6 POP_TOP
              8 LOAD_CONST               0 (None)
             10 RETURN_VALUE
```

Инструкции `*_GLOBAL` генерируются в том случае, когда компилятор может вывести, что некоторое имя используется внутри функции, но никогда не связывается с каким-либо объектом, как в нашем примере с переменной `global_var` и функцией `print`. Таким образом, `LOAD_GLOBAL` позволяет избежать поиска имени в локальном пространстве имен, а осуществляет поиск только в глобальном (`global_var`) и встроенном (`print`) пространствах имен (если имя не было найдено, то будет порождено исключение `NameError`).

Последняя пара опкодов, о которых мы поговорим, это `LOAD_DEREF` и `STORE_DEREF`. Python позволяет нам создавать вложенные функции. Давайте рассмотрим следующий простой пример:

```python
>>> def f():
...     some_variable = 'some_variable'
...     def h():
...         return some_variable
...     return h
...
>>> g = f()
>>> g()
some_variable
```

Может возникнуть вполне закономерный вопрос: «Как функция `h` может использовать значение перменной `some_variable`, если пространства имен, в которой эта переменная была создана, уже не существует?». Давайте посмотрим на список инструкций:

```python
>>> dis.dis(f)
  2           0 LOAD_CONST               1 ('some_variable')
              2 STORE_DEREF              0 (some_variable)

  3           4 LOAD_CLOSURE             0 (some_variable)
              6 BUILD_TUPLE              1
              8 LOAD_CONST               2 (<code object h at 0x10bd47d20 ...>)
             10 LOAD_CONST               3 ('f.<locals>.h')
             12 MAKE_FUNCTION            8
             14 STORE_FAST               0 (h)

  5          16 LOAD_FAST                0 (h)
             18 RETURN_VALUE

Disassembly of <code object h at 0x10bd47d20 ...>:
  4           0 LOAD_DEREF               0 (some_variable)
              2 RETURN_VALUE
```

Заметим, что переменная `some_variable` по отношению к функции `f` хоть и является локальной, но обрабатывается с помощью опкодов `LOAD_DEREF/STORE_DEREF`, а не `LOAD_FAST/STORE_FAST` как мы могли бы ожидать. Если на этапе компиляции становится ясно, что переменная используется во вложенных областях видимости, то для работы с ней не будут использованы обычные опкоды, вместо этого будет создан специальный объект [`cell`](https://docs.python.org/3/c-api/cell.html#cell-objects), который является контейнером для ссылки на другой объект (в нашем примере `some_variable`) и не зависит от сущестования стека объемлющей функции.

!!! note
    Для создания или изменения глобальных переменных в локальной области видимости используется ключевое слово [`global`](https://docs.python.org/3/reference/simple_stmts.html#global); аналогично для объемлющей облатси видимости существует ключевое слово [`nonlocal`](https://www.python.org/dev/peps/pep-3104/).

Если в функции присутствуют переменные, которые были определены в объемлющей области видимости, то такую функцию называют замыканием (closure):

```python
>>> g.__closure__
(<cell at 0x10c329f48: str object at 0x10c34e470>,)
>>> g.__closure__[0].cell_contents
'some_variable'
```
