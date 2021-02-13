benchfunc.py
===

a python3 module for quickly and easily benchmarking the elapsed time and memory usage of python functions


## quick start
make sure you have *memory-profiler* installed:

```sh
pip install -r requirements.txt
```

1. import benchmark
2. define a function to test
3. use `benchmark.run(func, [repeat], [name])`

```python
import benchmark

def search_list():
  "two" in ["one", "two", "three"]

results = benchmark.run(search_list, repeat=1000)
print(results)
```
*outputs*
```
<TestRun 'search_list'
  runs:     1,000
  avg time: 0.0002124s
  avg mem:  0.0Mib>
```

## benchmark.run()
this is the easiest way to use this tool. it generates a `TestRun` object, returns it, and adds it to `benchmark.history` for later review or analysis.

```python
run(func: Callable, repeat: int = 10, name: str = None) -> TestRun
```
pass in the function to be tested. optionally, specify the number of times to repeat the function and the name to use. if a name is not provided, it defaults to the function name ("&lt;lambda>" for lambda functions)

## TestRun
a `TestRun` object is returned by either `benchmark.run()` or `benchmark.Test.run()`. it contains all of the results from benchmarking a function.

### attributes
| attribute | type | description |
| --------- | ---- | ----------- |
| _mem_raw  | List[float] | the raw slices of the program's memory usage (MiB) during the function call (the entire program's memory usage) |
| _mem      | List[float] | "normalized" slices of the program's memory usage (MiB) during the function call (initial memory usage subtracted from subsequent memory usage values) |
| func      | Callable    | the benchmarked function |
| name      | str         | the function name *or* the name passed with `.run(func, name="foobar")` |
| time      | float       | the average time (seconds) of all runs |
| memory    | float       | the peak memory usage (MiB) of all runs |
| stdout    | str         | output is suppressed by `.run()` and stored here for later retrieval |
| repeat    | int         | number of tests run |

### initialization
```python
__init__(self,
    func: Callable,
    name: str = None,
    repeat: int = 1,
    mem: List[float] = [],
    time: float = 0.0,
    stdout: str = ""
)
```

a `TestRun` object is *not* meant to be initialized on its own! it is meant to be generated by either `benchmark.Test.run()` or `benchmark.run()`

### stdout
during a `.run()` call, all output to `sys.stdout` (e.g. `print()` statements) is temporarily redirected so that output can be captured. you can access it later using `TestRun.stdout`

## Test

a `Test` object is initialiezd with a function name and some default values. calling `Test.run()` will add a new `TestRun` object to `Test.history` and `benchmark.history` for later retrieval and then return it.

### attributes
| attribute | type | description |
| --------- | ---- | ----------- |
| _func     | Callable      | the benchmarked function |
| _repeat   | int           | the default number of times to run the function |
| _running  | bool          | boolean representing if a benchmark is currently running |
| name      | str           | the default name to use for tests |
| history   | List[TestRun] | all `TestRun`s generated by `Test.run()` |

### methods
```python
run(repeat: int = 100, name: str = None) -> TestRun
```
returns a new `TestRun` and appends it to `Test.history`. optionally, set the number of times to run repeat the test and the name to use for this `TestRun`.

### default values priority
priority for name/repeat values used are:

1. value passed in to `Test.run()` or `benchmark.run()`
2. value passed to `__init__()` (or `Test()`) during initialization
3. (for name only) function name if neither above provided
4. default value (if the function somehow doesn't have a name, "&lt;function>" is used, but i've never seen this be the case and don't know if it's even possible, but it's there just in case)

## history
there's a history object at the module level (`benchmark.history`) that contains a history of every `TestRun` object generated through `benchmark.Test.run()` and `benchmark.run()`.

### attributes
| attribute | type | description |
| --------- | ---- | ----------- |
| _history  | List[TestRun] | all `TestRun` objects |

### methods
```python
average_time(filter: Callable = None) -> float
```
return the average time of all `TestRun`s. optionally, pass a function to be used as a filter, e.g.: `history.average_time(lambda x: x.name == "str_list_test")`

```python
average_memory(filter: Callable = None) -> float
```
get the average memory usage in *MiB* of all `TestRun`s. optionally, pass a function to be used as a filter, e.g.: `history.average_memory(lambda x: x.stdout.contains("hello world")`

```python
add(run: TestRun) -> None
```
add a `TestRun` to the module level history

```python
get(filter: Callable = None) -> List[TestRun]
```
return all `TestRun` objects in history. optionally, pass a function to be used as a filter, e.g.: `history.get(lambda x: x.time > 30)`

# todo
- [ ] create a generic `_history` class to be used at the module level and by instances of `Test`
- [ ] add more analysis options for items in `history`
- [ ] capture `stderr` in `TestRun.run()`?
