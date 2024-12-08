---
title: "Measuring time and computational resources required by the software"
start: false
teaching: 15
exercises: 10
questions:
- "What is software profiling?"
- "What tools can we use to measure time and computational resources required by our software?"
objectives:
- "Use Jupyter magics and SnakeViz to profile time and computational resources."
keypoints:
- "For our software to be usable, we need to take care not only of its correctness, but also of its performance."
- "Profiling is necessary for large computationally expensive projects or for the software that will be processing large datasets."
- "Finding bottlenecks and ineffective subroutines should be done before parallelization of the code."
---

## What is Software Profiling?

We may have a software that is doing everything that we want it to do, with its codebase well-written 
and perfectly comprehensible, but this does not yet guarantee that this software will be applicable to the real-world problems.
For example, its interface can be so convoluted that the user can't figure out how to access the feature of interest. Other common
issue is when the software has such high [computational complexity](https://en.wikipedia.org/wiki/Computational_complexity) that it 
can be used only on small datasets, or on computer clusters with hundreds of GB of memory.

The process of estimating how much time the execution of the software will take and how much memory or other resources it will need
is called **profiling**. Profiling is a form of **dynamic program analysis**, meaning that it requires launching the software and measuring 
its performance and logging its activity as it runs. One of the main purposes of profiling is to identify *bottlenecks*,
inefficient operations, or high-latency components of the program, so that these parts could be optimized to run faster or
to use less resources. It is worth noting that for many problems there is a [space-time tradeoff](https://en.wikipedia.org/wiki/Space%E2%80%93time_tradeoff),
which essentially means that by optimizing the program in respect to the execution time we make it less memory-efficient, and vice versa.

How necessary profiling is for academic software development?
Well, it is ok to skip profiling when working on small projects that can be easily executed on your PC 
or within a Google Colab notebook. However, as your projects get larger, the execution of your code
may start taking days and weeks, or it may start crashing due to the lack of RAM or CPU power. Another
example when the efficiency is crucial is executing your code at astronomical data access portals,
such as [Rubin Science Platform](https://data.lsst.cloud/) or [Astro Data Lab](https://datalab.noirlab.edu/). 
These platforms provide an opportunity to work with large astronomical datasets without downloading them
to your machine, however, the CPU and memory allocated to each user are limited. Code profiling helps us 
to determine which implementation is more efficient and to make our code scalable. And if you are developing
software that will be used by someone else, it also makes the users happier (and in some cases
ensures that the software will be used at all).

In this introduction to profiling we mostly concentrate on time profiling, with some notes on memory profiling. 
We start with the Jupyter built-in tool, **Jupyter Magics**. 

## **What Are Jupyter Magics?**

Jupyter Magics are special commands in Jupyter Notebooks/Lab that extend functionality by providing shortcuts for tasks 
like timing, debugging, profiling, or interacting with the system, e.g. executing terminal commands from withing the notebook. They come in two forms:
- **Line Magics**: Prefixed with `%`, they operate on a single line of code.
- **Cell Magics**: Prefixed with `%%`, they apply to the entire cell.

> ## Why Magics aren't really part of the Jupyter
> 
> To be specific, Magic commands are part of the *Python kernels* that are used by the Jupyter Notebook or Lab.
> The *kernels* are separate processes, language-specific computational engines that are executing the commands you gave
> it with the code. The kernel with which your notebooks is launched is separate from the *frontend* processes that handle,
> for example, the tasks of adding or removing cells, or even rendering the characters when you type them during the execution of the
> previous cell. Think on how different it is from when you launch some commands in your PC terminal: there, you cannot
> type the next commands until the previous one finish executing. The separation of the Jupyter frontend from the kernel is what allows to
> avoid this behavior.
>
> By default, Jupyter uses [ipykernel](https://github.com/ipython/ipykernel), and Magic commands are a part of it. However,
> many different kernels for Jupyter exist, including some developed for other languages, such as R or Julia. Whether Magic commands are available for
> these kernels depends on their implementation.
> 
> If you want to know more how kernels work, [here](https://www.geeksforgeeks.org/managing-jupyter-kernels-a-comprehensive-guide/) is a generic overview with some
> practical examples.
> 
{: .callout}

There are plenty Magics cheatsheets online, however, the easiest way to look up what kinds of comands are there is to use Magics itself:
- `%lsmagic` - prints a list of all Magics available;
- `%quickref` - prints a reference card on the available Magic commands.

![`%lsmagic` prints a list of all Magics](../fig/imgDummy.png){: .image-with-shadow width="600px"}
![`%quickref` prints a reference card on Magic commands](../fig/imgDummy.png){: .image-with-shadow width="600px"}

## Time Profiling with Magics

For time profiling, the most useful Magics are:

- **`%time`**: Measures execution time of a single line.
- **`%%time`**: Measures execution time of an entire cell.
- **`%timeit`**: Repeats timing of a line for reliable results.
- **`%%timeit`**: Repeats timing of a cell for average results.

Let's use this function to profile some code. As an example we'll use a code that calculates the first
thousand of partial sums of [series](https://en.wikipedia.org/wiki/Series_(mathematics)#Partial_sum_of_a_series) of natural numbers
and saves it into a list.

First, we need to create a new branch:
~~~
$ git checkout develop
$ git checkout -b profiling
~~~
{: .language-bash}

Then let's write one possible implementation of the code for the problem above:

~~~
%%time
# Example: Timing a block of code
result = []
for i in range(1000):
    result.append(sum(range(i)))
~~~
{: .language-python}

This code creates an empty list `result`, and then launches a `for` loop in which for each `i` in the range from 0 to 1000
a sum of all numbers from 0 to `i` is calculated and appended to the list. This code produces the following  output:

~~~
CPU times: user 10.4 ms, sys: 0 ns, total: 10.4 ms
Wall time: 10.2 ms
~~~
{: .output}

As you can notice, there are several measurements taken:

- **Wall Time**: Total elapsed time from start to end of a task, including waiting time for I/O operations or other processes.
- **CPU Time**: The time the CPU spends executing the code, further divided into **User Time** 
(time spent executing user code) and **System Time** (time spent on system-level operations, such as I/O or memory management).
- In general, there is also **Overhead**, which is the additional time introduced by profiling tools themselves, which may slightly skew results.

The double `%%` symbol means that `%%time` is applied to entire cell. Pay attention 
that if you use `%time` instead, the result will be very different:

~~~
CPU times: user 2 μs, sys: 0 ns, total: 2 μs
Wall time: 4.05 μs
~~~
{: .output}

2 *micro*seconds (since it's `μs` and not `ms`) instead of ten *milli*seconds. These microseconds is the measurement of an execution time of an *empty line*,
since `%time` command does not care about the code in the following lines.

If you launch the code above several times, you'll notice that the measured time can differ quite a lot. 
Depending on the specifics of the code and on the usage of system resources by other processes, 
the runtime of a code block may vary. As a side-remark, if you develop code running on a spacecraft, 
you want to avoid unpredictable runtime at all cost. To get a better measurement, 
`%timeit` and `%%timeit` runs the code seven times and produces mean and standard deviation of the execution time. This command
also discards outlying measurements that are likely caused by temporary system slowdowns (e.g. caused by other software that is running on your PC).

~~~
%%timeit
# Example: Timing a block of code with 'timeit'
result = []
for i in range(1000):
    result.append(sum(range(i)))
~~~
{: .language-python}

~~~
4.43 ms ± 35.2 μs per loop (mean ± std. dev. of 7 runs, 100 loops each)
~~~
{: .output}

The `7 runs, 100 loops each` tells us that the code was executed 7x100 times in total. While 7 runs is the set default value, the number of the loops
is calculated automatically depending on how fast your code is, so that profiling didn't take too much time. You can change the number of
runs and loops using the flags `-n` and `-r`, for example, like this: `%%timeit -n 4 -r 10` to run the code 4 times, with 10 loops in each run. 
Another useful flag is `-o` that allows you to store the result of the profiling in a variable:

~~~
%%timeit -n 4 -r 10 -o
# Example: Timing a block of code with saving the profiling result
result = []
for i in range(1000):
    result.append(sum(range(i)))
~~~
{: .language-python}

~~~
# In a new cell we save the content of the temporary
# variable '_' into a new variable 'measure'
measure = _
measure
~~~
{: .language-python}

~~~
<TimeitResult : 5.16 ms ± 1.56 ms per loop (mean ± std. dev. of 10 runs, 4 loops each)>
~~~
{: .output}

`measure` variable is a `TimeitResult` object that has some useful methods that allow us to see e.g. the value of the worst measurement
and the measurements of all runs.

## Memory Profiling with Magics

On machines with limited RAM, we could also consider profiling memory usage. For this we can install more magic commands, e.g. from [Python Package Index](https://pypi.org/)

~~~
$ python -m pip install ipython-memory-magics
~~~
{: .language-python}

and load the external memory magic via

~~~
%load_ext memory_magics
~~~
{: .language-python}

Now we can analyze the memory usage in our cell

~~~
%%memory
# Example: Memory usage
result = []
for i in range(1000):
    result.append(sum(range(i)))
~~~
{: .language-python}

The output of this should look as follows:

~~~
RAM usage: cell: 35.62 KiB / 35.79 KiB
~~~
{: .output}

This reports *current* and *peak* memory usage of the code. Another useful
option is to use `%memory -n` command in an empty cell, which will print how much RAM the whole current *notebook*
is taking.

~~~
RAM usage: notebook: 158.61 MiB
~~~
{: .output}

> ## Execution time with Magics
> 
> Use Magic commands to measure execution time for functions `max_mag` and `plot_unfolded`.
>
> > ## Solution
> >
> > Either in a new section of the `light-curve-analysis.ipynb` notebook
> > or in a new notebook we can import the `max_mag` function and use `%time` command:
> > 
> > ~~~
> > from lcanalyzer.models import max_mag
> > ...
> > %time lcmodels.max_mag(lc[bands[b]],mag_col=mag_col)
> > ~~~
> > {: .language-python}
> > 
> > ~~~
> > CPU times: user 1.94 ms, sys: 155 μs, total: 2.1 ms
> > Wall time: 1.9 ms
> > np.float64(18.418037351622612)
> > ~~~
> > {: .output}
> >
> > And we see that this command takes only a few microseconds to run.
> > Pay attention that in order for this command to work, it should be in the same line as the
> > code you are profiling. Otherwise, you need to use a *cell* Magic preceded by `%%`.
> >
> > Next use `%timeit` on a plotting function:
> > ~~~
> > %timeit views.plot_unfolded(lc[bands[b]],time_col=time_col,mag_col=mag_col,color=plot_filter_colors[b],marker=plot_filter_symbols[b])
> > ~~~
> > {: .language-python}
> >
> > ~~~
> > 347 ms ± 9.54 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
> > ~~~
> > {: .output}
> > 
> > Wow, what happened? We got seven copies of the same plot! This is because `%timeit` repeats the execution and measures the average time.
> > Plotting also takes noticeably longer time that a simple calculation of a maximum value.
> > 
> {: .solution}
>  
{: .challenge}

## A Few Words on Optimization

Let's take our 'partial sums of series' problem and think on how we can optimize it to run faster. 
You may remember that Python has [list comprehensions](https://docs.python.org/2/tutorial/datastructures.html#list-comprehensions)
syntax that is often recommended as a faster tool than `for` loops. We can rewrite the code above to use list comprehensions
and use `%%timeit` magic to profile it.

~~~
%%timeit
# Implementation with list comprehension
result = [sum(range(i)) for i in range(1000)]
~~~
{: .language-python}

~~~
4.43 ms ± 71 μs per loop (mean ± std. dev. of 7 runs, 100 loops each)
~~~
{: .output}

Well... Actually, we got pretty much the same result. For this particular task, there is no computational
gain in using list comprehension, although the code is cleaner and more readable this way. For some 
problems, list comprehensions may work even slower than `for` loops, which is why time profiling
is something to do *before* you start optimization - it may turn out that the bottleneck is in a completely different
part of the program than you thought. In order to optimise this specific piece of code, we would have to be
smarter and use an equation instead of the bruteforce approach:

~~~
%%timeit
# Optimized implementation using mathematical formula for summation
result = [(i * (i - 1)) // 2 for i in range(1000)]
~~~
{: .language-python}

~~~
77.6 μs ± 812 ns per loop (mean ± std. dev. of 7 runs, 10,000 loops each)
~~~
{: .output}

Now the execution time is in dozens of *micro*seconds, which is two orders better than before! 
That's quite an improvement.

## **What If You Don’t Use Jupyter?**

Of course, Jupyter Magics isn't the only tool for time profiling. In fact, Python has built-in modules just for this, such as `time` and `timeit`, 
that can be used in any IDE. 

~~~
import time
...
start_time = time.time()
max_mag = lcmodels.max_mag(LC[bands[b]],mag_col=mag_col)
print(f"Execution time: {time.time() - start_time:.5f} seconds")
~~~
{: .language-python}

~~~
Execution time: 0.00083 seconds
~~~
{: .output}

In this snippet of code, `time.time()` function records the current time in seconds since the epoch (typically January 1, 1970).
 This value is stored in the variable `start_time` before the code block is executed. 
After the code block, `time.time()` is called again to get the current time.
The difference between the current time and `start_time` gives the total execution time.
This elapsed time is formatted to five decimal places and printed.

To get a more reliable estimate, for the small snippets of code we can use `timeit.repeat()` method. 
It executes the timing multiple times and provides a list of results, making it easier to analyze performance under changing environments.

~~~
import timeit
results = timeit.repeat('sum([i**2 for i in range(1000)])', repeat=5, number=1000)
print("Timing Results:", results)
print("Best Execution Time:", min(results))
~~~
{: .language-python}

~~~
Timing Results: [0.17868508584797382, 0.16405000817030668, 0.16924912948161364, 0.1637995233759284, 0.16636504232883453]
Best Execution Time: 0.1637995233759284
~~~
{: .output}

The `repeat` and `number` parameters of this function work similarly to the number of runs and number of loops for the `%%timeit`.
However, this method cannot be used conveniently for e.g. measuring execution time of functions. For this, we need a more advanced tool.

> ## Additional reading on 'time' module and 
> 
> Some additional sources to look into are:
>
> - [Python timeit Module](https://docs.python.org/3/library/timeit.html):  Detailed explanation of how to use the `timeit` module for benchmarking Python code.
> - [Python time Module](https://docs.python.org/3/library/time.html): Overview of the `time` module, including functions like `time()`, `sleep()`, and more.
> - [Profiling in Python](https://www.geeksforgeeks.org/profiling-in-python/): A beginner-friendly introduction to time profiling methods in Python.
>   
{: .callout}

## Resource profiling with offline profilers

Profiling is the essential technique for identifying bottlenecks and optimize performance in larger projects. This process can significantly impact both the success of a project and its environmental footprint, as high-performance code may consume more resources and energy. Finding a balance between the time it takes to develop code and the cost of running the code itself motivates us to analyze projects in a more detailed fashion. Relying on the pedestrian approach in the last section can be useful for benchmarking smaller blocks of code, but it is not a sustainable approach for larger projects. Prioritizing development requires to understand

* The frequency at which a function is called
* The execution time for each function
* The performance of different algorithms 
* Benchmarking pure python vs external C code
* Identify bottlenecks and gauge how involved a change is compared to the development effort

As a rule of thumb, software developers tend to profile work on improving 10 to 20% of their codebase. To accomplish this we can use `cProfile` to prepare the performance statistics and utilize `snakeviz` to visualize the statistics in a more user friendly way. To install `snakeviz` run

~~~
$ python -m pip install snakeviz
~~~
{: .language-bash}


You can generate performance statistics for a script `myscript.py` using `cProfile` with the following command:

~~~
$ python -m cProfile -o output.stats myscript.py
~~~
{: .language-bash}

An interactive view of these statistics can be obtained by running:

~~~
$ snakeviz output.stats
~~~
{: .language-bash}

This will open in your browser, showing an interactive table that is particularly useful for inspecting the number of function calls and elapsed time per function call (among other metrics). We are using our `squares` and `quadruples` functions to illustrate how the output looks like. The table can be sorted by any column. Performance statistics show not only your functions, but also functions from the standard library, e.g. list comprehensions are shown separately. 

![Snakeviz table](../fig/33_snakeviz_table.png){: .image-with-shadow}

Two visualization options are available: an icicle view, which displays the function calls as cumulative timings icicles with corresponding line numbers, and a sunburst diagram, both of which allow you to explore the call hierarchy.

![Snakeviz icicle](../fig/33_snakeviz_icicle.png){: .image-with-shadow }

The sunburst diagram looks as follows:

![Snakeviz sunburst](../fig/33_snakeviz_sunburst.png){: .image-with-shadow}

 
 If you prefer to run the profiler directly within a Jupyter Notebook, you need to load the external snakeviz magic:

~~~
%load_ext snakeviz
~~~
{: .language-python}

and test it using our example

~~~
def squares(n):
    squares = [x**2 for x in range(n)]

%snakeviz squares(777)
~~~
{: .language-python}

Based on the output, we would mention a tool providing call graphs in passing: [gprof2dot](https://pypi.org/project/gprof2dot/) can be used to plot call graphs in the following way, which some users could find more intuitive than the default `snakeviz` plots:

 ![gprof2dot example](../fig/33_gprof2dot.png){: .image-with-shadow}
 
We would like to conclude this session by illustrating how you can integrate and run the `cProfile` profiler into your main code, and we are testing two calls of our function squares

~~~
from cProfile import Profile
profiler = Profile()

profiler.run('squares(777777); squares(777777)')
profiler.disable()
profiler.print_stats(sort='cumulative')
~~~
{: .language-python}

this should return a list as follows which can also be written to disk using `profiler.dump_stats(filename)`:

~~~
 7 function calls in 0.522 seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.522    0.522 {built-in method builtins.exec}
        1    0.011    0.011    0.522    0.522 <string>:1(<module>)
        2    0.000    0.000    0.511    0.256 890459632.py:5(squares)
        2    0.511    0.256    0.511    0.256 890459632.py:6(<listcomp>)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
~~~
{: .output}

> ## Optional Exercise: Profile lightcurve
>
> Try to profile the methods devised for lightcurve analysis. 
>
> > ## Solution
> > tbd
> {: .solution}
> 
{: .challenge}


## Resource profiling with online profilers

Profiling a running project can be an invaluable tool for identifying and addressing issues, as it catches unusual events that may not be obvious during development. While this course focuses on offline profiling techniques using tools like `cProfile` and `snakeviz`, we would like to mention that monitoring performance in real-time can be crucial for certain projects. In this context, we would like to highlight two repositories as starting points for further reading:

1. [py-spy](https://github.com/benfred/py-spy)
2. [pyinstrument](https://github.com/joerick/pyinstrument)

{% include links.md %}
