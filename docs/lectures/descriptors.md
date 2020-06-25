В этой лекции мы рассмотрим такой важный механизм как дескрипторы, а также разберемся с тем как же устроены методы класса.

## Свойства

Перед тем как говорить о дескрипторах давайте поговорим о свойствах (property). Рассмотрим следующий пример: пусть у нас есть класс «Профиль пользователя», который включает следующие поля: имя, фамилия и дата рождения:

```py
class UserProfile:

    def __init__(
        self,
        user: 'User',
        first_name: str='',
        sur_name: str='',
        bdate: Optional[datetime.date]=None
    ) -> None:
        self._user = user
        self.first_name = first_name
        self.sur_name = sur_name
        self.bdate = bdate

        self._age = None
        self._age_last_recalculated = None
        self._recalculate_age()

    def _recalculate_age(self) -> None:
        if self.bdate is None:
            return

        today = datetime.date.today()
        age = today.year - self.bdate.year

        if today < datetime.date(today.year, self.bdate.month, self.bdate.day):
            age -= 1

        self._age = age
        self._age_last_recalculated = today

    def age(self) -> Optional[int]:
        if self._age is None:
            return None
        if datetime.date.today() > self._age_last_recalculated:
            self._recalculate_age()
        return self._age


class User:

    def __init__(self, username, email, password):
        ...
        self.profile = UserProfile(self)
        ...
```

Обратите внимание, что возраст пользователя, если была указана дата рождения, вычисляется при каждом обращении:

```python
>>> guido.profile.bdate = datetime.date(1956, 1, 31)
>>> guido.age()
63
```

Было бы удобно обращаться к возрасту не как к методу, а как к обычному атрибуту, таким образом, чтобы клиентский код «не знал», что возраст является вычисляемым (управляемым) атрибутом. Для создания управляемых атрибутов в Python используются свойства (property), которые позволяют определить поведение конкретного атрибута при попытке доступа, изменения и его удаления:

```py
class UserProfile:

    @property
    def age(self) -> Optional[int]:
        if self._age is None:
            return None
        if datetime.date.today() > self._age_last_recalculated:
            self._recalculate_age()
        return self._age
```

```python
>>> guido.profile.bdate = datetime.date(1956, 1, 31)
>>> guido.age
63
>>> guido.age = 35
...
AttributeError: can't set attribute
```

Как уже было сказано, мы можем определить поведение атрибута с помощью свойств не только при попытке доступа к нему, но также при попытке его изменения или удаления, например:

```py
class UserProfile:

    @property
    def fullname(self) -> str:
        return f'{self.first_name} {self.sur_name}'.title()

    @fullname.setter
    def fullname(self, value: str) -> None:
        name, surname = value.split(' ', maxsplit=1)
        self.first_name = name
        self.sur_name = surname

    @fullname.deleter
    def fullname(self) -> None:
        self.first_name = ''
        self.sur_name = ''
```

```python
>>> user.profile.fullname = 'Guido Van Rossum'
>>> user.profile.first_name
'Guido'
>>> user.profile.sur_name
'Van Rossum'
>>> del user.profile.fullname
>>> user.profile.first_name
''
>>> user.profile.sur_name
''
```

Свойства являются одним из распространённых способов применения дескрипторов.
