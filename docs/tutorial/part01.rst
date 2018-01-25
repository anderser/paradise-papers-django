=======================
Set up the environment.
=======================

The following section will have the steps to set your local environment, so you can begin to work on your Django-neomodel implementation.
If you want a point to start, we provide a base app to begin the tutorial. First clone the paradise-papers-django_ repo.

    .. _paradise-papers-django: https://github.com/neo4j-examples/paradise-papers-django

After you download the repo go to the ``start_app`` branch and you will get a working app that will be our start point.

.. [*] Note:
    If you use the start branch mentioned before you only will get just the front-end of the app.

Set your virtualenv
===================

First you need Python and pip install on your local environment. Here is the official download page of python_

    .. _python: https://www.python.org/downloads/

Here is a quick tutorial on how to install pip_

    .. _pip: https://pip.pypa.io/en/stable/installing/#installing-with-get-pip-py

Now we have everything we need to create our virtualenv. On the next part we are going to introduce the basic to get your app running; if you want
more information check this virtualenv_ tutorial

    .. _virtualenv: https://virtualenv.pypa.io/en/stable/

First go to your app folder on the case of the paradise-papers-django project you need to be on the main folder (localPath/ paradise-papers-django)
and run the following command to install virtualenv::

    pip install virtualenv

You can use this pip command to check the dependencies installed::

    pip freeze

After you install the virtualenv; you need to create a virtual environment for you app. On the same folder that you install virtualenv you need to run the
following command::

    virtualenv .

The dot means that you going to install your virtual environment on the current folder, but you can specify the path that you need to create your virtual environment.

After your virtual environment is ready your directory should look like this if you are using the example app::

    ├── paradise-papers-django/
    │   ├── Include/
    │   ├── Lib/
    │   ├── paradise_papers_search/
    |   ├── LICENSE
    │   ├── pip-selfcheck.json
    │   ├── PULL_REQUEST_TEMPLATE.md
    |   ├── README.md

Now you need to activate your brand new virtual environment with the following command::

    .\Scripts\activate

Now you are in your new virtual environment!!!

Add project dependencies
========================

The next step is install the other dependencies that we need. For this you can use our requirement file if you are using the example app otherwise you need to
install the following dependencies using the ``pip install name===version`` command::

    Django==1.11.2
    django-neomodel==0.0.4
    neo4j-driver==1.2.1
    neomodel==3.2.5

If you’re using your own requirement file you can use this command to get all your dependencies::

    pip install -r /path/to/requirements.txt

if you are using our staring app; go to the paradise-papers-search folder and run this command::

    pip install -r requirements.txt

Now you have all dependencies that you need for this implementation!!!!
