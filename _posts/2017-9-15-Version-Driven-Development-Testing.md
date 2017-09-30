An approach to testing and test suite construction and management via a Version-Driven development style. What is written is not a formalized pattern, or something that is solid. The ideas here can be considered fluid.

The core questions to premise our discussion is in regards to general application projects and test suite creation and management for those projects:

1. When to write a test?
2. How many tests to write (whether locally per function, or globally in terms of total number of tests)?

The common answer to these questions is: code coverage.

Code-Coverage-Driven-Development is this game a developer plays in which they see how close to "100% coverage" they can get so that the little testing badge on their GitHub repo turns the color green. It's not *necessarily* (in of itself) a useful (as in a good, let alone exceptional, ROI) thing.

So, how else can we answer our two core questions?

First, we need to define the properties of a "test" and a "test suite".

## Test Types

Let's differentiate between tests that are focused on a single internal package and tests that are focused on something about our application overall.

Tests that apply to internal packages for our application are "Intra-" tests. And tests that apply to multiple packages or our application overall are called "Inter-" tests.

We are testing for the existence of or the lacking of certain properties.

- `Unitary`: properties of a single code object (where code object is something like e.g. a function)
    + obviously, these tests always will be "intra-"
    + Unitary is not necessarily synonymous with the commonly known "Unit" test (discussed later). Although, the two are likely to be heavily correlated in a test suite.
- `Compositional`: properties of a composition of code objects (e.g.  a test that calls a sequence of code objects in a particular order)
    + `Intra-Compositional`: code objects from the same internal package composed.
    + `Inter-Compositional`: code objects from different packages composed.
- `Relational`: properties of the relationship between code objects
    + `Intra-Relational` properties for relationships between two code objects inside a single internal package, and `Inter-Relational` properties for two code objects located in different internal packages.
    + `Differential`: properties of a relationship that define how two code objects are different (e.g. a test for redundancy)
    + `Intersective`: properties of a relationship that define how two code objects must remain similar (e.g. to guarantee that they agree on certain values)
        * `Indempotence`: is an example of an intersective property of the relationship between a code object and any n-self-composition of itself.

It might also be useful to differentiate between:

- Time-Dependent (property) tests
- Time-Independent (property) tests. 

Time-dependent tests tend to take longer to test a particular property of a piece of a code. So, it would be good to ensure that a test is *time-consistent* i.e. it only asserts for either all time-dependent or time-independent properties.

It is also true that all of these Test "Types" are really: `Assertion Types` (asserted properties). And often it is the case that a Test block (test function) contains multiple assertions about a (set of) code block(s). 

Using Test Types allows us to group Test Blocks by assertion types (e.g. all Unitary property tests for a code block are in the same Test function).

## Test Purpose/Focus

Even though we have classified the types of tests we can have in the previous section, a test often has an aim/purpose/focus to its existence.

### Behavior Tests

- `Unit & Component`: focused on ensuring that a code block or composition of code blocks fulfills some desired properties and/or avoids some undesirable properties.
- `Fuzz`: not so much a "test" as it is data collection on finding corner-test-cases that crash the code block (or composition of code blocks).
- `Mutation`: tests the coverage of behavior tests, not so much a test as much as collecting coverage metrics for every piece of code (versus the set of behavior tests that apply to it).

### Performance Tests

- `benchmarking`: not really tests so much as collecting important data on a code block -- nonetheless, the data collected can still be Typed by the Assertion Types discussed in the previous section
    + Intra-Unitary (the most common -- benchmarking a single function)
    + Intra-Compositional (benchmarking a sequence of functions)
    + Inter-Compositional (benchmarking a sequence of functions)
- `profiling`: not really tests so much as collecting important data on resource usage (machine and runtime)
    + useful in designing and a necessity for running automated regression tests.
- `load`: for testing application response latency percentiles.

### External Tests

- `Integration` - tests how our application talks to other applications and services (using Test Doubles)
- `Contract` - tests how other applications, services, and users will interact with our application
    + tests for all previous versions of an API client release (to test against backwards-incompatibility)
    + "user-experience" contracts
    + [Consumer-Driven Contracts](https://www.martinfowler.com/articles/consumerDrivenContracts.html)

Contract testing an API client would usually revolve around making sure that the API has not undergone any breaking changes such that an existing client can no longer speak to it. In that case, we say that we are testing against an `objective breaking-change`. 

A `subjective breaking-change` is one that breaks the expected user-experience.

Why are contract tests important?

> "The underlying assumption here is that services, by themselves, are of no value to the business; their value is in being consumed"
> - https://martinfowler.com/articles/consumerDrivenContracts.html

### Regression Tests

Regression tests really do nothing more than give us pre-defined comfort/tolerance bounds for how much our code can "regress" per some metric e.g. performance, coverage, etc. Having these bounds allows for faster development cycles, instead of simply specifying that all code and tests should always be 100% perfection and never veer.

- `Behavioral Regressions`: e.g. using [Diffy](https://github.com/twitter/diffy) for automated regression testing between API versions
- `Coverage Regression`: an example is if we are used to using mutation tests to ensure that our unit test has 100% code coverage, but we would be comfortable with anything above 90%, then a coverage regression test simply fails if the latest mutation test is under 90% coverage for one of our behavior tests.
- `Performance Regressions`
    + `benchmark regressions`: ensuring that a modified piece of code still performs within certain bounds
        * Intra-Differential (e.g. confirming that one function must always be a faster option than another function)
    + `profiling regressions`: resource-usage regressions -- making sure we have not increased the computational time or memory usage of a code block more than a certain allowed delta.
    + `load regressions`: P9X latency category regressions
- `Integration Regressions`: [Integration-Contract](https://martinfowler.com/bliki/IntegrationContractTest.html) Tests

While regression tests can be run every patch change, it is usually best to have a policy of only using them every release.

## Test Maturity

A test has two primary maturity levels:

- **Developer Confidence:** often gained through manual tests -- or automation of workflows i.e. automating manual tests.
- **Application Invariant:** where the greatest value of a test can be derived, the longer the lifetime of the test the more valuable it is in regards to testing modifications to the application overtime.

A test will show its maturity if it moves from being an automation of a developer confidence workflow to being the automation of an application invariance check.

It can be useful to imagine every test like a singular invariant point in a complex, multi-dimensional geometric shape that is our application. And we are allowed to modify (feature additions, bug patching, etc.) and "rotate" (refactor/restructure) our application around these points.

## Test Suites

So now, we can answer our initial questions:

1. When to write a test?
    - when we want to increase developer confidence in the code
    - when we want to define a (long-term) invariant of the application
2. How many tests to write (whether locally per function, or globally in terms of total number of tests)?
    - the number of tests that cover the property types (unitary, compositional, relational) of the package and/or application that must remain invariant

If we were to be more specific: a good general rule is that the number of property assertions made for a code block should be equal to the [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity) of that code block. 

This usually translates to a single table-driven test, where the number of rows in the table is equivalent to the cyclomatic complexity of the code block being tested.

In the case of writing  (a complex) `regex` based code blocks, it might be useful to specify a table driven test that the regex should be able to pass before designing the regex so it can be easily tested on implementation iterations.

There are a million more random tips one could think up for all different kinds of code blocks. There is also no need to take anything written here as a hard and fast rule.

### Test Suite Maturation

- `(0.0.0 - 0.alpha.0/0.beta.0)` **Automated Workflow** focused (i.e. automating manual procedures of testing)
- `(0.alpha.0/0.beta.0 - 1.0.0)` (Pre-release defined) **Bug Class Catcher** focused
    + after coming to recognize real-world bugs (usually during a private or limited public alpha / beta pre-release); create test templates/formats that catch various bug classes recognized from issues encountered during that release phase.
- `(1.0.0 - 2.0.0)` **User-Experience Contract** focused
    + incompatibility is defined by a contract between what the user generally expects (i.e. the experience you want to guarantee the user -- within some bounds) and what the code actually does.
    + This is the key bridge between real-world user experiences with your application and the nitty-gritty of the code you write.

Once consumer-experienced-focused maturity has been reached *then* we can worry about the next "phase" of VDD i.e. how we focus our product revisions and releases around Semantic Version increments. This will be discussed in a later blog post.

### Test Suite manipulation

These rules below only apply to tests with an `Application Invariant` maturity level.

- **Invariant Testing Suite**
	- Test Additions
	- Test Modifications
	- Test Deletions
- **Test Additions:** *only* create a new test *iff* the *removal* of that test is a *deceleration* of backwards incompatibility. 
	- the addition of a test should change the minor version of a program/API
	- a set of additions can be a backwards-compatible version change (a Minor Version Update)
- **Test Modifications:** any modification to a test must *never* create/represent a breaking (backwards-incompatible) change/version update.
	- the modification of a test should represent a patch version update
	- a set of unique test modifications can also be a single patch version update
- **Test Deletion:** deleting a test should *always* represent/declare a breaking (backwards-incompatible) change i.e.  Major Version update
	- a set of test deletions can also be a single (or part of a) major version update

In other words: *only* deleting tests is equivalent to a breaking change. But, what if we make a modification to a test that acts as a breaking change? Then this is *not* a test modification (as defined above). That is the equivalent of a `test deletion + test addition`. So, if a released version of an API passes the current test suite then the latest version will *not* be a Major version update.

Development of an API should be `0.0.0` - `1.0.0` and the test suite should evolve accordingly corresponding to feature additions and patch updates. `1.0.0` is allowed to contain a breaking change relative to all `0.*.*` versions, but does not necessarily have to represent a breaking change. It is supposed to represent an initial release (not alpha or beta).

## Overview

```go

package test

// Test ...
type Test struct{
    ID int // must be unique
    Type string // should be a TestType enum
    Purpose string // e.g. unit, integration, fuzz, mutation, etc.
    Maturity string // should be a TestMaturity enum
}

// TestSuite ...
type TestSuite struct{
    Name string
    Tests []Test
    Maturity string // version-driven
}

// Application ...
type Application struct{
    Suite TestSuite
}
```

Simple test suite management package coming soon ...
