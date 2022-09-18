This is my summary of The Art of Unit Testing (Second Edition), by Roy Osherove.

I recommend you to buy and read the book to learn any concept directly from its author.

# Contents

- [Definitions](#definitions)
- [Properties of a good unit test](#properties-of-a-good-unit-test)
- [TDD](#tdd)
- [Writing tests tips](#writing-tests-tips)
- [Refactoring the design to be more testable](#refactoring-the-design-to-be-more-testable)
- [Interaction testing](#interaction-testing)
- [Mock vs Stub](#mock-vs-stub)
- [One mock per test](#one-mock-per-test)
- [Overspecifying the tests](#overspecifying-the-tests)
- [Tests readability](#tests-readability)
- [Building a test API for your application](#building-a-test-api-for-your-application)
- [The pillars of good unit tests](#the-pillars-of-good-unit-tests)
  - [Writing trustworthy tests](#writing-trustworthy-tests)
  - [Writing maintainable tests](#writing-maintainable-tests)
    - [Avoid testing private or protected methods](#avoid-testing-private-or-protected-methods)
    - [Remove duplications](#remove-duplications)
    - [Avoid using [SetUp]](#avoid-using-setup)
    - [Enforcing test isolation](#enforcing-test-isolation)
    - [Avoiding multiple asserts on different concerns](#avoiding-multiple-asserts-on-different-concerns)
    - [Testing multiple aspects of the same object in one test](#testing-multiple-aspects-of-the-same-object-in-one-test)
    - [Avoiding overspecification](#avoiding-overspecification)
  - [Writing readable tests](#writing-readable-tests)
    - [Naming unit tests](#naming-unit-tests)
    - [Naming variables](#naming-variables)
    - [Asserting yourself with meaning](#asserting-yourself-with-meaning)
    - [Separating asserts from actions](#separating-asserts-from-actions)
    - [Seting upd and tearing down](#seting-upd-and-tearing-down)

# Definitions

**SUT**  
Stands for system under test, and some people like to use CUT (class under test or code under test). When you test something, you refer to the thing you're testing as the SUT.

**End result**  
An end result of a system is any of the following:
- The invoked public method returns a value (a function that's not void).
- There's a noticeable change to the state or behavior of the system before and after invocation that can be determined without interrogating private state.
- There's a callout to a third-party system over which the test has no control, and that third-party system doesn't return any value, or any return value from that system is ignored.

**Unit of Work**  
Is the sum of actions that take place between the invocation of a public method in the system and a single noticeable end result by a test of that system. A noticeable end result can be observed without looking at the internal state of the system and only through its public APIs and behavior.

**Unit test**  
Is an automated piece of code that invokes the unit of work being tested, and then checks some assumptions about a single end result of that unit. A unit test is almost always written using 
a unit testing framework. It can be written easily and runs quickly. It's trustworthy, readable, and maintainable. It's consistent in its results as long as production code hasn't changed.

**Integration test**  
Is testing a unit of work without having full control over all of it and using one or more of its real dependencies, such as time, network, database, threads, random number generators, and so on.

**Refactoring**  
Refactoring is the act of changing code without changing the code's functionality. That is, it does exactly the same job as it did before. No more and no less. It just looks different. A refactoring example might be renaming a method and breaking a long method into several smaller methods.

**External dependency**  
An external dependency is an object in your system that your code under test interacts with and over which you have no control. (Common examples are filesystems, threads, memory, time, and so on.)

**Stub**  
A stub is a controllable replacement for an existing dependency (or collaborator) in the system. By using a stub, you can test your code without dealing with the dependency directly.

**Seams**  
Seams are places in your code where you can plug in different functionality, such as stub classes, adding a constructor parameter, adding a public settable property, making a method virtual so it can be overridden, or externalizing a delegate as a parameter or property so that it can be set from outside a class. Seams are what you get by implementing the Open-Closed Principle, where a class's functionality is open for  extenuation, but its source code is closed for direct modification.

**Interaction testing**  
Interaction testing is testing how an object sends messages (calls methods) to other objects. You use interaction testing when calling another object is the end result of a specific unit of work.

**Mock object**  
A mock object is a fake object in the system that decides whether the unit test has passed or failed. It does so by verifying whether the object under test called the fake object as expected. There's usually no more than one mock per test.

**Fake object**  
A fake is a generic term that can be used to describe either a stub or a mock object (handwritten or otherwise), because they both look like the real object. Whether a fake is a stub or a mock depends on how it's used in the current test. If it's used to check an interaction (asserted against), it's a mock object. Otherwise, it's a stub.

**Dynamic fake object**  
Is any stub or mock that's created at runtime without needing to use a handwritten (hardcoded) implementation of that object.

# Properties of a good unit test
A unit test should have the following properties:
- It should be automated and repeatable.
- It should be easy to implement.
- It should be relevant tomorrow.
- Anyone should be able to run it at the push of a button.
- It should run quickly.
- It should be consistent in its results (it always returns the same result if you don't change anything between runs).
- It should have full control of the unit under test.
- It should be fully isolated (runs independently of other tests).
- When it fails, it should be easy to detect what was expected and determine how to pinpoint the problem.

# TDD
- Just because you write your tests first doesn't mean they're maintainable, readable, or trustworthy.
- Just because you write readable, maintainable tests doesn't mean you get the same benefits as when writing them test-first. [Kent Beck's Test-Driven Development: by Example (Addison-Wesley Pro-
fessional, 2002)]
- Just because you write your tests first, and they're readable and maintainable, doesn't mean you'll end up with a well-designed system. [Growing Object-Oriented Software,
Guided by Tests by Steve Freeman and Nat Pryce (Addison-Wesley Professional,2009)]

# Writing tests tips
- It's common practice to have one test class per tested class, one unit test project per tested project (aside from an integration tests project for that tested project), and at least one test method per unit of work (which can be as small as a method or as large as multiple classes).
- Name your tests clearly using the following model: *UnitOfWork_Scenario_ExpectedBehavior*.
- Use factory methods to reuse code in your tests, such as code for creating and initializing objects all your tests use.
- Don't use [SetUp] and [TearDown] if you can avoid them. They make tests less understandable.

# Refactoring the design to be more testable
- Extract an interface to allow replacing underlying implementation
  - Inject a fake at the constructor level (constructor injection). Required dependency.
  - Injecting a fake as a property (property injection). Optional dependency.
  - Injecting a fake just before a method call.
    - Static factory with a seam to set the fake class (Add a seam to setup the stub to be retured ie: ExtensionManagerFactory.SetManager())
    - Factory method (add virtual method in the class under test, inherit from it in the test overriding the method to return the fake class)
- Using internal and [InternalsVisibleTo]. For only expose internal constructors, methods or properties to test projects.
- Using the [Conditional] attribute. Only adding methods or properties in certain build type.
- Using #if and #endif with conditional compilation. Same as previous but code looks messy.

# Interaction testing
You can also think of interaction testing as being action-driven testing. Action-driven testing means that you test a particular action an object takes.
Is preferable value-based or state-based tests over interaction testing.

Verify that methods were called on your interfaces doesn't necessarily mean that you're testing the right thing.
Testing that an object subscribed to an event doesn't tell you anything about the functionality of that object.
Testing that when the event is raised something meaningful happens is a better way to test that object.

Testing interactions is a double-edged sword: test it too much, and you start to lose sight of the big picture-the overall functionality; test it too little, and you'll miss the important interactions between objects

# Mock vs Stub
- Stubs can't fail a test. The asserts of the test are always against the SUT.
- Mocks can fail the test. The test uses the mock object to verify that the test passes.

# One mock per test
In a test where you test only one thing, there should be no more than one mock object. All other fake objects will act as stubs.

Having more than one mock per test usually means you're testing more than one thing, and this can lead to complicated or brittle tests.

Having more than one mock per test is the same as testing several end results of the same unit of work.

# Overspecifying the tests
Overspecification is the act of specifying too many things that should happen that your test shouldn't care about.

These extra specifications can make your test fail for all the wrong reasons. Then you'll have to constantly change your tests to suit the internal implementation of your code

# Tests readability
Having many mocks, or many expectations, in a single test can ruin the readability of the test so it's hard to maintain or even to understand what's being tested.

If you find that your test becomes unreadable or hard to follow, consider removing some mocks or some mock expectations or separating the test into several smaller tests that are more readable.
If you can't name your test because it does too many things, it's time to separate it into more than one test.

Avoid mock objects if you can. Tests will always be more readable and maintainable when you don't assert that an object was called.

# Building a test API for your application
As you write your tests, you'll create many simple utility methods that may or may not end up inside your test classes.
These utility classes become a big part of your test API, and they may turn out to be a simple object model you could use as you develop your tests.
You might end up with the following types of utility methods:
- Factory methods for objects that are complex to create or that routinely get createdby your tests.
- System initialization methods.
- Object configuration methods.
- Methods that set up or read from external resources such as databases, configurationfiles, and test input files. This is more commonly used in integration or system testing.
- Special assert utility methods, which may assert something that's complex or that's repeatedly tested inside the system's state.

# The pillars of good unit tests

The tests that you write should have three properties that together make them good:
- **Trustworthiness** -> Developers will want to run trustworthy tests, and they'll accept the test results with confidence. Trustworthy tests don't have bugs, and they test the right things.
- **Maintainability** -> Developers will simply stop maintaining and fixing tests that take too long to change or that need to change very often on very minor production code changes.
- **Readability** -> Maintaining tests becomes harder, and you can't trust them anymore because you don't understand them.

## Writing trustworthy tests
A trustworthy test is one that makes you feel you know what's going on and that you can do something about it. Guideline to achive that:

- Decide when to remove or change tests
  - Production bugs
  - Test bugs
  - Semantics or API changes
  - Conflicting or invalid tests
- Avoid test logic
  - if, else, switch, foreach, for, try-catch,...
  - Dynamically defining the expected result. If you already know how the result should look, hardcode it.
- Test only one concern
  - Avoid multiple asserts
  - You end up with generic test names that forces the reader to read the test code.
- Separate unit from integration tests
- Push for code reviews as much as you push for code coverage

## Writing maintainable tests

### Avoid testing private or protected methods
- Make methods public
- Extract methods to a new class
- Make method static (helper or utility)
- Make methods internal

### Remove duplications
Duplicated code means more code to change when one aspect you test against changes.

```cs
[Test]
public void IsValid_LengthBiggerThan8_IsFalse()
{
    LogAnalyzer logan = new LogAnalyzer();
    bool valid = logan.IsValid("123456789");
    Assert.IsFalse(valid);
}

[Test]
public void IsValid_LengthSmallerThan8_IsTrue()
{
    LogAnalyzer logan = new LogAnalyzer();
    bool valid = logan.IsValid("1234567");
    Assert.IsTrue(valid);
}
```

The main problem is that if the way you use _LogAnalyzer_ changes (its semantics, ie: if not initialized throws), the tests will have to be maintained independently of each other, leading to more maintenance work.

```cs
[Test]
public void IsValid_LengthBiggerThan8_IsFalse()
{
    LogAnalyzer logan = GetNewAnalyzer();
    bool valid = logan.IsValid("123456789");
    Assert.IsFalse(valid);
}

[Test]
public void IsValid_LengthSmallerThan8_IsTrue()
{
    LogAnalyzer logan = GetNewAnalyzer();
    bool valid = logan.IsValid("1234567");
    Assert.IsTrue(valid);
}

private LogAnalyzer GetNewAnalyzer()
{
    LogAnalyzer analyzer = new LogAnalyzer();
    analyzer.Initialize();
    return analyzer;
}
```

### Avoid using [SetUp]
Because, to read the tests for the first time and understand why they break, you need to do the following:

1. Go through the setup method to understand what's being initialized.
2. Assume that objects in the setup method are used in all tests.
3. Find out later you were wrong, and read the tests again more carefully to see which test uses the objects that may be causing the problems.
4. Dive deeper into the test code for no good reason, taking more time and effort to understand what the code does.

Always consider the readers of your tests when writing them. Imagine this is the first
time they read them. Make sure they don't get angry.

### Enforcing test isolation
There are several test "smells" that can hint at broken test isolation:
- Constrained test order -> Tests expecting to be run in a specific order or expecting
information from other test results
- Hidden test call -> Tests calling other tests
- Shared-state corruption -> Tests sharing in-memory state without rolling back
- External shared-state corruption -> Integration tests with shared resources and no rollback

### Avoiding multiple asserts on different concerns
```cs
[Test]
public void CheckVariousSumResultsIgnoringHigherThan1001()
{
    Assert.AreEqual(3, Sum(1001,1,2));
    Assert.AreEqual(3, Sum (1,1001,2));
    Assert.AreEqual(3, Sum (1,2,1001);
}
```

There is more than one test in this test method. The problems is that once an assert clause fails (throws an exception), no other line executes in the test method.

Sometimes people find bugs that they think are real, but when they "fix" them, the assert that previously failed passes and the other asserts in that test fail (or continue to fail). Sometimes you can't see the full problem, so fixing part of it can introduce new bugs into the system, which will only be discovered after you've uncovered each assert's result.

It's important that all the asserts have a chance to run in the case of multiple concerns, even if other asserts have failed before. In most cases, that means putting single asserts in tests.

There are several ways to achieve the same goal:
- Create a separate test for each assert.
- Use parameterized tests.
- Wrap the assert call with try-catch.

### Testing multiple aspects of the same object in one test

Instead of doing multiple assert over the same object:
```cs
[Test]
public void Analyze_SimpleStringLine_UsesDefaulTabDelimiterToParseFields()
{
    LogAnalyzer log = new LogAnalyzer();

    AnalyzedOutput output = log.Analyze("10:05\tOpen\tRoy");
    
    Assert.AreEqual(1,output.LineCount);
    Assert.AreEqual("10:05",output.GetLine(1)[0]);
    Assert.AreEqual("Open",output.GetLine(1)[1]);
    Assert.AreEqual("Roy",output.GetLine(1)[2]);
}
```
It's better to setup the expected object and compare the expected with the actual (it could require to override Equals method):

```cs
[Test]
public void Analyze_SimpleStringLine_UsesDefaulTabDelimiterToParseFields2()
{
    LogAnalyzer log = new LogAnalyzer();
    AnalyzedOutput expected = new AnalyzedOutput();
    expected.AddLine("10:05", "Open", "Roy");

    AnalyzedOutput output = log.Analyze("10:05\tOpen\tRoy");

    Assert.AreEqual(expected,output);
}
```

### Avoiding overspecification
An overspecified test is one that contains assumptions about how a specific unit under
test (production code) should implement its internal behavior, instead of only checking
that the end behavior is correct.

Here are ways unit tests are often overspecified:
- A test asserts purely internal state in an object under test.
- A test uses multiple mocks.
- A test uses stubs also as mocks.
- A test assumes specific order or exact string matches when it isn't required.

## Writing readable tests
Without readability the tests you write are almost meaningless. Readability is the connecting
thread between the person who wrote the test and the poor soul who has to
read it a few months later. Tests are stories you tell the next generation of programmers
on a project. They allow a developer to see exactly what an application is made
of and where it started.

### Naming unit tests
Naming standards are important because they give you comfortable rules and templates
that outline what you should explain about the test. The test name has three parts:

- **The name of the method being tested:** This is essential, so that you can easily see where the tested logic is. Having this as the first part of the test name allows easy navigation and as-you-type intellisense (if your IDE supports it) in the test class.
- **The scenario under which it's being tested:** This part gives you the "with" part of the name: "When I call method X with a null value, then it should do Y."
- **The expected behavior when the scenario is invoked:** This part specifies in plain English what the method should do or return, or how it should behave, based on the current scenario: "When I call method X with a null value, then it should do Y."

```cs
[Test]
public void AnalyzeFile_FileWith3LinesAndFileProvider_ReadsFileUsingProvider()
{

}
```

### Naming variables
Apart from their chief function of testing, tests also serve as a form of documentation for an API. By giving variables good names, you can make sure that people reading your tests understand what you're trying to prove as quickly as possible.

```cs
[Test]
public void BadlyNamedTest()
{
    LogAnalyzer log = new LogAnalyzer();

    int result= log.GetLineCount("abc.txt");

    Assert.AreEqual(-100,result);
}
```

Is -100 some sort of exception? Is it a valid return value? This is a more readable version of the test:

```cs
[Test]
public void BadlyNamedTest()
{
    LogAnalyzer log = new LogAnalyzer();

    int result= log.GetLineCount("abc.txt");

    const int COULD_NOT_READ_FILE = -100;
    
    Assert.AreEqual(COULD_NOT_READ_FILE,result);
}
```

### Asserting yourself with meaning

There are several key points to remember when writing a message for an assert clause:
- Don't repeat what the built-in test framework outputs to the console.
- Don't repeat what the test name explains.
- If you don't have anything good to say, don't say anything.
- Write what should have happened or what failed to happen, and possibly mention when it should have happened.

### Separating asserts from actions
For the sake of readability, avoid writing the assert line and the method call in the same statement.

```cs
[Test]
public void EasyToReadTest()
{
    const int COULD_NOT_READ_FILE = -100;

    LogAnalyzer log = new LogAnalyzer();

    int result= log.GetLineCount("abc.txt");
    
    Assert.AreEqual(COULD_NOT_READ_FILE, result);
}

[Test]
public void DifficultToReadTest()
{
    const int COULD_NOT_READ_FILE = -100;

    LogAnalyzer log = new LogAnalyzer();
    
    Assert.AreEqual(COULD_NOT_READ_FILE, log.GetLineCount("abc.txt"));
}
```

### Seting up and tearing down
If you have mocks and stubs being set up in a setup method, that means they don't get set up in the actual test. That, in turn, means that whoever is reading your test may not even realize that there are mock objects in use or what the expectations from them are in the test.

It's much more readable to initialize mock objects directly in the test itself, with all their expectations. If you're worried about readability, you can refactor the creation of the mocks into a helper method, which each test calls. That way, whoever is reading the test will know exactly what's being set up instead of having to look in multiple places.