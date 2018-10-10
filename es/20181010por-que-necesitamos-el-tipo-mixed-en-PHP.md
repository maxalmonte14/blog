# Por qué necesitamos el tipo mixed en PHP

Hemos estado usando la palabra mixed por muchos años en nuestros docblocks para especificar que un valor puede ser de varios tipos, con la adición de los tipos escalares y las declaraciones de tipo de retorno en PHP 7.0 y las subsecuentes mejoras (tipos nulo y void en PHP 7.1 y el pseudo-tipo object en PHP 7.2) muchos pensarían que la adición de un tipo _"mixed"_ no es necesaria, pero yo pienso que ahora más que nunca lo es.

### Echémosle un vistazo problema

Muchos dirán que si dependes de un tipo _"mixed"_ entonces tu diseño es inherentemente malo, después de todo, si necesitas un tipo entero, por qué esperar una cadena, ¿cierto? Bueno, yo digo que **todo depende del caso de uso**. Cuando trabajas en una aplicación web y tus necesidades son básicas quizás simplemente no necesites usar un tipo mixed en tus métodos, considera el siguiente ejemplo.

```php
/**
 * Obtiene la diferencia entre dos fechas.
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

Este es un problema muy común cuando deseas calcular el tiempo transcurrido entre la fecha de registro de un usuario y una fecha específica, o incluso calcular cuántos años tiene el usuario. Este es un problema de la vida real donde la declaración de tipos te ayuda a escribir código de manera más lógica y asegurarte de que tu programa va a trabajar de la manera esperada el 100% del tiempo.

¡Pero no todo es color de rosa en PHP! Echémosle un vistazo a esta interfaz sacada del proyecto [PHPCollections](https://www.devalmonte.com/es/posts/20181004php-collections-version-1-finalmente-disponible).

```php
interface CollectionInterface
{
    public function add($value): void;

    public function get(int $offset);

    public function remove(int $offset): void;

    public function update(int $offset, $value): bool;
}
```

Como puedes ver esta interfaz combina declariones de tipo y retorno, con simplemente la omisión de las mismas, ¿por qué? Simple, el método `add` que se supone agrega un nuevo valor en la colección **no sabe que tipo de dato va a recibir**, podría ser una cadena, un entero, un objeto, o cualquier tipo válido de PHP.

Pasa lo mismo con el método `get`, el cuál no especifica el tipo que va a retornar, y el método `update` el cuál no especifica de qué tipo va a ser el parámetro `$value`.

### Una vez más, ¿por qué necesitamos el tipo mixed?

Ya sé, seguro estás pensando _"bueno, simplemente no especifiques el tipo cuando quieras un valor mixed"_, o _"deberías evitar usar mixed tanto como te sea posible"_, o esta otra que particularmente me encanta, _"¡tu diseño apesta!"_, pero en los casos de la vida real las cosas no funcionan de esa manera.

Echémosle un vistazo a la interfaz anterior pero ahora con declaración de tipos mixed:

```php
interface CollectionInterface
{
    public function add(mixed $value): void;

    public function get(int $offset): mixed;

    public function remove(int $offset): void;

    public function update(int $offset, mixed $value): bool;
}
```

¿Qué cambió? Simple, **más claridad**, cuando estamos trabajando con datos tipados deberíamos ser tan explícitos como podamos, de esa manera cualquiera que lea nuestro código ya sea ahora o dentro de 20 años podrá entender lo que hace inmediatamente. Esa es la razón por la cual necesitamos el tipo mixed en PHP, para **más claridad**.

Quizás estés pensando ahora mismo, _"¡esto es innecesario y hace al lenguaje más complejo!"_, bueno, PHP suporta los pseudo-tipos void y object, ¿por qué no soportar mixed?

Sé que existen muchas maneras de resolver el problema con la interfaz `CollectionInterface`, como usar un objeto para almacenar cualquier valor primitivo (o cualquier valor en general) y luego usar el pseudo-tipo object o incluso un objeto de un tipo específico diseñado para este propósito en nuestros métodos, pero pienso que eso sería sobrecomplicar el diseño, y en este caso el tipo mixed podría perfectamente resolver este problema.

De hecho, actualmente existe una discusión sobre si añadir o no el tipo mixed, puedes ver la propuesta [aquí](https://wiki.php.net/rfc/mixed-typehint) y la implementación [aquí](https://github.com/php/php-src/pull/2603).

### Post-creditos

PHP ha mejorado un mundo en los últimos años, en PHP 4 ni siquiera teníamos Programación Orientada a Objetos, hoy en día tenemos sobrecarga de métodos vía métodos mágicos, traits, propiedades tipadas (¡pronto en PHP 7.4!), y una inmensa cantidad de hermosas y poderosas características que nos brindan gran flexibilidad, como el modo estricto por ejemplo. Pienso que el tipo mixed sería otra buena adición al lenguaje, ¿tú que piensas? ¡Gracias por leer hasta este punto!