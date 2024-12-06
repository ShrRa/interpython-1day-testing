---
title: "Section 3: Automatizing code quality checks"
colour: "#fafac8"
start: true
teaching: 5
exercises: 0
questions:
- "How to set up automatic testing for all contributors of the project?"
- "How to enforce coding style conventions?"
- "How to estimate computational effectiveness of the software and find bottlenecks?"
objectives:
- "Introduce the tools, techniques, and best practices for automatizing your work on ensuring the quality of the software."
keypoints:
- "Running tests every time when the code is updates is tiresome, and we can be tempted to skip this step.
We should use automatic **Continuous Integration** tools, such as **GitHub Actions**, to make sure that the tests are executed regularly."
- "**Continuous Integration** also allows us to set up an automatic code style checks."
- "To check how efficient our code is and find bottlenecks, we can use Jupyter Lab built-in **magic commands** and Python libraries,
  such as **cProfile** and **SnakeViz**"
---

After learning what testing is and how to write tests manually, we will learn how to make sure that you
do not skip the testing fase in a hurry. We also cover how to profile the time and computational resources your software requires.

In this section we will:

- Automatically run a set of unit tests using **GitHub Actions** -
  a **Continuous Integration** infrastructure that allows us to
  automate tasks when things happen to our code,
  such as running those tests when a new commit is made to a code repository.
- Use **Pylint** to check code style conventions.
- Use Jupyter **magic commands** to measure performance time of the code.
- Use library **SnakeViz** to profile resources taken by the software.

{% include links.md %}
