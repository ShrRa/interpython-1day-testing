---
title: "Setting the Scene"
start: false
colour: "#FBED65"
teaching: 15
exercises: 0
questions:
- "What are we teaching in this course?"
- "What motivated the selection of topics covered in the course?"
objectives:
- "Setting the scene and expectations"
- "Making sure everyone has all the necessary software installed"
keypoints:
- "For developing software that will be used by other people aside from you, it is not enough to write code that produces seemingly correct output in a few cases. You have to check that the software performs well in different conditions and with different input data, and if something goes wrong, the user is notified of this."
- "This lesson focuses on intermediate skills and tools for making sure that your software is correct, reliable and fast."
- "The lesson follows on from the novice Software Carpentry lesson, but this is not a prerequisite for
attending as long as you have some basic Python, command line and Git skills and you have been using them for a
while to write code to help with your work."
---

## Introduction
So, you have gained basic software development skills either by self-learning or attending,
e.g., a [novice Software Carpentry course][swc-lessons].
You have been applying those skills for a while by writing code to help with your work
and you feel comfortable developing code and troubleshooting problems.
However, your software has now reached a point when you have to use and maintain it 
for prolonged periods of time, or when you have to share it with other users who may apply it to different kinds of tasks or data. 
Perhaps your projects now involve more researchers (developers) and users,
and more collaborative development effort is needed to add new functionality
while ensuring that previous features remain functional and maintainable.

This single-day course is dedicated to basic software testing and profiling tools and techniques.
Both testing and code profiling are essential stages of the development of large software projects,
however, in smaller academic software we often skip them for the sake of speeding up the work. While reasonable 
when we are dealing with scripts that are only a few hundred lines long, this approach fails us once we begin developing 
more complicated and computationally heavy software. It is particularly important for collaborations in which the code produced by one
developer can break the code written by someone else.

The goals of software testing are:
- **to make sure that the developed code satisfies the requirements, i.e. does what it's supposed to do;**
- **to check that it produces the correct outputs for any valid input;**
- **to ensure that the user is warned when the input data is invalid.**

Code profiling, on the other hand, is the process of measuring **how much and which resources the software uses**. 
The most common resources measured are **time, memory and CPU load**. Profiling is necessary when developing
computationally expensive code or software that will be applied to large volumes of data.

The course uses a number of different **software development tools and techniques**
interchangeably as you would in a real life.
We had to make some choices about topics and tools to teach here,
based on established best practices,
ease of tool installation for the audience,
length of the course and other considerations.
Tools used here are not mandated though: 
alternatives exist and we point some of them out along the way.
Over time, you will develop a preference for certain tools and programming languages
based on your personal taste
or based on what is commonly used by your group, collaborators or community.
However, the topics covered should give you a solid foundation for producing 
high-quality software that is easier to sustain in the future by yourself and others.
Skills and tools taught here, while Python-specific,
are transferable to other similar tools and programming languages.

The course is organised into the following sections:

### [Section 1: Software project example](../10-section1-intro/index.html)
In the first section, we'll look into the software project that we will use for further testing and profiling
and set up our virtual environment.

- we can obtain the project from its **GitHub repository**.
- the structure of the software is determined by its **architecture**, which also affects how the testing is done.
- to avoid conflicts between different versions of Python distributions and packages, we will create a separate **virtual environment**.

### [Section 2: Unit testing](../20-section2-intro/index.html)
In this section we are going to establish what is included in software testing and how we can perform the basic type of it, **unit testing**.

- **Unit testing** for testing separate functions of the software;
- how to set up a **test framework** and write tests to verify the behaviour of our code is correct in different situations;
- what kind of cases should we test;
- what is **Test Driven Development**.

### [Section 3: Profiling](../30-section3-intro/index.html)
Once we know our way around different testing tools,
in this section we learn:

- how to automate and scale testing with **Continuous Integration (CI)** using
  **GitHub Actions** (a CI service available on GitHub).
- What is **profiling**
- how to use **Jupyter magicks** for performance time measurement
- how to use **SnakeViz** for resource profiling

## Before We Start

A few notes before we start.

> ## Prerequisite Knowledge
> This is an intermediate-level software development course
> intended for people who have already been developing code in Python (or other languages)
> and applying it to their own problems after gaining basic software development skills.
> So, it is expected for you to have some prerequisite knowledge on the topics covered,
> as outlined at the [beginning of the lesson](../index.html#prerequisites).
> Check out this [quiz](../quiz/index.html) to help you test your prior knowledge
> and determine if this course is for you.
{: .callout}

> ## Setup, Common Issues & Fixes
> Have you [setup and installed](../setup.html) all the tools and accounts required for this course?
> Check the list of [common issues, fixes & tips](../common-issues/index.html)
> if you experience any problems running any of the tools you installed -
> your issue may be solved there.
{: .callout}

> ## Compulsory and Optional Exercises
> Exercises are a crucial part of this course and the narrative.
> They are used to reinforce the points taught
> and give you an opportunity to practice things on your own.
> Please do not be tempted to skip exercises
> as that will get your local software project out of sync with the course and break the narrative.
> Exercises that are clearly marked as "optional" can be skipped without breaking things
> but we advise you to go through them too, if time allows.
> All exercises contain solutions but, wherever possible, try and work out a solution on your own.
{: .callout}

> ## Outdated Screenshots
> Throughout this lesson we will make use and show content
> from Graphical User Interface (GUI) tools (Jupyter Lab and GitHub).
> These are evolving tools and platforms, always adding new features and new visual elements.
> Screenshots in the lesson may then become out-of-sync,
> refer to or show content that no longer exists or is different to what you see on your machine.
> If during the lesson you find screenshots that no longer match what you see
> or have a big discrepancy with what you see,
> please [open an issue]({{ site.github.repository_url }}/issues/new) describing what you see
> and how it differs from the lesson content.
> Feel free to add as many screenshots as necessary to clarify the issue.
> 
{: .callout}

> ## Let Us Know About the Issues
> The original materials were adapted specifically for this workshop. They weren't used before,
> and it is possible that they contain typos, code errors, or underexplained or unclear moments.
> Please, let us know about these issues. It will help us to improve the materials and make
> the next workshop better.
> 
{: .testimonial}

{% include links.md %}
