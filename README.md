# .NET Testing Introduction & Guidelines 

This documentation is intended to give an introduction to writing tests for .NET, offer solutions to common problems when writing tests, and to outline some best practices to follow. 
Writing tests for your code is far more than just proving your code is functionally correct - testing provides many benefits.


## Why Write Tests?

Before we go into more detail, the benefits of testing can be summarised as:

* Code correctness
* Code confidence
* Cut down debugging time
* Raises awareness of complex areas of code
* Measuring code performance


### Code correctness

When thinking of writing tests for code, most people would consider tests to solely be a check for code correctness. While it is true that good tests can confirm that logic is correct, it is far from the most beneficial aspect of code testing. Regardless, correctness through tests is still a great reason to be writing tests for your code, but must not be depended on blindly - you can't assume you're testing all scenarios and that your tests themselves are correct. 
Careful thought before writing code and reviewing written code is still very important, with or without tests!


### Code confidence

After making a change within a large code-base, it is difficult without tests to be aware of any possible side effects of the change. Tests can help with this by confirming that code written across the entire code-base is still functioning identically to as before the change. 
This ability to quickly check that a change hasn't broken separated sections of code is extremely beneficial, especially when refactoring code, allowing you to ship changes with confidence.


### Reducing debugging time

If you're writing tests as you write your code (which you should be), you will find that the time spent writing tests can often be far less than time you would otherwise spend fixing bugs. 
If you're confident that your test is expecting a certain result, but the code that it was written to test is not returning the expected result, you can fix the code as you're writing it - not later down the line when it is identified as a bug, or worse as a support ticket!


### Raising awareness of complex code

Complex code is something that creeps into any project, whether we're aware we're writing complex code or not. Writing small, specific tests forces us to test only small, simple sections of code at a time. 
This granular approach to testing highlights areas of complex code, as it is extremely difficult to write a small, specific test for a large, complex section of code.
When we run into a scenario like this, it's a signal that we should consider refactoring the complex code into easier to manage, smaller chunks of code, reducing complexity and improving code quality.


### Measuring code performance

When we run any tests we've written, and it consumes code that is slow, it is a very clear identifier that the code is inefficient. This helps us to write faster code without having to benchmark. 
For the majority of unit tests we want them each to complete in approx. 25ms - any longer and you might want to look at your code! 
If we aim to keep each test as fast as possible, we'll keep the suite of tests running quickly (so we can run them often) and will also help speed up overall speed of our code.

 

In summary, writing tests not only confirms that our code is correct but it also reduces time spent debugging and helps us to write better, faster code. These concepts will be covered in more detail throughout this documentation along with examples.

# Writing Tests

Tests can be written and run on .NET code using MSTest & the Visual Studio test runner. Before we get into the syntax of writing tests, we will first define the two types of tests.

### Unit Tests

A *unit* test is a small, granular test designed to test the functionality of a single 'unit' of code (hence the name). 
A very important characteristic of a unit test is that it does not depend on any other areas of code. 

For example, if you're testing that a `CustomerController`'s `ChangeName` method is correctly updating a customer's name, you don't want the unit test to be making database calls - this would expand the scope of the test beyond one 'unit' of code. 
Once your unit tests begin depending on external code, they're no longer unit tests! Depending on external code introduces complexity into your simple tests, and reliance on external factors to pass, such as a valid database or internet connection.

### Integration Tests

While unit tests are small, they don't interact with the application in a larger scope. This is what *integration tests* are for. As a simple example, an integration test might ensure that the database is configured and connected correctly, by adding a record to the Customers table by using the `CustomerRepository`. 

While unit tests are testing the *functionality* of individual code blocks, integration tests are testing the *dependencies* between the different blocks and areas of code.

# Starting Testing

Beginning to write tests is easy! 

It's recommend to split your tests into a test project for your solution. For example, if you have `WebApp.Api` and `WebApp.Data`, you could create `WebApp.Test` too. 

Within your Test project, it is good practice to create two directories - *Unit* and *Integration* - a folder for both types of test. 

Going deeper into each directory, it is good practice to mirror the structure of the code you're testing. For example, adding a *Api* and *Data* directory - to break apart tests for different areas of code - is a good idea. From here on, you can add your test classes.

For example, as your *WebApp.Api* project might contain a `CustomerController`, you would want to add a test class `WebApp.Test/Unit/Api/CustomerController.cs` and add a reference to the project that you're going to be testing. 

Note that we're not adding 'Test' or 'Tests' to our test class name - this is important and will be covered later!

# Creating a Test File

Once you've added your test class (such as `CustomerController.cs`), we need to inform the Visual Studio test runner that the class is indeed a test class. We do this with the `[TestClass]` attribute on the class definition.

	namespace WebApp.Test.Unit.Api
	{
		[TestClass]
		public class CustomerController
		{
			...
		}
	}

This informs Visual Studio that this class includes *test methods*, defined by the `[TestMethod]` attribute.

	[TestMethod]
	public void OurFirstTest()
	{
		...
	}

Once we have our test method, we need to decide what we're going to test. As an example, we're going to test the `Calculator` class and it's very basic `Add` method:

	public class Caculator
	{
		public int Add(int numberOne, int numberTwo)
		{
			return numberOne + numberTwo;
		}
	}

In our test class, `Calculator.cs`, we have a method `Add_ShouldSumNumbersCorrectly()` where we evaluate weather our calculator is correct.

	[TestMethod]
	public void Add_ShouldSumNumbersCorrectly()
	{
		// Arrange
		int numberOne = 10;
		int numberTwo = 20;
		Calculator calc = new Calculator();

		// Act
		var sum = calc.Add(numberOne, numberTwo);

		// Assert
		Assert.AreEqual((numberOne + numberTwo), sum);
	}

What we've done in the test method is follow the AAA format of writing tests. 

It forces us to follow a structure to writing our tests, and helps us keep them small, clean, and easy to read. 

AAA means *Arrange, Act, Assert*, and these are the three steps you want every test to take.

**Arrange** is the step where we prepare the stage - we instantiate any objects or classes we need to carry out the test.

**Act** is where we carry out the action that we're testing. Ideally this should be one line. If it's overly complex to Act on your test, it's an indication that your code is complex and you need to refactor it!

Finally, **Assert** is where we carry out the actual evaluation of our result. Here we're using `Assert.AreEqual`, which will compare two input values and either pass or fail the test depending on the outcome. 

The test will be inconclusive without an Assert, so you'll always have at least one. 

Ideally each test can usually only include one assert statement, but don't be afraid to assert multiple times if the assertions are related - if you find you're asserting different functionality in a single test, it's an indication you need to split your test into multiple.

It's up to you whether you want to include the three AAA comments in your test code, however it's recommended for both readability and structure.

`.AreEqual` isn't the only assert in your testing toolbox. A few more are `.AreNotEqual(...)`,
`.IsNull(...)` and `.IsTrue(...)`.


# Keeping our unit tests as unit tests


When writing unit tests, it's easy to start instantiating objects and adding dependencies. For example, our `CustomerController` might include a `CustomerService`, injected into the constructor. 

While the `CustomerService` is required for our application to work, we don't want to load it into our test method as this would be bringing in code from outside our *unit* of code we're testing - our unit test would no longer be a unit test! 

While this seems like a complicated problem to solve, we can using a practice called mocking. 
Mocking involves creating a basic, *fake* version of the dependencies, and using those, allowing us to only test the code within our unit, removing the risk of external code causing our unit test to fail. 

For .NET, we recommend you use *Moq*, available as a nuget package.

Here's an example of using Moq.

	using Moq;

	namespace WebApp.Test.Unit
	{
		[TestClass]
		public class CustomerController
		{
			[TestMethod]
			public void ChangeCustomerName_ShouldUpdateCustomerName()
			{
				// Arrange
				var customerService = new Mock<CustomerService>();
				var controller = new CustomerController(customerService);
				
				var customer = new Customer { Name = "Barry" };
				var newName = "Harry";

				// Act
				controller.ChangeName(customer, newName);

				// Assert
				Assert.AreEqual(customer.Name, newName);
			}
		}
	}

In the above snippet, we're creating a mock class of `CustomerService` and passing it to the constructor of `CustomerController` (where it would *normally* be passed through dependency injection). This allows us to test our controller without breaking the rules of unit testing.

If you find that you're using a mock of a class across multiple tests within a single test class, you can add it to the `TestSetup()` method - which is called before the tests are ran:

	namespace WebApp.Test.Unit
	{
		[TestClass]
		public class CustomerController
		{
			private CustomerService customerService;

			[TestInitalize]
			public void TestSetup()
			{
				customerService = new Mock<CustomerService>();
			}

			[TestMethod]
			public void ChangeCustomerName_ShouldUpdateCustomerName()
			{
				// Arrange
				var controller = new CustomerController(customerService);
				
				var customer = new Customer { Name = "Barry" };
				var newName = "Harry";

				// Act
				controller.ChangeName(customer, newName);

				// Assert
				Assert.AreEqual(customer.Name, newName);
			}
		}
	}

This logic can be applied to almost any class - even an HttpClient! 

In the next example, we're again mocking an object but we're also calling `.Setup()` on the mock, defining what the class's method should do when called. 

When we call the .Get() method on our HttpClient in our test code, it will return the string "Hello, World!".

	[TestClass]
	public class CustomerController
	{
		private HttpClient http;

		[TestInitalize]
		public void TestSetup()
		{
			http = new Mock<HttpClient>();
			http.Setup(obj => obj.Get).Return("Hello, World!");	
		}
	}

In a similar way, we don't want to be using real EF database contexts either - imagine running tests that insert data into a database on a production database - Oops! 

Luckily EF offers the ability to instantiate a new, in-memory database context for your integration tests. 

Be aware that this is still spinning up a database and is such external code, so you'll only want to use this for integration tests - not your unit tests - but is still incredibly useful!

	[TestClass]
	public class CustomerController
	{
		private DbContext context;

		[TestInitalize]
		public void TestSetup()
		{
			var dbOptions = new DbContextBuilderOptions<DbContext>()
						.UseInMemoryDatabase(Guid)
						.Options;
			context = new DbContext(dbOptions);
		}
	}

## Testing Controllers

Writing unit tests for controllers is fairly simple. The only thing to pay attention to is the external calls - make sure to mock! Once you've created your mock, you can simply call it's methods as you would through a HTTP request.

	namespace WebApp.Test.Unit
	{
		[TestClass]
		public class CustomerController
		{
			// We're expecting Customer ID 1 to have the name 'Harry'.
			[TestMethod]
			public void CallViewCustomerById_ViewModelShouldHaveCorrectNameProperty()
			{
				// Arrange
				var customerService = new Mock<CustomerService>();
				var controller = new CustomerController(customerService);

				// Act
				var request = controller.ViewCustomer(customerId: 1);
				var model = request.Model;

				// Assert
				Assert.AreEqual(model.CustomerName, "Harry");
			}
		}
	}

## Naming your tests

When it comes to writing your unit tests, naming the test methods is very important. We need to take an opposite approach to writing normal .NET code - longer names are better!

In the above snippet the test was called `CallViewCustomerById_ViewModelShouldHaveCorrectNameProperty`. The name clearly outlines the functionality we're testing and the result we expect - when we see the test fail in the test runner it makes it a lot easier to find the affected code. A simple formula to follow for naming your tests is:

> What -> With -> Then

For example, `CallGetCustomerById_WithId_ShouldReturnCustomerWithId1`.

We mentioned earlier that we shouldn't add 'Test' or 'Tests' to the end of our test classes - this is to create a sentence-like structure to our class/method names.

For example when combined with the class, the above method name reads fluidly:
`CustomerController/CallGetCustomerById_WithId_ShouldReturnCustomerWithId1`. Adding 'Tests' to the class name would break that up and make it less fluent.

## Further tips to write better tests

Once you've got the basics, you're good to go! But there are some best practices and tips here to help you write better tests from the go.

* Avoid splitting test logic out into other classes or base classes. You might think it would be clever to create a base test class with mocking helpers, but that's spreading your test logic across other files and makes it harder to write and read your tests. Keep all your related test code in one file/location.


* If a test is difficult to setup, consider it an indicator that your code needs work. Tests provide *a lot* of feedback on code quality and complexity. Listen to your tests! If you find you're struggling to build a test, refactor! Don't just write a big, complex test - they're brittle.


* Treat your tests as first-class code. Keep it clean, decoupled and maintained. Your tests are important and an invaluable addition to your coding toolkit, don't let them get rusty.


* Making your code work is only half the work. Code needs to be efficient, readable, decoupled and maintainable. With the introduction of tests, you work towards meeting all of these requirements.


* If you're expecting an exception to be thrown from your code, you can add an attribute to the test method and call the throwing code. For example, `[ExpectedException(typeof(InvalidOperationException))]`.


* When we're deciding what code we need to test, it's important to look at how functional the code is. It's unnecessary to test a DTO/ POCO, but if we have a validation rule or an object builder, we might want to test that! There's no need to aim for 100% test coverage - a large portion of those tests would most likely be redundant. Testing key areas of code is enough - aiming for 70% is more than great!

# Test Driven Development (TDD)

Now that you're a master at writing good, clean tests for .NET, and you're well aware of all the incredible reasons to start writing tests, you're ready to begin TDD.

TDD is a methodology that flips the standard flow of writing code (write THEN test) around, so we write our tests first and THEN our code!

For example, if you're going to be writing a method to strip any invalid characters from an input string (to be used for cleaning input data), you would first define the test to test that method, such as:

		[TestMethod]
		public void StringCleanInput_PassInInvalidCharacters_AreSrippedFromString()
		{
			// Arrange
			var inputString = "This_£$_String_Contains_&*_Invalid_Characters";
			var expectedOutput = "This__String_Contains__Invalid_Characters";
			
			// Act
			var result = inputString.CleanInput(); // This won't compile - yet!
			
			// Assert
			Assert.AreEqual(result, expectedOutput);
		}

"But Chris, this test is testing code that doesn't even exist yet!"

Correct, the code won't compile, but already we've started to think about *what* the code will do, and *how* we intend to consume the functionality - without writing any code!
This non-compiling/ non-passing state of the test is called the 'Red' stage.

Now that we're in the red, we want to get ourselves into the green zone - make the test pass! As we've already thought about our code when we wrote our test, we know what we want to write - a string extension method:

	public static string CleanInput(this string input)
	{
		return input.Replace("$", "").Replace("£", "").Replace("&", "").Replace("*", "");
	}
	
Now when we run our test - the code passes! Hurray. But the code we wrote isn't that great. It only replaces 4 invalid characters - what if we don't want a "^" in our string? 

It would be inefficient to keep adding `.Replace()` calls to our code. This is where we enter the Blue stage - refactoring our code. Since we already have our test written and proving the the code is working, we can refactor and change our extension method as much as we like, cleaning it up and making it faster, and still be confident that our changes aren't breaking the functionality! 

Once we're happy with our test + refactored code, we can go back to the red stage and continue writing more tests and more code. TDD is a cycle - RED, GREEN, BLUE.

In summary, TDD helps us write cleaner code while also implementing tests at the same time. It provides us with confidence our code isn't breaking as we refactor, and it forces us to think how we're defining our code and how we want to consume it - before we even start to write it. Combine those benefits with the overall benefits of writing tests and you'd be mad not to do it! So go forth and test!
