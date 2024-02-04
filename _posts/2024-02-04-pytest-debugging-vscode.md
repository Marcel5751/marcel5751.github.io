---
layout: post
title: "Pytest Debugging in VS Code (Kedro Project)"
tags: Python vscode pytest kedro
---

Recently, I started a Machine learning projects with the Kedro Framework. Based on the example [spaceflights project](https://github.com/kedro-org/kedro-starters/tree/main/spaceflights-pandas-viz), kedro uses the pytest library for unit testing. For a nice development workflow, I wanted to see the following requirements fulfilled:
 - Unit testing with Pytest library
 - Import module `src` folder
 - Ability to debug single file

Sounds simple, but there is some specific configuration necessary to make it work...

 <!--more-->

## Problem No. 1: ModuleNotFound
For the purposes of this post, the relevant part of project structure looks like this:

{% highlight txt %}
project/
├── src/
│   └── pipelines
└── tests/
    └── test_pipelines
{% endhighlight %}

This means that when testing the modules in the `tests/pipelines` folder, the python scripts inside the `src/pipelines` module need to be found by pytest. Pytest scanns the code for test files to be executed. Given the folder structure, Pytest by default does not find the modules in the `src` folder. In order to let Pytest discover them, the following setting is needed in the `pyproject.toml` (see also [this SO post](https://stackoverflow.com/a/50610630/10152761) for more info).

{% highlight ini %}
[tool.pytest.ini_options]
pythonpath = [
  ".", "src",
]
{% endhighlight %}


## Problem No. 2: Debug/Run single Test

Assuming Python Debugging is generally already working (if not install the Python VS Code plugin and follow the [VS Code Guide](https://code.visualstudio.com/docs/python/debugging)) Hit `Ctrl + Shift + D` to open the debug window. 
The Running/Debugging can be customized by creating a `launch.json` configuration file.

In order to run/debug Pytest a single file, we can pass the [predefined vs Code Variable](https://code.visualstudio.com/docs/editor/variables-reference) as an argument `"${file}"` to the python. With this setup the currently selected file will be run when clicking the run/debug buttons.

## Problem No. 3: Make Debugging Work in VS Code

You may notice that even when a Breakpoint is set, the debugger does not stop at the specified position. This is because the debugging does not work with Pytest coverage on (As pointed out [here](https://github.com/microsoft/vscode-python/issues/693#issuecomment-367855391)).
This can be fixed by passing the `"--no-cov"` parameter to Pytest.

In total your `lanuch.json` should now contain a configuration like this:

{% highlight json %}
        {
            "name": "Pytest: Debug File",
            "type": "python",
            "request": "launch",
            "module": "pytest",
            "args": [
                "${file}",
                "--no-cov"
            ],
            "console": "integratedTerminal",
            "env": {}
        },
{% endhighlight %}
