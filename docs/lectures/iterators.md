В этой лекции, основанной на серии [тетрадок](https://github.com/jmoldow/jmoldow-python3-async-tutorial) Jordan Moldow, мы рассмотрим такие понятия как итераторы и итерируемые объекты.

## Итераторы

[Итератор](https://refactoring.guru/ru/design-patterns/iterator) — это поведенческий паттерн проектирования, который даёт возможность последовательно обходить элементы составных объектов, не раскрывая их внутреннего представления.

В Python [Iterator](https://docs.python.org/3.7/library/collections.abc.html#collections.abc.Iterator)[^1] является экземпляром любого класса, который реализует магические методы `__iter__()` и `__next__()` (`next()` в Python 2.x)[^2].

`iter(iterator)` или, что тоже самое `iterator.__iter__()`, должен всегда возвращать `iterator`, указывая тем самым, что объект является итератором по отношению к себе.

`next(iterator)` эквивалентен вызову `iterator.__next__()`, а `value = next(iterator, default)` эквивалентен записи вида:

```python
try:
    value = iterator.__next__()
except StopIteration:
    value = default
```

`__next__()` это метод, который «вычисляет» и возвращает следующий элемент итератора. Когда итератор исчерпан, то есть, нет больше элементов, которые он может вернуть, порождается исключение `StopIteration`. Таким образом, `__next__()` изменяет внутреннее состояние итератора и по умолчанию (и соглашению) итераторы исчерпываются после одного полного прохода по ним.

Давайте рассмотрим пример простого итератора:

```python
import collections.abc

class RangeIterator(collections.abc.Iterator):
    
    def __init__(self, stop):
        if not isinstance(stop, int):
            raise TypeError('stop must be an int')
        if stop < 0:
            raise ValueError('stop must be >= 0')
        super().__init__()
        self.stop = stop
        self.next_item = 0 if stop > 0 else StopIteration()
    
    def __repr__(self):
        return f"<{self.__class__.__name__}({self.stop!r}): next_item={self.next_item!r}>"
    
    # __iter__ is already defined in `collections.Iterator` as
    #
    # def __iter__(self):
    #     return self
    
    def __next__(self):
        item = self.next_item
        if isinstance(item, StopIteration):
            raise StopIteration
        self.next_item += 1
        if self.next_item >= self.stop:
            self.next_item = StopIteration()
        return item
```

Примеры использования:

```python
>>> range_iterator = RangeIterator(2)
>>> range_iterator
<RangeIterator(2): next_item=0>
```

```python
>>> iter(range_iterator), iter(range_iterator) is range_iterator
(<RangeIterator(2): next_item=0>, True)
```

```python
>>> next(range_iterator), range_iterator
(0, <RangeIterator(2): next_item=1>)
```

```python
>>> next(range_iterator), range_iterator
(1, <RangeIterator(2): next_item=StopIteration()>)
```

```python
>>> import traceback
>>> try:
...     next(range_iterator)
... except TypeError:
...     traceback.print_exc()
Traceback (most recent call last):
...
StopIteration
```

```python
>>> next(range_iterator, 2)
2
```

## Итерирумые объекты (Iterables)

В Python [Iterable](https://docs.python.org/3.6/library/collections.abc.html#collections.abc.Iterable) является экземляром любого класса, у которого определен магический метод `__iter__()`. `Iterator` является подклассом `Iterable`.

`iter(iterable)` тоже что и `iterable.__iter__()` и всегда должен возвращать итератор для итерируемого объекта. По этому итератору затем можно итерироваться, чтобы получить элементы итерируемого объекта в заданном итератором порядке.

Итерируемые объекты (влючая все итераторы) позволяют осуществить только один полный проход по ним, но они могут быть переиспользованы. Для повторного прохода по элементам необходимо вызвать магический метод `__iter__()` (у итерируемого объекта), который вернет новый итератор.

Далее приведен простой пример итерируемого объекта:

```python
import collections.abc

class RangeIterable(collections.abc.Iterable):
    
    def __init__(self, stop):
        super().__init__()
        self.stop = stop
    
    def __repr__(self):
        return f"{self.__class__.__name__}({self.stop!r})"
    
    def __iter__(self):
        return RangeIterator(stop=self.stop)
```

Примеры использования:

```python
>>> range_iterable = RangeIterable(2)
>>> range_iterable
RangeIterable(2)
```

```python
>>> import traceback
>>> try:
...     next(range_iterable)
... except TypeError:
...     traceback.print_exc()
Traceback (most recent call last):
...
TypeError: 'RangeIterable' object is not an iterator
```

```python
>>> iter(range_iterable)
<RangeIterator(2): next_item=0>
```

```python
>>> iter(range_iterable) is range_iterable
False
```

```python
>>> iter(range_iterable) is iter(range_iterable)
False
```

```python
>>> next(iter(range_iterable))
0
```

## Итерирование с помощью цикла for

Python позволяет «вручную» итерироваться по итераторам и итерируемым объектам с помощью `iter()` и `next()`. Однако, Python имеет встроенную поддержку для автоматического итерирования с использованием цикла `for`:

```python
for item in iterable:
    # Выполнить какое-то действие над `item`, например вывести
    print(item)
```

Если мы проигнорируем семантику с использованием `continue`, `break` и `else`, то цикл `for` в общем виде может быть записан как:

```python
for TARGET in ITER:
    BLOCK
```

Что является синтаксическим сахаром для чего-то вроде:

```python
iterable = (ITER)
iterator = iter(iterable)
running = True

while running:
    try:
        TARGET = next(iterator)
    except StopIteration:
        running = False
    else:
        BLOCK
```

Заметим, что цикл `for` имеет специальную обработку для `StopIteration`. Цикл `for` осведомлен о [протоколе итератор](https://docs.python.org/3/library/stdtypes.html#iterator-types) (iterator protocol) и знает как поймать исключение `StopIteration` и интерпретирует его как конец итерирования:

```python
>>> for item in RangeIterable(2):
...     print(item)
0
1
```

или:

```python
def manual_simplified_for_loop(iterable, function):
    iterator = iter(iterable)
    running = True
    while running:
        try:
            item = next(iterator)
        except StopIteration:
            running = False
        else:
            function(item)
```

```python
>>> manual_simplified_for_loop(RangeIterable(2), print)
0
1
```

Давайте посмотрим на список инструкций, которые будут выполнены для первого примера:

```python
>>> import dis
>>> dis.dis("for item in RangeIterable(2): print(item)")
  1           0 SETUP_LOOP              24 (to 26)
              2 LOAD_NAME                0 (RangeIterable)
              4 LOAD_CONST               0 (2)
              6 CALL_FUNCTION            1
              8 GET_ITER
        >>   10 FOR_ITER                12 (to 24)
             12 STORE_NAME               1 (item)
             14 LOAD_NAME                2 (print)
             16 LOAD_NAME                1 (item)
             18 CALL_FUNCTION            1
             20 POP_TOP
             22 JUMP_ABSOLUTE           10
        >>   24 POP_BLOCK
        >>   26 LOAD_CONST               1 (None)
             28 RETURN_VALUE
```

Инструкция [`GET_ITER`](https://github.com/python/cpython/blob/3.7/Python/ceval.c#L2763) получает итератор для объекта, который находится на вершине стека (в нашем примере это `RangeIterable(2)`). [`FOR_ITER`](https://github.com/python/cpython/blob/3.7/Python/ceval.c#L2806), получает следующее значение из итератора (в нашем примере это 0 и 1) и помещает его на вершину стека. Затем выполняется тело цикла (печать переменной `item`) и все повторяется снова до тех пор пока итератор не будет исчерпан.

И последнее, что следует иметь ввиду, большое число как встроенных функций, так и функций стандартной библиотеки, принимают в качестве аргументов итерируемые объекты и затем итерируются по ним или «вручную» или с использованием цикла `for`, например:

```python
>>> list(RangeIterable(5))
[0, 1, 2, 3, 4]
>>> list(filter(None, RangeIterable(5)))
[1, 2, 3, 4]
```

## Магический метод `__getitem__`

Есть еще один способ итерироваться по объекту без реализации протокола итераторов, переопределив магический метод `__getitem__`[^3]:

```python
class RangeIterable:

    def __init__(self, stop):
        self.stop = stop

    def __getitem__(self, index):
        if not isinstance(index, int):
            raise TypeError
        if index < self.stop:
            return index
        raise IndexError

>>> for i in RangeIterable(2): print(i)
0
1
>>> r = RangeIterable(2)
>>> it = iter(r)
>>> next(it)
0
>>> next(it)
1
>>> next(it)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

С точки зрения байт-кода ничего не меняется: мы получаем итератор (`GET_ITER`), а затем последовательно получаем элементы (`FOR_ITER`). Чтобы понять как это работает давайте рассмотрим инструкцию `GET_ITER`:

```c
case TARGET(GET_ITER): {
    /* before: [obj]; after [getiter(obj)] */
    PyObject *iterable = TOP();
    PyObject *iter = PyObject_GetIter(iterable);
    //...
}
```

Нас здесь интересует только вызов функции [`PyObject_GetIter`](https://github.com/python/cpython/blob/3.8/Objects/abstract.c#L2565), которая возвращает итератор для объекта на вершине стека:

```c
PyObject *
PyObject_GetIter(PyObject *o)
{
    PyTypeObject *t = o->ob_type;
    getiterfunc f;

    f = t->tp_iter;
    if (f == NULL) {
        if (PySequence_Check(o))
            return PySeqIter_New(o);
        return type_error("'%.200s' object is not iterable", o);
    }
    else {
        PyObject *res = (*f)(o);
        // ...
        return res;
    }
}
```

В функции `PyObject_GetIter` проверяется установлен ли слот `tp_iter` у итерируемого объекта (т.е., был ли переопределен метод `__iter__`). Если слот не установлен, то проверяется реализует ли объект [протокол последовательностей](https://docs.python.org/3/c-api/sequence.html)[^4] и, если реализует, то происходит вызов функции [`PySeqIter_New`](https://github.com/python/cpython/blob/master/Objects/iterobject.c#L7), в которой создается новый итератор:

```c
typedef struct {
    PyObject_HEAD
    Py_ssize_t it_index;
    PyObject *it_seq; /* Set to NULL when iterator is exhausted */
} seqiterobject;

PyObject *
PySeqIter_New(PyObject *seq)
{
    seqiterobject *it;
    // ...
    it = PyObject_GC_New(seqiterobject, &PySeqIter_Type);
    if (it == NULL)
        return NULL;
    it->it_index = 0;
    Py_INCREF(seq);
    it->it_seq = seq;
    _PyObject_GC_TRACK(it);
    return (PyObject *)it;
}
```

## Что почитать?
- Модуль [itertools](https://docs.python.org/3/library/itertools.html)

[^1]:
    [PEP 234 -- Iterators](https://www.python.org/dev/peps/pep-0234/)
[^2]:
    [PEP 3114 -- Renaming iterator.next() to iterator.\_\_next\_\_()](https://www.python.org/dev/peps/pep-3114/)
[^3]:
    [Описание `__getitem__` в официальной документации](https://docs.python.org/3/reference/datamodel.html#object.__getitem__)
[^4]:
    ==In Python code, when `__getitem__` is defined, when the class is instanticated, it calls `type_call` in `Objects/typeobject.c`. It assigns the address of `as_sequence` of a `PyHeapTypeObject` to the class's `tp_as_sequence` field. The `PySequenceMethods` struct it points to is initially all zeros, so `tp_as_sequence->sq_item` is `NULL`. Then, in `update_one_slot` called from `fixup_slot_dispatchers` called from `type_new` as the type's `tp_new` field called from `type_call`, it checks if `__getitem__` is defined. If that is true, it assigns the `slot_sq_item` function to `tp_as_sequence->sq_item`, to make `PySequence_Check` return `True`.==