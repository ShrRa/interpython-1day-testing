---
title: "Scaling Up Unit Testing"
teaching: 10
exercises: 5
questions:
- "How can we make it easier to write lots of tests?"
- "How can we know how much of our code is being tested?"
objectives:
- "Use parameterisation to automatically run tests over a set of inputs"
- "Use code coverage to understand how much of our code is being tested using unit tests"
keypoints:
- "We can assign multiple inputs to tests using parametrisation."
- "It's important to understand the **coverage** of our tests across our code."
- "Writing unit tests takes time, so apply them where it makes the most sense."
---

## Introduction

We're starting to build up a number of tests that test the same function,
but just with different parameters.
However, continuing to write a new function for every single test case
isn't likely to scale well as our development progresses.
How can we make our job of writing tests more efficient?
And importantly, as the number of tests increases,
how can we determine how much of our code base is actually being tested?

## Parameterising Our Unit Tests

So far, we've been writing a single function for every new test we need.
But when we simply want to use the same test code but with different data for another test,
it would be great to be able to specify multiple sets of data to use with the same test code.
Test **parameterisation** gives us this.

So instead of writing a separate function for each different test,
we can **parameterise** the tests with multiple test inputs.
For example, in `tests/test_models.py` let us rewrite
the `test_max_mag_zeros()` and `test_max_mag_integers()`
into a single test function:

~~~
@pytest.mark.parametrize(
    "test_df, test_colname, expected",
    [
        (pd.DataFrame(data=[[1, 5, 3], 
                            [7, 8, 9], 
                            [3, 4, 1]], 
                      columns=list("abc")),
        "a",
        7),
        (pd.DataFrame(data=[[0, 0, 0], 
                            [0, 0, 0], 
                            [0, 0, 0]], 
                      columns=list("abc")),
        "b",
        0),
    ])
def test_max_mag(test_df, test_colname, expected):
    """Test max function works for array of zeroes and positive integers."""
    from lcanalyzer.models import max_mag
    assert max_mag(test_df, test_colname) == expected
~~~
{: .language-python}

Here, we use Pytest's **mark** capability to add metadata to this specific test -
in this case, marking that it's a parameterised test.
`parameterize()` function is actually a
[Python **decorator**](https://www.programiz.com/python-programming/decorator).
A decorator, when applied to a function,
adds some functionality to it when it is called, and here,
what we want to do is specify multiple input and expected output test cases
so the function is called over each of these inputs automatically when this test is called.

We specify these as arguments to the `parameterize()` decorator,
firstly indicating the names of these arguments that will be
passed to the function (`test_df`, `test_colname`, `expected`),
and secondly the actual arguments themselves that correspond to each of these names -
the input data (the `test_df` and `test_colname` arguments),
and the expected result (the `expected` argument).
In this case, we are passing in two tests to `test_max_mag()` which will be run sequentially.

So our first test will run `max_mag()` on `pd.DataFrame(data=[[1, 5, 3], 
                            [7, 8, 9], 
                            [3, 4, 1]], 
                      columns=list("abc"))` (our `test_df` argument),
and check to see if it equals `7` (our `expected` argument) with `test_colname` set to `'a'`.
Similarly, our second test will run `max_mag()`
with `pd.DataFrame(data=[[0, 0, 0], 
                            [0, 0, 0], 
                            [0, 0, 0]], 
                      columns=list("abc"))` and check it produces `0` with `test_colname` set to `'b'`.

The big plus here is that we don't need to write separate functions for each of the tests -
our test code can remain compact and readable as we write more tests
and adding more tests scales better as our code becomes more complex.

> ## Exercise: Write Parameterised Unit Tests
>
> Rewrite your test functions for `mean_mag()` to be parameterised,
> adding in new test cases. A suggestion: instead of filling the DataFrames manually,
> you can use `numpy.random.randint()` and `numpy.random.rand()` functions.
> When developing these tests you are likely to see a situation when
> the expected value is a float. In some cases your code may produce
> the output that has some uncertainty; how do you test such functions?
> For this situation, `pytest` has a special function called `approx`.
> It allow you to assert _similar_ values with some degree of precision;
> e.g. `assert func(input) == pytest.approx(expected,0.01))` returns `True`
> in when `(expected-0.01)<=func(input)<=(expected+0.01)`. Similar solutions
> exist for `numpy.testing` and other testing tools.
>
> > ## Solution
> > ~~~
> > ...
> > # Parametrization for mean_mag function testing
> > @pytest.mark.parametrize(
> >     "test_df, test_colname, expected",
> >     [
> >         (pd.DataFrame(data=[[1, 5, 3], 
> >                             [7, 8, 9], 
> >                             [3, 4, 1]], 
> >                       columns=list("abc")),
> >         "a",
> >         pytest.approx(3.66,0.01)),
> >         (pd.DataFrame(data=[[0, 0, 0], 
> >                             [0, 0, 0], 
> >                             [0, 0, 0]], 
> >                       columns=list("abc")),
> >         "b",
> >         0),
> >     ])
> > def test_mean_mag(test_df, test_colname, expected):
> >     """Test mean function works for array of zeroes and positive integers."""
> >     from lcanalyzer.models import mean_mag
> >     assert mean_mag(test_df, test_colname) == expected
> > ~~~
> > {: .language-python}
> {: .solution}
>
{: .challenge}

Let's commit our revised `test_models.py` file and test cases to our `test-suite` branch
(but don't push them to the remote repository just yet!):

~~~
$ git add tests/test_models.py
$ git commit -m "Add parameterisation mean, min, max test cases"
~~~
{: .language-bash}


## Code Coverage - How Much of Our Code is Tested?

Pytest can't think of test cases for us.
We still have to decide what to test and how many tests to run.
Our best guide here is economics:
we want the tests that are most likely to give us useful information that we don't already have.
For example, if testing our `max_mag` function with a DataFrame filled with integers works,
there's probably not much point testing the same function with a DataFrame filled with other integers,
since it's hard to think of a bug that would show up in one case but not in the other. Note, however,
that for other function this statement may be incorrect (e.g. if your function is supposed to discard values
above a certain threshold, and your test case input does not contain such values at all).

Now, we should try to choose tests that are as different from each other as possible,
so that we force the code we're testing to execute in all the different ways it can -
to ensure our tests have a high degree of **code coverage**.

A simple way to check the code coverage for a set of tests is
to use `pytest` to tell us how many statements in our code are being tested.
By installing a Python package to our virtual environment called `pytest-cov`
that is used by Pytest and using that, we can find this out:

~~~
$ pip3 install pytest-cov
$ python -m pytest --cov=lcanalyzer.models tests/test_models.py
~~~
{: .language-bash}

So here, we specify the additional named argument `--cov` to `pytest`
specifying the code to analyse for test coverage.

~~~
==================================== test session starts ====================================
platform linux -- Python 3.11.5, pytest-8.0.0, pluggy-1.4.0
rootdir: /home/alex/InterPython_Workshop_Example
plugins: anyio-4.2.0, cov-4.1.0
collected 9 items                                                                           

tests/test_models_full.py .........                                                   [100%]

---------- coverage: platform linux, python 3.11.5-final-0 -----------
Name                   Stmts   Miss  Cover
------------------------------------------
lcanalyzer/models.py      12      1    92%
------------------------------------------
TOTAL                     12      1    92%


===================================== 9 passed in 0.70s =====================================
~~~
{: .output}

Here we can see that our tests are doing well - 92% of statements in `lcanalyzer/models.py` have been executed.
But which statements are not being tested?
The additional argument `--cov-report term-missing` can tell us:

~~~
$ python -m pytest --cov=lcanalyzer.models --cov-report term-missing tests/test_models.py
~~~
{: .language-bash}

~~~
...
==================================== test session starts ====================================
platform linux -- Python 3.11.5, pytest-8.0.0, pluggy-1.4.0
rootdir: /home/alex/InterPython_Workshop_Example
plugins: anyio-4.2.0, cov-4.1.0
collected 11 items                                                                          

tests/test_models.py ..                                                               [ 18%]
tests/test_models_full.py .........                                                   [100%]

---------- coverage: platform linux, python 3.11.5-final-0 -----------
Name                   Stmts   Miss  Cover   Missing
----------------------------------------------------
lcanalyzer/models.py      12      1    92%   20
----------------------------------------------------
TOTAL                     12      1    92%


==================================== 11 passed in 0.71s =====================================

...
~~~
{: .output}

So there's still one statement not being tested at line 20,
and it turns out it's in the function `load_dataset()`.
Here we should consider whether or not to write a test for this function,
and, in general, any other functions that may not be tested.
Of course, if there are hundreds or thousands of lines that are not covered
it may not be feasible to write tests for them all.
But we should prioritise the ones for which we write tests, considering
how often they're used,
how complex they are,
and importantly, the extent to which they affect our program's results.

Again, we should also update our `requirements.txt` file with our latest package environment,
which now also includes `pytest-cov`, and commit it:

~~~
$ pip3 freeze > requirements.txt
$ cat requirements.txt
~~~
{: .language-bash}

You'll notice `pytest-cov` and `coverage` have been added.
Let's commit this file and push our new branch to GitHub:

~~~
$ git add requirements.txt
$ git commit -m "Add coverage support"
$ git push origin test-suite
~~~
{: .language-bash}

> ## What about Testing Against Indeterminate Output?
>
> What if your implementation depends on a degree of random behaviour?
> This can be desired within a number of applications,
> particularly in simulations or machine learning.
> So how can you test against such systems if the outputs are different when given the same inputs?
>
> One way is to *remove the randomness* during testing.
> For those portions of your code that
> use a language feature or library to generate a random number,
> you can instead produce a known sequence of numbers instead when testing,
> to make the results deterministic and hence easier to test against.
> You could encapsulate this different behaviour in separate functions, methods, or classes
> and call the appropriate one depending on whether you are testing or not.
> This is essentially a type of **mocking**,
> where you are creating a "mock" version that mimics some behaviour for the purposes of testing.
>
> Another way is to *control the randomness* during testing
> to provide results that are deterministic - the same each time.
> Implementations of randomness in computing languages, including Python,
> are actually never truly random - they are **pseudorandom**:
> the sequence of 'random' numbers are typically generated using a mathematical algorithm.
> A **seed** value is used to initialise an implementation's random number generator,
> and from that point, the sequence of numbers is actually deterministic.
> Many implementations just use the system time as the default seed,
> but you can set your own.
> By doing so, the generated sequence of numbers is the same,
> e.g. using Python's `random` library to randomly select a sample
> of ten numbers from a sequence between 0-99:
>
> ~~~
> import random
>
> random.seed(1)
> print(random.sample(range(0, 100), 10))
> random.seed(1)
> print(random.sample(range(0, 100), 10))
> ~~~
> {: .language-python}
>
> Will produce:
>
> ~~~
> [17, 72, 97, 8, 32, 15, 63, 57, 60, 83]
> [17, 72, 97, 8, 32, 15, 63, 57, 60, 83]
> ~~~
> {: .output}
>
> So since your program's randomness is essentially eliminated,
> your tests can be written to test against the known output.
> The trick of course, is to ensure that the output being testing against is definitively correct!
>
> The other thing you can do while keeping the random behaviour,
> is to *test the output data against expected constraints* of that output.
> For example, if you know that all data should be within particular ranges,
> or within a particular statistical distribution type (e.g. normal distribution over time),
> you can test against that,
> conducting multiple test runs that take advantage of the randomness
> to fill the known "space" of expected results.
> Note that this isn't as precise or complete,
> and bear in mind this could mean you need to run *a lot* of tests
> which may take considerable time.
{: .callout}

## Package-specific asserts

Let us add a new function to our jupyter notebook called `calc_stats()` 
that will calculate for us all three statistical indicators (min, max and mean) for all
bands of our light curve.
(Make sure you create a new feature branch for this work off your `develop` branch.)

~~~
def calc_stats(lc, bands, mag_col):
    # Calculate max, mean and min values for all bands of a light curve
    stats = {}
    for b in bands:
        stat = {}
        stat["max"] = models.max_mag(lc[b], mag_col)
        stat["mean"] = models.max_mag(lc[b], mag_col)
        stat["min"] = models.mean_mag(lc[b], mag_col)
        stats[b] = stat
    return pd.DataFrame.from_records(stats)
~~~
{: .language-python}

**Note:** *there are intentional mistakes in the above code,
which will be detected by further testing and code style checking below
so bear with us for the moment!*

This code accepts a dictionary of DataFrames that contain observations of a single object in all bands.
Then this code iterates through the bands, calculating the requested statistical values and storing them
in a dictionary. At the end, these dictionaries are converted into a DataFrame, where column names are the
keys of the original `lc` dictionary, and the index ('row names') are the names of the statistics ('max',
'mean' and 'min'). Pass one of our previously designed light curves to this function
to see that the result is an accurate and informative pandas table.

> ## Can't we save them directly into a DataFrame?
> Technically, we can. However, editing DataFrames row by row or element by element
> is inefficient from the computational point of view. For this reason, when creating a frame
> row by row is inevitable, storing data
> in a list, dictionary or array and then converting them in a DataFrame is the preferred
> solution. It is also worth noting that in many cases iterations in a loop
> through the rows of some kind of a table can be avoided entirely with a better
> design of the algorithm.
{: .callout}

Now let's design a test case for this function:

~~~
test_cols = list("abc")
test_dict = {}
test_dict["df0"] = pd.DataFrame(
    data=[[8, 8, 0], 
          [0, 1, 1], 
          [2, 3, 1], 
          [7, 9, 7]], columns=test_cols
)
test_dict["df1"] = pd.DataFrame(
    data=[[3, 8, 2], 
          [3, 8, 0], 
          [3, 9, 8], 
          [8, 2, 5]], columns=test_cols
)
test_dict["df2"] = pd.DataFrame(
    data=[[8, 4, 3], 
          [7, 6, 3], 
          [4, 2, 9], 
          [6, 4, 0]], columns=test_cols
)
~~~
{: .language-python}

Remember, that we don't have to fill the data manually, but can use built-in `numpy`
random generator. For example, for the data above `size = (4,3); np.random.randint(0, 10, size)` 
was used.

The expected output for these data will look like this:

~~~
test_output = pd.DataFrame(data=[[9,9,6],[5.25,6.75,4.],[1,2,2]],columns=['df0','df1','df2'],index=['max','mean','min'])
~~~
{: .language-python}

Finally, we can use `assert` statement to check if our function produces what we expect...
~~~
assert calc_stats(test_dict, test_dict.keys(), 'b') == test_output
~~~
{: .language-python}

...and get a `ValueError`:
~~~
...
ValueError: The truth value of a DataFrame is ambiguous. Use a.empty, a.bool(), a.item(), a.any() or a.all().
~~~
{: .output}

The reason for this is that `assert` takes a condition that produces a _single_ boolean value,
but using `==` for two DataFrames results in an element-wise comparison and produces a _DataFrame_
filled with booleans. 

This is the case when we need to use a more powerful `assert` function, the one that is developed specifically 
for a certain variable type. `Pandas` has its own module called `testing` that contains a number of type-specific
`assert` functions. Let's import this module:

~~~
import pandas.testing as pdt
~~~
{: .language-python}

And use `assert_frame_equal` function that can compare DataFrames in a meaningful way:
~~~
pdt.assert_frame_equal(calc_stats(test_dict, test_dict.keys(), 'b'),
                              test_output,
                             check_exact=False,
                             atol=0.01)
~~~
{: .language-python}

The first two arguments of this function are just what we would expect: the call of our `calc_stats`
function and the expected `test_output`. `assert_frame_equal` will be comparing these two DataFrames.
The next two arguments allow this function to compare the DataFrames with only some degree of precision.
This precision is determined by the argument `atol`, which stands for 'absolute tolerance'. The DataFrames
will be considered equal if their elements differ no more than by `atol` value. This is similar to the 
`pytest.approx` that we encountered in the previous episodes. 

This assertion is falining with an error message: apparently, our function produces `mean` that equals `9.0` 
instead of 5.25. 

~~~
...
AssertionError: DataFrame.iloc[:, 0] (column name="df0") are different

DataFrame.iloc[:, 0] (column name="df0") values are different (66.66667 %)
[index]: [max, mean, min]
[left]:  [9.0, 9.0, 5.25]
[right]: [9.0, 5.25, 1.0]
At positional index 1, first diff: 9.0 != 5.25
~~~
{: .output}

Apparently, there are differences between the two DataFrames in the column 'df0';
the values in the 'max' row are the same, but the 'mean' and 'min' values are different.
Going back to the code of the function, we discover that these two lines:
~~~
...
        stat["mean"] = models.max_mag(lc[b], mag_col)
        stat["min"] = models.mean_mag(lc[b], mag_col)
...
~~~
{: .language-python}
use incorrect functions - a common case when e.g. part of the code was copy-pasted. Now we can fix these errors and 
relaunch the tests to make sure everything else is correct.

{% include links.md %}
