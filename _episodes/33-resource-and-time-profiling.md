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



## Jupyter Magics

- What is magics
- Which kinds of magics exist
- How to use them for time profiling
- What if you don't use Jupyter?

---

## What is Software Profiling?

Profiling is...

Time profiling is the process of measuring the time taken by a program, 
a specific block of code, or individual functions to execute. It helps 
developers analyze the performance of their software by identifying bottlenecks, 
inefficient operations, or high-latency components. Time profiling provides insights 
into how the program uses computational resources (CPU, memory, I/O), enabling performance optimization.

It is ok to skip profiling when working on small projects that can be easily executed on your PC 
or within a Google Colab notebook. However, as your projects get larger, the execution of your code
may start taking days and weeks, or it may start crashing due to the lack of RAM or CPU power. Another
example when the efficiency is crucial is executing your code at the astronomical Data Acess portals,
such as [Rubin Science Platform](https://data.lsst.cloud/) or [Astro Data Lab](https://datalab.noirlab.edu/). 
These platforms provide an opportunity to work with large astronomical datasets without downloading them
on your machine, however, the CPU and memory allocated to each user are limited. Code profiling helps us to find bottlenecks
in our code, to determine which implementation is more efficient and to make our code scalable. And if you are developing
software that will be used by someone else, it also helps you to make your users happier (and in some cases
ensures that the software will be used at all).

## **What Are Jupyter Magics?**

Jupyter Magics are special commands in Jupyter Notebooks that extend functionality by providing shortcuts for tasks 
like timing, debugging, profiling, or interacting with the system, e.g. executing terminal commands from withing the notebook. They come in two forms:
- **Line Magics**: Prefixed with `%`, they operate on a single line of code.
- **Cell Magics**: Prefixed with `%%`, they apply to the entire cell.

There are plenty Magics cheatsheets online, however, the easiest way to look up what kinds of comands are there is to use Magics itself:
- `%lsmagic` - prints a list of all Magics available;
- `%quickref` - prints a reference card on the available Magic commands.

![`%lsmagic` prints a list of all Magics](../fig/imgDummy.png){: .image-with-shadow width="600px"}
![`%quickref` prints a reference card on Magic commands](../fig/imgDummy.png){: .image-with-shadow width="600px"}

For time profiling, the most useful Magics are:

- **`%time`**: Measures execution time of a single line.
- **`%%time`**: Measures execution time of an entire cell.
- **`%timeit`**: Repeats timing of a line for reliable results.
- **`%%timeit`**: Repeats timing of a cell for average results.

Let's try to profile execution time for some functions of our `lcanalyzer` module. 

> ## Execution time with Magics
> Create a new branch called `profiling`
> and execute the Magic commands above to measure execution time for functions `max_mag` and `plot_unfolded`.
>
> > ## Solution
> >
> > First, we need to create a new branch:
> > ~~~
> > $ git checkout develop
> > $ git checkout -b profiling
> > ~~~
> > {: .language-bash}
> >
> > Next, in the `light-curve-analysis.ipynb` we can import the `max_mag` function and use `%time` command:
> > from lcanalyzer.models import max_mag
> >
> > ~~~
> > from lcanalyzer.models import max_mag
> > ...
> > %time lcmodels.max_mag(LC[bands[b]],mag_col=mag_col)
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
> > And we see that this command takes only a few milliseconds to run.
> > Pay attention that in order for this command to work, it should be in the same line as the
> > code you are profiling. Otherwise, you need to use a *cell* Magic preceded by double `%`.
> >
> > Next use `%timeit` on a plotting function:
> > ~~~
> > %timeit views.plot_unfolded(LC[bands[b]],time_col=time_col,mag_col=mag_col,color=plot_filter_colors[b],marker=plot_filter_symbols[b])
> > ~~~
> > {: .language-python}
> >
> > ~~~
> > 347 ms ± 9.54 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
> > ~~~
> > {: .output}
> > Wow, what happened? We got seven copies of the same plot! This is because `%timeit` repeats the execution and measures the average time.
> > Plotting also takes noticeably longer time that a simple calculation of a maximum value.
> {: .solution}
>
{: .challenge}

As you can notice, the output of these Magic commands contains several different time values:

- **Wall Time**: Total elapsed time from start to end of a task, including waiting time for I/O operations or other processes.
- **CPU Time**: The time the CPU spends executing the code, further divided into **User Time** 
(time spent executing user code) and **System Time** (time spent on system-level operations, such as I/O or memory management).
- In general, there is also **Overhead**, which is the additional time introduced by profiling tools themselves, which may slightly skew results.

`timeit` Magics also report standard deviation of the execution time, calculated from the best run from a set of iterations,
so that outliers from temporary system slowdowns (e.g. caused by other software that is running on your PC) are not included.

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

  - The `time.time()` function records the current time in seconds since the epoch (typically January 1, 1970).
  - This value is stored in the variable `start_time` before the code block is executed.
  - After the code block, `time.time()` is called again to get the current time.
  - The difference between the current time and `start_time` gives the total execution time.
  - This elapsed time is formatted to five decimal places and printed.

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

The parameters of this function:
- `repeat=5`: Runs the benchmark 5 times, producing a list of 5 results. Each result corresponds to the total execution time for the specified number of iterations.
- `number=1000`: Executes the code snippet 1000 times per benchmark run.

The output:
- **List of Results**: Each value in the `results` list represents the total time for 1000 executions of the code snippet.
- **Best Execution Time**: Use `min(results)` to find the fastest execution time across the 5 runs. This is often the most reliable measure of the code's performance as it minimizes the impact of temporary system noise or background processes.

However, this method cannot be used conveniently for e.g. measuring execution time of functions. For this, we need a more advanced tool.

## **References on Time Profiling**

### **1. Official Documentation**
- [Python timeit Module](https://docs.python.org/3/library/timeit.html)  
  Detailed explanation of how to use the `timeit` module for benchmarking Python code.
- [Python time Module](https://docs.python.org/3/library/time.html)  
  Overview of the `time` module, including functions like `time()`, `sleep()`, and more.

### **2. Jupyter Notebook Magics**
- [IPython Magic Commands Documentation](https://ipython.readthedocs.io/en/stable/interactive/magics.html)  
  Comprehensive list of magic commands available in Jupyter and IPython, including `%time`, `%timeit`, and others.
- **GeeksforGeeks: Time Profiling in Python**  
  [Link](https://www.geeksforgeeks.org/profiling-in-python/)  
  A beginner-friendly introduction to time profiling methods in Python.


{% include links.md %}
