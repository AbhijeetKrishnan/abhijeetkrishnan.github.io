---
title: Integer Objects in Python
tags: python deep-dive
mathjax: true
---

Try predicting the output of the following Python snippet -

```python
a = 1
b = 1
a is b
```

`is` is the Python identity operator. Similar to `===` in Javascript or `==` in Java, it tests whether two variables reference the same underlying object.[^1] If I had the below snippet instead -

```python
a = [1]
b = [1]
a is b
```

The last line would clearly return `False` since `a` and `b` both point to different list objects (albeit with the same value). In the original snippet, however, it is not so clear. We don't usually think of integers as being objects. It was surprising to me to discover that Python actually stores integers as objects.[^3] I recommend watching Ned Batchelder's PyCon 2015 talk on [Facts and Myths About Python Names and Values](https://www.youtube.com/watch?v=_AEJHKGk9ns), which, in addition to many things, talks about how *everything* in Python is a value, including booleans, floats and integers. In light of this, we can now deduce that the output of the first snippet would be `False`, which we can check.

```python
>>> a = 1
>>> b = 1
>>> a is b
>>> True
```

Why do we get back `True`? This would mean that both `a` and `b` in fact point to the *same* object. The statements `a = 1` and `b = 1` have assigned `a` and `b` with references to the same object. This did not happen in the case of lists (`a = [1]` and `b = [1]` assigned `a` and `b` with references to *different* list objects). Let's try to play around with this some more.

```python
>>> a = -10
>>> b = -10
>>> a is b
>>> False
```

Why do we get a different answer by merely changing the literal `1` to `-10`? This implies that the integer objects created for `1` are merely references to the same object, while the objects created for `-10` are in fact unique. If we look through the documentation[^2], we find the following -

> The current implementation keeps an array of integer objects for all integers between `-5` and `256`, when you create an int in that range you just get back a reference to the existing object.

This explains our findings perfectly. Python maintains an array of integer objects between $-5$ to $256$. In the first snippet, we were attempting to create integer objects for $1$, which lies in that range, so we were simply getting back a reference to the pre-allocated object. In the second snippet, when we attempted to create an integer object for $-10$, we got back a reference for a completely new object, since $-10$ lies outside the range of these pre-allocated objects.

This begs the question of *why* the Python implementers decided to do things this way. Was it for performance reasons, as a kind of cache mechanism to speed-up object creation for oft-used integers? If so, how was this range determined? Was it arbitrarily chosen or were there quantitative experiments conducted? Was it something else entirely?

It turns out that this pre-allocated integer object creation (which I'll refer to as PAIOC for brevity) is an implementation quirk of CPython (the reference Python interpreter) and is not actually guaranteed by the language specification. Note the following from the Python documentation[^3] -

> Types affect almost all aspects of object behavior. Even the importance of object identity is affected in some sense: for immutable types, operations that compute new values may actually return a reference to any existing object with the same type and value, while for mutable objects this is not allowed.[^5] E.g., after `a = 1; b = 1`, `a` and `b` may or may not refer to the same object with the value one, depending on the implementation, but after `c = []; d = []`, `c` and `d` are guaranteed to refer to two different, unique, newly created empty lists. (Note that `c = d = []` assigns the same object to both `c` and `d`.)

Although I recall a tweet by [Raymond Hettinger](https://rhettinger.wordpress.com/about/), one of Python's core developers, on the reasoning behind this range, and how it was determined (performance, and testing using the top 200 or so most popular Python packages), I cannot find that tweet now. I will link it when possible.

[This](https://bugs.python.org/issue1436243) exchange on an issue on the Python Bug Tracker explains how the range used to be $[-5, 100]$, but was increased to $[-5, 256]$ due to the introduction of the Python `bytes` object. No comment on how the $-5$ was chosen, though.

So how might we explore this range further? Is it possible to optimize the range, find one which gives an even better performance speed-up? We first need to define what we mean by "performance speed-up", and then instrument the CPython source code to accommodate the tests. Concretely, we would want to -

1. disable PAIOC in the $[-5, 256]$ range
2. instrument integer object creation to record frequency of object creation per value
3. use the resulting build of CPython to execute a variety of Python programs
4. observe the measured frequencies and use it to determine a range.

Let's examine each step in detail.

## Disabling PAIOC

We first need to obtain the CPython source code, which is now helpfully hosted on [GitHub](https://github.com/python/cpython).[^4] Following the instructions in the [Python Developer's Guide](https://devguide.python.org/), we can quickly attempt to build a vanilla copy of Python and run the tests to ensure our build system is working correctly. To ensure stability, I'm working with the version tagged [v3.9.1](https://github.com/python/cpython/releases/tag/v3.9.1), instead of the latest release ([v3.10.0a3](https://github.com/python/cpython/releases/tag/v3.10.0a3)).

To find the relevant part of the source code, I was led by [this](https://stackoverflow.com/a/15172182) StackOverflow answer to [this](https://github.com/python/cpython/blob/4830f581af57dd305c02c1fd72299ecb5b090eca/Objects/longobject.c#L18-L23) part of the CPython source code. It clearly defines `NSMALLPOSINTS` and `NSMALLNEGINTS` as `257` and `5`respectively. The integer $257$ itself doesn't seem to have a pre-allocated integer object for it, so maybe it acts as a non-inclusive upper bound for the integer values for which pre-allocated objects are created.

However, this version of the code is old ([v3.7.0a1](https://github.com/python/cpython/tree/v3.7.0a1) released on Sep 18, 2017), and the latest version instead defines the [constants](https://github.com/python/cpython/blob/v3.9.1/Objects/longobject.c#L20-L21) in terms of other constants defined [elsewhere](https://github.com/python/cpython/blob/v3.9.1/Include/internal/pycore_interp.h#L67-L68). [ripgrep](https://github.com/BurntSushi/ripgrep) helps a lot with navigating a codebase as large as this.

It turns out disabling PAIOC is easy. Just change the constants `_PY_NSMALLPOSINTS` and `_PY_NSMALLNEGINTS` to `0`. This is because the Python devs, in their infinite wisdom, have guarded all accesses to the pre-allocated integer array (names `small_ints` in the code) with the check that the range be of length $> 1$.

## Instrumenting Integer Object Creation

Our goal here is to -

1. maintain a frequency counter for every unique (in terms of value) integer object for the lifetime of the interpreter
2. update this counter whenever the corresponding integer object is created
3. output the frequency when the interpreter closes

The first step requires the implementation of a hashmap in the [Python/C API](https://docs.python.org/3/c-api/index.html). If a `PyLong` object with any value is created, we need to initialize our frequency map as `frequency[value] = 1`. Writing this implementation without affecting other parts of the API seems cumbersome.

Another solution is to modify the `small_ints` array to also record the frequency of creation, re-enable PAIOC, and increase the range.[^8] This should force object creation for most created integers while allowing us to record their frequencies, which can be logged upon interpreter exit. This solution is much more straightforward to implement, and you can see the relevant commit in my CPython fork [here](https://github.com/AbhijeetKrishnan/cpython/commit/a48b805a2d74f835df7760c6c2e8437448cfee17). The information provided in Guido van Rossum's [Dropbox paper](https://paper.dropbox.com/doc/Yet-another-guided-tour-of-CPython-XY7KgFGn88zMNivGJ4Jzv) on how the prompt in the CPython interpreter gets printed helped me locate the exit function for the interpreter.

The frequency statistics for each interpreter call during the benchmarking is appended to a log file, and a simple Python [script](https://github.com/AbhijeetKrishnan/cpython/blob/int-obj/Tools/scripts/log_analyzer.py) is used to tally the results of the separate calls together. I chose to use a range of $[-1000, 1000]$.

## Profile the resulting CPython build's performance

Since the entire point of maintaining the pre-allocated integer objects is to achieve execution speed-up, we can define that to be our performance metric (along with perhaps memory usage). The tweet I had vaguely recalled earlier mentioned testing this using the top 200 Python packages. We can pull up the list of the most "trending" Python packages on [PyPI](https://pypi.org/search/?q=&o=-zscore&c=Development+Status+%3A%3A+5+-+Production%2FStable), which is the official third-party software repository for Python, and is what `pip` uses as its source for packages. However, this doesn't give us a list of the *most downloaded* Python packages. For that, we must turn to Hugo van Kemenade's [project](https://hugovk.github.io/top-pypi-packages/) which provides a monthly dump of the 4,000 most-downloaded packages from PyPI.

However, these packages are designed to be used as libraries, and they require some driver code to function as a test. I'd prefer to use a real-world (or similar) application as a benchmark since we can easily construct a script that just creates a huge number of integer objects outside the pre-allocated range as a pathological test. We can find this in [The Python Performance Benchmark Suite](https://pyperformance.readthedocs.io/index.html), which

> \[...\] is intended to be an authoritative source of benchmarks for all Python implementations. The focus is on real-world benchmarks, rather than synthetic benchmarks, using whole applications when possible.

It meets our requirements nicely. We can run the benchmark for the default CPython build, a build without pre-allocated integers (to measure the speed-up obtained), and a build with the new range calculated from our frequency analysis.

The command used to generate the benchmark results is -

```bash
pyperformance run --fast --python=[path_to_executable] -o output-[build_type].json
```

Generating these benchmarks such that they can be reproduced is a tricky affair[^6]. I opted to not pursue it since reproducibility is not valuable for this application. I just tuned it to my system using the provided command (which had to be run under `sudo`) -

```bash
sudo python -m pyperf system tune
```

A comparison generated using `pyperformance` between the v3.9.1 build and the version without pre-allocated integers can be found [here](/assets/docs/default_vs_no_int_comparison.txt). The latter is significantly[^7] slower on almost all benchmarks. A comparison between the v3.9.1 build and a version with PAIOC range $[-10^6, 10^6]$ is given [here](/assets/docs/default_vs_wide_int_comparison.txt). This too is significantly slower on all benchmarks, probably due to the time taken for PAIOC (the `python_startup` benchmarks are more than 20x slower). We will have to balance the additional start-up time incurred from increasing the range with the speed-up gained due to fewer PAIOC calls.

## Results & Analysis

The frequency of integer object creation calls plotted against the value of the integer is shown below.

![A graph of the frequency of integer object creation calls vs. the value of the integer](/assets/images/hist1.png "Histogram of PAIOC calls")

There is a huge drop-off in the frequency as we move upward from the positive integers, while the frequencies of the negative integers are all $0$. In fact, the biggest drop-off is between $256$ and $257$, where the frequency falls by nearly x24. This lends evidence to the limits for the original range being well-chosen. Zooming into just the positive integers, we see the following -

![A graph of the frequency of positive integer object creation calls vs. the value of the integer](/assets/images/hist2.png "Histogram of +ve PAIOC calls")

The frequencies are fairly level from $257$ all the way to $1000$, but there's a slight bump between $[400, 600]$ that we could take advantage of. Let's increase the PAIOC range to $512$ and see if we get any significant speedup.

Unfortunately, the [comparison](/assets/docs/default_vs_opt_int_comparison.txt) does not indicate a clearly better result. While the results are more mixed, interpreter start-up costs still dominate the speed-ups from a bigger integer object cache.

How can we explain the frequency pattern we got though? It certainly seems strange that the negative integers, even the small ones like $-1$, which might be used to iterate in reverse, reported $0$ frequency. It's also interesting that $256$ and $512$ have anomalously high frequencies compared to the surrounding values (and presumably other powers of $2$ also have similar behaviour). What makes their creation so much more frequent?

## Limitations

- This analysis was done using the `--fast` option provided by `pyperformance`. Removing it might give more stable results.
- Not much care was given to making the benchmarks reproducible. System variability might make the results between runs different.
- The frequency logs also include interpreter calls made during package installation for the benchmarks. These are not strictly part of the benchmarks.
- Memory usage was not measured or compared during any of the benchmarks, although it is possible to do so.[^9]

## Conclusion

Based on the graph of observed frequencies, it seems likely that the current range is the best one. The upper bound captures the maximum area under the graph, while the lower bound seems like a fair arbitrary value given that negative integers had $0$ frequency. It is unlikely that twiddling the upper bound between $256$ and $512$ would provide a significant speed-up, even if an optimum was achieved.

[^1]: Brett Cannon, a Python core developer, advises against using `is` carelessly in this [blog post](https://devblogs.microsoft.com/python/idiomatic-python-boolean-expressions/).
[^2]: [https://docs.python.org/3/c-api/long.html](https://docs.python.org/3/c-api/long.html)
[^3]: [https://docs.python.org/3/reference/datamodel.html#objects-values-and-types](https://docs.python.org/3/reference/datamodel.html#objects-values-and-types)
[^4]: Brett Cannon's [blog post](https://snarky.ca/the-history-behind-the-decision-to-move-python-to-github/) details the history behind this move
[^5]: Al Sweigart, the author of the popular Python book [Automate the Boring Stuff With Python](https://automatetheboringstuff.com/), has a nice [blog post](https://inventwithpython.com/blog/2018/02/05/python-tuples-are-immutable-except-when-theyre-mutable/) explaining the difference between mutable and immutable types in Python and how operators which use them work.
[^6]: [https://pyperf.readthedocs.io/en/latest/run_benchmark.html#how-to-get-reproductible-benchmark-results](https://pyperf.readthedocs.io/en/latest/run_benchmark.html#how-to-get-reproductible-benchmark-results)
[^7]: From the `pyperformance` [usage notes](https://pyperformance.readthedocs.io/usage.html#notes) -
    > pyperformance will run Student’s two-tailed T test on the benchmark results at the 95% confidence level to indicate whether the observed difference is statistically significant.

    The comparison outputs the t-statistic directly.

[^8]: We would want to increase it to as wide of an interval as permissible. [Comments in the code](https://github.com/python/cpython/blob/v3.9.1/Include/longintrepr.h#L37-L38) mention that it can be as wide as a `digit` type. A `digit` is a Python/C API-specific datatype which [seems to be equivalent](https://github.com/python/cpython/blob/v3.9.1/Include/longintrepr.h#L45) to a C `int`.

[^9]: From the `pyperformance` [usage notes](https://pyperformance.readthedocs.io/usage.html#notes) -
    > If `--track_memory` is passed, pyperformance will continuously sample the benchmark’s memory usage. This currently only works on Linux 2.6.16 and higher or Windows with PyWin32. Because `--track_memory` introduces performance jitter while collecting memory measurements, only memory usage is reported in the final report.

*[PAIOC]: pre-allocated integer object creation
