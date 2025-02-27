---
title: How to write a SapphireTest
summary: Learn the basics of unit testing in Silverstripe
---

# How to write a sapphireTest

Here is an example of a test which extends [SapphireTest](api:SilverStripe\Dev\SapphireTest) to test the URL generation of the page. It also showcases
how you can load default records into the test database.

```php
// app/tests/PageTest.php
namespace App\Test;

use Page;
use SilverStripe\Dev\SapphireTest;

class PageTest extends SapphireTest
{
    /**
     * Defines the fixture file to use for this test class
     * @var string $fixture_file
     */
    protected static $fixture_file = 'SiteTreeTest.yml';

    /**
     * Test generation of the URLSegment values.
     *
     * Makes sure to:
     *  - Turn things into lowercase-hyphen-format
     *  - Generates from Title by default, unless URLSegment is explicitly set
     *  - Resolves duplicates by appending a number
     */
    public function testURLGeneration()
    {
        $expectedURLs = [
            'home' => 'home',
            'staff' => 'my-staff',
            'about' => 'about-us',
            'staffduplicate' => 'my-staff-2',
        ];

        foreach ($expectedURLs as $fixture => $urlSegment) {
            $obj = $this->objFromFixture(Page::class, $fixture);

            $this->assertEquals($urlSegment, $obj->URLSegment);
        }
    }
}
```

```yml
# app/tests/SiteTreeTest.yml
Page:
    home:
      Title: Home
    staff:
      Title: My Staff
    about:
      Title: About Us
    staffduplicate:
      Title: My Staff
```

```xml
<!-- phpunit.xml -->
<phpunit bootstrap="vendor/silverstripe/framework/tests/bootstrap.php" colors="true">
    <testsuites>
        <testsuite name="Default">
            <directory>app/tests/</directory>
        </testsuite>
    </testsuites>
    <filter>
        <whitelist addUncoveredFilesFromWhitelist="true">
            <directory suffix=".php">app/src</directory>
            <exclude>
                <directory suffix=".php">app/tests/</directory>
            </exclude>
        </whitelist>
    </filter>
</phpunit>
```

Firstly we define a static `$fixture_file`, this should point to a file that represents the data we want to test,
represented as a YAML [Fixture](../fixtures). When our test is run, the data from this file will be loaded into a test
database and discarded at the end of the test.

[notice]
The `fixture_file` property can be path to a file, or an array of strings pointing to many files. The path must be
absolute from your project root folder.
[/notice]

The second part of our class is the `testURLGeneration` method. This method is our test. When the test is executed,
methods prefixed with the word `test` will be run.

[notice]
The test database is rebuilt every time one of these methods is run.
[/notice]

Inside our test method is the `objFromFixture` method that will generate an object for us based on data from our fixture
file. To identify to the object, we provide a class name and an identifier. The identifier is specified in the YAML file
but not saved in the database anywhere, `objFromFixture` looks the [DataObject](api:SilverStripe\ORM\DataObject) up in memory rather than using the
database. This means that you can use it to test the functions responsible for looking up content in the database.

The final part of our test is an assertion command, `assertEquals`. An assertion command allows us to test for something
in our test methods (in this case we are testing if two values are equal). A test method can have more than one
assertion command, and if any one of these assertions fail, so will the test method.

The example **phpunit.xml** file should be placed in the root folder of your project. PHPUnit 9 should be included by default, as a dev dependency, in the **composer.json** file.

## Caching

Just like on web requests, Silverstripe CMS caches metadata about the execution context. This cache can get stale, e.g. when you change YAML configuration or add certain types of PHP code. In order to flush the cache, the **first time** this test is run use the `flush=1` CLI parameter:

```bash
vendor/bin/phpunit app/tests/PageTest.php '' flush=1
```

[info]
For more information on PHPUnit's assertions see the [PHPUnit manual](https://docs.phpunit.de/en/9.6/assertions.html).
[/info]

## Related documentation

- [Unit Testing](../unit_testing)
- [Fixtures](../fixtures)

## API documentation

- [SapphireTest](api:SilverStripe\Dev\SapphireTest)
- [FunctionalTest](api:SilverStripe\Dev\FunctionalTest)
