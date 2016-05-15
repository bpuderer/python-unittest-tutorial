# Python unittest Tutorial

Python unittest aka "PyUnit" is the python version of JUnit by Beck and Gamma and supports test fixtures, test cases, test suites, and test runners.

## Simple Test Structure

Test cases are created by subclassing unittest.TestCase and tests are methods which start with "test".  Methods which do not start with "test" will not be executed by the test runner.

```python
import unittest

class BasicTestCase(unittest.TestCase):

    def test_something(self):
        self.assertTrue(True)
```


## Assertions

The TestCase class provides a number of assert methods including the common assertEqual(), assertNotEqual(), assertTrue(), and assertFalse().  See the [official documentation] (https://docs.python.org/2.7/library/unittest.html#assert-methods).

As of python 2.7, assertEqual() automatically selects type-specific methods if the two args are the same type including strings, sequences, lists, tuples, sets or frozensets, dicts.  See [docs](https://docs.python.org/2.7/library/unittest.html#type-specific-methods) for details.  These methods construct error messages detailing the differences between the two arguments.

```python
import unittest

class BasicTestCase(unittest.TestCase):

    def test_using_asserttrue(self):
        list_a = [0, 1, 2]
        list_b = [1, 2]
        self.assertTrue(list_a == list_b)

    def test_using_assertequal(self):
        list_a = [0, 1, 2]
        list_b = [1, 2]
        self.assertEqual(list_a, list_b)
```

When run:

```
$ python -m unittest discover
FF
======================================================================
FAIL: test_using_assertequal (test_example.BasicTestCase)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/home/ben/wip/test_example.py", line 13, in test_using_assertequal
    self.assertEqual(list_a, list_b)
AssertionError: Lists differ: [0, 1, 2] != [1, 2]

First differing element 0:
0
1

First list contains 1 additional elements.
First extra element 2:
2

- [0, 1, 2]
?  ---

+ [1, 2]

======================================================================
FAIL: test_using_asserttrue (test_example.BasicTestCase)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/home/ben/wip/test_example.py", line 8, in test_using_asserttrue
    self.assertTrue(list_a == list_b)
AssertionError: False is not true

----------------------------------------------------------------------
Ran 2 tests in 0.001s

FAILED (failures=2)
```

To assert that an exception is raised, a context manager with the assertRaises method can be used.  assertRaisesRegexp can also check the raised exception's string representation.

```python
    def test_exception(self):
        with self.assertRaises(TypeError):
            # statements that raise exception
            raise TypeError
```

## Running Tests

The most basic way of running tests is to include unittest.main() then execute the script.

```python
if __name__ == '__main__':
    unittest.main()
```

Another way is to utilize [unittest discovery](https://docs.python.org/2.7/library/unittest.html#test-discovery):

```
$ python -m unittest discover -v
```

[nose](https://nose.readthedocs.io/en/latest/), in maintenance mode for several years, can be used to run tests and generate xUnit style xml reports.  These reports can then be used by [Jenkins](https://github.com/jenkinsci/jenkins) to publish test results for jobs (Post-build Actions > Publish JUnit test result report > Test report XMLs).

```
$ nosetests --with-xunit
```

[nose2](https://github.com/nose-devs/nose2) is the next generation of nose and is actively maintained.

```
$ nose2 --plugin nose2.plugins.junitxml --junit-xml
```

## Test Outcomes

There are three outcomes for a test: pass, fail, and error.  Tests which do not raise errors pass and display a ".".  Tests which raise AssertionError fail and display "F".  Tests which raise an error other than AssertionError report error display an "E".  Tests can also be skipped and denoted as expected failures.  More on these later.

First line of output where five tests pass, one fails, and one reports an error.

```
$ nose2
....E.F
```

## Test Fixture

setUp(), tearDown(), setUpClass(), tearDownClass(), setUpModule(), tearDownModule() are used to prepare and cleanup the test fixture.

setUp() and tearDown() run before and after ever test.  tearDown() only runs if setUp() was successful and runs regardless of the test result.

setUpClass() and tearDownClass() are run before/after an individual test class and must be decorated as a classmethod().  If an error occurs in setUpClass(), the tests and tearDownClass() are not run.  Also, neither are run if the class is marked skip.

setUpModule() and tearDownModule() are run before/after the test classes and are implemented as functions.  If an error occurs in setUpModule(), the tests and tearDownModule() are not run.  

```python
import unittest

def setUpModule():
    print "in setUpModule"

def tearDownModule():
    print "\nin tearDownModule"

class BasicTestCase1(unittest.TestCase):

    @classmethod
    def setUpClass(cls):
        print "in setUpClass"
 
    @classmethod
    def tearDownClass(cls):
        print "\nin tearDownClass"

    def setUp(self):
        print "in setUp"

    def tearDown(self):
        print "in tearDown"

    def test_something(self):
        print "in BasicTestCase1.test_something"
        self.assertTrue(True)

class BasicTestCase2(unittest.TestCase):

    def test_something(self):
        print "in BasicTestCase2.test_something"
        self.assertTrue(True)
```

When run:

```
in setUpModule
in setUpClass
in setUp
in BasicTestCase1.test_something
in tearDown
.
in tearDownClass
in BasicTestCase2.test_something
.
in tearDownModule

----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
```

## Test Skipping

Tests can marked for unconditional skipping by including the skip() decorator and display "s" or "S" when run.

```python
    @unittest.skip("good reason for skipping")
    def test_something(self):
        self.assertTrue(True)
```

Tests can also be conditionally skipped using skipIf() and skipUnless()

```python
    @unittest.skipIf(platform.system() == 'Windows', 'does not run on windows')
    def test_skip_if_windows(self):
        self.assertTrue(True)

    @unittest.skipUnless(platform.system() == 'Linux', 'Linux required')
    def test_skip_unless_linux(self):
        self.assertTrue(True)
```

Tests can also be skipped during execution by calling skipTest()

```python
    def test_skip_during_test(self):
        self.skipTest('skip mid-test')
```

## Expected Failure

Tests decorated with expectedFailure do not count as a failure if they fail.  If a test passes which is decorated with expectedFailure it registers as an 'unexpected success' and may or may not count as a failure depending on the runner.

```python
    @unittest.expectedFailure
    def test_fails(self):
        self.assertTrue(False)

    @unittest.expectedFailure
    def test_should_fail_but_doesnt(self):
        self.assertTrue(True)
```

When run using unittest:
```
$ python test_expected_failure.py 
xu
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK (expected failures=1, unexpected successes=1)
```

When run using nose:
```
$ nosetests
/usr/lib/python2.7/unittest/case.py:342: RuntimeWarning: TestResult has no addExpectedFailure method, reporting as passes
  RuntimeWarning)
./usr/lib/python2.7/unittest/case.py:350: RuntimeWarning: TestResult has no addUnexpectedSuccess method, reporting as failures
  RuntimeWarning)
F
======================================================================
FAIL: test_should_fail_but_doesnt (test_expected_failure.BasicTestCase)
----------------------------------------------------------------------
_UnexpectedSuccess

----------------------------------------------------------------------
Ran 2 tests in 0.003s

FAILED (failures=1)
```

When run using nose2:
```
$ nose2
xu
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK (expected failures=1, unexpected successes=1)
```


## Base Test Class

A base test case can be added to make custom assertions and helpers available to a number of tests.

```python
import unittest

class BaseTestCase(unittest.TestCase):
    def assertEndsInR(self, seq):
        if seq[-1].lower() != 'r':
            raise AssertionError("{} does not end in 'r'".format(seq))

class ExampleTestCase(BaseTestCase):
    def test_str_ends_in_r(self):
        self.assertEndsInR('Doctors')
```

## Best Practices

* Tests should be small and test only one thing.
* Tests should run fast.  This is essential for inclusion in a CI environment.
* Tests should be fully independent.  Tests should not depend on each other and can run in any order any number of times.
* Tests should be fully automated and not require manual interaction or checks to determine if they pass or fail.
* Tests should not include unnecessary assertions such as "checkpoints" in the test. Assert *only* what the test is testing.  See first item in list.
* Tests should be portable and easily run in different environments.
* Tests should setup what they require to run.  Tests should not make assumptions about particular resources existing.
* Tests should cleanup afterwards.
* Test names should clearly describe what they are testing.  See first item in list.
* Tests should produce meaningful messages when they fail.  Try causing the test to fail and see if the reason for the failure can be determined by reading the output of the test case alone.  See example in Assertions section above - assertTrue vs assertEqual
* Tests should use the strongest assertions possible.  e.g. checking the contents of a list.  Does order matter? Are duplicates allowed? Are additional items allowed?  The answers determine the assertion that should be used, assertEqual, assertItemsEqual, assertEqual with set operations, etc.

## Links

[PSF unittest documentation](https://docs.python.org/2.7/library/unittest.html)

[Ned Batchelder: Getting Started Testing](http://nedbatchelder.com/text/test0.html)

[Doug Hellmann's PyMOTW unittest](https://pymotw.com/2/unittest/index.html)
