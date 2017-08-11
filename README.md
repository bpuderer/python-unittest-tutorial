# Python unittest Tutorial

Python unittest aka "PyUnit" is the python version of Beck and Gamma's JUnit and supports test fixtures, test cases, test suites, and test runners.

## Simple Test Structure

Test cases are created by subclassing unittest.TestCase and tests are methods which start with "test".  Methods which do not start with "test" will not be executed by the test runner.

```python
import unittest

class BasicTestCase(unittest.TestCase):

    def test_something(self):
        self.assertTrue(True)
```


## Assertions

The TestCase class provides a number of assert methods including the common assertEqual(), assertNotEqual(), assertTrue(), and assertFalse().  See the [official documentation](https://docs.python.org/2.7/library/unittest.html#assert-methods).

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

To unconditionally fail a test, use the fail method.

```python
    def test_failure(self):
        self.fail('optional message')
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

The three test outcomes are pass, fail, and error.  Tests which do not raise an error pass and display a ".".  Tests which raise AssertionError fail and display an "F".  Tests which raise an error other than AssertionError report error and display an "E".  Tests can also be skipped or denoted as expected failures.  More on these later.

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

Tests and test cases can marked for unconditional skipping by including the skip() decorator and display "s" or "S" when run.

```python
    @unittest.skip("good reason for skipping")
    def test_something(self):
        self.assertTrue(True)
```

Tests and test cases can also be *conditionally* skipped using skipIf() and skipUnless()

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

## Tagging Tests

Tests (and TestCases) can be tagged several different ways enabling dynamic selection with nose and nose2.

The examples below do not use the attr decorator available in nose.plugins.attrib.

A Boolean attribute can be used to tag a test so it can be chosen for execution as well as being excluded with a logical NOT.  In the example below, setting the nose/nose2 attribute argument (-a,--attr for nose, -A,--attribute for nose2) to "slow" selects this test and "\\!slow" (escape the !) selects everything but this test assuming they don't have the slow attribute at all or it is set to False.  Truthiness is used so it doesn't have to be a Boolean.

```python
    def test_timeouts(self):
        self.assertTrue(1)
    test_timeouts.slow = True
```

Specific values can also be used.  Setting the attribute arg to life=42 selects the following test.  The example uses an int but the match is *case insensitive*.

```python
    def test_something(self):
        self.assertTrue(1)
    test_something.life = 42
```

List attributes can also be used to tag tests for execution.  Setting the attribute arg to "tags=tag1" selects this test.  The match is *case insensitive* and there's nothing special about the name "tags".  NOTE: nose2 as tested with version 0.6.4 supports a logical NOT with lists but nose as of version 1.3.7 does not.  In the example, "\!tags=tag1" with nose2 selects tests *with* the "tags" list attribute which do not contain the "tag1" element.

```python
    def test_something_else(self):
        self.assertTrue(1)
    test_something_else.tags = ['tag1', 'tag2']
```

To run tests where the attribute args are logically ANDed, separate them with commas.  With nose, "-a tags=tag1,tags=tag2" only selects the second test.  To logically OR, pass multiple attribute args  With nose, "-a tags=tag1 -a tags=tag2" selects both.

```python
    def test_feature_scenario1(self):
        self.assertTrue(1)
    test_feature_scenario1.tags = ['tag1']

    def test_feature_scenario2(self):
        self.assertTrue(1)
    test_feature_scenario2.tags = ['tag1', 'tag2']
```

Nose and nose2 also support python expressions with attributes with the -A,--eval-attr and -E,--eval-attribute arguments respectively.  Example with nose: "-A 'slow or life==42'".  Example with nose2: "-E 'slow or life==42'".

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
        self.assertEndsInR('Doctor')
```

## Best Practices

* Tests should be small and test only one thing.
* Tests should run fast.  This is essential for inclusion in a CI environment.
* Tests should be fully independent.  Tests should not depend on each other and can run in any order any number of times.
* Tests should be fully automated and not require manual interaction or checks to determine if they pass or fail.
* Tests should not include unnecessary assertions such as "checkpoints" in the test. Assert *only* what the test is testing.  See first item in list.
* Tests should be portable and easily run in different environments.
* Tests should setup what they require to run.  Tests should not make assumptions about particular resources existing.
* Tests should cleanup created resources afterwards.
* Test names should clearly describe what they are testing.
* Tests should produce meaningful messages when they fail.  Try causing the test to fail and see if the failure reason can be determined by reading the test case output.  See example in Assertions section above - assertTrue vs assertEqual
* Tests should use the strongest assertions possible.  e.g. checking the contents of a list.  Does order matter? Are duplicates allowed? Are additional items allowed?  The answers determine the assertion that should be used, assertEqual, assertItemsEqual, assertEqual with set operations, etc.

## Links

[PSF unittest documentation](https://docs.python.org/2.7/library/unittest.html)

[Ned Batchelder: Getting Started Testing](http://nedbatchelder.com/text/test0.html)

[C. Titus Brown's An Extended Introduction to the nose Unit Testing Framework](http://ivory.idyll.org/articles/nose-intro.html)

[Doug Hellmann's PyMOTW unittest](https://pymotw.com/2/unittest/index.html)
