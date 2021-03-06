# Amet

Amet is a Python 2/3 library to read and write complex configuration from and to environment variables.

## Motivation

Some time ago I had to deploy an application to Heroku. This meant that I had to read the application's configuration from environment variables instead of the JSON files I was previously using for configuration. I wrote this library so that I could dump those configuration files into environment variables that I could easily read and assemble back into the original structure.

This means that you do not need to give up the benefits of using a structured JSON file while using environment variables for configuration.

## How to use

First of all, install the library with a simple `pip install amet`.

### Loading - The `load_from_environment` function

When loading a configuration object from the environment, the first thing that is needed is, of course, to define what needs to be read. The first thing we need to create is a dictionary to hold our configuration values as well as specify what type each value is. We will call this dictionary our prototype:

```python
CONFIGURATION_PROTOTYPE = {
    'database': {
        'hostname': str,
        'port': int,
        'username': str,
        'password': str,
        'use_ssl': bool
    }
}
```

In order to load configuration values from environment variables, the `load_from_environment` function is provided. This function will iterate over our prototype and read environment variables that can be used to fill it. The format of the environment variables that are read can be customized by setting the `prefix` and `separator` string values and the `force_uppercase` boolean value

`load_from_environment(proto, prefix='', force_uppercase=True, separator='_')`

The environment variables that will be read are expected to have a particular format:

`<prefix><separator><key-1><separator>...<key-n>`

where `key-1` through `key-n` correspond to the keys of nested dictionaries. If `force_uppercase` is set, the environment variables will be converted to uppercase.

For example, out previously defined configuration prototype would be filled with the value from the following environment variables (assuming `prefix` is left empty, `separator` is `_` and `force_uppercase` is `True`):

* `DATABASE_HOSTNAME`
* `DATABASE_PORT`
* `DATABASE_USERNAE`
* `DATABASE_PASSWORD`
* `DATABASE_USE_SSL`

`load_from_environment` will always return a new dictionary, the prototype will remain unchanged.

### Conversion of values

Amet will attempt to convert values to Python primitive types. If the value for a given key is of type `int`, `float`, or `bool` (or the types themselves), upon reading the corresponding environment variables, Amet will attempt to parse those values. If the type of the value is `str` or `NoneType` the read value if left as is (which is `str`).

In the previous example, `database.hostname` is assumed to be a string, however, `database.password` will be converted to `int` as that is the value given to that key. If a number such as `0` or `1` had been set then the value of `integer` would have also been converted.

```python
CONFIGURATION_PROTOTYPE = {
    'database': {
        'hostname': str,    # Read as str
        'port': int,        # Read as int
        'username': None,   # Read as str
        'password': None,   # Read as str
        'use_ssl': bool     # Read as bool
    }
}
```

Possible values for `True` are `T`, `True`, `TRUE`, `true`, `Y`, `Yes`, `YES`, `yes` and `1`. The same goes for `False`.

### Writing - The `dump` function

The `dump` function is also provided.

`dump(config, prefix='', force_uppercase=True, separator='_')`

This function will return a dictionary of the key-value pairs that should be set as environment variables. If we called `dump` with the dictionary in the previous example, the output would be:

```python
{
	`DATABASE_HOSTNAME`: ...
	`DATABASE_PORT`: ...
	`DATABASE_USERNAE`: ...
	`DATABASE_PASSWORD`: ...
	`DATABASE_USE_SSL`: ...
}
```

### Exceptions

When a problem occurs loading or dumping a dictionary, errors may be raised.

* `IncompatibleTypeError`, a subclass of `TypeError` will be raised if a key in the dictionary is not a `str` object or if a value is not a `dict`, `str`, `int`, `float` or `bool` value.
* `KerError` is raised if an environment variable cannot be read.
* `ValueError` is raised if a value cannot be parsed, for example if an integer is expected but the value is a string not corresponding to a number.

## TODO

- [x] Improve this README.
- [ ] Add some useful examples for `dump` such as setting those variables in Heroku.
- [ ] Add support for default values.spell
- [ ] Check for possible name clashes when loading or dumping variables.
- [ ] Improve unit tests.

## Contributing

If you find any issues feel free to report them [here](https://github.com/maurodec/amet/issues/new). If you want you can also fork the project and submit a pull request.
