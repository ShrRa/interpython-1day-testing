---
title: "Measuring time and computational resources required by the software"
start: false
teaching: 15
exercises: 10
questions:
- "***"
objectives:
- "Use Jupyter magics and SnakeViz to profile time and computational resources."
keypoints:
- "***."
---

## IPython Magics for benchmarking

Before discussing the most important tools for profiling, we are briefly introducing a more pedestrian approach to assess the runtime of a single code block. For this purpose we are going to use IPython Magics, which are part of the respective IPython kernel or specified by users. Either way, the command needs to be prefixed with at least one `%` symbol and can run various operations. A simple example is to list the files in the current working directory:

~~~
%ls
~~~

On most operating systems you can execute the following function to analyze the runtime

~~~
$ time python myscript.py
~~~

If we are using a juypyter notebook with an IPython kernel, the following the `%time` magic can be used which is used for timing code blocks. 

~~~
%time [x**2 for x in range(7777777)]
~~~

It reports the so-called wall clock time, i.e. the elapsed actual time, but also the user CPU time. The single `%` symbol means that `%time` is applied to single cell. To apply magic commands to an entire cell, we use `%%` 

~~~
%%time 
squares = [x**2 for x in range(7777777)]
quadrupels = [x**4 for x in range(7777777)]
~~~

Depending on the structure of the datas ets in question or if stochastic processes are involved, the runtime of a code block may vary. As a side-remark, if you develop code running on a spacecraft, you want to avoid unpredictable runtime at all cost. To get a better measurement, `%%timeit` comes with extra options to test a block multiple times, which also allows us to get a mean and standard deviation. 


~~~
%%timeit
squares = [x**2 for x in range(7)]
quadrupels = [x**4 for x in range(7)]
~~~

The output of this reveals that after 7 runs with 100,000 the following measurement is available:

~~~
4.51 μs ± 16 ns per loop (mean ± std. dev. of 7 runs, 100,000 loops each)
~~~

The pedestrian approach of using `%timeit` works for blocks of code but testing multiple functions, the number of calls and time it takes can be helpful. The goal is usually to identify the slowest function, but the difference between wall time and user CPU time can reveal that file operations with local or remote drives may be a limiting factor. On machines with limited RAM, we could also consider profiling memory usage. For this we can install more magic commands,, e.g. from [https://pypi.org/](https://pypi.org/)

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



## Resource profiling

Profiling is the art of finding bottlenecks in a larger project to speed up computation as opposed to benchmarking different code blocks as shown before. This step can be critical to the success of a project, but it also has an environmental impact. High performance code can leave a significant carbon footprint. The trade-off between the time it takes to develop code and the cost of running the code itself motivates us to analyze projects in a more detailed fashion. The pedestrian approach is not a sustainable way of analyzing any larger project. Prioritizing development requires to understand

* How often a function is called 
* How long it takes to run a certain function
* Identify bottlenecks and gauge how involved a change is compared to the development effort

To accomplish this we use `cProfile` to prepare the performance statistics and install and use `snakeviz` to visualize the statistics in a more user friendly way.

~~~
$ python -m pip install snakeviz
~~~

The statistics of `myscript.py` can be produced by calling

~~~
$ python -m cProfile -o output.stats myscript.py

~~~

An interactive view can be produced using

~~~
$ snakeviz output.stats
~~~

and will start in your browser. One way to interpret is to use the table which is particularly useful to inspect the number of function calls and the elapsed time per function call. We are again using our `squares` and `quadruples` functions to illustrate how the output looks like. The table is interactive and can be sorted by any column:

![Snakeviz table](../fig/33_snakeviz_table.png){: .image-with-shadow width="800px"}

The top part of the snakeviz page illustrates the function name as icicle with cumulative times as callstack and the respective line number:

![Snakeviz icicle](../fig/33_snakeviz_icicle.png){: .image-with-shadow width="800px"}

Alternatively a clickable sunburst diagram can be selected:

![Snakeviz sunburst](../fig/33_snakeviz_sunburst.png){: .image-with-shadow width="800px"}

Both diagrams can be interpreted depending on the aformentioned use case, i.e. if the total runtime is in focus or if a single function should be optimized - a decision to be made after exploring the used ressources. 


{% include links.md %}
