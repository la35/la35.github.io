---
layout: post
title:  "MIPS: código máquina"
author: "Santiago Trini"
categories: orga
tags: [mips,orga]
image: binario.png
date:   2020-10-15 00:00:00 -0300
---

Para una computadora la representación de un programa con una serie de números es la más natural. Para los humanos en cambio es mucho más cómodo trabajar con símbolos. El lenguaje ensamblador es una representación simbólica del lenguaje máquina, que siempre es numérico. En este artículo voy a explicar brevemente como la arquitectura MIPS codifica numéricamente las instrucciones que vimos cuando programamos en _assembler_.

## Campos y formatos de instrucción

Una instrucción en MIPS está formada por un conjunto de campos, cada uno con una función definida. En MIPS tenemos los siguientes campos.

- _op_: código de operación o _opcode_. 6 bits.
- _rs_: un registro, el primer operando (_register source_). 5 bits.
- _rt_: otro registro, el segundo operando. 5 bits.
- _rd_: otro registro, el destino de la operación (_register destination_). 5 bits.
- _shamt_: cantidad de desplazamiento o _shift amount_, solo para las instrucciones que realizan _shifts_. 5 bits.
- _funct_: código de función, se usa para distinguir instrucciones con el mismo _opcode_. 6 bits
- _imm_: una constante o valor inmediato (_immediate_). 16 bits.
- _addr_: una dirección de memoria (_address_). 26 bits.

Todas las instrucciones en MIPS tienen 32 bits de longitud, o sea una palabra de memoria (4 bytes). Esto es una decisión de diseño, no es obligatorio. La arquitectura x86 a diferencia de MIPS, tiene instrucciones de longitud variable.

Pero no todas las instrucciones usan todos los campos que enumeramos arriba. Entonces tenemos formatos o tipos de instrucción, que en MIPS son tres llamados R, I y J.

|Tipo |Campos |
|-----|-------|
|R| _op rs rt rd shamt funct_ |
|I| _op rs rt imm_ |
|J| _op addr_ |

El formato R es por registro y es el formato utilizado por todas las instrucciones que operan con dos registros y ponen el resultado en un tercer registro.

El formato I por _immediate_ o inmediato es el formato de instrucciones como _addi_ que operan con un registro y una constante y ponen el resultado en otro registro.

El formato J es por _jump_ y corresponde a las instrucciones de saltos donde se usan casi todos los bits de la instrucción para especificar la dirección de memoria a la que queremos saltar.

Noten que para cada formato la suma de la longitud de los campos que lo componen debe sumar 32 bits. Además los tres formatos tienen en común los primeros 6 bits del _opcode_. Si no sería imposible distinguir un formato del otro. Cuando la CPU ve el _opcode_ en los bits 31 - 26 de la instrucción debe saber como intepretar los próximos bits.

## Un ejemplo

Teniendo en cuenta los campos y formatos de instrucción podemos entender como se traduce de _assembler_ a código máquina. Por ejemplo la siguiente instrucción `add $t0, $s1, $s2` equivale al número 36847648. La representación en decimal en general no es muy útil. En hexadecimal tenemos `0x02324020` y en binario `0000 0010 0011 0010 0100 0000 0010 0000`. El porqué de ese número hay que buscarlo separando la instrucción en campos. La instrucción `add` es de tipo R. Entonces tenemos:

```
Campo:        op     rs    rt    rd  shamt  funct
Binario:    000000 10001 10010 01000 00000 100000
Decimal:       0     17    18    8     0     32   
```

Los números 17, 18 y 8 corresponden a los registros `$s1`, `$s2` y `$t0`. Los 32 registros están numerados del 0 al 31. En esta instrucción el campo _shamt_ no se usa así que vale cero. La combinación de _op_ y _funct_ le indican a la CPU que se trata de un `add`.
La misma lógica se aplica a cualquier instrucción de formato R.

## Instrucciones con constantes

Las instrucciones de tipo I y J usan 16 y 26 bits respectivamente para dar un número. En las instrucciones de tipo I ese número es una constante y en las de tipo J representa una dirección de memoria.

Estos formatos imponen una restricción al número en ese campo. En el caso de `addi` por ejemplo que suma una constante a un registro y lo guarda en otro, la constante puede estar en el rango de $[-2^{15}, 2^{15} -1]$ o $[-32768, 32767]$ si usamos números con signo.

Las instrucciones de transferencia de datos como `lw` y `sw` también usan el formato I, ya que operan con dos registros y un _offset_ o corrimiento de una dirección base.

Veamos un par de ejemplos, en primer lugar la instrucción `lw $t0, 32($s3)` o `0x8e680020`.

```
Campo:        op     rs    rt        imm
Binario:    100011 10011 01000 0000000000100000
Decimal:      35     19    8          32       
```

Noten que el significado de _rt_ aquí ya no es un operando como en una instrucción de tipo R, si no que es el destino de la operación.

De manera similar la instrucción `addi $a0, $t7, 1200` o `0x21e404b0` se codifica de la siguiente manera.

```
Campo:        op     rs    rt        imm
Binario:    001000 01111 00100 0000010010110000
Decimal:       8     15    4         1200       
```

En este caso también _rt_ es el destino de la operación, el registro `$a0` es el registro número 4.

Por último veamos una instrucción de tipo J. La instrucción `j exit` o `0x0810000c` donde `exit` es una etiqueta o _label_ en el código de _assembler_ y su valor depende del resto del programa.

```
Campo:        op             addr
Binario:    000010 00000100000000000000001100
Decimal:       2            1048588       
```

El valor 1048588 es una dirección de memoria, pero en base 10 no es muy informativo, corresponde a `0x0010000c` en hexadecimal (completando con ceros para llegar a 32 bits). Pero ese no es la dirección de memoria a la que está saltando el programa, en realidad la etiqueta `exit` equivale a `0x00400030` según marca el simulador.

La CPU de MIPS tiene que transformar esos 26 bits de la instrucción tipo J en una dirección de 32 bits. Para hacerlo usa los 4 bits más significativos del PC (_program counter_) y desplaza a la izquierda 2 bits el campo _addr_ de 26 bits. Vamos por partes.

Primero desplazamos 2 bits a la izquierda.

```
  00000100000000000000001100  addr 26 bits
0000010000000000000000110000  addr << 2
```

Desplazar a la izquierda o hacer un _shift left_ de dos bits se ilustra arriba, los espacios vacíos se completan con ceros y ahora tenemos una dirección de 28 bits. Para llegar a 32 bits concatenamos al principio los 4 bits de mayor valor del PC, que en este caso son cuatro ceros.

```
     0000010000000000000000110000  addr << 2 (28 bits)
0000 0000010000000000000000110000  PC bits 31-28 + addr

   separando para leer mejor

00000000 01000000 00000000 00110000  
   00       40       00       30       pasando a hex
              00400030                 jump target de 32 bits
```

Y así tenemos de 26 bits una dirección de memoria de 32 bits para el _jump_. Esto se conoce en MIPS como direccionamiento pseudodirecto o _pseudodirect addressing_ y es uno de los cinco modos de direccionamiento de la CPU de MIPS.

## Conclusión

¿Por qué es importante todo esto? Bueno, como sabemos el ciclo de instrucción de una CPU puede dividirse en _fetch_, _decode_ y _execute_. La fase de _fetch_ trae una instrucción de la memoria, una cadena de 32 bits, unos y ceros. Y una parte de la CPU que conocemos como unidad de control se encarga de extraer sentido de estos unos y ceros en base a las reglas que vimos arriba en la fase de _decode_.
