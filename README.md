# P+ individual contribution

Original report is found [here](https://github.com/zsabbagh/dd2480-assignment-4).

P+ work for Assignment 4. Main files, from branch [`2-implement-limiter-class`](https://github.com/Glace97/loguru/tree/2-implement-limiter-class):
- [`loguru/_logger.py`](https://github.com/Glace97/loguru/blob/2-implement-limiter-class/loguru/_logger.py)
- [`loguru/_limiter.py.py`](https://github.com/Glace97/loguru/blob/2-implement-limiter-class/loguru/_limiter.py)
- [`tests/test_limiter.py`](https://github.com/Glace97/loguru/blob/2-implement-limiter-class/tests/test_limiter.py)

Note that the time spent in the original assignment report
does *not* include this work.
This work is for the purpose of achieving P+.

## 2. Work in relation to project

The chosen structure was such that as little as possible
would change in the main logger, i.e. `_logger.py` file.
By creating a separate class, it would be easy to ensure functionality.
Furthermore, by doing so, it was simple to just add the functionality
of adding a `_limit` field in the main class.
This could be done during initialisation with the `limit=(None, None)`
field, where the tuple signifies `(count, interval)`.
There is a function added for adding a `limit` after creation:

```python
     def limit(self, count=None, interval=None, sliding=False, message=None, copy=0):
        limiter = Limiter(count, interval, sliding, message=message)
        if copy < 1:
            self._limit = limiter
            return self
        # check if soft-copy
        core = Core() if copy == 1 else self._core
        new_logger = Logger(core, *self._options)
        new_logger._limit = limiter
        return new_logger
```

The code above does limit the current logger per default, but one could create a new logger if one would like, separating logging.
Further alteration includes `opt` to take a `limit` field and
also the main `_log` function as following:

```python
    if self._limit.reached(message):
        message = self._limit.get_overflow_message()
        if message is None:
            return
```

This is the only added lines.
What is great about this, is the fact that there is a minimum
of lines added, but the functionality is the same as described in the 
issue.

There are, however, areas of improvement seen from a broader perspective.
One could add a limit per-`sink`, however this would be more complex as
one then would, most likely, need to restructure the project extensively.
There would be needed a functionality of 'wrapping' a `Limiter`
object within a `sink`.
This is not a simple task, as it would require a more profound
understanding of the ins-and-outs of the project.
Further, it is worth mentioning that this, really, is already possible
with the new-`Logger` creation with the flag `copy=2`, etc.

## 3. Relevant Test Cases

| ID | Title | Description |
|---|---|---|
| 1 | New feature | The new feature will be implemented as a new function named `limit`, which is callable from a `Logger` object. |
| 2 | Object returned | The implemented `limit` function returns a new `Logger` object, wrapping the existing object. |
| 3 | Functionality | Given a number of logs (a `frequency_limit` or N), and given a `time_limit` in minutes, no more than `frequency_limit` logs should be logged within a period of `time_limit`. |
| 4 | Time restriction | The specified `time_limit` shall be a "moving window", meaning it is not a sequential restriction but a rate of how often the logger can log per minute. |
| 5 | Logging levels | The returned limited object should be able to limit all logging functions such as `warning`, `debug`, etc |
| 6 | Overflow | When `frequency_limit` has been reached, an overflow message is logged once, all other logging calls will be surpressed until `time_limit` allows to log again. |
| 7 | Testing | Write unit tests for implemented functionality, attempting at 100% code coverage. |

All added test cases in [`tests/test_limiter.py`](https://github.com/Glace97/loguru/blob/2-implement-limiter-class/tests/test_limiter.py).
verifies the functionality in accordance with specification.
Below are the current test cases and functions and its motivation:
1. Limiting. `test_limiter_count`. This verifies the simple
functionality requirement 1 of limiting a message sent multiple times. `test_logger_count_limit`. This verifies that a random
log file and random hexstring as a message does not appear
more than the expected amount of times in the output file,
unbounded by time. Random values added to `.log` file under
`/tmp`.
2. New object. `test_logger_limit_copy`. This verifies the 2nd requirement, i.e. one could create a new object.
3. Time. `test_limiter_time`. This verifies
functionality 3 of being able to limit messages to 'intervals',
i.e. not bound solely by count. When an interval is passed,
the basic functionality is to reset the count. See `window`-test for
alternative. `test_logger_time_limit` verifies that a time limit
added does indeed limit the amount of output messages.
Once again, this is done with random values.
4. Window. `test_window`. Verification of the sliding window interval
algorithm, i.e. how the interval is constantly pushed forward.
This is done by first adding a message, then waiting for half the time,
adding another message, then passing the window and seeing that only
the message that was added first is "pushed out" of the window. Also, `test_logger_window` does this testing but on the `logger`. Similar to the `window`-test, but this 
is associated with a `logger`, i.e. that the logger does indeed
write as many values as expected.
5. Levels. `test_logger_levels` specifically tests that all logging functions are limited.
6. Overflow message. `test_logger_overflow_message` tests this functionality by
asserting that there are only one overflow message.
7. 100% code coverage. `wipe()` for wiping `__tracker` is tested by
`test_logger_wipe_limit`. Currently 97%, the only non-tested ones are minutes and hours.
The reason for this is that it would be very inefficient tests, cut the code is very
simple and could be reasoned about.


## 4. Clean Code

The code is clean, and verified as such using
`pylint` tool which is also used in the project.
Latest check gave `10.0/10.0` score of code
formatting when checking implemented limiting class
[`loguru/_limiter.py`](https://github.com/Glace97/loguru/blob/2-implement-limiter-class/loguru/_limiter.py)
and tests [`tests/test_limiter.py`](https://github.com/Glace97/loguru/blob/2-implement-limiter-class/tests/test_limiter.py).
It follows the requirements since it:
1. removes but does not comment out obsolete code and
2. does not produce extraneous output that is not required for your task (such as debug output) and
3. does not add unnecessary whitespace changes (such as adding or removing empty lines)

As mentioned, `pylint` verifies linting passes.

## 6. Benefits and Drawbacks

One of the benefits with the current implementation
and solution of the issue, is the fact that it is refactored
to an own class `Limiter`.
The purpose of this is to enabled unit-testing of the 
code and make the functionality more modifiable.
Also, as discussed in section 2 concerning work
in relation to the project, creating a separate class
would enable to alter `sink`'s to be wrapped in a 
class with a `sink`-specific limiter.
The drawbacks of such a change, however, is that
it might cause problems with the current design of the
project.
However, it is possible to create new objects of type `Logger`
which essentially enables copying handlers so that
one could limit only on a specific instance of the logger.

Other drawbacks of the current design choice is
the fact that it limits messages and logs
based on the hash of its contents, which might,
for bigger projects, be inefficient memory-wise.
Furthermore, the window-sliding functionality
is linear in time-complexity in regards to the `count`
limit.
The alternative and default is to make `sliding=False`,
which is exactly a default in the purpose of improving
performance.
If count is high, the `Limiter` would be time- and space
inefficient.

Another quite important note on the current design,
is that it limits on identical messages.
However, if the message includes a timestamp or such,
it would not be limited as a result of this.
This is really an issue on use rather than design problem,
i.e. that the user should have a sufficient understanding of
using the library before logging such 'distinct' messages.

## 7. Extraordinary

I feel that my work is great and is thoroughly tested.
I am proud of my work, and believe I have done something
extraordinary in regards to the Assignment description.
