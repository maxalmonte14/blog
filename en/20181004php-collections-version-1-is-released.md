# PHPCollections version 1.0.0 is finally released

PHPCollections is a set of Java/C#-like collections written in PHP, the idea behind this is to have a similar way of dealing with large and complex amounts of data in a powerful way with a simple API instead of working with plain PHP arrays.

I started this project like one year ago (the first commit date is 10/10/2017!), and by one or another reason I never found the time to complete it, until now of course.

### Why PHPCollections?

Maybe you're thinking right now _"why to work with collections?", "What is the big deal?", "We have beautiful native arrays!"_, and you are totally right, in a dynamic language like PHP, arrays fit almost any need.

But! Every day more people are using PHP to build larger and more complex applications, and manipulating data can turn into a really awful experience. So, here is when PHPCollections come to scene.

PHPCollections offers a set of different well-documented classes to fit different needs, do you need a stack to represent a set of descendant ordered data? Do you need to only store a certain type of object into a collection? Do you need a collection as flexible as native PHP arrays but with the power of object-oriented? PHPCollections has you cover!

### Different type of collections

Right now the package offers 4 different types of collections with different use cases and capabilities:

- ArrayList
- Dictionary
- GenericList
- Stack

### Less talk, more code!

My favorite type of collection without any doubt is `GenericList`, we all know that PHP doesn't support generic types, in languages like Java we can do something like this `ArrayList<User> users = new ArrayList<User>();` to store just objects of type `User` into our collection, in PHP **you simply can't**, well, you couldn't until now.

```php
use PHPCollections\Collections\GenericList;
use Fake\Namespace\User;

$users = new GenericList(
    User::class,
    new User('Robert Smith'),
    new User('Siouxsie Sioux'),
);
```

Beautiful, now we have a collection of `User` objects and any intent to add some other type of data to the class will fail!

```php
$users->add(new \StdClass()); // This line throws an InvalidArgumentException!
```

With this verification you can be more confident your program will run the right way because you're removing an error source.

**But wait, we love arrays!**

That's right! That's the reason PHPCollections offers `ArrayList`

```php
use PHPCollections\Collections\ArrayList;
use Fake\Namespace\User;

$users = new ArrayList(
    new User('Patricia Morrison'),
    new User('Peter Murphy'),
);

/*
* I'll add this array as an user and it's ok because YOLO
*/
$users->add(['name' => 'Sid Vicious']); 
```

Most collections offer a variety of useful methods, and even a fluent interface to manipulate your data.

```php
$users->map(function ($user) {
    $user->name = sprintf('%s %s', 'Sr.', $user->name);
    return $user;
})->merge([
    new User('Ian Curtis'),
    new User('Andrew Eldritch'),
])->forEach(function ($user, $key) {
    print($user->name);
})
```

If you like the package you can find it on [Github](https://github.com/maxalmonte14/phpcollections), and check the documentation in [here](https://github.com/maxalmonte14/phpcollections/tree/master/docs).

### Post-credits

I put a lot of effort in building PHPCollections, if you come from the Java/C# world I hope you like it, if you're a pure PHP developer I hope you like it too! And if you haven't work with collections before I hope you give it a try. Thanks for staying at this point!