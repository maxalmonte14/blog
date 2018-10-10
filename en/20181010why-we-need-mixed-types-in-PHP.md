# Why we need mixed types in PHP

We have been using the mixed keyword for years into our PHP docblocks to specify that a value can be, well, kinda _"mixed"_. With the addition of scalar and return type declarations in PHP 7.0, and the subsequent improvements (nullable and void types in PHP 7.1 and object pseudo-type in PHP 7.2) many people may think that a mixed type is not necessary, but I think that now more than ever it is.

### Let's see the problem

People may say that if you depend on a _"mixed"_ type then your design is bad, after all, if you need an integer, why to expect a string, right? That's when I say **it always depends on the use case**, when you're creating a web application and doing some basic operations maybe you simply won't need a mixed type hinting, consider the following example.

```php
/**
 * Get the difference between two dates.
 *
 * @param DateTime $firstDate
 * @param DateTime $secondDate
 *
 * @return DateInterval
 */
public function getDateDifference(DateTime $firstDate, DateTime $secondDate): DateTime
{
    return $firstDate->diff($secondDate);
}
```

This is a very common problem to solve when you're calculating the time passed between a user registered to your site and a specific date, or even how old the user is. That's a real-life example where type hinting helps you to write a more logical piece of code and make sure your program is going to work as you expected 100% of the time.

But it's not all a bed of roses with PHP! Let's take a look at this interface from the [PHPCollections](https://www.devalmonte.com/en/posts/20181004php-collections-version-1-is-released) project.

```php
interface CollectionInterface
{
    public function add($value): void;

    public function get(int $offset);

    public function remove(int $offset): void;

    public function update(int $offset, $value): bool;
}
```

As you can see this interface is a combination of typed and not typed hinting and return declarations, why? Simple, the `add` method which is supposed to add a new value into the collection **does not know what type of data is going to receive**, it could be a string, an integer, an object, or any valid PHP type.

The same thing happens with the `get` method, which hasn't a return type, and the `update` method which hasn't a `$value` property type hinted.

### So, again, why do we need a mixed type?

I know, maybe you're thinking right now _"well, just don't type hint when you want a mixed"_, or _"you should avoid using mixed types as much as you can"_, or this one that I love, _"your design just sucks!"_, but in real life programming things don't work that way.

Let's take a look again at the same interface with mixed type hints:

```php
interface CollectionInterface
{
    public function add(mixed $value): void;

    public function get(int $offset): mixed;

    public function remove(int $offset): void;

    public function update(int $offset, mixed $value): bool;
}
```

What changed? Simple, **explicitness**, when we are working with type hints we should be as explicit as we can, so anybody reading the code now, or 20 years in the future can easily understand what is going on immediately. That's why we need mixed types in PHP, for **explicitness**.

You might think, _"this is unnecessary and make the language more complex!"_, well, PHP supports void and object as pseudo-type, why not support mixed?

I know there is a lot of ways to solve the `CollectionInterface` problem, like wrapping any primitive value (or any value at all) in an object, and just use the object pseudo-type or even a specific object designed for this purpose, but I think that's overcomplicating the design, and the mixed type can perfectly address this problem.

Actually, there is an ongoing conversation about adding mixed or not, [here](https://wiki.php.net/rfc/mixed-typehint) you can see the proposal, and [here](https://github.com/php/php-src/pull/2603) the implementation.

### Post-credits

PHP has improved a lot in the last few years, in PHP 4 we even hadn't Object Oriented Programming, today we have overloading via magic methods, traits, typed properties (coming soon in PHP 7.4!) and a lot of beautiful and powerful features that give us flexibility like the strict mode. I think adding the mixed type would be another good addition to the language, what do you think?. Thanks for staying this long!