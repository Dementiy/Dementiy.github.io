Эта первая практическая работа, которая посвящена базовым конструкциям языка Python. Также в этой работе описан общий подход к выполнению заданий.

Все исходники к работам можно найти в [репозитории](https://github.com/Dementiy/pybook-assignments) (следите за его обновлениями). Чтобы получить все шаблоны к работам достаточно выполнить следующую команду:

!!! note
    Описание работы с командой `git` читай в разделе [«Настройка рабочего окружения»](../assignments/setup_env.md).

```sh
$ git clone https://github.com/Dementiy/pybook-assignments.git
```

Все дальнейшие изменения в репозитории можно получать с помощью команды `git pull` (её рекомендуется использовать каждый раз, когда вы приступаете к выполнению новой работы):

```sh
$ git pull
```

## Прежде чем приступить к выполнению работы

Перед тем как начать выполнять задания не забудьте перейти в рабочую директорию и активировать ваше виртуальное окружение:

```sh
$ cd путь/к/рабочей_директории
$ workon cs102
```

Также создадим ветку разработки (подробнее про git workflow можно прочитать [тут](https://habrahabr.ru/post/106912/)):

```sh
(cs102) $ git checkout -b develop master
Switched to a new branch 'develop'
```

Теперь создадим ветку новой функциональности:
```sh
(cs102) $ git checkout -b feature/caesar develop
Switched to a new branch 'feature/caesar'
```

И, наконец, создадим папку, в которой будем хранить все файлы, относящиеся к первой работе:

```
(cs102) $ mkdir homework01
```

## Шифр Цезаря

[Шифр Цезаря](https://ru.wikipedia.org/wiki/Шифр_Цезаря) является одним из самых простых методов шифрования. Для кодирования сообщения все буквы алфавита сдвигают на три символа вперед:

```
A -> D, B -> E, C -> F, и так далее
```

Сдвиг трёх последних букв алфавита:

```
X -> A, Y -> B, Z -> C
```

Используя шифр Цезаря, слово `PYTHON` будет закодировано следующим образом:

```
PYTHON
||||||
SBWKRQ
```

Вам необходимо написать тело для следующих двух функций:

```python
def encrypt_caesar(plaintext: str) -> str:
    """
    >>> encrypt_caesar("PYTHON")
    'SBWKRQ'
    >>> encrypt_caesar("python")
    'sbwkrq'
    >>> encrypt_caesar("Python3.6")
    'Sbwkrq3.6'
    >>> encrypt_caesar("")
    ''
    """
    # PUT YOUR CODE HERE
    return ciphertext


def decrypt_caesar(ciphertext: str) -> str:
    """
    >>> decrypt_caesar("SBWKRQ")
    'PYTHON'
    >>> decrypt_caesar("sbwkrq")
    'python'
    >>> decrypt_caesar("Sbwkrq3.6")
    'Python3.6'
    >>> decrypt_caesar("")
    ''
    """
    # PUT YOUR CODE HERE
    return plaintext
```

!!! hint
    Воспользуйтесь встроенными функциями `ord()` и `chr()`. Функция `ord()` позволяет получить код указанного символа, а `chr()` работает наоборот - возвращает символ по его коду.

!!! note
    О кодировках можно почитать [тут](http://kunststube.net/encoding/) и [тут](http://farmdev.com/talks/unicode/).

В результате переменные `ciphertext` и `plaintext` должны содержать зашифрованное и расшифрованное сообщения соответственно.

В папке `homework01` создайте файл с именем `caesar.py` и шаблоном приведенным выше. Сделайте коммит с первоначальным шаблоном:

```sh
(cs102) $ git add homework01/caesar.py
(cs102) $ git commit -m "Добавлен шаблон для шифра Цезаря"
```

По завершении работы над каждой функцией не забудьте также сделать коммит, например:

```sh
(cs102) $ git commit -am "Реализована функция encrypt_caesar()"
```

и аналогично:

```sh
(cs102) $ git commit -am "Реализована функция decrypt_caesar()"
```

Проверить работу функций можно с помощью примеров, приведенных в [доктестах](https://docs.python.org/3.5/library/doctest.html) (текст внутри функции, который заключен в тройные кавычки и похож на работу с интерпретатором в интерактивном режиме). Запустить доктесты можно с помощью следующей команды (при условии, что файл с программой называется `caesar.py`):

```sh
$ python3 -m doctest -v caesar.py
```

Также обратите свое внимание на официальное руководство по стилю [pep8](https://www.python.org/dev/peps/pep-0008/) (некоторые пояснения по оформлению кода можно найти [здесь](https://github.com/Dementiy/pydonts)). Соответствие вашего кода PEP8 можно узнать «прогнав» исходный код вашей программы через утилиту `pep8`.

Если все тесты были пройдены успешно, то объедините (merge) ваши изменения с веткой `develop`:

```sh
(cs102) $ git checkout develop
(cs102) $ git merge --no-ff feature/caesar
```

!!! note
    Вы можете воспользоваться приложением [Source Tree](https://www.sourcetreeapp.com/) для наглядного отслеживания вносимых изменений.

В качестве практики измените ваши функции так, чтобы размер сдвига был произвольным:

```
encrypt_caesar(plaintext, shift)
decrypt_caesar(plaintext, shift)
```

## Шифр Виженера

[Шифр Виженера](https://ru.wikipedia.org/wiki/Шифр_Виженера) очень похож на шифр Цезаря, за тем исключением, что каждый символ сообщения сдвигается на определяемое ключом значение. Ключ - это слово, каждый символ которого указывает на сколько позиций должен быть сдвинут соответствующий символ в шифруемом сообщении. Так, `A` означает сдвиг на `0` символов, `B` на `1` и т.д.

Если длина ключа меньше длины слова, подлежащего шифрованию, то ключ повторяется необходимое число раз, например:

```
Простой текст: ATTACKATDAWN
Ключ: LEMONLEMONLE
Зашифрованный текст: LXFOPVEFRNHR
```

Ваша задача написать тело для следующих двух функций так, чтобы переменные `ciphertext` и `plaintext` содержали зашифрованное и расшифрованное сообщения соответственно.

```python
def encrypt_vigenere(plaintext: str, keyword: str) -> str:
    """
    >>> encrypt_vigenere("PYTHON", "A")
    'PYTHON'
    >>> encrypt_vigenere("python", "a")
    'python'
    >>> encrypt_vigenere("ATTACKATDAWN", "LEMON")
    'LXFOPVEFRNHR'
    """
    # PUT YOUR CODE HERE
    return ciphertext


def decrypt_vigenere(ciphertext: str, keyword: str) -> str:
    """
    >>> decrypt_vigenere("PYTHON", "A")
    'PYTHON'
    >>> decrypt_vigenere("python", "a")
    'python'
    >>> decrypt_vigenere("LXFOPVEFRNHR", "LEMON")
    'ATTACKATDAWN'
    """
    # PUT YOUR CODE HERE
    return plaintext
```

!!! note
    Обратите внимание, что символы `A` и `a` в ключе не оказывают никакого влияния на шифруемое сообщение. Если же в качестве ключа мы будем использовать `C` или `c`, то получим шифр Цезаря.

Перед выполнением задания не забудьте создать новую ветвь функциональности `feature/vigenere` и сделать коммит с шаблоном (файл с шаблоном должен быть размещен в каталоге `homework01` под именем `vigener.py`):

```sh
(cs102) $ git checkout -b feature/vigener develop
(cs102) $ git add homework01/vigener.py
(cs102) $ git commit -m "Добавлен шаблон для шифра Виженера"
```

По окончании работы над каждой функцией не забудьте сделать соответствующие коммиты, как в примере с шифром Цезаря, а также объединенить изменения с веткой `develop`:

```sh
(cs102) $ git checkout develop
(cs102) $ git merge --no-ff feature/vigener
```

## RSA шифрование

Одним из современных методов шифрования является алгоритм шифрования RSA, названный так по первым буквам фамилий его авторов.

Мы не будем здесь вдаваться в [подробности работы](http://kpfu.ru/docs/F366166681/mzi.pdf) этого алгоритма (хотя и рассмотрим техническую часть), но [следующего объяснения](https://www.quora.com/How-do-you-explain-how-an-RSA-public-key-works-to-a-child) должно быть достаточно для понимания принципов шифрования с открытым ключом:

!!! quote
    ![](https://qph.fs.quoracdn.net/main-qimg-987fa169ca2f92dfaad606b2217c3bd1-c)

    Show your kid a padlock. This is a kind of lock that locks when you click it (i.e it doesn't require a key) but requires the key to open the lock.
    
    So, I can send these padlocks to all my friends who want to communicate with me. I will send them only the lock but will keep the key with me.
    
    My friends can write me messages, put it in a box, lock it with my padlock (by clicking it) and send it to me, even over high risk networks. If the box is intercepted, it's contents will not be compromised since I still have the key with me.
    
    When the box reaches me, I can open my padlock with my key and read the contents. This way, I can send padlocks (public keys) to people outside which they can use to lock boxes (encrypt messages) without being in danger of the contents being compromised as the padlock key (the private key) is always with me and never exchanged over the network.

Работу алгоритма можно разбить на три шага:

1. Генерация ключей
2. Шифрование
3. Расшифровка

От вас в этом задании требуется выполнить только шаг генерации ключей, остальные два шага уже даны (см. исходники к работе).

На этапе генерации создаётся два ключа: открытый (public key, с помощью которого каждый сможет зашифровать сообщение и отправить его нам) и закрытый (private key, которым мы можем расшифровать полученные сообщения). Для этого выбирается два [простых числа](https://ru.wikipedia.org/wiki/Простое_число) `p` и `q`. Позволим пользователю вводить эти числа, но их необходимо проверять на простоту. Для этого напишем функцию, но предварительно создадим новую ветку функциональности и сделаем коммит с шаблоном для этого задания:

```sh
(cs102) $ git checkout -b feature/rsa develop
(cs102) $ git add homework01/rsa.py
(cs102) $ git commit -m "Добавлен шаблон для RSA шифрования"
```

Теперь можно начать работать над функцией `is_prime(n)`:

```python
def is_prime(n: int) -> bool:
    """
    >>> is_prime(2)
    True
    >>> is_prime(11)
    True
    >>> is_prime(8)
    False
    """
    # PUT YOUR CODE HERE
    pass
```

Если вы закончили работу над функцией `is_prime(n)` - сделайте коммит:

```sh
(cs102) $ git commit -am "Реализована функция is_prime(n)"
```

После того как были выбраны два простых числа находится их произведение `n = p * q` (по ходу объяснения заменяйте комментарий со словами `PUT YOUR CODE HERE` в приведенной ниже функции на соответствующее решение).

```python
def generate_keypair(p: int, q: int) -> Tuple[Tuple[int, int], Tuple[int, int]]:
    if not (is_prime(p) and is_prime(q)):
        raise ValueError('Both numbers must be prime.')
    elif p == q:
        raise ValueError('p and q cannot be equal')

    # n = pq
    # PUT YOUR CODE HERE

    # phi = (p-1)(q-1)
    # PUT YOUR CODE HERE

    # Choose an integer e such that e and phi(n) are coprime
    e = random.randrange(1, phi)

    # Use Euclid's Algorithm to verify that e and phi(n) are comprime
    g = gcd(e, phi)
    while g != 1:
        e = random.randrange(1, phi)
        g = gcd(e, phi)

    # Use Extended Euclid's Algorithm to generate the private key
    d = multiplicative_inverse(e, phi)
    # Return public and private keypair
    # Public key is (e, n) and private key is (d, n)
    return ((e, n), (d, n))
```

Затем вычисляется функция Эйлера по формуе:

$$\phi = (p-1)(q-1)$$

Далее выбирается число `e`, отвечающее следующим критериям:

* `e` — простое;
* `e < phi`;
* `e` [взаимно простое](https://ru.wikipedia.org/wiki/Взаимно_простые_числа) с `phi`.

Определить, являются ли числа взаимно простыми можно с помощью алгоритма Евклида. Для этого необходимо вычислить наибольший общий делитель (НОД) и проверить, **равен ли он единице**. На этом этапе вашей задачей является реализация данного алгоритма:

```python
def gcd(a: int, b: int) -> int:
    """
    >>> gcd(12, 15)
    3
    >>> gcd(3, 7)
    1
    """
    # PUT YOUR CODE HERE
    pass
```

Не забудьте закоммитить реализацию функции `gcd(a, b)`:

```sh
(cs102) $ git commit -am "Реализована функция поиска НОД"
```

Заключительным этапом на шаге генерации ключей является вычисление `d` такого что `d * e mod phi = 1`. Для его вычисления используется расширенный (обобщенный) алгоритм Евклида (см. стр. 23 [этого учебного пособия](http://kpfu.ru/docs/F366166681/mzi.pdf) с подробными объяснениями).

```python
def multiplicative_inverse(e: int, phi: int) -> int:
    """
    >>> multiplicative_inverse(7, 40)
    23
    """
    # PUT YOUR CODE HERE
    pass
```

Таким образом, полученные пары `(e,n)` и `(d,n)` являются открытым и закрытым ключами соответственно.

Снова закоммитьте изменения:

```sh
(cs102) git commit -am "Реализованы функции multiplicative_inverse() и generate_keypair()"
```

Не забудьте сделать коммит для объединения изменений с веткой `develop`:

```sh
(cs102) $ git checkout develop
(cs102) $ git merge --no-ff feature/rsa
```

## После выполнения всех заданий

После выполнения всех заданий создайте новый релиз:

```sh
(cs102) $ git checkout -b release-1.0 develop
(cs102) $ git commit -m "Релиз 1.0"
```

После чего закройте ветвь релиза:

```sh
(cs102) $ git checkout master
(cs102) $ git merge --no-ff release-1.0
(cs102) $ git tag -a 1.0
```

Соответствующие ветви функциональности можно удалить, например:

```sh
(cs102) $ git branch -d feature/caesar
```
