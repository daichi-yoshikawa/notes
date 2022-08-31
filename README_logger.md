https://docs.python.org/3/howto/logging.html

## Level

Level | When it's used
--- | ---
DEBUG | Detailed information, typically of interest only when diagnosing problems.
--- | ---
INFO | COnfirmation that things are working as expected.
--- | ---
WARNING | An indication that something unexpected happened, or indicative of some problem in the near future. The software is still working as expected.
--- | ---
ERROR | Due to a more serious problem, the software has not been able to perform some function.
--- | ---
CRITICAL | A serious error, indicating that the program itself may be unable to continue running.

## Configur logging system
1. Creating loggers, handlers, and formatters explicitly using Python code that calls the configuration methods.
2. Creating a logging config file and reading it using the *fileConfig()* function.
3. Creating a dictionary of configuration information and passing it to the *dictConfig()* function.

## Configure logger through logging
### Logging to a file
logging.basicConfig configures logger globally.
```
logging.basicConfig(
    filename='example.log',
    encoding='utf-8', # Caution: This argument is for Python 3.9+
    filemode='w', # If 'a', logs are appended to existing log file.
    format='%(levelname)s:%(asctime)s - %(message)s',
    datefmt='%m/%d/%Y %I:%M:%S %p',
    level=logging.DEBUG,)
```

### Logging from multiple modules
```
# main.py
import logging
import mylib

if __name__ == '__main__':
  logging.basicCOnfig(...)
  logging.info('Started')
  mylib.do_something()
  logging.info('Finsihed')
```

```
# mylib.py
import logging

def do_something():
  logging.info('Doing something')
```

### Default format of log message
```
levelname:loggername name:message
```

## Configure logger by Python code
Each logger instance has a name, and they are conceptually arranged in a namespace hierarchy. For example, a logger named 'scan' is the parent of loggers 'scan.text'.

A good convention to use when namoing loggers is to use module-level logger as below, since logger names can track the package/module hierarchy.
```
# mylib.py
logger = logging.getLogger(__name__) # __name__ == '__mylib__' # getLogger returns a reference to a logger instance with the specified name if it's provided, or root if not.
```

* The root of the hierarchy of loggers is called the root logger, whose name is printed as 'root' in the logged output.
* Destinations of log messages are served by *handler* classes.
* Child loggers propagate message up to the handlers associated with their ancestor logger. It's sufficient to configure handlers for a top-level logger and create child loggers as needed.

### logging.Formatter
```
formatter = logging.Formatter(
    fmt='%(asctime)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
    style='%')
```

### logging.Handler
```
handler = logging.StreamHandler()
handler.setLevel(logging.DEBUG/NOTSET/INFO/WARNING/ERROR/CRITICAL)
handler.setFormatter(formatter)
logger.addHandler(handler)
```

## Configure logger with config file
logging.conf
```
[loggers]
keys=root,example

[handlers]
consoleHandler

[formatters]
keys=exampleFormatter

[logger_root]
level=DEBUG
handlers=consoleHandler

[logger_example]
level=DEBUG
handlers=consoleHandler
qualname=example
propagate=0

[handler_consoleHandler]
class=StreamHandler
level=DEBUG
formatter=exampleFormatter
args=(sys.stdout,)

[formatter_simpleFormatter]
format=%(asctime)s - %(name)s [%(levelname)s] %(message)s
```

```
...
logging.config.fileConfig('logging.conf')
logger = logging.getLogger('example')
...
```

## Configure logger with dictConfig and yaml
```
$ pip install pyyaml
```

```
# logger.yaml
version: 1
formatters:
  default:
    datefmt: '%Y-%m-%d %H:%M:%S'
    format: '%(asctime)s %(name)s[%(levelname)s]: %(message)s'
handlers:
  console:
    class: logging.StreamHandler
    level: DEBUG
    formatter: default
    stream: ext://sys.stdout
  file:
    class: logging.FileHandler
    level: INFO
    formatter: default
    filename: ./log/application.log
    encoding: utf8
    mode: w # or a if you'd like to clear log file every time when your app runs
loggers:
  default:
    level: DEBUG
    handlers: [console,file]
    propagate: no
root:
  level: DEBUG
  handlers: [console,file]
```
