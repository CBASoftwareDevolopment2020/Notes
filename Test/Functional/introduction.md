# [Introduction](https://datsoftlyngby.github.io/soft2020spring/TST/week-07/#1-introduction)

10-02-2020

-   Robustness
    -   Defensive programming
    -   Encapsulation
    -   Complexity
    -   Murphy
-   Tests
    -   Test Types
    -   Test Levels

## Levels and Types

| Level/Type  | Functional | Operational<br>Security<br>Performance | Developmental |
| ----------- | ---------- | -------------------------------------- | ------------- |
| Unit        |            |                                        |               |
| Component   |            |                                        |               |
| Integration |            |                                        |               |
| System      |            |                                        |               |
| Acceptance  |            |                                        |               |

### Test Types

-   Functional Tests
-   Operational Tests
    -   Performance
    -   Scalability
    -   Security
-   Developmental Tests
    -   Maintainability
    -   Technical debt
-   Manual Tests
-   Automatic Tests
-   Regression Tests
    -   Tests to run every time you push
-   Black-box Testing
-   White-box Testing
    -   Code Coverage

### Test Levels

-   Unit Tests
    -   A function or method
    -   A class or an object
    -   Several highy cohesive classes
    -   Uses mocking to focus on relevant unit
    -   But <span style="color:red">**not**</span> a component
-   Component Tests
    -   Reusable pieace of software
    -   **Not** an application in itself
    -   Hidden implementation
    -   Has a well defined interface
    -   Is detachable from other code
    -   When tested, stored as binary
    -   Saves tests in future uses
    -   Software Factory
-   Integration Tests
    -   Narrow
        -   Exercise only that portion of the code in my service that talks to a separate service
        -   Uses test doubles af those services, either in process or remote
        -   Thus consist of many narrowly scoped tests, often no larger in scope than a unit test (and usually run with the same test fraework that's used for unit tests)
    -   Broad
        -   Require live versionof all services, requiring subsantial test environment and network access
        -   Exercise code paths through all services, not just code responsible for interactions
-   System Tests
    -   Very broad integration tests involving the whole system
    -   Contract tests
    -   Subcutaneous test
        -   Testing everything from just under the skin of UI
-   Acceptance Tests
    -   Business facing tests
        -   A business-facing test is a test that’s intended to be used as an aid to communicating with the non-programming members of a development team such as customers, users, business analysts and the like
        -   When automate, theydescribe the system in domain-oriented terms, ignoring the component architecture of the system itself
        -   Business-facing tests are often used as acceptance criteria, having such tests pass indicate the system provides the functionality that the customer expects
    -   User journey tests

![Agile Testing](http://www.agilebuddha.com/wp-content/uploads/2015/03/Agil-Testing-Quadrants.png)

#### Security Tests

CIA: **C**onfidentiality, **I**ntegrity, **A**vailability

## Aspects of tests

-   Testing of Critiques
    -   Specifications as Tests
    -   Verification
    -   Validation
-   Testing as Support
    -   Requirements as Tests
    -   Test Driven Development
    -   Characterization tests

## Robustness

-   What is robustness?
    -   Fault tolerant
    -   Graceful Shutdown
    -   Meaningful feedback on errors
    -   Recoverable
-   How to obtain robustness

    -   Program defensively
        -   Require your preconditions to hold
        -   Ensure your postconditions hold
        -   Defend your gates against intruders
        -   Let no spies get out
    -   Hide information
        -   Use the languag's build in incapsulation mechanisms
        -   or trick the language to hide the information
        -   With no walls, defending the gates alone is not very effective
        -   Never reveal your Weaknesses
    -   Reduce complexity
        -   Bundle parts of code with high cohesion
        -   Decouple bundle parts
        -   Be sure thatyour units are wellorganized and cooperating
        -   Don't let one unit's defeat lose the battle
    -   Expect the impossible
        -   Remember Murphy's law

### Defensive Programming

-   Always check the preconditions, <span style="color:red">**trust nobody!**</span>
-   Check your postconditions, <span style="color:red">**doubt yourself!**</span>
-   Check your invariants
-   Remember the ACID rules
    -   **A**tomicy
    -   **C**onsistency
    -   **I**solation
    -   **D**urability

### Encapsulation

-   Onlyuse public where necesary accordig to contract
-   Use Facade objects
-   Use Data Transfer Objects and tokens
-   Be as abstract as possible
-   Don't promise more than required

## What to test

-   Preconditions and invatiants will be assumptions, and it should be tested, that they hold, at least in the preproduction phase. A defensive approach might be fine here = preconditions will be tested in production as well
-   Postconditions and invariants will be assertions. Postcondition tests shouldbe made for both happy and less happy scenarios
-   Defining what is precondtion, postcondition, and invariants also clarifies the requests

## Documents

-   Specification of invariants, pre, and postcondtions
-   Specification of test to run (perform)
-   Specification of the coverage of the tests
-   Specification of expected output of the tests
