В лекции [«ООП. Разрешение имен атрибтутов»](../lectures/attribute_lookup.md) мы упомянули о том, что поиск атрибутов осуществляется сначала в словаре класса, а затем в родительских классах в соответствии с [MRO](https://www.python.org/download/releases/2.3/mro/) (Method Resolution Order).

```python
def mro(cls):
    return [cls] + merge(list(map(mro, cls.__bases__)) + [list(cls.__bases__)])


def merge(seqs):
    if not seqs:
        return []
    if len(seqs) == 1:
        return seqs[0]
    res = []
    while seqs:
        for seq in seqs:
            head = seq[0]
            if not any(head in tail for _, *tail in seqs):
                break
        else:
            raise TypeError("Inconsistent hierarchy")
        res.append(head)
        for seq in seqs:
            if seq[0] == head:
                del seq[0]
        seqs = [s for s in seqs if s]
    return res
```