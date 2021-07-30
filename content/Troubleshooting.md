In this page, we attempt to address known issues that could occur while deploying Lingua Franca.

#### [ProtoNoPacking.lf](https://github.com/icyphy/lingua-franca/blob/master/test/Python/ProtoNoPacking.lf) fails to compile in the Python target.
```
File "xxx/python3.7/site-packages/typing.py", line 1004, in __new__
self._abc_registry = extra._abc_registry
AttributeError: type object 'Callable' has no attribute '_abc_registry'
```
##### All Platforms
This case can arise when the `typing` package is installed in combination with Python 3.7. This can be remedied by:
```
sudo pip3 uninstall typing
```
and
```
pip3 uninstall typing
```

#### Single-quoted strings (Python)

The LFC lexer does not support single-quoted strings. This affects most strongly the Python target, where using single-quoted strings is very common. Please use double-quoted strings instead. This limitation will hopefully be lifted in the future.
