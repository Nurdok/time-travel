[![Build Status](https://travis-ci.org/snudler6/time-travel.svg?branch=master)](https://travis-ci.org/snudler6/time-travel) [![Build status](https://ci.appveyor.com/api/projects/status/y13ewnvmj0muoapf/branch/master?svg=true)](https://ci.appveyor.com/project/snudler6/time-travel/branch/master)

# time-travel
python time libraries mocking

### time-travel is the fun and easy way to unit-test time sensetive modules.
### time-travel lets you write determenistic test for long and even infinite time scenarios.

## Usage

`TimeTravel` is context manager patching all* time and I\O related modules in a single line.
When inside the time travel context manager, the initial time is set to 
86,400.0 seconds in order to support windows (this is the lowest acceptable 
value by the OS). This value is exported via ``time_travel.MIN_START_TIME``.

\* All modules currently patched :) .

In order to improve TimeTravel's performance, you can give it the names of modules you want to patch (in list, tuple or single name). If you want to patch the current module, you can use the local variable `__name__`.

### Examples

#### Skip timeouts

Tests are deterministic and take no time with time travel. For example:

```python
with TimeTravel():  
    assert time.time() == 86400
    time.sleep(200)
    assert time.time() == 86600    
```

```python
with TimeTravel(modules_to_patch=__name__):
    assert datetime.today() == datetime.fromtimestamp(86400)
    time.sleep(250)
    assert datetime.today() == datetime.fromtimestamp(86650)
```

```python
import module1
import module2

with TimeTravel(modules_to_patch=['module1', 'module2']) as time_machine:
    time_machine.set_time(100000)
    module1.very_long_method()
    module2.time_sensitive_method()
```

#### Patching event based modules

Can Patch and determine future events for event based modules using select:

```python
with TimeTravel() as t:
    event = mock.MagicMock()
    t.events_pool.add_future_event(86402, event, t.events_types.select.WRITE)
    assert select.select([], [event], []) == ([], [event], [])
    assert time.time() == 86402
```

Or using ``poll`` (linux only):

```python
with TimeTravel() as t:
    fd = mock.MagicMock()
    t.events_pool.add_future_event(86402, fd, select.POLLIN)

    poll = select.poll()
    poll.register(fd, select.POLLIN | select.POLLOUT)

    assert poll.poll() == [(fd, select.POLLIN)]
    assert time.time() == 86402
```

## List of currently patched modules and functions

- time.time
- time.sleep
- datetime.date.today
- datetime.datetime.today
- datetime.datetime.now
- datetime.datetime.utcnow
- select.select
- select.poll (linux only)

### Add your own patches to time-travel

time-travel uses python "entry points" to add external patches to it.

#### Example
my_new_patcher.py:
```python
from time_travel.patchers.basic_patcher import BasicPatcher

class MyNewPatcher(BasicPatcher):
    def __init__(self, *args, **kwargs):
        pass
```

To add the new patcher automatically to time-travel, only add the new class to the "time_travel.patchers" entry point in your setup.py:
```python
from setuptools import setup

setup(
    name=...
    .
    .
    .
    entry_points={
        'time_travel.patchers' : [
            'my_new_patcher = my_new_patcher:MyNewPatcher',
        ],
    }
)
```


