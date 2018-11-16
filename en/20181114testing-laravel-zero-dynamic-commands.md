# Testing Laravel Zero dynamic commands

**Note**: most of the concepts exposed in here can also be applied in the Laravel framework.

[Laravel Zero](https://laravel-zero.com) is a Laravel-based framework for building command line applications. You can find applications built with it and resources to start with it in [here](https://github.com/laravel-zero/awesome-laravel-zero).

Testing commands can be a pretty straightforward task or a total headache. Let's see why.

## The problem

Laravel provides an easy way [to test your commands](https://laravel.com/docs/5.7/console-tests) as you can see in the following example extracted from the official documentation.

```php
public function test_console_command()
{
    $this->artisan('question')
         ->expectsQuestion('What is your name?', 'Taylor Otwell')
         ->expectsQuestion('Which language do you program in?', 'PHP')
         ->expectsOutput('Your name is Taylor Otwell and you program in PHP.')
         ->assertExitCode(0);
}
```

This is an efficient way to test when you are generating the command output by yourself, but what if you depend on some external source like a database or web service? You can't simply "expect an output" when you don't even know what the output will be.

## Fake it 'till you make it

So, what should we do? Simple, **fake it**. To demonstrate my approach we will build a simple weather cli using the [Meta Weather public API](https://www.metaweather.com/api/).

## Starting from the start

### Step 1

The first thing we need to do is create a new project.

`composer create-project --prefer-dist laravel-zero/laravel-zero weather-cli`

The second thing to do if we follow the [TDD](https://en.wikipedia.org/wiki/Test-driven_development) approach is to create a feature test to define our _walking skeleton_.

```php
namespace Tests\Feature;

use Tests\TestCase;

class WeatherCommandTest extends TestCase
{
    /** @test */
    public function it_can_get_the_weather()
    {
        $this->artisan('weather', ['city' => 'San Diego'])
             ->expectsOutput('Thunderstorm, Min temperature: 14.50, Max temperature: 24.80')
             ->assertExitCode(0);
    }
}

```

### Step 2

Perfect, now if we run your tests obviously we are going to have errors because the `weather` command **does not exist yet**, the following command will create it.

`php artisan weather-cli make:command WeatherCommand`

### Step 3

Now we have the command, before to start consuming the API we need an HTTP client, we could install an existing one like [Guzzle HTTP](http://docs.guzzlephp.org/en/stable/), but we will stick to [The KISS principle](https://en.wikipedia.org/wiki/KISS_principle) for this example, so we will define the most basic implementation for our needs.

We will define an interface and a concrete implementation for our HTTP client, the most simple implementation could look like this:

```php
namespace App\Http;

interface HttpClientInterface
{
    public function get(string $url): string;
}
```

```php
namespace App\Http;

class HttpClient implements HttpClientInterface
{
    public function get(string $url): string
    {
        $curl = curl_init();

        curl_setopt_array($curl, [
            CURLOPT_RETURNTRANSFER => 1,
            CURLOPT_URL => $url
        ]);

        return curl_exec($curl);
    }
}
```

Perfect, now we have to request data from two endpoints.

### Step 4

```
This endpoint will give us a woeid to represent the city.

https://www.metaweather.com/api/location/search/?query=our-city

This endpoint will give us the actual information about the weather in the city.

https://www.metaweather.com/api/location/our-woeid/
```

As the API will return the data in JSON format we will need to parse it, we can define a simple `JsonParser` like the following:

```php
namespace App\Parsers;

class JsonParser
{
    public static function parse(string $JSONString): array
    {
        return json_decode($JSONString, true);
    }
}
```

Now we have to define two [DTOs](https://en.wikipedia.org/wiki/Data_transfer_object) to store the response of the API calls, the first will be `CityInformation` and the second `Weather`, they will look like this.

```php
namespace App\DTO;

class CityInformation
{
    public $title;
    public $locationType;
    public $woeid;
    public $lattLong;

    public function __construct($title, $locationType, $woeid, $lattLong)
    {
        $this->title = $title;
        $this->locationType = $locationType;
        $this->woeid = $woeid;
        $this->lattLong = $lattLong;
    }
}
```

```php
namespace App\DTO;

class Weather
{
    public $stateName;
    public $minTemp;
    public $maxTemp;
    public $theTemp;

    public function __construct($stateName, $minTemp, $maxTemp, $theTemp)
    {
        $this->stateName = $stateName;
        $this->minTemp = $minTemp;
        $this->maxTemp = $maxTemp;
        $this->theTemp = $theTemp;
    }
}
```

### Step 5

Now we should bind our `HttpClient` to the service container, that way we can use it via [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) in our command. This is one of the most important steps in our path to test dynamic commands.

```php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->bind(
            'App\Http\HttpClientInterface',
            'App\Http\HttpClient'
        );
    }
}
```

### Step 6

Perfect, now (finally) we are ready to get back to our `WeatherCommand` and consume the API. This will be the implementation:

```php
namespace App\Commands;

use App\Http\HttpClientInterface;
use App\DTO\CityInformation;
use App\DTO\Weather;
use App\Parsers\JsonParser;
use Illuminate\Console\Scheduling\Schedule;
use LaravelZero\Framework\Commands\Command;

class WeatherCommand extends Command
{
    protected $signature = 'weather {city}';

    protected $description = 'Get the weather for a city';

    private $client;

    public function __construct(HttpClientInterface $client)
    {
        $this->client = $client;

        parent::__construct();
    }

    public function handle()
    {
        $cityResponse = $this->client->get(
            "https://www.metaweather.com/api/location/search/?query=".urlencode($this->argument('city'))
        );
        $cityData = JsonParser::parse($cityResponse);
        $cityInformation = new CityInformation(
            $cityData[0]['title'],
            $cityData[0]['location_type'],
            $cityData[0]['woeid'],
            $cityData[0]['latt_long']
        );

        $weatherResponse = $this->client->get("https://www.metaweather.com/api/location/{$cityInformation->woeid}/");
        $weatherData = JsonParser::parse($weatherResponse);
        $weather = new Weather(
            $weatherData['consolidated_weather'][0]['weather_state_name'],
            number_format($weatherData['consolidated_weather'][0]['min_temp'], 2),
            number_format($weatherData['consolidated_weather'][0]['max_temp'], 2),
            number_format($weatherData['consolidated_weather'][0]['the_temp'], 2)
        );

        $this->line(sprintf(
            '%s, Min temperature: %s, Max temperature: %s',
            $weather->stateName,
            $weather->minTemp,
            $weather->maxTemp
        ));
    }
}
```

At this point if you run the weather command it should work, for example:

```
php weather-cli weather "San Diego"

Output:
Light Cloud, Min temperature: 12.82, Max temperature: 20.78
```

But our test is still not working:

```
vendor/bin/phpunit

Output:
PHPUnit 7.4.3 by Sebastian Bergmann and contributors.

.F                                                                  2 / 2 (100%)

Time: 1.43 seconds, Memory: 12.00MB

There was 1 failure:

1) Tests\Feature\WeatherCommandTest::it_can_get_the_weather
Output "Thunderstorm, Min temperature: 14.50, Max temperature: 24.80" was not printed.

/home/max/projects/weather-cli/vendor/laravel-zero/foundation/src/Illuminate/Foundation/Testing/Concerns/InteractsWithConsole.php:51
/home/max/projects/weather-cli/vendor/laravel-zero/foundation/src/Illuminate/Foundation/Testing/TestCase.php:139

FAILURES!
Tests: 2, Assertions: 4, Failures: 1.

   PHPUnit\Framework\AssertionFailedError  : Output "Thunderstorm, Min temperature: 14.50, Max temperature: 24.80" was not printed.
```

### Step 7

Now we have to fake the implementation of our `HttpClient`, since we are binding this class via the [service container](https://laravel.com/docs/5.7/container) we have to create a [service provider](https://laravel.com/docs/5.7/providers) and bind it to the container while we're testing.

```php
namespace Tests\Fakes;

use App\Http\HttpClientInterface;
use App\Providers\AppServiceProvider;

class FakeServiceProvider extends AppServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register(): void
    {
        $this->app->bind('App\Http\HttpClientInterface', function () {
            return new class implements HttpClientInterface
            {
                public function get(string $url): string
                {
                    if ($url == 'https://www.metaweather.com/api/location/search/?query=San+Diego') {
                        return $this->getCityInformationData();
                    }

                    if ($url == 'https://www.metaweather.com/api/location/2487889/') {
                        return $this->getWeatherData();
                    }
                }

                private function getCityInformationData()
                {
                    return '[{"title":"San Diego","location_type":"City","woeid":2487889,"latt_long":"32.715691,-117.161720"}]';
                }

                private function getWeatherData()
                {
                    return '{"consolidated_weather":[{"id":4866121622618112,"weather_state_name":"Thunderstorm","weather_state_abbr":"lc","wind_direction_compass":"NE","created":"2018-11-14T12:08:35.108768Z","applicable_date":"2018-11-14","min_temp":14.50000000,"max_temp":24.80000,"the_temp":19.075,"wind_speed":5.2266084718735915,"wind_direction":49.77025406533666,"air_pressure":1007.6800000000001,"humidity":21,"visibility":17.14238845144357,"predictability":70}]}';
                }
            };
        });
    }
}
```

First we are extending our `AppServiceProvider` and overriding the `register` method, then we are telling the container to use an anonymous class when we require an implementation of `App\Http\HttpClientInterface`.

The final step is to register the `FakeServiceProvider` to the container, but only while we are testing:

```php
namespace Tests;

use Illuminate\Contracts\Console\Kernel;
use Tests\Fakes\FakeServiceProvider;

trait CreatesApplication
{
    public function createApplication()
    {
        $app = require __DIR__.'/../bootstrap/app.php';

        $app->register(FakeServiceProvider::class);
        $app->make(Kernel::class)->bootstrap();

        return $app;
    }
}
```

Now if we run the our tests, everything should be just fine.

```
vendor/bin/phpunit

Output:
PHPUnit 7.4.3 by Sebastian Bergmann and contributors.

..                                                                  2 / 2 (100%)

Time: 159 ms, Memory: 12.00MB
```

### Post-credits

This is a **sample** application made with the sole purpose of explaining how to test commands with dynamic output in Laravel and Laravel Zero, there are a lot of things that should be done in another way in a real application, for example our `HttpClient` is not making any kind of validation or catching any kind of exceptions, if something goes wrong the program will fail; we're not defining an interface for our parser, in a real application probably you want to do that; we are hardcoding a fake implementation of `HttpClient` in our `FakeServiceProvider`, in a real application you don't want to do that; and finally, in a real application you want to write unit tests for every one of your classes.

Also in a real application you probably want to use a real Http client like GuzzleHttp and focus on your business logic instead of waste time reinventing the wheel.

That's the way I test commands with dynamic output, do you know a better approach? I would love to learn yours, don't doubt in reaching me!

You can find the source code of this project on [Github](https://github.com/maxalmonte14/weather-cli)

I hope you have found this article useful. Thanks for staying this long!