# Nosetests

Nosetest is a utility to unit test python code. 

We use nosetests and not `__main__` to test all functionality so they can me
automatically run and reports can be generated. A project that does not have a
sufficient number of tests to make sure the module works can not be accepted.

## Installation

The nose module can be installed with the help of pip utility

```python
$ pip install nose
```

This will install the nose module in the current Python distribution as well 
as a nosetest.exe, which means the test can be run using this utility as well as using `–m` switch.
All nose tests are included in the folder `tests`.

* <https://github.com/cloudmesh/cloudmesh-cm4/tree/master/tests>

```
+cm
  + cloudmesh
  + tests
    - test_01_topic1.py
    - test_02_topic2.py
    - test_03_topic2.py
```

Note: That at this time we have not yet introduced the order of the tests by
introducing numbers in the tests.

## Test Specification and Execution

A simple example for a test is 

* <https://github.com/cloudmesh/cloudmesh-cm4/blob/master/tests/test_key.py>

Note that all test python programs have specific function names 
of the form

`def test_number_topic (self)`

The number is defined to order them and is typically something like `001`, note
the leading spaces. The topic is a descriptive term on what we test.

Each test starts with a setup function `def setup(self)` we declare a setup that
is run prior to each test being executed. Other functions will use the setup
prior to execution.

A function includes one or multiple asserts that check if a particular test
succeeds and reports this to nose to expose the information if a tess succeds or
fails, when running it

Note that all nosetest functions start with a `HEADING()` in the body which conveniently
prints a banner with the function name and thus helps in debugging in case of
errors.


Invocation is simply done with the comment lines you see on top that you will include.

in our case the test is called test_key.py so we include on the top

```
#############################################
# nosetest -v --nopature
# nosetests -v --nocapture tests/test_key.py
#############################################
```

You can than execute the test with either command. More information is printed
with the command

Make sure that you place this comment in your nosetests.

The following is our simple nosetests for key. THe file is stored at 
`tests/test_key.py`

First, we import the needed classes and methods we like to test. 
We define a class, and than we define the methods. such as the setup and the actual tests.

your run it with 

```bash
$ nosetests -v --nocapture tests/test_key.py
```

```python
############################################
# nosetest -v --nocapture 
# nosetests tests/test_key.py
# nosetests -v --nocapture tests/test_key.py
############################################
from pprint import pprint
from cloudmesh.common.Printer import Printer
from cloudmesh.common.util import HEADING
from cloudmesh.management.configuration.SSHkey import SSHkey
from cloudmesh.management.configuration.config import Config


class TestKey:

    def setup(self):
        self.sshkey = SSHkey()


    def test_01_key(self):
        HEADING()
        pprint(self.sshkey)
        print(self.sshkey)
        print(type(self.sshkey))
        pprint(self.sshkey.__dict__)

        assert self.sshkey.__dict__  is not None


    def test_02_git(self):
        HEADING()
        config = Config()
        username = config["cloudmesh.profile.github"]
        print ("Username:", username)
        keys = self.sshkey.get_from_git(username)
        pprint (keys)
        print(Printer.flatwrite(keys,
                            sort_keys=("name"),
                            order=["name", "fingerprint"],
                            header=["Name", "Fingerprint"])
              )

        assert len(keys) > 0

```

The output in with `nosetests tests/test_key.py` does not provide any detail,
but just reports if tests fail or succeed.

```
----------------------------------------------------------------------
Ran 2 tests in 0.457s

OK
```

The output with  `nosetests -v tests/test_key.py`

results in 

```
tests.test_key.TestName.test_01_key ... ok
tests.test_key.TestName.test_02_git ... ok

----------------------------------------------------------------------
Ran 2 tests in 1.072s

OK
```

During development phase you want to use `nosetests -v --nocapture tests/test_key.py`

WHich prints all print statements also

## Timed decorator

In some cases you may want to use a timed decorator that limits the time a test
is executed for. An example is given next:

```python
############################################
# nosetest -v --nocapture 
# nosetests tests/test_timed.py
# nosetests -v --nocapture tests/test_timed.py
############################################

class TestTimed:

    @timed(1.0)
    def test_10_sleep_which_fails():
        time.sleep(2.0)
```

## Test Setup

The setup in a class can be controlled by the following functions. We include in
the print statement when they are called:

```python
   def setup(self):
        print ("setup() is called before each test method")
 
    def teardown(self):
        print ("teardown() is called after each test method")
 
    @classmethod
    def setup_class(cls):
        print ("setup_class()is called before any methods in this class")
 
    @classmethod
    def teardown_class(cls):
        print ("teardown_class() is called after any methods in this class")
```

## Test Timer

The following extension adds timers to nosetests

* <https://github.com/mahmoudimus/nose-timer>

It is installed with 

```bash
$ pip install nose-timer
```

It is started with the flag `--with-timer`

Thus, 

```bash
$ nosetests -v --nocapture --with-timer tests/test_key.py
```

Will print the time for each test as shown in this partial output:

```
...
[success] 58.61% tests.test_key.TestName.test_02_git: 0.1957s
[success] 41.39% tests.test_key.TestName.test_01_key: 0.1382s
----------------------------------------------------------------------
Ran 2 tests in 0.334s
```

## Doctests in Cloudmesh

Out of principal we will not create an test running doctests.


## Sniffer (not tested)

often we make changes frequently and like to get an imediate feedback on the
changes made. For the automatic repeated execution on change we can use the tool
`sniffer`.

```bash
$ sniffer -x--with-spec -x--spec-color
```

This will execute the nosetests upon change. To specify a specific test you can
pass along the name of the python test.

To install it use

```bash
$ pip install sniffer
```

The manual page can be called with 

```bash
$ sniffer --help
```

## Profiling

Nosetest can be augmented with profiles that showcase some of the internal time
spend on different functions and methods. To do so install 

* <https://github.com/msherry/nose-cprof>

```bash
$ pip install nose-cprof
$ pip install cprofilev
$ pip install snakeviz
$ pip install profiling
```

Then call the nosetest with the additional option 

```bash
$ nosetests --with-cprofile
```

The output will be stored by default in `stats.dat`

To view it use cprofilev

```bash
$ cprofilev -f stats.dat
```

To view it with snakeviz use 

```bash
$ snakeviz -f stats.dat
```


To visualize the call graph we use pygraphviz. Unfortunatley it has an error and only produces png files.
Thus we use a modified version and instal it from source:

```bash
cd /tmp
$ git clone git@github.com:laszewsk/pycallgraph.git
$ cd pycallgraph
$ pip install .
```

Next we go to the cm directory and can creat a call graph from a python program
and open the output

```bash
$ pycallgraph graphviz -- tests/test_key.py 
$ open pycallgraph.pdf 
```

Another very usefule module is `profiling` which can be invoked with

```bash
$ profiling tests/test_key.py
```


To do live profiling you can use

```bash
$ profiling live-profile tests/test_key.py
```

