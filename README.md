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
This could be done during initialisation with the `limit=None`, `interval=(int, str)`, `overflow_msg=None`
fields, where the tuple signifies `(time:int, unit:str)`.
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
with the new-`Logger` creation with the flag `copy=2` paremeter.

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
`test_logger_wipe_limit`. Minutes and hours are also tested during the 
time testing.


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
Proof of this lies in the [diff.txt](diff.txt) file,
which is basically a `git diff master 2-implement-limiter-class`
command run.

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

Below are further benefits and drawbacks of the current
implementation, when seen from different Alpha's within the 
SEMAT kernel framework.

### Requirements Alpha

The benefits of the current solution, when seen from the
perspective of requirements, is that it fulfils all specified technical 
requirements. Since testing is extensive and coverage is 100%,
one could argue that the algorithms are correct and thoroughly tested,
which again is another requirement.
Further potential requirements are the [way of contributing](https://loguru.readthedocs.io/en/stable/project/contributing.html),
which arguably are fulfilled to the extent seen appropriate.
The loose coupling of the class `Limiter` to the 
main class `Logger` is also beneficial in seen of
more general software engineering practices.

There are several potential drawbacks, as mentioned earlier.
One is the fact that testing has not been done on co-routines
and concurrent threads, which arguably is less convenient and
possibly redundant when testing the actual class `Limiter`.
It might be necessary to implement locks to prevent data races,
but as this has not been part of the main requirements,
this has not been highly prioritised. This is one drawback,
or rather *potential* lack of functionality.

### Software System Alpha

When seen from this Alpha, loose coupling towards the already
existing classes and functions is preferred.
The reason for this, is that it for each stakeholder (including
the owner of the project), user, and contributor, a separate 
class for the work improves the overview of the work.
Unit testing becomes easier and verification that the requirements
are fulfilled becomes arguably more doable.
Further, the very least modifications have been done in
regards to the `master` branch, which
could be seen in the [diff.txt](diff.txt) file.

Problems with the current implementation, when seen from this perspective,
might be a matter of naming and such practices.
Currently, for example, the input parameters in the `Logger` class
are separate in the form of `limit`, which is the count limit, 
`interval` which specifies the time interval, and `overflow_msg`, 
which specifies the message to be received when the limit is reached.
There might be thoughts and ideas on how one could do this differently,
but these are minor modifications and really is something that one could
hold a discussion about in the open source project.

### Opportunity & Stakeholder Alpha

These two Alphas are combined since the benefits of the current
solution concern both of them.
The main benefit is exactly the refactoring of the solution
to a separate class. For each stakeholder, and for impact on
future work on the project (the value here is seen as an opportunity),
such loose coupling and also high cohesion of the purpose of each
class is valued highly.
When such separate and well defined functionalities are refactored
to separate classes, such in this case the `Logger` and `Limiter`,
it is very easy to abstract away such functionality
and continue the work on the project as an *outsider* (i.e. 
someone with very little previous knowledge of the project).
If one can entrust the previous work and rely on that it is 
properly tested, which gets much easier when something
is a separate class, it gets much more feasible to also
start contributing and expanding the project.

Whether the actual solution holds the exact same standard as
the owner (Delgan) would want, it is hard to tell, so there
might obviously be some areas of improvement.
However, I truly believe that the design choices made in this
solution is beneficial for the stakeholders (users, contributors, etc)
and therefore drawbacks are a matter of further discussion and
input from such stakeholders.

## 7. Something Remarkable

I feel that my work is great and is thoroughly tested.
I am proud of my work, and believe I have done something
extraordinary in regards to the Assignment description.
