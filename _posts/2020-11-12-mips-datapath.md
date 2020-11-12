---
layout: post
title:  "MIPS: datapath"
author: "Santiago Trini"
categories: orga
tags: [mips,datapath]
image: datapath.png
date:   2020-11-12 00:00:00 -0300
---

En el artículo anterior vimos como cualquier instrucción de MIPS puede representarse con un número. Un número binario de 32 bits que en definitiva no es más que una señal eléctrica digital. La CPU de MIPS debe utilizar esa señal para ejecutar las instrucciones de un programa en el _hardware_.

Como dijimos anteriormente una CPU tiene tres partes: la unidad de control, los registros y la ALU. Estas dos últimas forman lo que se conoce como el _datapath_. En este artículo vemos como podemos implementar este _datapath_, cuáles son sus partes constituyentes y como funcionan en conjunto. La implementación de la unidad de control la dejamos para el próximo artículo.

Hay que aclarar que esta implementación del _datapath_ de MIPS ejecuta cada instrucción en un ciclo de reloj pero no usa _pipelining_. Esta técnica, el _pipelining_, sirve para mejorar sustancialmente el rendimiento de la CPU y es lo que usa una CPU **real** de MIPS. El _datapath_ real de MIPS usa una tubería o _pipeline_ de cinco etapas que desarrollaremos en otro artículo.

El circuito de Logisim de este _datapath_ simple pueden encontrarlo en [GitHub](https://github.com/santiagotrini/mips-datapath).

## Unidades funcionales

Para ilustrar el _datapath_ de MIPS vamos a concentrarnos en un subconjunto de las instrucciones. Nos interesan las siguientes instrucciones:

- Instrucciones de referencia a memoria: _load word_ y _store word_.
- Instrucciones lógicas y aritméticas: _add_, _sub_, _and_ y _or_.
- Instrucciones de saltos: _branch on equal_ y _jump_.

Si bien es un subconjunto reducido sirve perfectamente para mostrar las unidades funcionales de la CPU de MIPS. Además representan los tres tipos de instrucciones, R, I y J.

En primer lugar tenemos que preguntarnos que necesitamos para ejecutar estas instrucciones. Para todas las instrucciones necesitamos una **memoria** que las contenga y un registro con la dirección de la próxima instrucción a ejecutar. Este registro es el que tradicionalmente llamamos PC por _program counter_. En el primer paso del ciclo de instrucción entonces leemos una memoria con instrucciones usando el valor del PC.

Los próximos pasos dependen de la instrucción en cuestión, pero salvo para _jump_ en todas tenemos que usar entre dos y tres registros.

```
# instrucciones de tipo R usan tres registros
add   $t1, $t2, $t3
sub   $t3, $t3, $a0
and   $s1, $zero, $a1
# instrucciones de tipo I usan dos registros
beq   $zero, $t4, exit      
lw    $t1, 4($t0)
```

Por este motivo tenemos un **archivo de registros**, que no es más que un circuito con los 32 registros enteros de MIPS, desde `$zero` hasta `$ra`. El archivo de registros nos permite leer dos registros y escribir en un tercer registro. Para especificar un registro bastan cinco bits porque $2^5=32$.

Para las instrucciones aritméticas y lógicas está claro que necesitamos una ALU que realice las operaciones. Si miramos con atención la sintaxis de `lw` y `sw` vemos que hay una suma también en esas instrucciones. Porque la dirección de memoria sobre la que leemos o escribimos se calcula sumando una dirección base más un _offset_. Incluso en el _branch on equal_ tenemos que utilizar la ALU, porque una manera muy simple de comparar dos registros para ver si son iguales es restándolos y viendo si el resultado es cero. Por este motivo la ALU de MIPS tiene una salida de 1 bit que nos indica si el resultado de la operación dió cero. Este tipo de señales dentro de una CPU se conocen como _flags_.

Por último tenemos para las instrucciones de acceso a memoria la **memoria de datos**, tanto para leer en `lw` como para escribir en `sw`. Conectando estas unidades funcionales tenemos distintos grupos de cables, la mayor parte de ellos son de 32 bits de ancho ya que tanto las instrucciones como los datos con los que opera MIPS son de 32 bits. El manual de Patterson y Hennessy ilustra muy bien las partes del _datapath_ de MIPS y sus conexiones con el siguiente diagrama.

![datapath](../assets/img/mips-datapath/mips-datapath.png){:.zoom}

Salvo que se aclare en el diagrama los buses que conectan las distintas partes de la CPU son de 32 bits. Las líneas azules representan señales de control, todas de 1 bit de ancho salvo por las que sirven para controlar la ALU.

## Los multiplexores

En el diagrama de arriba se aprecian varios multiplexores. Estos están ahí porque existe una asimetría entre entradas y salidas. Por ejemplo si yo quiero que el resultado de la ALU vaya a parar al archivo de registros para el caso de un `add` o a la dirección de la memoria de datos para un `lw` no hay problema. Simplemente divido el bus en dos y listo.

El problema se presenta cuando yo quiero que dos señales potencialmente diferentes vayan a la misma entrada. Por ejemplo para un `add` como dijimos queremos que el resultado vaya a la entrada etiquetada como _Write data_ en el archivo de registros. Pero para un `lw` necesitamos que _Write data_ en el archivo de registros reciba la salida _Read data_ de la memoria de datos. Seguramente estos dos buses de 32 bits llevan distintas secuencias de unos y ceros. ¿Qué es lo que va a ver el archivo de registros en _Write data_ si estas señales llegan las dos juntas al mismo tiempo?

Para evitar esto usamos multiplexores en estos casos, y la unidad de control elige con estos multiplexores cuál señal pasa y cuál se queda. Existen tres casos, en el que mencionamos arriba usamos la señal `MemToReg`. El segundo caso para diferenciar entre una instrucción de tipo I y una de tipo R con `RegDst`. Y el tercero para elegir el segundo operando de la ALU, entre un registro o una constante o valor inmediato con `ALUSrc`.

## La próxima instrucción

Para la gran mayoría de las instrucciones de MIPS el PC tiene que incrementar en cuatro su valor para apuntar a la próxima instrucción del programa. Para eso está el sumador que está arriba a la izquierda. Este sumador tiene como entradas el valor actual del PC y el número cuatro.

Pero para las instrucciones `beq` y `j` el valor del PC tiene que ser distinto. En el caso de `beq` solo si los dos registros que se comparan son iguales hay que sumar al PC el valor de los primeros 16 bits de la instrucción extendidos a 32 bits y desplazados 2 bits a la izquierda. Eso es lo que se ve en la esquina superior derecha del diagrama.

El caso de _jump_ no aparece en el diagrama de arriba pero si está implementado en el circuito de Logisim. El cálculo de la dirección es un poco más complicado en este caso. Lo que la CPU hace es usar los primeros 26 bits de la instrucción, desplazarlo dos bits a la izquierda para obtener 28 bits y concatenarlo con los últimos 4 bits del PC para obtener una dirección de 32 bits.
