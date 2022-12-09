# tomlconfig

Parse TOML config file into a dataclass like object

## Usage

Annotate a class with `@configclass` decorator.
The decorator takes a validation function of type `T -> None` as its optional
argument.
The validation function should accept one argument (parsed config class) and
raise `ConfigError` exception if the config is not valid.

The configclass class can contain an attribute of any type as long as the type
annotation can be used as constructor and the constructor can accept a value
from TOML (`str`, `int`, `float`, `list`, `dict`).
Nested configclass classes are also supported (the `@configclass` decorator
creates the `__init__` method).

When attributes of type `list` or `dict` are overridden in subsequent files the
attribute value is expanded (with `list.extend` or `dict.update` respectively),
values of attributes of other types are replaced.

## Example

```toml
# /etc/example/config.toml
host = "localhost"
debug = True
files = ["a", "b"]
```

```toml
# /etc/example/config.d/10-custom.toml
host = "127.0.0.1"
files = ["b", "c"]
```

```python
from dataclasses import field
from tomlconfig import configclass, parse


@configclass
class Config:
    host: str = "0.0.0.0"
    port: int = 8080
    debug: bool = False
    files: list[str] = field(default_factory=list)


config = parse(Config, "/etc/example/config.toml", "/etc/example/config.d")


assert config.host == "127.0.0.1"
assert config.port == 8080
assert config.debug == True
assert config.files == ["a", "b", "c", "d"]
```
