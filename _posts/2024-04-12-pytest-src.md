---
layout: post
title: Pytest and the src
subtitle: Testing the packaging
date: 2024-04-12
background: /images/default_post.jpg
---

This week, I was working on getting a testing stage in CI fixed. It sounds trivial but lead to a couple of insights again how tests are executed in Python.

Python with its dynamic nature and unopiniated tooling give a lot of room for interpretation how tests should be executed. Today most people will rely on `pytest` as the standard tool to write and execute tests. But what does that execution actually mean?

## Code

In compiled languages, the tests need to be compiled along with the application code to ensure execution in the same environment. In Python, there is no compilation step involved -- as long as your library does not use compiled code from Cython, C or Rust for example -- and compilation might be mapped to packaging in Python. To be able to install a Python package as a module into a new environment it can be packaged into a `wheel` and this is a huge other story.

Important here is, we can have executable Python code in the code repository and in the form of an installable package (that contains the same source code files).

## Goal

Why are we actually doing tests? Most importantly, we want to make sure that the code works for its users. That means, we should run the tests on the code that gets shipped and installed by users.

Therefore, one might want to run tests rather on an installed package than the source code in the project repository.

Unfortunately, there are some surprises in that matter as laid out by in an interesting [blog article](https://hynek.me/articles/testing-packaging/) by Hynek Schlawack. It argues why it might be important to test not only the code but also the packaging to make sure that the package contains all required files.

## Implementation

In the article, a central part of the solution was to use the [`src` layout](https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/). It helps in distinguishing the installed package of your project from the directory with the same name in the code repository. In Python, the working directory is added to the [Pythonpath per default](https://docs.python.org/3/library/sys.html#sys.path) and then your directory is added as a module overwriting the installed module of same name.

In pytest, we can configure that path and when running it via [`pytest tests/`](https://docs.pytest.org/en/7.1.x/how-to/usage.html#calling-pytest-through-python-m-pytest) only the configured directories are added to the Pythonpath.

```toml
[tool.pytest.ini_options]
pythonpath = ["tests"]
```

Now, we should be sure that the tests are running on the installed package version of your module which sits most probably in the `site-packages` directory of the virtual environment. Even without the `src` layout, right?

## Coverage

When adding the coverage plugin to pytest to collect test coverage another surprise awaits.

```bash
pytest -m 'unit' --cov
```

Executes the tests on the code of the installed package. The coverage part checks again code execution of code in you working directory. You will see names of the tested code files that point at your directory and not the virtual environment. There is one coverage setting that takes care for us

```toml
[tool.coverage.run]
source_pkgs = ["project"]
```

Here, we make sure that it uses the package of name `project` and not the directory of the same name.

Finally, we have a setup that works locally when installing your project as an editable package (via poetry for example) and in the production environment when the package is installed as a wheel.
