# should

Minimal xUnit library for C programmers.

I think today's most xUnit libraries are too much complex and there are
some good reasons to "reinvent the wheel". `shoud` simply and justly supports 
the key concepts of [Kent Beck's original testing framework]
(http://www.xprogramming.com/testfram.htm).

## Installation

```bash
$ # get
$ git clone git@github.com:9beach/should.git
$ # build
$ cd should
$ mkdir build
$ cd build
$ cmake ..
$ make
$ # test
$ ctest # `make test' or `ctest --output-on-failure' or `ctest -V', as you like
$ # install
$ sudo make install
```

## Glossary

### Test Suite

A test suite can hold multiple test cases. Test is performed by running a test 
suite.

### Test Case

Test case is void (*)(void *) type function including at least one `should_be` 
macro.

### `should_be` macros

`should_be` macros verify a expression. If the expression is not true, 
it prints error message (does not abort).

* `should_be(expr)` Verifies that the expression is true.
* `should_be_msg(expr, msg)` Verifies that the expression is true. Prints 
the given message if fails.
* `should_be_eq(val1, val2)` Verifies that two integers are the same.
* `should_be_ne(val1, val2)` Verifies that two integers are not the same.
* `should_be_lt(val1, val2)` Verifies that the first integer is less than the 
second.
* `should_be_le(val1, val2)` Verifies that the first integer is less than or 
equal to the second.
* `should_be_eq_str(val1, val2)` Verifies that two strings contains the same 
letters.
* `should_be_ne_str(val1, val2)` Verifies that two strings contains the
different letters.

### Setup, Teardown, and Fixture

Fixture is a environment which is shared among test cases and/or suites of a 
test suite.

You can initialize/finalize the fixture with the setup/teardown function. If a 
suite has many suites and/or cases, the setup/teardown functions are called 
for each of the suites and/or cases.

## Quick Start

### Simple Example
#### `test_simple.c`
```C
#include <should/should.h>

#include <stdlib.h>

void case_hello(void *fxtr)
{
        const char *a = "HELLO";
        const char *b = "hello";

        should_be_ne_str("hello", a);
        should_be_ne_str("HELLO", b);
        should_be_eq_str("hello", b);
}

void case_world(void *fxtr)
{
        int a = 10;
        int b = 20;

        should_be(a != b);
        should_be(a == 10);
        should_be_msg(a != 9, "OMG");

        should_be_eq(a, 10);
        should_be_eq(b, 20);
        should_be_ne(a, 20);
        should_be_ne(b, 10);
        should_be_ne(10, 20);
        should_be_ne(20, 10);
        should_be_lt(9, 20);
        should_be_le(9, 20);
        should_be_le(9, 9);
}

int main()
{
        /* as a example, we gonna make two test suites */
        should_suite_t *s0;
        should_suite_t *s1;

        s0 = should_create_suite("simple test A");
        if (!s0) {
            abort();
        }

        s1 = should_create_suite("simple test B");
        if (!s1) {
            abort();
        }

        /* suite s0 has two test cases */
        should_add_case(s0, case_hello);
        should_add_case(s0, case_world);

        /* suite s1 has one test case */
        should_add_case(s1, case_hello);

        /* and finally ... */
        return !(should_run_and_destroy_suite(s0) == 0 
                        && should_run_and_destroy_suite(s1) == 0);
}
```

#### Expected Outputs
```bash
$ gcc test_simple.c -lshould && ./a.out && rm -f a.out
*** Running should_suite "simple test A"...
*** Results: 0 failures, 15 successes
*** Running should_suite "simple test B"...
*** Results: 0 failures, 3 successes
```

### Example with a Fixture
#### `test_fixture.c`
```C
#include <should/should.h>

#include <stdlib.h>

void case_hello(void *fxtr)
{
        const char *a = "HELLO";
        const char *b = "hello";

        should_be_ne_str("hello", a);
        should_be_ne_str("HELLO", b);
        should_be_eq_str("hello", b);
}

void case_world(void *fxtr)
{
        int a = 10;
        int b = 20;

        should_be(a != b);
        should_be(a == 10);
        should_be_msg(a != 9, "OMG");

        should_be_eq(a, 10);
        should_be_eq(b, 20);
        should_be_ne(a, 20);
        should_be_ne(b, 10);
        should_be_ne(10, 20);
        should_be_ne(20, 10);
        should_be_lt(9, 20);
        should_be_le(9, 20);
        should_be_le(9, 9);
}

int main()
{
        int ret0, ret1;

        /* as a example, we gonna make two test suites */
        should_suite_t *s0;
        should_suite_t *s1;

        s0 = should_create_suite("simple test A");
        if (!s0) {
            abort();
        }

        s1 = should_create_suite("simple test B");
        if (!s1) {
            abort();
        }

        /* suite s0 has two test cases */
        should_add_case(s0, case_hello);
        should_add_case(s0, case_world);

        /* suite s1 has one test case */
        should_add_case(s1, case_hello);

        /* and finally ... */
        ret0 = should_run_and_destroy_suite(s0);
        ret1 = should_run_and_destroy_suite(s1);

        return ret0 || ret1;
}
```

#### Expected Outputs
```bash
$ gcc test_fixture.c -lshould && ./a.out && rm -f a.out
*** Running should_suite "fixture test"...
*** Results: 0 failures, 6 successes
```
### Failure Examples
```
*** Running should_suite "simple test A"...
should_be failed: ("hello" == a) -> not ("hello" == "HELLO") in "case_hello", /home/seafarer/should/tests/test_simple.c (10)
should_be failed: "OMG" (a == 9) in "case_world", /home/seafarer/should/tests/test_simple.c (22)
should_be failed: (b != 20) -> not (20 != 20) in "case_world", /home/seafarer/should/tests/test_simple.c (25)
*** Results: 3 failures, 12 successes
*** Running should_suite "simple test B"...
should_be failed: ("hello" == a) -> not ("hello" == "HELLO") in "case_hello", /home/seafarer/should/tests/test_simple.c (10)
*** Results: 1 failures, 2 successes
```
## Reference
* [Kent Beck's original testing framework paper]
(http://www.xprogramming.com/testfram.htm)
