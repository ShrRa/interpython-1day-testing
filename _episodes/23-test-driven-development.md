---
title: "Edge-Cases, Test Driven Development and Defensive Programming"
teaching: 30
exercises: 20
questions:
- "How can we make our programs more resilient to failure?"
- "Can we use testing to speed up our work?"
objectives:
- "Describe and identify edge and corner test cases and explain why they are important"
- "Apply defensive programming techniques to improve robustness of a program"
- "Learn what Test Driven Development is"
keypoints:
- "Ensure that unit tests check for **edge** and **corner cases** too."
-  "Use **preconditions** to ensure correct behaviour of code."
- "Write tests before the code itself to think of the functionality and desired behaviour in advance."
---

## Corner or Edge Cases

The test cases that we have written so far 
are parameterised with a fairly standard DataFrames filled with random integers or floats.
However, when writing your test cases,
it is important to consider parameterising them by unusual or extreme values,
in order to test all the edge or corner cases that your code could be exposed to in practice.
Generally speaking, it is at these extreme cases that you will find your code failing,
so it's beneficial to test them beforehand.

What is considered an "edge case" for a given component depends on
what that component is meant to do. 
For numerical values, extreme cases could be zeros,
very large or small values,
not-a-number (`NaN`) or infinity values.
Since we are specifically considering *arrays* of values,
an edge case could be that all the numbers of the array are equal.

For all the given edge cases you might come up with,
you should also consider their likelihood of occurrence.
It is often too much effort to exhaustively test a given function against every possible input,
so you should prioritise edge cases that are likely to occur. 
Let's consider a new function, which purpose is to normalize a single light curve:

~~~
def normalize_lc(df,mag_col):
    # Normalize a single light curve
    min = min_mag(df,mag_col)
    max = max_mag((df-min),mag_col)
    lc = (df[mag_col]-min)/max
    return lc
~~~
{: .language-python}

For a function like this, a common edge case might be the occurrence of zeros,
and the case where all the values of the array are the same. Indeed, if we passed 
to such a function a 'light curve' where all measurements are zeros, we would expect
to have zeros in return. However, since we have division in this function, it will return
and array of 'NaN' instead of this.

With this in mind,
let us create a test `test_normalize_lc` with parametrization
corresponding to an input array of random integers, an input array of all 0,
and an input array of all 1.

~~~
# Parametrization for normalize_lc function testing
@pytest.mark.parametrize(
    "test_input_df, test_input_colname, expected",
    [
        (pd.DataFrame(data=[[8, 9, 1], 
                            [1, 4, 1], 
                            [1, 2, 4], 
                            [1, 4, 1]], 
                      columns=list("abc")),
        "b",
        pd.Series(data=[1,0.285,0,0.285])),
        (pd.DataFrame(data=[[1, 1, 1], 
                            [1, 1, 1], 
                            [1, 1, 1], 
                            [1, 1, 1]], 
                      columns=list("abc")),
        "b",
        pd.Series(data=[0.,0.,0.,0.])),
        (pd.DataFrame(data=[[0, 0, 0], 
                            [0, 0, 0], 
                            [0, 0, 0], 
                            [0, 0, 0]], 
                      columns=list("abc")),
        "b",
         pd.Series(data=[0.,0.,0.,0.])),
    ])
def test_normalize_lc(test_input_df, test_input_colname, expected):
    """Test how normalize_lc function works for arrays of positive integers."""
    from lcanalyzer.models import normalize_lc
    import pandas.testing as pdt
    pdt.assert_series_equal(normalize_lc(test_input_df,test_input_colname),expected,check_exact=False,atol=0.01,check_names=False)
~~~
{: .language-python}

Pay attention, that since this our `normalize_lc` function returns a `pandas.Series`, we have to use the corresponding assert function
(`pdt.assert_series_equal`). Another thing to pay attention to is the arguments of this function. Not only we specify the `atol` for 
ensuring that there will be no issues when comparing floats, but also set `check_names=False`, since by default the `Series` returned from
the `normalize_lc` function will have the name of the column for which we performed the normalization. Custom assert functions, such as
`assert_series_equal`, often take a large number of arguments that specify which parameters of the objects have to be compared. E.g. you can 
opt out of comparing the `dtypes` of a `Series`, the column orders of a `DataFrame` and so on.

Running the tests now from the command line results in the following assertion error,
due to the division by zero as we predicted. Note that not only the test case with all zeros
fails, but also the test with all ones too, due to the extraction of the `min` value!

~~~
E   AssertionError: Series are different
E   
E   Series values are different (100.0 %)
E   [index]: [0, 1, 2, 3]
E   [left]:  [nan, nan, nan, nan]
E   [right]: [0.0, 0.0, 0.0, 0.0]
E   At positional index 0, first diff: nan != 0.0

testing.pyx:173: AssertionError
====================================================================================== short test summary info ======================================================================================
FAILED tests/test_models_full.py::test_normalize_lc[test_input_df1-b-expected1] - AssertionError: Series are different
FAILED tests/test_models_full.py::test_normalize_lc[test_input_df2-b-expected2] - AssertionError: Series are different
~~~
{: .output}

How can we fix this? For example, we can replace all the NaNs in the
return Series with zeros using `pandas` function `fillna`. 
~~~
...
def normalize_lc(df,mag_col):
    # Normalize a single light curve
    min = min_mag(df,mag_col)
    max = max_mag((df-min),mag_col)
    lc = (df[mag_col]-min)/max
    lc = lc.fillna(0)
    return lc
...
~~~
{: .language-python}

## Defensive Programming

In the previous section, we made a few design choices for our `normalize_lc` function:

1. We are implicitly converting any `NaN` to 0,
2. Normalising a constant array of magnitudes in an identical array of 0s,
3. We don't warn the user of any of these situations.

This could have be handled differently.
We might decide that we do not want to silently make these changes to the data,
but instead to explicitly check that the input data satisfies a given set of assumptions
(e.g. no negative values or no values outside of a certain range)
and raise an error if this is not the case.
Then we can proceed with the normalisation,
confident that our normalisation function will work correctly.

Checking that input to a function is valid via a set of preconditions
is one of the simplest forms of **defensive programming**
which is used as a way of avoiding potential errors.
Preconditions are checked at the beginning of the function
to make sure that all assumptions are satisfied.
These assumptions are often based on the *value* of the arguments, like we have already discussed.
However, in a dynamic language like Python
one of the more common preconditions is to check that the arguments of a function
are of the correct *type*.
Currently there is nothing stopping someone from calling `normalize_lc` with
a string, a dictionary, or another object that is not a DataFrame, or from
passing a DataFrame filled with strings or lists.

As an example, let us change the behaviour of the `normalize_lc()` function
to raise an error if some magnitudes are smaller than '-90' (since in astronomical data '-99.' or '-99.9' are
common filler values for 'NaNs').
Edit our function by adding a precondition check like so:

~~~
...
    if any(df[mag_col].abs() > 90):
        raise ValueError(mag_col+' contains values with abs() larger than 90!')
...
~~~
{: .language-python}

We can then modify our test function
to check that the function raises the correct exception - a `ValueError` -
when input to the test contains '-99.9' values.
The [`ValueError`](https://docs.python.org/3/library/exceptions.html#ValueError) exception
is part of the standard Python library
and is used to indicate that the function received an argument of the right type,
but of an inappropriate value.

In `lcanalyzer/models.py`
~~~
def normalize_lc(df,mag_col):
    # Normalize a light curve
    if any(df[mag_col].abs() > 90):
        raise ValueError(mag_col+' contains values with abs() larger than 90!')
    min = min_mag(df,mag_col)
    max = max_mag((df-min),mag_col)
    lc = (df[mag_col]-min)/max
    lc = lc.fillna(0)
    return lc
~~~
{: .language-python}

Here we added a condition that if our input data contains values that are larger than 90 or smaller than -90,
we should raise a `ValueError` with the corresponding message.

In `tests/test_models.py`
~~~
# Parametrization for normalize_lc function testing with ValueError
@pytest.mark.parametrize(
    "test_input_df, test_input_colname, expected, expected_raises",
    [
        (pd.DataFrame(data=[[8, 9, 1], 
                            [1, 4, 1], 
                            [1, 2, 4], 
                            [1, 4, 1]], 
                      columns=list("abc")),
        "b",
        pd.Series(data=[1,0.285,0,0.285]),
        None),
        (pd.DataFrame(data=[[1, 1, 1], 
                            [1, 1, 1], 
                            [1, 1, 1], 
                            [1, 1, 1]], 
                      columns=list("abc")),
        "b",
        pd.Series(data=[0.,0.,0.,0.]),
        None),
        (pd.DataFrame(data=[[0, 0, 0], 
                            [0, 0, 0], 
                            [0, 0, 0], 
                            [0, 0, 0]], 
                      columns=list("abc")),
        "b",
        pd.Series(data=[0.,0.,0.,0.]),
        None),
        (pd.DataFrame(data=[[8, 9, 1], 
                            [1, -99.9, 1], 
                            [1, 2, 4], 
                            [1, 4, 1]], 
                      columns=list("abc")),
        "b",
        pd.Series(data=[1,0.285,0,0.285]),
        ValueError),
    ])
def test_normalize_lc(test_input_df, test_input_colname, expected,expected_raises):
    """Test how normalize_lc function works for arrays of positive integers."""
    from lcanalyzer.models import normalize_lc
    import pandas.testing as pdt
    if expected_raises is not None:
        with pytest.raises(expected_raises):
            pdt.assert_series_equal(normalize_lc(test_input_df,test_input_colname),expected,check_exact=False,atol=0.01,check_names=False)
    else:
        pdt.assert_series_equal(normalize_lc(test_input_df,test_input_colname),expected,check_exact=False,atol=0.01,check_names=False)
~~~
{: .language-python}

And in the `test_models` we had to add to our parametrization a new function argument 
called `expected_raises`, which is equal to `None`
for the test cases where our function should not invoke any raises. In the testing function itself
we are adding an `if` statement to separately handling the situation when a raise is expected.

Be sure to commit your changes so far and push them to GitHub.

> ## Optional Exercise: Add a Precondition to Check the Correct Type and Column Names
>
> Add preconditions to check that input data is `DataFrame` object and that its columns
> contain the column for which we have to perform normalization.
> Add corresponding tests to check that the function raises the correct exception.
> You will find the Python function
> [`isinstance`](https://docs.python.org/3/library/functions.html#isinstance)
> useful here, as well as the Python exception
> [`TypeError`](https://docs.python.org/3/library/exceptions.html#TypeError).
> Once you are done, commit your new files,
> and push the new commits to your remote repository on GitHub.
>
{: .challenge}

If you do the challenge, again, be sure to commit your changes and push them to GitHub.

You should not take it too far by trying to code preconditions for every conceivable eventuality.
You should aim to strike a balance between
making sure you secure your function against incorrect use,
and writing an overly complicated and expensive function
that handles cases that are likely never going to occur.
For example, it would be sensible to validate the values of your
light curve measurements
when it is actually read from the file,
and therefore there is no reason to test this again in `normalize_lc` or 
any of our functions related to statistics.
You can also decide against adding explicit preconditions in your code,
and instead state the assumptions and limitations of your code
for users of your code in the docstring
and rely on them to invoke your code correctly.
This approach is useful when explicitly checking the precondition is too costly.

## Test Driven Development

In the previous episodes
we learnt how to create *unit tests* to make sure our code is behaving as we intended.
**Test Driven Development** (TDD) is an extension of this.
If we can define a set of tests for everything our code needs to do,
then why not treat those tests as the specification.

With Test Driven Development,
the idea is to write our tests first and only write enough code to make the tests pass.
It is done at the level of individual features -
define the feature,
write the tests,
write the code.
The main advantages are:

- It forces us to think about how our code will be used before we write it
- It prevents us from doing work that we don't need to do, e.g. "I might need this later..."
- It forces us to test that the tests _fail_ before we've implemented the code, meaning we
   don't inadvertently forget to add the correct asserts.

You may also see this process called **Red, Green, Refactor**:
'Red' for the failing tests,
'Green' for the code that makes them pass,
then 'Refactor' (tidy up) the result.

> ## Trying the TDD approach
>
> TDD exercise
>
{: .challenge}

## Limits to Testing

Like any other piece of experimental apparatus,
a complex program requires a much higher investment in testing than a simple one.
Putting it another way,
a small script that is only going to be used once,
to produce one figure,
probably doesn't need separate testing:
its output is either correct or not.
A linear algebra library that will be used by
thousands of people in twice that number of applications over the course of a decade,
on the other hand, definitely does.
The key is identify and prioritise against
what will most affect the code's ability to generate accurate results.

It's also important to remember that unit testing cannot catch every bug in an application,
no matter how many tests you write.
To mitigate this manual testing is also important.
Also remember to test using as much input data as you can,
since very often code is developed and tested against the same small sets of data.
Increasing the amount of data you test against - from numerous sources -
gives you greater confidence that the results are correct.

Our software will inevitably increase in complexity as it develops.
Using automated testing where appropriate can save us considerable time,
especially in the long term,
and allows others to verify against correct behaviour.

{% include links.md %}
