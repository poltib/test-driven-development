## TDD

![Smaller icon](http://upload.wikimedia.org/wikipedia/en/9/9c/Test-driven_development.PNG "TDD cycle")

Test-driven development (TDD) is a software development process that relies on the repetition of a very short development cycle: 

1. first the developer writes an (initially failing) automated test case that defines a desired improvement or new function.
2. Then produces the minimum amount of code to pass that test
3. Finally refactors the new code to acceptable standards. 

Kent Beck, who is credited with having developed or 'rediscovered' the technique, stated in 2003 that TDD encourages simple designs and inspires confidence.
<http://en.wikipedia.org/wiki/Test-driven_development>

### Profits

-   Force yourself to think before coding: it actually improve the quality of the code.
-   Testability improve architecture. You need to keep that question in mind *How might I test this?* Tests encourage structure by making you design before coding.
-   It's a free documentation like an explanatory list of features provided by the system. If every tests' names are properly chosen you simply need to read it to understand the functionality of a class.

### BDD?

Behavior-driven developpment

TDD is code-centric while BDD is communication-centric.


## Testing in Laravel

### Write a testable code

(synthesis of *Testing Decoded* Ch1-6 Signs of Untestable Code)

#### 1. Use dependency injection instead of 'new' operator
Not testable

    public function getSponsorshipMention($data)
    {
        //we can't test this!
        $sponsorshipservice = new PlaceSponsorShipService;
        
        if ($sponsor = $sponsorshipservice->getSponsor($data['id'])) {
        
            return $sponsorshipservice->getMention($sponsor, $data);
        }
    }

Testable (because of the injection throught the constructor)

    protected $file;
    
    public function __construct(PlaceSponsorShipService $sponsorshipservice)
    {
        $this->sponsorshipservice = $sponsorshipservice;
    }
    
    public function getSponsorshipMention($data)
    {
        if ($sponsor = $this->sponsorshipService->getSponsor($data['id'])) {

            return $this->sponsorshipService->getMention($sponsor, $data);
        }
    }

#### 2. Constructor assign dependencies, nothing else

Sometimes (in 1 or 2% of cases) it may be different. 

-   When a constructor also calls the parent constructor.
-   Or when it calls a method that initializes internal stuff in the object, among other matters of syntax. See for example the DOM crawler Symfony (\ Symfony \ Component \ DomCrawler \ Crawler).

#### 3. Don't forget *The Single Responsibility Principle*

When you write a methode if you should describe what it does and say *and, and, and...* you should refactor it.

*In object-oriented programming, the single responsibility principle states that every class should have a single responsibility, and that responsibility should be entirely encapsulated by the class. All its services should be narrowly aligned with that responsibility.*

In others words, a class shouldn't need to be updated in response to multiple, unrelated changes to your application, such as modifying business logic, or how output is formatted,...

[source](http://en.wikipedia.org/wiki/Single_responsibility_principle) first principle of SOLID 

SOLID: <http://en.wikipedia.org/wiki/SOLID_(object-oriented_design)> 

<https://laracasts.com/tags/principles>

#### 4. If your class have to many paths use polymorphism

Polymorphism refers to the act of breaking a complex class into sub-classes which share a common interface, but can have unique functionality. ([example](http://code.tutsplus.com/tutorials/understanding-and-applying-polymorphism-in-php--net-14362))

#### 5. Too many dependencies

Sometimes if a class requires too many  dependencies, maybe it's a sign that this class is asking for too much.

#### 6. Too many bugs

If you notice that you often need to debug a specific class, you should refactor it.

#### TTC

The fact that we inject the database layer into the controller through the constructor make them testable.

### Mocks

A mock objet do nothing more than simulating the behavior of real objects.

Therefore during your tests when you don't want to execute a methode but just ensure that it was called; you mock it.

#### Mocking facades

Laravel's facades are extremely simple to test. Because every Laravel facades extends a parent Facade class that offers, among other things, a shouldReceive method. When called, Laravel will automatically swap out the registered instance with a mock, using Mockery.

-   Note: You should not mock the `Request` facade. Instead, pass the input you desire into the `call` method when running your test.

example:

    // app/routes.php
    
    Route::get('foo', function()
    {
        File::put(__DIR__./file.txt', 'Lorem ipsum');
    });
And the test file

    <?php //app/tests/Footest.php
    
    use \Mockery as m;
    
    class FooTest extends TestCase {
    
        public function tearDown()
        {
            m::close;
        }
        
        public function testCreatesFile()
        {
            File::shouldReceive('put')->once();
            
            $this->call('GET', 'foo');
        }
    }

Explanation:

-    Every test class is named after the class under test and must end with ‘Test’. Example: FooBar -> FooBarTest.
-   Every tests class extends the TestCase class already present in the tests folder.
-   When you use mocks don't forget to call Mockery::close() inside tearDown method in your test file. This call will verify your expectations.

### Test

#### Unit testing

Unit Tests are written from a programmers perspective. They are made to ensure that a particular method (or a unit) of a class performs a set of specific tasks.

By default Laravel use [PhpUnit](http://phpunit.de/) but we could decide to select an other framework like [PhpSpec](http://www.phpspec.net/) which is more focused on communication. [intro](https://laracasts.com/lessons/phpspec-is-so-good)

Every classes should be tested in isolation. (using mocks for example).

Testing one object, and one only (if your test fails, you know exactly where to look).

Test files are located at app/tests directory and your test structure should mirror your application structure.

ex: `app/tests/Foo/Controllers/Partners/LocationControllerTest.php`

It isn't an obligation some people structure their tests according to the test type eg: 

-   `app/tests/unit` 
-   `app/tests/acceptance`
-   `app/tests/integration`

Preparing a unit test can be reduced to three simple actions:

1. Arrange: Set the stage, instantiate objects, pass in mocks.
2. Act: Execute the *thing* that you wish to test.
3. Assert: Compare your expectes output to what was returned.

dummy example phpunit

    public function testFetchesItemsInArrayUntilKey()
    {
        //Arrange
        $names = ['Taylor', 'Dayle', 'Matthew'];
        
        //Act
        $result = array_until('Matthew', $names);
        
        //Assert
        $expected = ['Taylor', 'Dayle'];
        $this->assertEquals($expected, $result);
    }

dummy example phpspec   
    
    function it_fetches_items_in_array_until_key()
    {
        $names = ['Taylor', 'Dayle', 'Matthew'];
        
        $this->array_until('Matthew', $names)->shouldBe(['Taylor', 'Dayle']);
    }
    
##### PhpSpec 

PhpSpec is really interesting and helpful during TDD (BDD). For example when you begin by writting your tests you have a failure. When this failure comes PhpSpec propose you to create the missing class for you.

This is not a huge advantage but using it can boost your work.

Unfortunately the documentation is light and to fully understand it you need to practise and make some tests.



#### Functional (controllers) testing

Like unit testing, functional ensure that the code does what you expect. But functional can trigger multiple piece of your application.

edit: Usually controllers are tested with [Behat](http://behat.org/) or [Codeception](http://codeception.com/).

-> [Behat good introduction](https://tutsplus.com/tutorial/behat-for-the-rest-of-us/)


    <?php

    use \Mockery as m;
    use Way\Tests\Factory;

    class MapControllerTest extends TestCase
    {

        public $mock;

        public function setUp()
        {
            parent::setUp();
            
            $this->place = $this->mock('Foo\Repositories\Place\PlaceRepositoryInterface');
        }

        public function mock($class)
        {
            $mock = m::mock($class);

            $this->app->instance($class, $mock);

            return $mock;
        }

        function tearDown()
        {
            m::close();
        }
    
        /**
         * Check that the controller can return details by place id.
        */
        public function testGetPlaceDetailsById()
        {
            //Arrange
            $place_details = Factory::make('Foo\Repositories\Place\Place', ['id'=>1]);

            $this->place
                 ->shouldReceive('findPlaceDetailsById')
                ->once()
                ->andReturn($place_details);
        
            //Act
            $response = $this->action('GET','MapController@getPlaceDetailsById', ['placeId' => 1]);
        
            //Assert
            $this->assertResponseOk();
        }
    }

#### Integration testing

These tests will flex multiple parts of your application, and typically won't rely on mocks or stubs. As such, be sure to create a special test database.

In other words the purpose is to verify if your unit tests work when grouped together.

When testing repositories, you can sometimes fall into that trap of writing a test that basically reproduces exactly what your repository method is going to do. In those cases, consider integration tests against a DB in memory instead.

example of functional testing whit codeception (registration)

    <?php
    
    $I = new TestGuy($scenario);
    $I->wantTo('register for a new account');
    $I->lookForwardTo('be a member');
    
    $I->amOnPage('/register');
    $I->see('Register', 'h1');
    $I->fillField('Email:', 'foo@sample.com');
    $I->fillField('Password:', '1324');
    $I->click('Register Now');
    
    $I->seeCurrentUrlEquals('/login');
    $I->see('You may now sign in', '.flash');
    $I-seeInDatabase('users', ['email' => 'foo@sample.com']);

#### Acceptance testing

Acceptance Tests are written from the user's perspective. They ensure that the system is functioning as users are expecting it to.

Greate package to do acceptance tests in Laravel is [Codeception](http://codeception.com/).

Codeception provides browser emulation powered by [Mink](http://mink.behat.org/).

Example of acceptance tests:


1. Test that check the home page and verify if it's possible to go on /map page 
        
        <?php

        $I = new WebGuy($scenario);
        $I->wantTo('Check the home page and go to map page');
        $I->amOnPage('/');
        $I->see('Foo', 'h1');
        $I->click('#locate_btn');
        $I->amOnPage('/map');

2. Test that check if it's possible to log in in partner site.

        <?php

        $I = new WebGuy($scenario);
        $I->wantTo('Login to partner area');

        $I->amOnPage('/partners');
        $I->seeCurrentUrlEquals('/partners/login');

        $I->fillField('email', 'poltib@plop.com');
        $I->fillField('password', 'plop');
        $I->click('Log in');

        $I->seeCurrentUrlEquals('/partners');
        $I->see('My Locations', 'h1');

### Continuous Integration

After every merges/commit we can chose an external service (eg: [Travis](http://docs.travis-ci.com)) that rebuilt the application on their server (in an environment that's as close to production as possible) and run all tests and send us the results.


#### Notes

#### Xdebug

We have it but we don't use it.

#### Bugsnag

[Bugsnag](https://bugsnag.com) is a service that send notifications when you website crash.

#### Phpstorm testing

-> [PhpStorm test workflow](https://laracasts.com/lessons/phpstorm-testing-workflow)

#### Tips and thoughts

-   Test tips
    -   Don't test more than one file per test file
    -   Don't test more than one function per test method
    -   If I have more than 3 or 4 assertions per test, I should probably break it into two tests
    -   Write multiple tests for functions with complex input parameters
    -   Write a separate test for success/failure (validation, if statment, etc)

-   Factory

    If you use factories: `use Way\Tests\Factory;` just add `"doctrine/dbal": "2.4.*"` to your composer.json file, and run `composer update`.
    
    
I only read testing stuffs for two weeks and I think that I still have some way to become confortable with this subject.

##### Team 

Some people do acceptance test for Controlers/Routes/views and unit test for Models/

For now we need to concert ourselves to decide what we want to test and how. 

Our main goal is to have our code looking like it was written by a single person. Moreover, the list of features it provides should be understandable just by looking at the test files.

## Links / Ressources

-   Testing decoded (Jeffrey Way)

-   [Video of TDD explanation by Jeffrey Way](http://code.tutsplus.com/tutorials/test-driven-development-in-php-first-steps--net-25796)

-   [PhpUnit doc](http://phpunit.de/manual/3.7/en/index.html"PhpUnit doc")

-   [PhpSpec doc](http://www.phpspec.net/docs/introduction.html )

-   [Behat screencast](https://tutsplus.com/tutorial/behat-for-the-rest-of-us/)

-   [Laravel-test-helpers](https://github.com/JeffreyWay/Laravel-Test-Helpers"Laravel-test-helpers doc")

-   [Testing controllers](http://code.tutsplus.com/tutorials/testing-laravel-controllers--net-31456"Testing controllers tuto")

-   [Testing models](http://code.tutsplus.com/tutorials/testing-like-a-boss-in-laravel-models--net-30087)

-   [Structure testable Controllers](http://culttt.com/2013/07/15/how-to-structure-testable-controllers-in-laravel-4/)

-   [Laravel doc:test](http://laravel.com/docs/testing"Laravel doc test")

-   [Mockery doc](https://github.com/padraic/mockery"Mockery doc")

-   [AspectMock mocking framework](https://github.com/Codeception/AspectMock)

-   [Mockery explanation](http://culttt.com/2013/07/22/getting-started-with-mockery/)

-   [Mock/Stubs](http://martinfowler.com/articles/mocksArentStubs.html"Mock/Stubs") 
    
    Difference between Mocks and Stubs. (Examples are in Java, but the principles make sense with any object-oriented language.)

-   [Mock/Stubs/fakes](http://stackoverflow.com/questions/346372/whats-the-difference-between-faking-mocking-and-stubbing?rq=1)

-   [Explanation of IoC container and unit testing by Taylor Otwell](http://taylorotwell.com/full-ioc-unit-testing-with-laravel/")

-   [Explanation of IoC container by Dayle Rees](https://github.com/daylerees/inversion-of-control-container-example") 
    
-   [Test driven development example by Dayle Rees](https://github.com/daylerees/test-driven-development-example)

    All explanations are in the tests files.

-   [Better testing in Laravel by Jeffrey Way](http://www.youtube.com/watch?v=ajoFwWwSHTI)

-   [Laravel framework tests](https://github.com/laravel/framework/tree/master/tests)

-   [The calculator example step by step](https://github.com/poltib/calc)