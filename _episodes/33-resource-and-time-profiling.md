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

## IPython Magics

Before discussing the most important tools for profiling, we are briefly introducing a more pedestrian approach to assess the runtime of a single code block. For this purpose we are going to use IPython Magics, which are part of the respective IPython kernel or specified by users. Either way, the command needs to be prefixed with at least one `%` symbol and can run various operations. A simple example is to list the files in the current working directory:

~~~
%ls
~~~

On most operating systems you can execute the following function to analyze the runtime

~~~
time python myscript.py
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

The pedestrian approach of using `%timeit` works for blocks of code but testing multiple functions, the number of calls and time it takes can be helpful. The goal is usually to identify the slowest function, but the difference between wall time and user CPU time can reveal that file operations with local or remote drives may be a limiting factor. On machines with limited RAM, we could also consider profiling memory usage. For this we can install more magic commands,, e.g. from [https://pypi.org/](https://pypi.org/)

~~~
pip install ipython-memory-magics
~~~

and load the external memory magics via

~~~
%load_ext memory_magics
~~~

Now we can analyze the memory usage in our cell

~~~
%%memory
squares = [x**2 for x in range(7777)]
quadrupels = [x**4 for x in range(7777)]
~~~


## Resource profiling

-What is profiling?

Profiling is the art of finding bottlenecks in code to speed up computation. This step can be critical to the success of a project, but it also has an environmental impact. High performance code can leave a significant carbon footprint. The trade-off between the time it takes to develop code and the cost of running the code itself motivates us to analyze projects in a more detailed fashion.

- What is profiling
- What kinds of resources can we measure
- What are the complications
- Main tools
- Using SnakeViz

{% include links.md %}
