---
title: "Measuring time and computational resources required by the software"
start: false
teaching: 20
exercises: 5
questions:
- "***"
objectives:
- "Use Jupyter magics and SnakeViz to profile time and computational resources."
keypoints:
- "***."
---

## IPython Magics for casual profiling

Before discussing the most important tools for profiling, we are briefly introducing a more pedestrian approach to assess the runtime of a single code block. For this purpose we are going to use IPython Magics, which are part of the respective IPython kernel or specified by users. Either way, the command needs to be prefixed with at least one `%` symbol and can run various operations. A simple example is to list the files in the current working directory:

~~~
%ls
~~~

On most operating systems you can execute the following function to analyze the runtime:

~~~
$ time python myscript.py
~~~

If we are using a juypyter notebook with an IPython kernel, the following the `%time` magic can be used which is used for timing code blocks. 

~~~
%time [x**2 for x in range(7777777)]
~~~

It reports the so-called wall clock time, i.e. the elapsed actual time, but also the user CPU time. The single `%` symbol means that `%time` is applied to single cell. To apply magic commands to an entire cell, we use `%%`:

~~~
%%time 
squares = [x**2 for x in range(7777777)]
quadrupels = [x**4 for x in range(7777777)]
~~~

Depending on the structure of the datas ets in question or if stochastic processes are involved, the runtime of a code block may vary. As a side-remark, if you develop code running on a spacecraft, you want to avoid unpredictable runtime at all cost. To get a better measurement, `%%timeit` comes with extra options to test a block multiple times, which also allows us to get a mean and standard deviation:

~~~
%%timeit
squares = [x**2 for x in range(7)]
quadrupels = [x**4 for x in range(7)]
~~~

The output of this reveals that after 7 runs with 100,000 the following measurement is available:

~~~
4.51 μs ± 16 ns per loop (mean ± std. dev. of 7 runs, 100,000 loops each)
~~~

The pedestrian approach of using `%timeit` works for blocks of code but testing multiple functions, the number of calls and time it takes can be helpful. The goal is usually to identify the slowest function, but the difference between wall time and user CPU time can reveal that file operations with local or remote drives may be a limiting factor. On machines with limited RAM, we could also consider profiling memory usage. For this we can install more magic commands, e.g. from [Python Package Index](https://pypi.org/)

~~~
$ python -m pip install ipython-memory-magics
~~~

and load the external memory magic via

~~~
%load_ext memory_magics
~~~

Now we can analyze the memory usage in our cell

~~~
%%memory
squares = [x**2 for x in range(7777)]
quadruples = [x**4 for x in range(7777)]
~~~

The output of this should look as follows:

~~~
RAM usage: cell: 590.7 KiB / 590.94 KiB
~~~


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


You can generate performance statistics for a script `myscript.py` using `cProfile` with the following command:

~~~
$ python -m cProfile -o output.stats myscript.py
~~~

An interactive view of these statistics can be obtained by running:

~~~
$ snakeviz output.stats
~~~

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

and test it using our example

~~~
def squares(n):
    squares = [x**2 for x in range(n)]

%snakeviz squares(777)
~~~

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
