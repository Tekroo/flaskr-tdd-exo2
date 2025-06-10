# Flaskr - Intro to Flask, Test-Driven Development, and JavaScript

As many of you know, Flaskr -- a mini-blog-like-app -- is the app that you build for the official Flask [tutorial](https://flask.palletsprojects.com/en/3.0.x/tutorial/). I've gone through the tutorial more times than I care to admit. Anyway, I wanted to take the tutorial a step further by adding [Test-Driven Development](https://testdriven.io/test-driven-development/) (TDD), a bit of JavaScript, and deployment. This article is that tutorial. Enjoy.

Also, if you're completely new to Flask and/or web development in general, it's important to grasp these basic fundamental concepts:

1. The difference between HTTP GET and POST requests and how functions within the app handle each.
1. What HTTP "requests" and "responses" are.
1. How HTML pages are rendered and/or returned to the end user.

> This project is powered by **[TestDriven.io](https://testdriven.io/)**. Please support this open source project by purchasing one of our Flask courses. Learn how to build, test, and deploy microservices powered by Docker, Flask, and React!

## What you're building

You'll be building a simple blogging app in this tutorial:

![flaskr app](/flaskr-app.png)

## Changelog

This tutorial was last updated on October 17th, 2023:

- **10/17/2023**:
  - Updated to Python 3.12.0 and bumped all other dependencies.
- **06/03/2022**:
  - Updated to Python 3.10.4 and bumped all other dependencies.
- **10/14/2020**:
  - Renamed *app.test.py* to *app_test.py*. (Fixed issue #[58](https://github.com/mjhea0/flaskr-tdd/issues/58).)
  - Updated to Python 3.9 and bumped all other dependencies.
  - Added pytest v7.1.2. (Fixed issue #[60](https://github.com/mjhea0/flaskr-tdd/issues/60))
  - Migrated from `os.path` to `pathlib`.
- **11/05/2019**:
  - Updated to Python 3.8.0, Flask 1.1.1, and Bootstrap 4.3.1.
  - Replaced jQuery with vanilla JavaScript.
  - Added Black and Flake8.
  - Used Postgres in production.
  - Restricted post delete requests.
- **10/07/2018**: Updated to Python 3.7.0.
- **05/10/2018**: Updated to Python 3.6.5, Flask 1.0.2, Bootstrap 4.1.1.
- **10/16/2017**:
  - Updated to Python 3.6.2.
  - Updated to Bootstrap 4.
- **10/10/2017**: Added a search feature.
- **07/03/2017**: Updated to Python 3.6.1.
- **01/24/2016**: Updated to Python 3 (v3.5.1)!
- **08/24/2014**: PEP8 updates.
- **02/25/2014**: Upgraded to SQLAlchemy.
- **02/20/2014**: Completed AJAX.
- **12/06/2013**: Added Bootstrap 3 styles
- **11/29/2013**: Updated unit tests.
- **11/19/2013**: Fixed typo. Updated unit tests.
- **11/11/2013**: Added information on requests.

## Contents

1. [Test Driven Development?](#test-driven-development)
1. [Download Python](#download-python)
1. [Project Setup](#project-setup)
1. [First Test](#first-test)
1. [Flaskr Setup](#flaskr-setup)
1. [Second Test](#second-test)
1. [Database Setup](#database-setup)
1. [Templates and Views](#templates-and-views)
1. [Add Some Style](#add-some-style)
1. [JavaScript](#javascript)
1. [Deployment](#deployment)
1. [Bootstrap](#bootstrap)
1. [SQLAlchemy](#sqlalchemy)
1. [Search Page](#search-page)
1. [Login Required](#login-required)
1. [Postgres Heroku](#postgres-heroku)
1. [Linting and Code Formatting](#linting-and-code-formatting)
1. [Conclusion](#conclusion)

## Requirements

This tutorial utilizes the following requirements:

1. Python v3.12.0
1. Flask v3.0.0
1. Flask-SQLAlchemy v3.1.1
1. Gunicorn v21.2.0
1. Psycopg2 v2.9.9
1. Flake8 v6.1.0
1. Black v23.10.0
1. pytest v7.4.2



## Dockerisation
Use this cmd  :
  docker run "image_name"

## Test Driven Development?

![tdd](https://raw.githubusercontent.com/mjhea0/flaskr-tdd/master/tdd.png)

Test-Driven Development (TDD) is an iterative development cycle that emphasizes writing automated tests before writing the actual feature or function. Put another way, TDD combines building and testing. This process not only helps ensure correctness of the code -- but also helps to indirectly evolve the design and architecture of the project at hand.

TDD usually follows the "Red-Green-Refactor" cycle, as shown in the image above:

1. Write a test
1. Run the test (it should fail)
1. Write just enough code for the test to pass
1. Refactor code and retest, again and again (if necessary)

> For more, check out [What is Test-Driven Development?](https://testdriven.io/test-driven-development/).

## Download Python

Before beginning make sure you have the latest version of [Python 3.12](https://www.python.org/downloads/release/python-3120/) installed, which you can download from [http://www.python.org/download/](http://www.python.org/download/).

> This tutorial uses Python v3.12.0.

Along with Python, the following tools are also installed:

- [pip](https://pip.pypa.io/en/stable/) - a [package management](http://en.wikipedia.org/wiki/Package_management_system) system for Python, similar to gem or npm for Ruby and Node, respectively.
- [venv](https://docs.python.org/3/library/venv.html) - used to create isolated environments for development. This is standard practice. Always, always, ALWAYS utilize virtual environments. If you don't, you'll eventually run into problems with dependency conflicts.

> Feel free to swap out virtualenv and Pip for [Poetry](https://python-poetry.org) or [Pipenv](https://github.com/pypa/pipenv). For more, review [Modern Python Environments](https://testdriven.io/blog/python-environments/).

## Postgres Heroku

SQLite is a great database to use in order to get an app up and running quickly. That said, it's not intended to be used as a production grade database. So, let's move to using Postgres on Heroku.

Start by provisioning a new `mini` plan Postgres database:

```sh
(env)$ heroku addons:create heroku-postgresql:mini
```

Once created, the database URL can be access via the `DATABASE_URL` environment variable:

```sh
(env)$ heroku config
```

You should see something similar to:

```sh
=== glacial-savannah-72166 Config Vars

DATABASE_URL: postgres://zebzwxlootewbx:da5c19a66cd4765dd39aed40abb06dff10682c3213501695c4b98612de0dfac9@ec2-54-208-11-146.compute-1.amazonaws.com:5432/d77tnmeavvasm0
```

Next, update the `SQLALCHEMY_DATABASE_URI` variable in *app.py* like so:

```python
url = os.getenv('DATABASE_URL', f'sqlite:///{Path(basedir).joinpath(DATABASE)}')

if url.startswith("postgres://"):
    url = url.replace("postgres://", "postgresql://", 1)

SQLALCHEMY_DATABASE_URI = url
```

So, `SQLALCHEMY_DATABASE_URI` now uses the value of the `DATABASE_URL` environment variable if it's available. Otherwise, it will use the SQLite URL.

Make sure to import `os`:

```python
import os
```

Run the tests to ensure they still pass:

```sh
(env)$ python -m pytest

=============================== test session starts ===============================
platform darwin -- Python 3.10.4, pytest-7.4.2, pluggy-1.0.0
rootdir: /Users/michael/repos/github/flaskr-tdd
collected 6 items

tests/app_test.py ......                                                    [100%]

================================ 6 passed in 0.32s ================================
```

Try logging in and out, adding a few new entries, and deleting old entries locally.

Before updating Heroku, add [Psycopg2](http://initd.org/psycopg/) -- a Postgres database adapter for Python -- to the requirements file:

```
Flask==3.0.0
Flask-SQLAlchemy==3.1.1
gunicorn==21.2.0
psycopg2-binary==2.9.9
pytest==7.4.2
```

Commit and push your code up to Heroku.

Snce we're using a new database on Heroku, you'll need to run the following command *once* to create the tables:

```sh
(env)$ heroku run python create_db.py
```

Test things out.

## Linting and Code Formatting

Finally, we can lint and auto format our code with [Flake8](http://flake8.pycqa.org/) and [Black](https://black.readthedocs.io/), respectively:

```sh
(env)$ pip install flake8==6.1.0
(env)$ pip install black==23.10.0
```

Run Flake8 and correct any issues:

```sh
(env)$ python -m flake8 --exclude env --ignore E402,E501 .

./create_db.py:5:1: F401 'project.models.Post' imported but unused
./tests/app_test.py:2:1: F401 'os' imported but unused
./project/app.py:2:1: F401 'sqlite3' imported but unused
./project/app.py:6:1: F401 'flask.g' imported but unused
./project/app.py:7:19: E126 continuation line over-indented for hanging indent
```

Update the code formatting per Black:

```sh
$ python -m black --exclude=env .

reformatted /Users/michael/repos/github/flaskr-tdd/project/models.py
reformatted /Users/michael/repos/github/flaskr-tdd/project/app.py
All done! ✨ 🍰 ✨
2 files reformatted, 4 files left unchanged.
```

Test everything out once last time!

## Conclusion

1. Want my code? Grab it [here](https://github.com/mjhea0/flaskr-tdd).
1. Want more Flask fun? Check out [TestDriven.io](https://testdriven.io/). Learn how to build, test, and deploy microservices powered by Docker, Flask, and React!
1. Want something else added to this tutorial? Add an issue to the repo.
