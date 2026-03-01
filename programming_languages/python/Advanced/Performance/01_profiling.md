# Profiling

Optimization must be based on data, not guesses.

## cProfile
The standard deterministic profiler.

```bash
python -m cProfile -s tottime myscript.py
```
*   `tottime`: Total time spent in the function (excluding sub-calls).
*   `cumtime`: Cumulative time (including sub-calls).

## line_profiler
Shows line-by-line execution time. Installing: `pip install line_profiler`.

```python
@profile
def slow_function():
    ...
```
Run with `kernprof -l -v script.py`.

## timeit
For micro-benchmarks of small snippets.

```python
import timeit
print(timeit.timeit('"_".join(str(n) for n in range(100))', number=10000))
```
