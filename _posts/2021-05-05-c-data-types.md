---
layout: post
title:  "Tipos de datos en C"
author: "Santiago Trini"
categories: c
tags: [algo,c]
image: c.png
date:   2021-05-05 00:00:00 -0300
---

|Tipo    |Valores     |Operadores comunes |Literales
|--------|------------|-------------------|---------------------|
|`int`   |$\mathbb{Z}$|`+` `*` `-` `/` `%`|`124` `42` `5355`    |
|`double`|$\mathbb{R}$|`+` `*` `-` `/`    |`3.14` `2.5` `1.2e11`|
|`bool`  |$\{ 0, 1 \}$|`!` `&&` `||`      |`true` `false`       |
|`char`  |carácteres  |                   |`'a'` `'4'` `'!'`    |


## Números enteros

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

## Coma flotante

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

## Booleanos

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
