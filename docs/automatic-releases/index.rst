Automatic releases
------------------

The key point with using this package is to automate your releases and stop worrying about
version numbers. Different approaches to automatic releases and publishing with the help of
this package can be found below. Using a CI is the recommended approach.


Environment checks
~~~~~~~~~~~~~~~~~~
On publish, a few environment checks will run. Below are descriptions of what the different checks
do and under what condition they will run.

frigg
^^^^^
*Condition:* Environment variable ``FRIGG`` is ``'true'``

Checks for frigg to ensure that the build is not a pull-request and on the correct branch.
The branch check, checks against the branch that frigg said it checked out, not the current
branch.

semaphore
^^^^^^^^^
*Condition:* Environment variable ``SEMAPHORE`` is ``'true'``

Checks for semaphore to ensure that the build is not a pull-request and on the correct branch.
The branch check, checks against the branch that semaphore said it checked out, not the current
branch. It also checks that the thread state is not failure.

travis
^^^^^^
*Condition:* Environment variable ``TRAVIS`` is ``'true'``

Checks for travis to ensure that the build is not a pull-request and on the correct branch.
The branch check, checks against the branch that travis said it checked out, not the current
branch.

CircleCI
^^^^^^^^
*Condition:* Environment variable ``CIRCLECI`` is ``'true'``

Checks for circle-ci to ensure that the build is not a pull-request and on the correct branch.
The branch check, checks against the branch that circle-ci said it checked out, not the current
branch.

Publish with CI
~~~~~~~~~~~~~~~
Add ``python setup.py publish`` or ``semantic-release publish`` as an after success task on your
preferred Continuous Integration service. Ensure that you have configured the CI so that it can
upload to pypi and push to git and it should be ready to role.

Configuring pypi upload
^^^^^^^^^^^^^^^^^^^^^^^
In order to upload to pypi python-semantic-release needs credentials to an account that
have access to the given package. Either by being logged in through a pip configuration file
or through environment variables. The latter is most often preferable in an CI environment.
You will need to set ``PYPI_USERNAME`` and ``PYPI_PASSWORD``. Make sure that you mark it
as a secret on your CI service so that your password will be left out of the build logs.

Configuring push to Github
^^^^^^^^^^^^^^^^^^^^^^^^^^
In order to push to Github and post the changelog to Github the environment variable
``GH_TOKEN`` has to be set. It needs access to the ``public_repo`` scope for public repositories
and ``repo`` for private repositories.


Step-by-Step guides
^^^^^^^^^^^^^^^^^^^
* :doc:`travis`


Publish with cronjobs
~~~~~~~~~~~~~~~~~~~~~

This is for you if for some reason you cannot publish from your CI or you would like releases to
drop at a certain interval. Before you start, answer this: Are you sure you do not want a CI to
release for you? (high version numbers are not a bad thing).

The guide below is for setting up scheduled publishing on a server. It requires that the user
that runs the cronjob has push access to the repository and upload access to pypi.

1. Create a virtualenv::

    virtualenv semantic_release -p `which python3`

2. Install python-semantic-release::

    pip install python-semantic-release

3. Clone the repositories you want to have scheduled publishing.
3. Put the following in ``publish``::

    VENV=semantic_release/bin

    $VENV/pip install -U pip python-semantic-release > /dev/null

    publish() {
      cd $1
      git stash -u # ensures that there is no untracked files in the directory
      git fetch && git reset --hard origin/master
      $VENV/semantic-release publish
      cd ..
    }

    publish <package1>
    publish <package2>

4. Add cronjob::

    /bin/bash -c "cd <path> && source semantic_release/bin/activate && ./publish 2>&1 >> releases.log"
