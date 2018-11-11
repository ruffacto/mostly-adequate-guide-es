# Hindley-Milner y yo

## ¿Cuál es tu tipo?
Si eres nuevo en el mundo funcional, no pasará mucho tiempo para que te encuentres inmerso pensando en los tipos que definen una función: [Firma de tipos](https://es.wikipedia.org/wiki/Signatura_(inform%C3%A1tica)). Los tipos son el meta lenguaje que permite a gente con diferentes antecedentes o formación comunicarse sucinta y efectivamente. En gran medida, están escritos en un sistema llamado "Hindley-Milner", el cual vamos a revisar en este capítulo.

Cuando trabajamos con funciones puras, su firma de tipos expresa tácitamente conceptos que el lenguaje Español necesitaría párrafos enteros. Estos tipos susurran secretos íntimos de cada función. En una sola línea exponen comportamiento e intención, y nos permiten derivar "Teoremas gratuitos" con tan solo los tipos. Los tipos pueden ser inferidos por lo que no hay necesidad de incluir anotaciones de tipo de forma explícita. Pueden ser afinados para incrementar su precisión o mantenerse genéricas y abstractas. Son útiles no solo para verificaciones al compilar, además, son la mejor documentación disponible. Por lo anterior, la firma de tipos juega un rol esencial en la programación funcional - mucho más de lo que se hubiese esperado inicialmente.

JavaScript es un lenguaje dinámico, pero eso no implica que evita utilizar tipos. Trabajamos con cadenas de texto, números, valores de verdad, etc. El asunto es que no están integrados de manera directa en el lenguaje por lo que siempre debemos recordar esa información mentalmente. Pero no debemos preocuparnos demasiado, ya que además de utilizar firma de tipos para documentar, también podemos usar comentarios a nuestro favor.  

Existen herramientas de verificación de tipos para JavaScript, tales como: [Flow](http://flowtype.org/) o el dialecto con tipos, [TypeScript](http://www.typescriptlang.org/). El objetivo de este libro es equiparte con las herramientas necesarias para escribir código funcional así que utilizaremos el sistema de tipos estandard comúnmente usado en lenguajes de programación funcional.

## Historias algo crípticas

Podemos encontrar firmas de tipos de Hindley-Milner por todos lados. Desde los polvorientos libros de matemáticas, pasando por mares interminables de artículos y publicaciones científicas, entre casuales entradas en blogs sabatinos y hasta en código fuente de programación. El sistema es sumamente simple; requiere una descripción rápida y algo de práctica para poder dominarlo.

```js
//  Corrige nombres propios: Primera letra en mayúsculas
//  capitalize :: String -> String
var capitalize = function(s){
  return toUpperCase(head(s)) + toLowerCase(tail(s));
}

capitalize("smurf");
//=> "Smurf"
```

Aquí, `capitalize` recibe un `String` y regresa un `String`. Olvidémonos de la implementación, es la firma de tipos lo que nos interesa.

Es HM, las funciones se escriben así: `a -> b` donde `a` y `b` son variables de cualesquier tipo. Por lo tanto, la firma de tipos de `capitalize` se puede leer como "una función de `String` a `String`". Es decir, recibe un `String` como entrada y retorna un `String` como salida.

Veamos otros ejemplos de firma de tipos:

```js
//  strLength :: String -> Number
var strLength = function(s){
  return s.length;
}

//  join :: String -> [String] -> String
var join = curry(function(what, xs){
  return xs.join(what);
});

//  match :: Regex -> String -> [String]
var match = curry(function(reg, s){
  return s.match(reg);
});

//  replace :: Regex -> String -> String -> String
var replace = curry(function(reg, sub, s){
  return s.replace(reg, sub);
});
```

`strLength` al igual que anteriormente: recibe un `String` pero en esta ocasión retorna un `Number`.

Las otras quizás te desconcierten a primera vista. Sin entender completamente los detalles, siempre podemos ver el último tipo como el retorno de la función. Así `match` se puede interpretar como: Toma un `Regex` y un `String` y devuelve `[String]`. Permíteme detallar algo interesante que está pasando aquí.

En la función `match` tenemos la libertad de agrupar la firma de tipos de la forma siguiente:

```js
//  match :: Regex -> (String -> [String])
var match = curry(function(reg, s){
  return s.match(reg);
});
```
Agrupando la última parte en paréntesis nos revela más información. Ahora la podemos ver como una función que toma un `Regex` y retorna otra función que a su vez, toma un `String` y devuelve `[String]`. Debido a que usamos _currying_, le pasamos un `Regex` y obtenemos una función que espera un argumento `String`. No es estrictamente necesario pensarlo de esta manera, pero es benéfico entender por que el último tipo es el que regresa.

```js
//  match :: Regex -> (String -> [String])

//  onHoliday :: String -> [String]
var onHoliday = match(/holiday/ig);
```
Cada argumento que pasamos a la función, elimina un tipo al principio de la firma de tipos. `onHoliday` es equivalente a `match` que ya recibió un `Regex`.

```js
//  replace :: Regex -> (String -> (String -> String))
var replace = curry(function(reg, sub, s){
  return s.replace(reg, sub);
});
```

Como se observa con todos los paréntesis en `replace`, incluirlos ensucia y fuerza la interpretación y se vuelve redundante así que preferimos omitirlos. De esa manera, podemos proveer todos los argumentos de una vez si así lo queremos y simplemente tomar en cuenta que: `replace` toma un `Regex`, un `String`, otro `String` y retorna nuevamente un `String`.

Finalmente, un par de cosas adicionales:


```js
//  id :: a -> a
var id = function(x){ return x; }

//  map :: (a -> b) -> [a] -> [b]
var map = curry(function(f, xs){
  return xs.map(f);
});
```

La función `id` toma cualquier tipo `a` y retorna algo del mismo tipo `a`. Esto muestra la posibilidad que tenemos de usar variables en tipos de igual manera que en código. Por convención, se usa el nombre de las variables `a` y `b`, pero son arbitrarios y pueden reemplazarse por cualquier otro. Si tienen el mismo nombre de variable, deberán tener el mismo tipo. Esto último es sumamente importante así que reiteremos: `a -> b` representa una función que toma cualquier tipo `a` y retorna cualquier tipo `b`. `a -> a` recibe cualquier tipo `a` y retorna el mismo tipo. Por ejemplo, `id` puede ser `String -> String` o `Number -> Number`, pero no `String -> Bool`.

De igual manera, `map` usa variables en sus tipos, solo que además incluye `b` que puede ser o no del mismo tipo que `a`. Podemos leerlo así: `map` toma una función que espera cualquier tipo `a` y retorna el mismo u otro tipo `b`, después toma una lista de `a`s y retorna una lista de `b`s.

Espero que para este momento, hallas encontrado satisfactoria la belleza expresiva de la firma de tipos. Literalmente nos dice lo que la función hace al pie de la letra. Tenemos una función de `a` a `b`, una lista de `a`s, y nos retorna una lista de `b`'s. Lo lógico que viene a la mente es aplicar la función en cada `a`. Cualquier otra cosa es pura fantasía.

La habilidad de razonar sobre los tipos y sus implicaciones te hará adentrarte considerablemente en el mundo funcional. No solo artículos, blogs, documentos, etc. te parecerán más fácilmente digeribles, adicionalmente, la firma de tipos por si sola te sugerirá la funcionalidad. Toma cierto tiempo convertirse en lector fluido, pero si perseveras, torrentes de información estarán a tu alcance sin necesidad de cargar documentación.

A continuación, unos cuantos ejemplos más para ver si eres capaz de dilucidar la funcionalidad tú solo.


```js
//  head :: [a] -> a
var head = function(xs){ return xs[0]; }

//  filter :: (a -> Bool) -> [a] -> [a]
var filter = curry(function(f, xs){
  return xs.filter(f);
});

//  reduce :: (b -> a -> b) -> b -> [a] -> b
var reduce = curry(function(f, x, xs){
  return xs.reduce(f, x);
});
```

`reduce` es posiblemente el más expresivo de todos. Sin embargo, trae jiribilla, así que no te preocupes si se te dificulta. Trataré de explicarlo en simple Español para los curiosos, aunque descifrarlo por uno mismo es mucho más instructivo.

Primeramente observando la firma de tipos vemos que el primer argumento es una función que espera `b`, `a` y produce `b`. ¿De dónde podría obtener esas `b`'s y `a`'s? Bien, pues los siguientes argumentos son una `b` y una lista de `a`s, asī que podemos asumir que esa `b` y cada una de esas `a`s en la lista serán aplicados a la función. Además vemos que el resultado de la función en el primer argumento es `b` al igual que la respuesta esperada de `reduce` por tanto, el resultado de la última aplicación de la función en el primer argumento puede ser también el resultado final esperado. Sabiendo lo que hace reduce, podemos decir con plena certeza que la descripción anterior es precisa. 


## Reduciendo las posibilidades

Una ves que una variable de tipo es incluida, una curiosa propiedad emerge, llamada: [parametricidad](http://en.wikipedia.org/wiki/Parametricity). Esta propiedad describe que toda función *actuará de forma uniforme en cualesquier tipo*. Investiguemos:

```js
// head :: [a] -> a
```

Observando `head`, vemos que va de `[a]` a `a`. A parte del tipo concreto `lista []`,no tenemos ninguna información adicional, por lo tanto, su funcionalidad está limitada a trabajar solamente en la lista de valores. ¿Qué podría hacer con la variable `a` si no sabe absolutamente nada de ella? Es decir, `a` nos dice que puede ser *cualquier tipo* pero ninguno en particular, lo que nos lleva a pensar que realizará una acción de manera uniforme para *todo* tipo imaginable. Esto es a lo que *parametricidad* se refiere. Adivinando acerca de la implementación, lo único razonable en asumir es que retornará el primero, el último o un valor aleatorio intermedio de la lista. El nombre `head` sugiere que es lo que hace.

Aquí tenemos otro ejemplo:

```js
// reverse :: [a] -> [a]
```

Tomando en cuenta la firma de tipos, ¿Qué es lo que `reverse` pretenderá hacer? Nuevamente, no puede hacer nada específico a `a` porque no sabe que es. No lo puede transformar en en otro tipo, si no, tendríamos una `b` en la firma. ¿Podría ordenar la lista? Pues no, no tiene información suficiente para ordenar cualquier tipo de datos. ¿Qué tal re ordenar la lista? Si, es posible, pero solo si lo hace de forma exactamente predecible cada vez que la usemos. Otra posibilidad es que podría remover o duplicar un elemento. En cualquier caso, el rango de posibilidades se reduce considerablemente por su tipo polimórfico.

Esta reducción de posibilidades nos permite usar un buscador de firma de tipos como por ejemplo: [Hoogle](https://www.haskell.org/hoogle) para encontrar la función que nos interesa. La información inherente, contenida en la firma de tipos es verdaderamente poderosa.

## Teoremas gratuitos y sin esfuerzos

Además de poder deducir posibilidades de implementación, este forma de razonamiento nos acarrea *teoremas gratuitos*. A continuación presentamos algunos ejemplos aleatorios de teoremas obtenidos directamente del [ artículo de Wadler sobre la materia](http://ttic.uchicago.edu/~dreyer/course/papers/wadler.pdf).

```js
// head :: [a] -> a
compose(f, head) == compose(head, map(f));

// filter :: (a -> Bool) -> [a] -> [a]
compose(map(f), filter(compose(p, f))) == compose(filter(p), map(f));
```

No se necesita código para entender estos teoremas ya que se derivan directamente de los tipos. El primero dice que si aplicamos `head` a una lista, y posteriormente aplicamos una función `f` sobre el resultado, esto es equivalente, y por cierto más eficiente, a aplicar primero `map(f)` sobre todos los elementos de la lista y después tomar `head` del el resultado.

Cualquiera podría pensar que esto es solo sentido común, pero la última vez que verificamos, las computadoras no tienen ese sentido. De hecho para hacerlo, deben tener una forma de automatizar este tipo de optimizaciones. Usando matemáticas es posible encontrar maneras de formalizar estas reglas intuitivas que son de gran ayuda en el rígido campo de la lógica de computadoras.

El teorema `filter` es similar. Nos dice que si componemos `f` y `p` para validar cual debe ser filtrada, y después aplicamos `f` usando `map` (recuerda que filter no transforma elementos - su firma de tipos asegura que `a` no se va a tocar); siempre va a ser equivalente a mapear `f` y despues filtrar usando el predicado `p`.

Estos solo son dos ejemplos, pero se puede aplicar el mismo razonamiento a cualquier firma de tipos polimórfica y siempre será valida. En JavaScript, existen herramientas para declarar reglas de re escritura. También se puede usar `compose` para el mismo propósito. Las posibilidades son infinitas y están a tu alcance.

## Restricciones

La última cosa que queremos mencionar es que podemos restringir tipos que satisfacen una interfase.

```js
// sort :: Ord a => [a] -> [a]
```

Lo que vemos a la izquierda de la flecha gruesa es un lineamiento: `a` debe ser un `Ord`. O en otras palabras, `a` debe implementar la interfase `Ord`. ¿Qué es `Ord` y de dónde viene? En un lenguaje tipado estaría definido como una interfase que asegura poder ordenar valores. Lo anterior, no solo nos da información adicional sobre `a` y lo que `sort` pretende hacer, además, restringe el dominio de la función. A esta declaración de interfases le llamamos *Restricción de tipos*.


```js
// assertEqual :: (Eq a, Show a) => a -> a -> Assertion
```

Aquí tenemos dos restricciones: `Eq` y `Show`. Las cuales se van a asegurar que podamos evaluar igualdad en `a`s y que podamos imprimir la diferencia si no son iguales.

Veremos más ejemplos de restricciones en capítulos siguientes para reforzar la idea.

## En Resumen

La firma de tipos Hindley-Milner es ubicua en el mundo funcional. Aunque son simples de leer y escribir, toma tiempo dominar la técnica de entender programas solamente a partir de los tipos. Incluiremos firma de tipos en código de aquí en adelante.

[Chapter 8: Tupperware](ch8.md)
