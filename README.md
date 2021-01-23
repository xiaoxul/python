# python3-s2i-starter
Cirrus uses Red Hat's Python Universal Base Image(s) as the foundation for this 
s2i container runtime. The following README is composed of fragments from the
[Software Collective's Repository](https://github.com/sclorg/s2i-python-container)
and Red Hat's [Guide on Universal Base Images](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/building_running_and_managing_containers/using_red_hat_universal_base_images_standard_minimal_and_runtimes).

Usage
-----

When creating a Cirrus build, select the **Python 2.7 S2I** or **Python 3.6 S2I** for the container runtime. Note, this example is tailored for Python 3.

If these files exist they will affect the behavior of the build process:

* **requirements.txt**

  List of dependencies to be installed with `pip`. The format is documented
  [here](https://pip.pypa.io/en/latest/user_guide.html#requirements-files).


* **Pipfile**

  The replacement for requirements.txt, project is currently under active
  design and development, as documented [here](https://github.com/pypa/pipfile).
  Set `ENABLE_PIPENV` environment variable to true in order to process this file.


* **setup.py**

  Configures various aspects of the project, including installation of
  dependencies, as documented
  [here](https://packaging.python.org/en/latest/distributing.html#setup-py).
  For most projects, it is sufficient to simply use `requirements.txt` or
  `Pipfile`. Set `DISABLE_SETUP_PY_PROCESSING` environment variable to true
  in order to skip processing of this file.

Environment variables
---------------------

The variables below can affect the build process and runtime of your application, 
and can be added via _Build Environment Vars_ for build, or in _Secrets_ for runtime.

* **APP_SCRIPT**

    Used to run the application from a script file.
    This should be a path to a script file (defaults to `app.sh` unless set to null) that will be
    run to start the application.

* **APP_FILE**

    Used to run the application from a Python script.
    This should be a path to a Python file (defaults to `app.py` unless set to null) that will be
    passed to the Python interpreter to start the application.

* **APP_MODULE**

    Used to run the application with Gunicorn, as documented
    [here](http://docs.gunicorn.org/en/latest/run.html#gunicorn).
    This variable specifies a WSGI callable with the pattern
    `MODULE_NAME:VARIABLE_NAME`, where `MODULE_NAME` is the full dotted path
    of a module, and `VARIABLE_NAME` refers to a WSGI callable inside the
    specified module.
    Gunicorn will look for a WSGI callable named `application` if not specified.

    If `APP_MODULE` is not provided, the `run` script will look for a `wsgi.py`
    file in your project and use it if it exists.

    If using `setup.py` for installing the application, the `MODULE_NAME` part
    can be read from there. For an example, see
    [setup-test-app](https://github.com/sclorg/s2i-python-container/tree/master/3.6/test/setup-test-app).

* **APP_HOME**

    This variable can be used to specify a sub-directory in which the application to be run is contained.
    The directory pointed to by this variable needs to contain `wsgi.py` (for Gunicorn) or `manage.py` (for Django).

    If `APP_HOME` is not provided, the `assemble` and `run` scripts will use the application's root
    directory.

* **APP_CONFIG**

    Path to a valid Python file with a
    [Gunicorn configuration](http://docs.gunicorn.org/en/latest/configure.html#configuration-file) file.

* **DISABLE_MIGRATE**

    Set this variable to a non-empty value to inhibit the execution of 'manage.py migrate'
    when the produced image is run. This only affects Django projects. See
    "Handling Database Migrations" section of [Django blogpost on OpenShift blog](
    https://blog.openshift.com/migrating-django-applications-openshift-3/) on suggestions
    how/when to run DB migrations in OpenShift environment. Most importantly,
    note that running DB migrations from two or more pods might corrupt your database.

* **DISABLE_COLLECTSTATIC**

    Set this variable to a non-empty value to inhibit the execution of
    'manage.py collectstatic' during the build. This only affects Django projects.

* **DISABLE_SETUP_PY_PROCESSING**

    Set this to a non-empty value to skip processing of setup.py script if you
    use `-e .` in requirements.txt to trigger its processing or you don't want
    your application to be installed into site-packages directory.

* **ENABLE_PIPENV**

    Set this variable to use [Pipenv](https://github.com/kennethreitz/pipenv),
    the higher-level Python packaging tool, to manage dependencies of the application.
    This should be used only if your project contains properly formated Pipfile
    and Pipfile.lock.

* **ENABLE_INIT_WRAPPER**

    Set this variable to a non-empty value to make use of an init wrapper.
    This is useful for servers that are not capable of reaping zombie
    processes, such as Django development server or Tornado. This option can
    be used together with **APP_SCRIPT** or **APP_FILE**. It never applies
    to Gunicorn used through **APP_MODULE** as Gunicorn reaps zombie
    processes correctly.

* **PIP_INDEX_URL**

    Set this variable to use a custom index URL or mirror to download required packages
    during build process. This only affects packages listed in requirements.txt.
    Pipenv ignores this variable.

* **UPGRADE_PIP_TO_LATEST**

    Set this variable to a non-empty value to have the 'pip' program and related
    python packages (setuptools and wheel) be upgraded to the most recent version
    before any Python packages are installed. If not set it will use whatever
    the default version is included by the platform for the Python version being used.

* **WEB_CONCURRENCY**

    Set this to change the default setting for the number of
    [workers](http://docs.gunicorn.org/en/stable/settings.html#workers). By
    default, this is set to the number of available cores times 2, capped
    at 12.
    
