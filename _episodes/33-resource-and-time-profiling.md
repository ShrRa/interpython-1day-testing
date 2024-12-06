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

## Jupyter Magics

- What is magics
- Which kinds of magics exist
- How to use them for time profiling
- What if you don't use Jupyter?






---

## **What Are Jupyter Magics?**

Jupyter Magics are special commands in Jupyter Notebooks that extend functionality by providing shortcuts for tasks like timing, debugging, profiling, or interacting with the system. They come in two forms:
- **Line Magics**: Prefixed with `%`, they operate on a single line of code.
- **Cell Magics**: Prefixed with `%%`, they apply to the entire cell.

### **Why Use Magics?**
- Simplifies repetitive or complex tasks.
- Integrates external tools into notebooks.
- Provides performance insights.

---

## **Types of Magics**

### **1. Timing Magics**
- **`%time`**: Measures execution time of a single line.
- **`%%time`**: Measures execution time of an entire cell.
- **`%timeit`**: Repeats timing of a line for reliable results.
- **`%%timeit`**: Repeats timing of a cell for average results.

### **2. Debugging Magics**
- **`%debug`**: Launches an interactive debugger after an exception.
- **`%pdb`**: Enables debugging upon errors.

### **3. System and Shell Magics**
- **`%lsmagic`**: Lists available magics.
- **`%env`**: Displays or modifies environment variables.
- **`!command`**: Runs shell commands.

---

## **Using Magics for Time Profiling**
### **Definition of Time Profiling**

Time profiling is the process of measuring the time taken by a program, a specific block of code, or individual functions to execute. It helps developers analyze the performance of their software by identifying bottlenecks, inefficient operations, or high-latency components. Time profiling provides insights into how the program uses computational resources (CPU, memory, I/O), enabling performance optimization.



# **1. Why Time Profiling is Important**

## **Performance Optimization**
- Helps identify bottlenecks in the code.(we mentioned this)
- Enables developers to optimize resource usage (CPU, memory).(we mentioned this)

## **Scalability**
- Ensures the application performs well as the dataset or workload grows.

## **Cost Efficiency**
- Reduces execution time, saving computational resources and costs, especially in cloud environments.

## **User Experience**
- Enhances application responsiveness, critical in real-time or interactive systems.

---

# **2. Key Metrics in Time Profiling**

## **Wall Time**
- **Definition**: Total elapsed time from start to end of a task, including waiting time for I/O operations or other processes.

## **CPU Time**
- **Definition**: The time the CPU spends executing the code, further divided into:
  - **User Time**: Time spent executing user code.
  - **System Time**: Time spent on system-level operations, such as I/O or memory management.

## **Overhead**
- **Definition**: The additional time introduced by profiling tools themselves, which may slightly skew results.
---

### **Key Characteristics of Time Profiling**

#### **Precision**
- Measures execution time at granular levels, from individual operations to entire workflows.
- Can use high-resolution timers like `time.perf_counter()` for microsecond-level precision.


#### **Tools and Methods**
- **Jupyter Notebook Magics**: `%time`, `%timeit`, and `%%timeit`.
- **Python Modules**: `time`, `timeit`, `cProfile`.
- **System Monitors**: Tools like `perf` (Linux) or Task Manager (Windows) for system-level profiling.



### **Line Timing with `%time`**

```python
# Example: Calculate sum of squares
%time sum([i**2 for i in range(1000000)])

## **Output Explanation**

**CPU times:**
- **User time (208 ms)**: Time spent executing the Python code by the CPU directly.
- **System time (16.6 ms)**: Time spent by the CPU performing system-level operations such as memory management or I/O.
- **Total CPU time (225 ms)**: The sum of user and system times.
- **Wall time (223 ms)**: The actual elapsed time, including any time waiting for resources like I/O or other processes.

### **Key Notes About `%timeit` and Your Output:**
- **Variability**: The results can vary depending on several factors:
  - **System load**: Background processes running on your machine can affect execution times.
  - **CPU and memory resources**: Hardware specifications can influence performance.
  - **Python optimizations**: Certain operations may trigger optimizations like caching.

### Factors Affecting Performance

- **System Load**: Background processes running on your machine can affect the execution time.
- **CPU and Memory Resources**: Hardware specifications can significantly influence performance.
- **Python Optimizations**: Certain operations may trigger optimizations such as caching.

### **Wall vs. CPU Time**

- The Wall time is slightly shorter than the total CPU time, which may seem counterintuitive. This discrepancy is often due to parallel processing or efficient CPU scheduling by the operating system.

### **Why `%timeit` Is Reliable**

- `%timeit` executes the code multiple times and averages the execution times, effectively minimizing the impact of transient system variations.
- The best run from a set of iterations is typically reported, ensuring that outliers from temporary system slowdowns are not included.

### **Result Interpretation**

- **Computation Details**: Your code computes the sum of squares for 1,000,000 numbers in approximately 223 ms (real time), while the CPU spent 225 ms in combined user and system processes.
- **Operation Intensity**: The sum operation is computationally intensive, as it iterates through a large range of numbers and calculates squares.

### **Repeated Timing with `%timeit`**

```python
# Example: Repeated timing for more reliable results
%timeit sum([i**2 for i in range(1000000)])

Output
189 ms ± 6.13 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)


### **Detailed Explanation**

#### **Mean Execution Time**
- **Duration**: The mean execution time for the command was 189 ms. This represents the average time it took to execute the sum operation across all the timing runs.

#### **Standard Deviation**
- **Consistency**: The ± 6.13 ms indicates the standard deviation of the execution times across the runs. A small standard deviation relative to the mean suggests that the execution times were consistent.

#### **Number of Runs**
- **Repetition Details**: `%timeit` performed 7 runs, with each run consisting of a single execution of the operation (1 loop each).

#### **How `%timeit` Works**
- **Methodology**: `%timeit` executes the command multiple times to ensure the reliability of timing data. It provides the mean time (average) and the standard deviation (variability) for the runs.

#### **Resources Used**
- **Resource Intensity**: This operation is CPU-intensive but not memory-intensive because the intermediate list of squares is created in memory.

### **Result Interpretation**
- **Performance**: The average time it took to compute the operation during the 7 runs was 189 ms per loop. The standard deviation of 6.13 ms suggests consistent performance across the runs.

### **System Influence**
- **Factors Affecting Performance**:
  - **Background Processes**: Timing results can vary based on the background processes running on the system.
  - **Hardware Specifications**: The machine's CPU and memory specifications can significantly impact performance.
  - **Python Optimizations**: Internal optimizations for list comprehensions and summations in Python can also affect execution times.

## **Timing Full Cells with `%%time`**

The `%%time` magic command in Jupyter Notebook is used to measure the execution time of the entire code cell. This command is very useful for obtaining a quick measure of the time it takes for a cell to run from start to finish.

### **Usage Example**

Here is an example of how to use `%%time` in a Jupyter Notebook cell to measure the execution time of code that sums the numbers from 0 to 999 using a for loop:

```python
%%time
# Example: Timing a block of code
result = []
for i in range(1000):
    result.append(sum(range(i)))


## **Detailed Analysis Using `%%time` Magic Command**

### **Execution Times and System Interaction**

- **CPU Times**:
  - **User time (10.2 ms)**: Represents the time spent directly executing Python code on the CPU.
  - **System time (1.68 ms)**: Time the CPU dedicated to system-level operations such as memory management or I/O handling.
  - **Total CPU time (11.9 ms)**: Aggregate of user time and system time, reflecting total CPU engagement for this block of code.
  - **Wall time (10.5 ms)**: Real-time elapsed from the start to the end of the execution, capturing delays beyond CPU processing like waiting for I/O resources.

### **How `%%time` Works**

- The `%%time` magic provides comprehensive timing metrics, capturing both CPU engagement times and real-world wall time. This gives a full spectrum of execution duration, from computational effort to external wait times.

### **Code Behavior Analysis**

- **Functionality**:
  - Initializes a list named `result` to store the sums computed.
  - Executes a for loop 1,000 times (`range(1000)`), each loop:
    - Generates a `range` object from 0 to i - 1.
    - Calculates the total of these numbers using the `sum` function.
    - Appends the calculated sum to the `result` list.

### **Resource Usage and System Efficiency**

- **CPU Usage**:
  - Primarily engaged in mathematical operations (summing and iterating over ranges) and managing list append operations.
- **System Calls**:
  - Involves minor system-level operations, likely linked to memory allocation for the growing `result` list.

### **Performance Insights**

- **CPU vs. Wall Time**:
  - Wall time (10.5 ms) is slightly less than the total CPU time (11.9 ms), a phenomenon possibly explained by effective parallel processing or efficient scheduling by the operating system.
- **System Time Proportion**:
  - System time (1.68 ms) is significantly less than user time (10.2 ms), indicating that most computational effort is spent within user-mode code execution, not system-level tasks.

### **Scalability Considerations**

- The execution time scales with the size of the range processed; thus, expanding the range (e.g., to `range(10000)`) would proportionally increase the execution time due to the quadratic nature of the operations involved.

### **Key Takeaways**

- **Efficiency**:
  - The operation is relatively lightweight, requiring only 10.5 ms of wall time.
- **Resource Allocation**:
  - A majority of the time is utilized in the Python execution layer (10.2 ms user time) with minimal overhead from system tasks (1.68 ms system time).
- **Performance Character**:
  - This result underscores that the code is predominantly CPU-bound with low dependence on system I/O, emphasizing its computational intensity rather than resource wait or idle times.

### **Optimization Suggestion**

#### **List Comprehension**

Replace the `for` loop with a list comprehension for potentially faster execution:

```python
%%time
# Optimized implementation with list comprehension
result = [sum(range(i)) for i in range(1000)]

### **Output**
```text
CPU times: user 10.1 ms, sys: 553 µs, total: 10.7 ms
Wall time: 10.3 ms



### **Key Observations**

#### **User Time**
- Both implementations (simple `for` loop and list comprehension) have nearly identical user time:
  - `for` loop: **10.2 ms**
  - List comprehension: **10.1 ms**
- This similarity occurs because the primary computation (`sum(range(i))`) remains unchanged.

#### **System Time**
- The list comprehension significantly reduces system time:
  - `for` loop: **1.68 ms**
  - List comprehension: **553 µs**
- The improvement results from eliminating the explicit `.append()` operation in the loop, which reduces system calls (e.g., memory allocation).

#### **Total CPU Time**
- The list comprehension achieves a modest reduction in total CPU time:
  - `for` loop: **11.9 ms**
  - List comprehension: **10.7 ms**
- This reduction is due to streamlined memory operations and reduced overhead.

#### **Wall Time**
- The Wall time is slightly faster for the list comprehension:
  - `for` loop: **10.5 ms**
  - List comprehension: **10.3 ms**
- This reflects the overall efficiency improvement in the list comprehension approach.

---

### **Why List Comprehension Is Faster**

#### **Streamlined Execution**
- List comprehensions are optimized for creating lists in Python.
- They avoid the overhead of repeatedly calling `.append()` in a loop.

#### **Fewer System Calls**
- By pre-allocating memory for the entire list, list comprehensions reduce the need for multiple memory allocations.
- This leads to significantly lower system time.

#### **Better Integration with Python Internals**
- The concise syntax of list comprehensions allows Python's interpreter to apply internal optimizations, improving performance.

---

### **When to Use List Comprehensions**

#### **For Simple Iterative Operations**
- Use list comprehensions when transforming or filtering data within a single concise operation.

#### **When Performance Matters**
- In performance-critical code, list comprehensions can provide small but meaningful speedups over explicit loops.

#### **For Readability**
- List comprehensions are often more readable and considered more Pythonic compared to explicit `for` loops.

## **Full Block Timing with `%%timeit`**

### **Code Example**
```python
%%timeit
# Example: Average execution time for a full block
result = []
for i in range(1000):
    result.append(sum(range(i)))

### **Output**
```text
6.59 ms ± 105 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)

### **Explanation**

1. **Mean Execution Time**:
   - **6.59 ms** is the mean (average) time it took to execute the entire block of code for one loop.
   - This mean was calculated across **100 loops**, providing a robust estimate of the execution time for the operation.

2. **Standard Deviation**:
   - The **± 105 µs** represents the standard deviation of the execution times across the runs.
   - A smaller standard deviation (relative to the mean) indicates that the results are consistent across runs, with minimal variability.

3. **Number of Runs**:
   - The operation was executed **7 runs**, and each run consisted of **100 loops**.
   - This gives a total of **700 executions** of the code block, ensuring a reliable measurement.

---

### **What the Code Does**
1. Initializes an empty list `result`.
2. Iterates over a range of **1000 values**.
3. For each iteration:
   - Creates a `range` object (`range(i)`) of integers from `0` to `i-1`.
   - Computes the sum of the range using `sum(range(i))`.
   - Appends the result of the sum to the `result` list.

---

### **Why the Code Takes Time**

#### **Iteration Over 1000 Elements**:
- A loop with **1000 iterations** inherently requires time, especially when performing computational tasks in each iteration.

#### **Summation of Ranges**:
- The `sum(range(i))` operation is executed **1000 times**.
- Its computational complexity increases as `i` grows.

#### **List Appending**:
- Appending to the list involves **dynamic memory allocation** for the growing `result` list.

---

### **Performance Breakdown**

#### **Mean Time (6.59 ms)**:
- This average time includes the time spent on all iterations, summations, and appending to the list.
- It is a reliable measure because it is averaged over **7 runs of 100 loops each**.

#### **Standard Deviation (105 µs)**:
- The low standard deviation indicates that the timing results were stable across all runs.
- Variability could arise from factors like background system processes or temporary resource contention.

#### **Number of Loops and Runs**:
- **100 loops per run** ensure that transient noise (e.g., CPU spikes) has minimal impact on the measurement.
- **7 runs** provide sufficient repetition to calculate a robust average and standard deviation.

### **Optimization Suggestion**

#### **Using a Mathematical Formula**

Replace the iterative summation with a direct mathematical formula for efficiency. This approach avoids repeated calculations and reduces computational complexity.

```python
%%timeit
# Optimized implementation using mathematical formula for summation
result = [(i * (i - 1)) // 2 for i in range(1000)]

### **Output**
```text
505 µs ± 8.01 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)

### **Result Interpretation**
- The optimized code reduces execution time by approximately **13x**, achieving a mean time of **505 µs** compared to the original **6.59 ms**.
- It also simplifies the logic and reduces system resource usage, making it a more efficient solution for generating the desired result.

---

### **Performance Improvement**

#### **Why It’s Faster**
1. **Reduced Complexity**:
   - The original code involves:
     - Creating a `range` object.
     - Iterating through it.
     - Performing summation for each element.
   - This results in a time complexity of \(O(n^2)\) across all iterations.
   - The optimized code uses a direct formula with \(O(1)\) complexity for each iteration, reducing the overall time complexity to \(O(n)\).

2. **Resource Savings**:
   - No intermediate objects like `range` are created, saving memory.
   - Avoids Python’s built-in `sum` function overhead by replacing it with a direct mathematical operation.

3. **Python Optimization**:
   - Arithmetic operations (multiplication and division) are highly optimized in Python.
   - These operations are faster compared to iterative constructs like `for` loops and repeated function calls.

---

### **Key Takeaways**
- **Execution Time**:
  - Original: **6.59 ms**
  - Optimized: **505 µs**
- **Efficiency**:
  - The mathematical formula eliminates intermediate operations, reducing overhead.
- **Applicability**:
  - Ideal for scenarios requiring high performance and minimal resource usage.


## **What If You Don’t Use Jupyter?**

If you’re not using Jupyter Notebooks, you can still measure execution time using Python's built-in modules like `time`. Below is an example using the `time` module.

### **Code Example**
```python
import time

start_time = time.time()
result = sum([i**2 for i in range(1000000)])
print(f"Execution time: {time.time() - start_time:.2f} seconds")

Execution time: 0.21 seconds


### **Explanation**

#### **1. Code Breakdown**

- **Start Timer**:
  - The `time.time()` function records the current time in seconds since the epoch (typically January 1, 1970).
  - This value is stored in the variable `start_time` before the code block is executed.

- **Operation**:
  - A list comprehension `[i**2 for i in range(1000000)]` generates a list of squares for integers from `0` to `999,999`.
  - The `sum` function calculates the total sum of these squared values.

- **End Timer and Elapsed Time**:
  - After the code block, `time.time()` is called again to get the current time.
  - The difference between the current time and `start_time` gives the total execution time.
  - This elapsed time is formatted to two decimal places and printed.

---

#### **2. Why It Takes 0.21 Seconds**

- **List Comprehension**:
  - The list comprehension `[i**2 for i in range(1000000)]` iterates over a range of 1,000,000 integers.
  - For each integer `i`, it calculates `i**2` (a multiplication operation) and stores the result in a list.
  - The time complexity of this operation is \(O(n)\), where \(n = 1000000\).

- **Summation**:
  - The `sum` function iterates through the list of squared values and accumulates the total.
  - This operation also has a time complexity of \(O(n)\).

- **Combined Complexity**:
  - The combined time complexity of generating the list and summing it is \(O(n)\), where \(n = 1000000\).

- **Hardware Influence**:
  - Execution time depends on the system's CPU performance and memory speed.
  - A faster CPU or lower system load can reduce execution time.

---

### **Key Notes**
- The operation involves both computational (squaring) and summation steps, each contributing to the overall execution time.
- Hardware and system load play a critical role in determining the actual elapsed time.
- Optimization techniques, such as using generator expressions or mathematical formulas, can further reduce execution time.


### **Simple Generator (No List)**

#### **Code Example**
```python
import time

start_time = time.time()
result = sum(i**2 for i in range(1000000))  # Generator expression
print(f"Execution time: {time.time() - start_time:.2f} seconds")

Execution time: 0.20 seconds
### **Why Use a Generator Instead of a List Comprehension?**

#### **Key Differences**
- **Memory Usage**:
  - Unlike a list comprehension, a generator expression does not create the entire list in memory.
  - Instead, it generates each value on demand, significantly reducing memory overhead.

---

#### **How the Generator Works**
- **Summation**:
  - The `sum` function iterates over the values produced by the generator and accumulates the total.
  
- **End Timer and Elapsed Time**:
  - After summation, the current time is recorded.
  - The elapsed time is calculated as the difference between the current time and `start_time`.

---

#### **Advantages of On-the-Fly Computation**
1. **Dynamic Value Generation**:
   - Values are computed one at a time and passed directly to `sum`.
   - Avoids the need to store all intermediate results in memory.

2. **Memory Efficiency**:
   - Memory usage remains constant regardless of the size of the `range`, as only one value is processed at a time.

---

#### **Scalability**
- For very large ranges, the generator expression avoids the memory limitations of a list comprehension.
- It is ideal for situations where memory efficiency is critical.

---

#### **Efficiency**
- While the reduction in execution time compared to the list comprehension is modest (from **0.21 seconds** to **0.20 seconds**), the savings in memory usage can be substantial.
- This makes generator expressions highly suitable for scenarios involving large data sets or constrained system resources.


### **Replacing Iterative Code with a Mathematical Formula**

#### **Code Example**
```python
import time

start_time = time.time()
n = 999999
result = (n * (n + 1) * (2 * n + 1)) // 6  # Formula for sum of squares
print(f"Execution time: {time.time() - start_time:.6f} seconds")

Execution time: 0.001124 seconds
### **Eliminating Iteration with a Mathematical Formula**

#### **Key Advantages of the Formula**
- **No Iteration or Summation**:
  - This formula eliminates the need for looping or accumulating values, reducing the time complexity to \(O(1)\).
  - It performs a constant number of arithmetic operations regardless of the range size.

---

#### **Explanation**

1. **Assigning Variables**:
   - The variable `n` is set to **999999**, representing the range of numbers for which the sum of squares is calculated.

2. **Start Timer**:
   - The `time.time()` function records the start time before the computation begins.

3. **Perform Calculation**:
   - The formula \((n \times (n + 1) \times (2 \times n + 1)) // 6\) computes the sum of squares directly.
   - Since Python handles large integers efficiently, this operation is executed very quickly.

4. **End Timer and Elapsed Time**:
   - After the computation, the current time is recorded using `time.time()`.
   - The elapsed time is calculated as the difference between the current time and `start_time`.

---

#### **Why it is so fast**

1. **No Iteration**:
   - Unlike list comprehensions or generator expressions, the mathematical formula doesn’t involve looping through the range.
   - The entire calculation is performed in a single operation.

2. **Constant Time Complexity O(1)**:
   - The execution time remains constant regardless of the value of (n).
   - Whether n = 100 or n = 10^9, the number of operations stays the same.

3. **Efficient Arithmetic Operations**:
   - Multiplication and division are highly optimized in Python, even for large integers.
   - These operations are faster compared to iterative constructs and avoid intermediate steps.

---

#### **Performance Highlights**

| Metric                     | Iterative Approach       | Generator Expression    | Mathematical Formula   |
|----------------------------|--------------------------|--------------------------|------------------------|
| **Execution Time**         | 0.21 seconds            | 0.20 seconds            | 0.001124 seconds      |
| **Memory Usage**           | High (stores full list)  | Low (on-demand values)  | Minimal (constant)     |
| **Complexity**             | O(n)                | (O(n)                | O(1)               |

---

### **Key Notes**
- The mathematical formula offers unparalleled performance by eliminating iteration altogether.
- Ideal for scenarios requiring efficiency and scalability, especially when working with extremely large values of n.
- The minimal memory usage and constant execution time make it the optimal choice for calculating arithmetic series like the sum of squares.

## **Prefer `repeat()` for Reliable Results**

### **Why Use `repeat()`?**
The `repeat()` method in the `timeit` module is an effective way to handle variability in system conditions. It executes the timing multiple times and provides a list of results, making it easier to analyze performance under changing environments.

- **Handles Variability**: Useful when background processes may affect timing accuracy.
- **Provides Detailed Insights**: Offers multiple results instead of a single value, allowing you to analyze the range of execution times.
- **Highlights the Best Case**: Helps identify the fastest execution time, which is typically the most accurate representation of your code's performance.

---

### **Example**
```python
import timeit

# Benchmark a simple operation multiple times
results = timeit.repeat('sum([i**2 for i in range(1000)])', repeat=5, number=1000)

print("Timing Results:", results)
print("Best Execution Time:", min(results))

Timing Results: [0.17868508584797382, 0.16405000817030668, 0.16924912948161364, 0.1637995233759284, 0.16636504232883453]
Best Execution Time: 0.1637995233759284

### **Explanation**

- **`repeat=5`**: Runs the benchmark 5 times, producing a list of 5 results. Each result corresponds to the total execution time for the specified number of iterations.
- **`number=1000`**: Executes the code snippet 1000 times per benchmark run.

---

### **Results Analysis**
- **List of Results**: Each value in the `results` list represents the total time for 1000 executions of the code snippet.
- **Best Execution Time**: Use `min(results)` to find the fastest execution time across the 5 runs. This is often the most reliable measure of the code's performance as it minimizes the impact of temporary system noise or background processes.

---

### **Benefits of `repeat()`**
1. **Accurate Results**:
   - Captures the fastest execution time, which reflects the true performance of the code under ideal conditions.
2. **Noise Reduction**:
   - Minimizes the effect of outliers caused by temporary system interruptions, such as CPU spikes or memory contention.
3. **Flexible Analysis**:
   - Allows comparison of results across multiple runs to assess consistency and variability in execution times.

---

### **Explore `autorange()` for Automatic Loop Determination**

The `autorange()` method in the `timeit` module automatically calculates the optimal number of loops to ensure that the total execution time is at least 0.2 seconds. This eliminates the need for manual adjustment of the `number` parameter and simplifies the benchmarking process.

---

### **How It Works**
1. **Automatic Calculation**:
   - `autorange()` starts with a single loop and increases the loop count using the sequence: 1, 2, 5, 10, 20, 50, ….
   - It continues until the total time for executing the code snippet exceeds 0.2 seconds.

2. **Output**:
   - Returns a tuple containing:
     - The optimal number of loops.
     - The total time taken for that number of loops.

---

### **Example**
```python
import timeit

# Define a Timer object
t = timeit.Timer('sum([i**2 for i in range(1000)])')

# Use autorange() to determine the optimal number of loops
loops, time_taken = t.autorange()

print(f"Optimal number of loops: {loops}")
print(f"Time taken for {loops} loops: {time_taken:.6f} seconds")

Optimal loops: 2000, Time taken: 0.364260 seconds



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


## Resource profiling

- What is profiling
- What kinds of resources can we measure
- What are the complications
- Main tools
- Using SnakeViz

{% include links.md %}
