# PHPCollections versión 1.0.0 finalmente disponible

PHPCollections es un conjunto de collecciones al estilo de lenguajes como Java o C# escritas en PHP, la idea detrás de esto es tener una manera de manejar grandes cantidades de datos con un API simple y poderosa en vez de trabajar con los simples arrays nativos de PHP.

Empecé este proyecto hace más o menos un año (¡la fecha del primer commit es 10/10/2017!), y por alguna u otra razón nunca encontré el tiempo para terminarlo, hasta ahora claro está.

### ¿Por qué PHPCollections?

Seguramente estás pensando _"¿Por qué trabajar con colleciones?"_, _"¡Tenemos los arrays nativos de PHP!"_, y estás en lo correcto, en un lenguaje dinámico como PHP los arrays cubren casi cualquier necesidad.

¡Pero! Cada día más personas están usando PHP para construir aplicaciones más grandes y complejas, y la manipulación de datos se puede convertir en una experiencia horrible. Aquí es donde PHPCollections entra en escena.

PHPCollections ofrece un conjunto de clases bien documentadas para cubrir diferentes necesidades, ¿Necesitas una pila (_stack_) para representar un conjunto de datos ordenados de manera descendente? ¿Necesitas únicamente guardar datos de un cierto tipo en tu colección? ¿Necesitas una colección tan flexible como los arrays nativos de PHP pero con todo el poder de la orientación a objetos? ¡PHPCollections lo tiene!

### Diferentes tipos de colecciones

Por el momento PHPCollections ofrece 4 tipos diferentes de colecciones, cada una cubre diferentes tipos de necesidades y tiene características diferentes:

- ArrayList
- Dictionary
- GenericList
- Stack

### ¡Menos charla, más código!

Mi colección favorita es sin duda `GenericList`, todos sabemos que PHP no soporta genéricos de forma nativa, en lenguajes como Java podemos hacer algo como esto `ArrayList<User> users = new ArrayList<User>();` para crear una colección que solo guarde objetos del tipo `User`, en PHP **simplemente no se puede**, bueno, no se podía hasta ahora.

```php
use PHPCollections\Collections\GenericList;
use Fake\Namespace\User;

$users = new GenericList(
    User::class,
    new User('Robert Smith'),
    new User('Siouxsie Sioux'),
);
```

Hermoso, ahora tenemos una colección de objetos `User` y cualquier intento de añadir datos de otro tipo fallará.

```php
$users->add(new \StdClass()); // ¡Esta línea lanzará una excepción del tipo InvalidArgumentException!
```

Con esta verificación puedes estar más seguro de que tu programa funcionará de la manera correcta ya que estás removiendo una fuente de posibles errores.

_- "¡Pero espera, amamos los arrays!"_

¡Ciertamente! Por esa razón PHPCollections cuenta con la clase `ArrayList`

```php
use PHPCollections\Collections\ArrayList;
use Fake\Namespace\User;

$users = new ArrayList(
    new User('Patricia Morrison'),
    new User('Peter Murphy'),
);

/*
* Añadiré este array como si fuera un
* usuario porque solo se vive una vez
*/
$users->add(['name' => 'Sid Vicious']); 
```

La mayoría de las colecciones ofrecen una gran cantidad de métodos realmente útiles, e incluso una interfaz fluida para manipular tus datos de una manera sencilla.

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
Si te gusta el paquete puedes encontrarlo en [Github](https://github.com/maxalmonte14/phpcollections), y revisar la documentación [aquí](https://github.com/maxalmonte14/phpcollections/tree/master/docs).

### Post-creditos

Puse mucho esfuerzo en construir PHPCollections, si vienes de Java o C# espero que te guste, si eres un desarrollador puramente PHP ¡también espero que te guste! Y si no has trabajado con colecciones nunca antes espero que te animes a probarlo. ¡Gracias por leer hasta este punto!