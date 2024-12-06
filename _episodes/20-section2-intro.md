---
title: "Section 2: Ensuring Correctness of Software at Scale"
colour: "#fafac8"
start: true
teaching: 5
exercises: 0
questions:
- "What should we do to ensure our code is correct?"
objectives:
- "Introduce the testing tools, techniques, and infrastructure that will be used in this section."
keypoints:
- "Using testing requires us to change our practice of code development, but saves time in the long run by
allowing us to more comprehensively and rapidly find faults in code, as well as giving us greater confidence in the correctness of our code."
- "Writing **parametrized** tests makes sure that you are testing your software in different scenarios."
- "Writing tests **before** the features forces you to think of the requirements and best possible implementations in advance."
---

We've just set up a suitable environment for the development of our software project
and are ready to start developing new features.
However, we want to make sure that the new code we contribute to the project
is actually correct and is not breaking any of the existing code.
So, in this section,
we'll look at testing approaches that can help us ensure
that the software we write is behaving as intended,
and how we can diagnose and fix issues once faults are found.
Using such approaches requires us to change our practice of development.
This can take time, but potentially saves us considerable time
in the medium to long term
by allowing us to more comprehensively and rapidly find such faults,
as well as giving us greater confidence in the correctness of our code -
so we should try and employ such practices early on.
We will also make use of techniques and infrastructure that allow us to do this
in a scalable, automated and more performant way as our codebase grows.

In this section we will:

- Make use of a **test framework** called Pytest,
  a free and open source Python library to help us structure and run automated tests.
- Design, write and run **unit tests** using Pytest
  to verify the correct behaviour of code and identify faults,
  making use of test **parameterisation**
  to increase the number of different test cases we can run.
- Try out **Test-Driven Development**, an work approach based on developing the checks
  before writing the code itself.

{% include links.md %}
