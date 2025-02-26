Xnat4Tests
==========
.. image:: https://github.com/australian-imaging-service/xnat4tests/actions/workflows/test.yml/badge.svg
   :target: https://github.com/Australian-Imaging-Service/xnat4tests/actions/workflows/test.yml
.. image:: https://img.shields.io/pypi/v/xnat4tests.svg
   :target: https://pypi.python.org/pypi/xnat4tests/

Xnat4Tests provides a helper functions for testing third party tools that access the XNAT
API or container service, primarily a means to launch a basic XNAT repository instance
within a single Docker container.

The XNAT container service plugin is installed by default and is configured to use
the same Docker host as the XNAT instance.

The 'home/logs', 'home/work', 'build', 'archive', 'prearchive' directories are
mounted in from the host for direct access under ``xnat4tests.config["xnat_root_dir"]``,
which can be useful for debugging and enables the environment in which containers
run in within XNAT's container service to be mocked.

In addition to the ``launch_xnat`` function, which launches the XNAT instance, a ``connect``
function is supplied that returns an XnatPy connection object to the test instance

Installation
------------

Xnat4Tests is available on PyPI so to install, simply use pip

.. code-block:: bash

    $ pip3 install xnat4tests
    
or include in your package's ``test_requires``.

Usage
-----

.. code-block:: python

    # Import xnat4tests functions
    from xnat4tests import launch_xnat, stop_xnat, connect, config

    # Launch the instance (NB: it takes quite while for an XNAT instance to start). If an existing
    # container with the reserved name is already running it is returned instead
    launch_xnat()

    # Run your tests
    with connect() as login:
        PROJECT = 'MYPROJECT'
        SUBJECT = 'MYSUBJECT'
        SESSION = 'MYSESSION'
    
        login.put(f'/data/archive/projects/MY_TEST_PROJECT')

        # Create subject
        xsubject = login.classes.SubjectData(label=SUBJECT,
                                             parent=login.projects[PROJECT])
        # Create session
        login.classes.MrSessionData(label=SESSION, parent=xsubject)

    assert [p.name for p in (config.XNAT_ROOT_DIR / 'archive').iterdir()] == [PROJECT]

    # Remove the container after you are done (not strictly necessary)
    stop_xnat()

Alternatively, if you are using Pytest then you can set up the connection as
a fixture in your ``conftest.py`` with

.. code-block:: python

    @pytest.fixture(scope='session')
    def xnat_login():
        xnat4tests.launch_xnat()
        yield xnat4tests.connect()
        xnat4tests.stop_xnat()
