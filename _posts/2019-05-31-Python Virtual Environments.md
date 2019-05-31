A frined of mine was signing the prasises of Python virtual environments recently so I decided to try and figure out what he was talking about.  Turns out creating a virtual environment is pretty straightforward (as least in Anaconda) and super useful.

# Some Resources:

* [https://uoa-eresearch.github.io/eresearch-cookbook/recipe/2014/11/20/conda/](https://uoa-eresearch.github.io/eresearch-cookbook/recipe/2014/11/20/conda/)
*[Why you need Python environments and how to manage them with Conda](https://www.freecodecamp.org/news/why-you-need-python-environments-and-how-to-manage-them-with-conda-85f155f4353c/)
* [From the Anaconda repository](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html)

# Answers to some simple questions

Really, I've just been monkeying with separate environments for a couple days so answers to simple questions are really all I got at this point.

## What is an environment?

*A named, isolated, working copy of Python that that maintains its own files, directories, and paths so that you can work with specific versions of libraries or Python itself without affecting other Python projects. Virtual environmets make it easy to cleanly separate different projects and avoid problems with different dependencies and version requiremetns across components.*

## Why do I/do I need one?

1. Maybe you're testing somebody else's application that has lots of library dependencies and you don't really want to install all those libraries in your base environment.

2. Maybe you're collaborating with somebody who's project requires a specific version of Python and you don't want it to conflict with your prefered Python version.

3. Maybe you want to share a project or app.  It works great on your system but to make things idiot-proof for your potential users you might want to make sure they are running it the same way you test it.  Then you might want to make sure they run it from the proper environment. See the [Import and export a conda environment](https://medium.com/@__pamaron__/understanding-and-use-python-virtualenvs-from-data-scientist-perspective-bfed61faeb3f) section.  

## But do I really need separate environments?

Maybe not.  I just had a hallway conversation with 2 colleagues:

One was adament that you should create a virtual environment for every project [using a .yml file] then, upon project completion, delete the virtual environment because it can always be recreated with .yml file.  This way you don't clutter your system with packages and libraries that might be project specific and the virtual environment helps ensure reproducability of the work.

The other felt that virtual environements are clunky, have a tendency to muck up your system with mulitple copies of the same library, and aren't really necessary unless your dealing with lots of versioning conflicts...which in this colleagues experience was kind of rare.  The individual prefers to create virtual environments sparingly and only in the small number of cases where a versioning conflict actually arises.

I can't say I have a good sense of whether or not virtual environments will become part of my regular workflow...but I'm kinda glad I at least know what they are and how to create them.

# My Example:

## Overview

I have a machine learning application that I'm working on with a colleague.  In this case, my co-author is doing most of the heavy lifting and I'm mostly just testing code, interpreting results, and doing the writing.  I need to be able to run his code which depends on a bunch of ML and AI libraries.  Also, my co-author uses Python 3.5 while my Anaconda root environment is currently using 3.7.

## Create the environment

[Per the Conda documentation](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html), creating a virtual environment is rediculously easy.  Open the terminal (I prefer to open a command prompt from with the Anaconda Dashboard because - for a variety of Windows related reasons that I don't like - this is how I have to do things at work) and do:

```python
conda create --name py35  python=3.5
```

In this case, I'm create a virtual environment called *py35* that will use Python version 3.5.  Note that I have not yet declared any libraries to be included in this environment.

[cond1](/images/conda1.png)

[pyenv1](/images/pyenv1.png)










