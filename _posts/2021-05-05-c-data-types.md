---
layout: post
title:  "Tipos de datos en C"
author: "Santiago Trini"
categories: c
tags: [algo,c]
image: data-types.png
date:   2021-05-05 00:00:00 -0300
---

Los programas del artículo anterior operaban sobre cadenas de caracteres, llamadas _strings_ por los programadores. En C y en cualquier lenguaje de programación los datos sobre los que operamos tienen un **tipo**. Un lenguaje de programación define un puñado de **tipos de datos básicos**. En C son los siguientes: `char`, `int`, `float`, `double` y `bool`.

Un tipo de dato puede definirse como un **conjunto de valores** y un **conjunto de operaciones** sobre esos valores. Por ejemplo en matemática tenemos los números naturales, el conjunto $\mathbb{N}$. Y sobre esos números podemos definir la suma y la multiplicación, pero no la resta o la división. Al menos no para cualquier valor dentro de $\mathbb{N}$.

En C el tipo de dato `int` (por _integer_) está pensado para representar los números enteros y los tipos `float` y `double` para los números reales.
La diferencia entre `float` y `double` tiene que ver con la precisión del formato. El primero implementa precisión simple y el segundo precisión doble como su nombre lo indica. Los detalles están definidos en el estándar [IEEE 754](https://es.wikipedia.org/wiki/IEEE_754).

Claro que en matemática los conjuntos de números son infinitos y en C o cualquier otro lenguaje de programación no lo son. Uno no puede tener memoria infinita en una computadora para representar cualquier número. Siempre hay limites, definidos por la arquitectura de cada computadora.
Por ejemplo, si una computadora representa un `int` con 4 bytes o 32 bits, los números representables están en el rango de $[-2^{31}, 2^{31}-1]$.

El tipo de dato `char` es un número de 8 bits o un byte. Está pensado para representar caracteres y lo normal es que corresponda a la codificación de caracteres [ASCII](https://es.wikipedia.org/wiki/ASCII) y UTF-8.

Por último, el `bool` o booleano es un tipo de dato que se agregó en [C99](https://es.wikipedia.org/wiki/ANSI_C#C99) y existe para representar los valores $\\{0, 1\\}$ o verdadero y falso. En resumen, todos los tipos de datos básicos en C son números.

|Tipo    |Valores       |Operadores comunes |Ejemplos de constantes
|:------:|:------------:|:-----------------:|:-------------------:|
|`int`   |$\mathbb{Z}$  |`+` `*` `-` `/` `%`|`124` `42` `5355`    |
|`double`|$\mathbb{R}$  |`+` `*` `-` `/`    |`3.14` `2.5` `1.2e11`|
|`bool`  |$\\{ 0, 1 \\}$|`!` `&&` `||`      |`true` `false`       |
|`char`  |caracteres    |                   |`'a'` `'4'` `'!'`    |

## Algunas definiciones

Antes de ver algunos programas de ejemplo conviene definir algunos términos técnicos que aplican a cualquier lenguaje de programación.

### Variables

Una **variable** es un objeto que contiene un valor de un tipo de dato dado. Podemos referirnos en el código a una variable a través de un nombre.
Por ejemplo una variable de tipo `int` puede guardar el valor `1234` pero no el valor `3.14159` o `"Hola mundo"`. El valor de una variable puede cambiar a lo largo de la ejecución del programa, de ahí su nombre.

Para crear una variable usamos un **enunciado de declaración**, que no es más que un tipo seguido de un nombre para la variable como en `int num;`. El nombre debe ser un **identificador** válido. En C un identificador debe comenzar con una letra o un `_`. Los identificadores distinguen entre mayúsculas y minúsculas. Por ejemplo `Abc`, `ABC` y `abc` son tres identificadores distintos. Tener en cuenta que no podemos usar palabras reservadas o _keywords_ del lenguaje para definir identificadores. No podemos usar como nombre de una variable `double` o `void` por ejemplo.



### Enunciados de asignación

Para **asignar** un valor a una variable usamos el operador `=` como en el siguiente fragmento de código.

```c
int a;        // declara a como int
double b;     // declara b como double
char c;       // declara c como char
a = 1234;     // asigna 1234 a la variable a
b = 321.2;    // asigna 321.2 a la variable b
c = 'S';      // asigna 'S' a la variable c
a = 4321;     // cambia el valor de a
c = 2.11e-13; // no hay error, pero tampoco funciona como esperamos
```

El operador `=` (asignación) lleva una variable a su izquierda y una **expresión** a su derecha.

Una expresión puede ser una **constante**, también llamadas literales, como en el ejemplo de arriba. Un número como `1234` es una constante de tipo `int`. En cambio si hay punto decimal (en Estados Unidos usan el `.` en vez de la `,`) representa un `float` o un `double`. Los números de coma flotante también pueden representarse con exponentes como en una calculadora. En el código de arriba `2.11e-13` quiere decir $2,11 \times 10^{-13}$.

Para representar constantes de tipo `char` usamos comillas simples, como en `'S'`. Las comillas no forman parte del valor. Si usamos comillas dobles tenemos una constante de tipo `string`, como en `"Hola mundo\n"`. Aunque en C no existe algo como el tipo de dato `string` en el mismo sentido que `char` o `int`. Algunos caracteres especiales, como el caracter de nueva línea o la tabulación deben escribirse de manera especial, como `\n` o `\t`.

Para el tipo de dato `bool` tenemos las palabras `true` y `false` para los dos valores posibles.

Podemos construir expresiones más complicadas usando **operadores** y variables. En el siguiente fragmento de código declaramos tres variables `a`, `b` y `c`.

```c
int a, b, c;  // declaramos tres variables en la misma linea
a = 12;
b = 13;
c = a + b;    // asignamos a c la suma de a y b
b = c - 12;   // podemos combinar variables y constantes
```

En las últimas dos líneas usamos operadores aritméticos (suma y resta) para crear expresiones más complejas. Podemos usar constantes y variables como operandos, siempre cuidando de utilizar tipos compatibles.

También podemos declarar e **inicializar** el valor de una variable en el mismo enunciado.

```c
int num = 42;
```

Con estas definiciones veamos algunos programas de ejemplo usando los distintos tipos que tenemos en C.

## Números enteros

El siguiente programa lee dos números como argumentos en la línea de comandos e imprime 4 líneas de texto a la consola. El producto, el cociente y el resto de estos números. La última línea imprime que el dividendo es igual al cociente por el divisor más el resto. Las variables `p`, `q` y `r` aluden a producto, cociente y resto.

Los argumentos leídos por el programa en la CLI llegan como _strings_, por eso hay que convertirlos a tipo `int` antes de poder operar. Para eso usamos la función `atoi()` que significa _ASCII to integer_. Esa función está definida en el _header_ `stdlib.h`.

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
  int a = atoi(argv[1]);
  int b = atoi(argv[2]);
  int p = a * b;
  int q = a / b;
  int r = a % b;
  printf("%d * %d = %d\n", a, b, p);
  printf("%d / %d = %d\n", a, b, q);
  printf("%d %% %d = %d\n", a, b, r);
  printf("%d = %d * %d + %d\n", a, q, b, r);
  return 0;
}
```

Este ejemplo también ilustra el uso de _format strings_ con la función `printf()`. Esta función acepta como primer argumento un _string_ con caracteres especiales, como `%d`. Por ejemplo, en el primer `printf()` la primer ocurrencia de `%d` se reemplaza con `a`, la segunda con `b` y la tercera con `p`. En el tercer `printf()` como queremos imprimir el carácter `%` debemos escribirlo `%%` porque ese carácter tiene un significado especial.

El operador `%` significa resto de la división entera. No aplica cuando estamos dividiendo dos `float` o `double`.

Compilando y ejecutando el programa deberíamos ver algo así.

```console
$ gcc -o intops intops.c
$ ./intops 9 2
9 * 2 = 18
9 / 2 = 4
9 % 2 = 1
9 = 4 * 2 + 1
```

Como segundo ejemplo leemos un número entero $n$ como argumento e imprimimos un número aleatorio entre $0$ y $n-1$.

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main(int argc, char *argv[]) {
  int n = atoi(argv[1]);
  srand(time(NULL));
  printf("%d\n", rand() % n);
  return 0;
}
```

La función `rand()` devuelve un número aleatorio entre 0 y `RAND_MAX`. El valor de `RAND_MAX` depende de la computadora, en mi caso es 2147483647. Si dividimos por 10 y nos quedamos con el resto de esa división nos quedamos con el último dígito del número. En general si dividimos por $n$ y nos quedamos con el resto acotamos el resultado al intervalo $[0, n - 1]$.

Antes de usar `rand()` debemos darle una semilla o _seed_ al generador de números aleatorios. Una manera de hacer esto es tomar la fecha al momento de ejecutar el programa y pasarla como argumento de `srand()`. Por eso incluimos el _header_ `time.h`.  

## Coma flotante

Los tipos `float` y `double` existen para representar números con parte fraccionaria. El ejemplo que sigue usa el tipo `double` para leer tres argumentos $a$, $b$ y $c$, los coeficientes de una función cuadrática $ax^2+bx+c$. El programa calcula las raíces de la función siempre y cuando sean raíces reales. Si la función tiene raíces complejas el resultado será `nan` (_not a number_) que es uno de los valores que puede tomar un `double` (y un `float`). Para calcular las raíces aplicamos la conocida fórmula.

$$\frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$$

Una función cuadrática tiene raíces complejas si el discriminante es menor a 0. El discriminante es el término dentro de la raíz cuadrada: $b^2 - 4ac$. Si el discriminante es cero las dos raíces son idénticas.

```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

int main(int argc, char *argv[]) {
  double a = atof(argv[1]);
  double b = atof(argv[2]);
  double c = atof(argv[3]);
  double discriminant = b * b - 4.0 * a * c;
  double root = sqrt(discriminant);
  printf("x1: %.2f\n", (-b + root) / (2.0 * a));
  printf("x2: %.2f\n", (-b - root) / (2.0 * a));
  return 0;
}
```

Noten el uso del _header_ `math.h` que nos da entre otras cosas, la función `sqrt()` para calcular raíces cuadradas. También podemos ver en las líneas de `printf()` el uso de `%.2f` que quiere decir número de coma flotante con dos lugares después del punto decimal.

Para compilar este programa es necesario proveer la opción `-lm` a `gcc` porque estamos usando `math.h`. En la consola compilamos y ejecutamos de la siguiente manera.

```console
$ gcc -lm -o quad quad.c
$ ./quad 1 -3 2
x1: 2.00
x2: 1.00
```

## Booleanos

El tipo booleano por el [álgebra de Boole](https://es.wikipedia.org/wiki/%C3%81lgebra_de_Boole) es un tipo de dato que usamos mucho en computación. Un `bool` solo puede tomar uno de dos valores, internamente o el número cero o el número uno. Pero en definitiva lo que queremos decir con un booleano es algo verdadero o algo falso. Para eso C nos permite usar las palabras `true` y `false`. Como el tipo `bool` es una adición más reciente al lenguaje para usarlo necesitamos incluir el _header_ `stdbool.h`.

El siguiente ejemplo es un programa que decide si un año es bisiesto o no. Usamos dos variables, una para el año y otra llamada `isleap` (es bisiesto en inglés) para indicar si es o no bisiesto. En primer lugar para que un año sea bisiesto tiene que ser divisible por 4. Pero además no tiene que ser divisible por 100, al menos que también sea divisible por 400.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

int main(int argc, char *argv[]) {
  int year = atoi(argv[1]);
  bool isleap;
  isleap = (year % 4 == 0);
  isleap = isleap && (year % 100 != 0);
  isleap = isleap || (year % 400 == 0);
  printf("%s\n", isleap ? "Es bisiesto" : "No es bisiesto");
  return 0;
}
```

Para esto usamos los operadores `&&` y `||` que corresponden a las operaciones AND y OR del álgebra de Boole. El funcionamiento de los operadores lógicos es fácil de entender con una tabla de verdad.

|`a`    |`b`    |`a && b`|`a || b`
|:-----:|:-----:|:------:|:-------:|
|`false`|`false`|`false` |`false`  |
|`false`|`true` |`false` |`true`   |
|`true` |`false`|`false` |`true`   |
|`true` |`true` |`true`  |`true`   |

Los otros operadores que aparecen son operadores relacionales como `!=` (distinto) y `==` (igual) comparan dos valores, en este caso números enteros. Estos operadores siempre arrojan como resultado o verdadero o falso.

Al final del programa en el `printf()` podríamos imprimir el valor final de la variable `isleap` pero eso mostraría un 0 para falso y un 1 para verdadero, porque internamente el `bool` no es más que un número. En vez de imprimir un número imprimimos los mensajes "Es bisiesto" o "No es bisiesto". Para eso usamos el operador ternario: `? :`. El operador ternario, también llamado operador condicional lleva tres operandos, de ahí el nombre. El primer operando es una condición, algo que pueda ser verdadero o falso. Si la condición es verdadera el resultado es lo que sigue al `?`, y si es falso el resultado es lo que está a la derecha de los `:`. El `%s` en `printf()` quiere decir que se reemplaza por un _string_.

Compilando y ejecutando vemos lo siguiente.

```console
$ gcc -o isleap leap.c
$ ./isleap 2000
Es bisiesto
$ ./isleap 1000
No es bisiesto
```
