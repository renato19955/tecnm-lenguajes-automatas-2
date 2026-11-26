# Bitácora de Requerimientos y Auditoría IA — Sesión 28
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Optimización: Constant Folding  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Plegado y simplificación de constantes  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue distinguir los valores enteros de las variables, aplicar correctamente las identidades algebraicas y evitar errores provocados por divisiones entre cero o desbordamientos aritméticos.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Reducción estática en compilación |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un optimizador de código
de tres direcciones (TAC) en lenguaje C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa la función plegar(), respetando
la firma del archivo api_sesion28.h.

Identifica si ambos operandos son números
enteros completos mediante strtol().

Realiza operaciones de suma, resta,
multiplicación y división cuando ambos
operandos sean constantes válidas.

Aplica las siguientes identidades algebraicas:
x + 0 = x
x * 1 = x
x * 0 = 0

Conserva la instrucción original cuando
no sea posible realizar la optimización.

Evita divisiones entre cero y desbordamientos
aritméticos utilizando las funciones
de comprobación de GCC.

Valida punteros NULL y capacidades inválidas.

Utiliza snprintf() para evitar desbordamientos
en el búfer de salida.

Output:
Entrega únicamente código modular en C
y su análisis de complejidad.
No incluyas la función main().
El código debe compilar con
-Wall -Wextra -Werror.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion28.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <limits.h>

/* Verificar si una cadena es un entero completo. */
static int es_entero(const char *texto, long *valor) {

    if (texto == NULL || *texto == '\0')
        return 0;

    const char *p = texto;

    if (*p == '+' || *p == '-')
        p++;

    if (*p == '\0')
        return 0;

    while (*p != '\0') {

        if (*p < '0' || *p > '9')
            return 0;

        p++;
    }

    char *fin;

    errno = 0;

    long numero = strtol(texto, &fin, 10);

    if (errno == ERANGE || *fin != '\0')
        return 0;

    *valor = numero;

    return 1;
}

/* Realizar operaciones sin desbordamientos. */
static int calcular(long x, long y,
                    const char *op, long *resultado) {

    if (strcmp(op, "+") == 0) {

        return !__builtin_add_overflow(x, y, resultado);
    }

    if (strcmp(op, "-") == 0) {

        return !__builtin_sub_overflow(x, y, resultado);
    }

    if (strcmp(op, "*") == 0) {

        return !__builtin_mul_overflow(x, y, resultado);
    }

    if (strcmp(op, "/") == 0) {

        if (y == 0 || (x == LONG_MIN && y == -1))
            return 0;

        *resultado = x / y;

        return 1;
    }

    return 0;
}

/* Plegado de constantes e identidades algebraicas. */
void plegar(char *dst, int cap, const char *res,
            const char *op, const char *a,
            const char *b) {

    if (dst == NULL || cap <= 0)
        return;

    if (res == NULL || op == NULL ||
        a == NULL || b == NULL) {

        dst[0] = '\0';

        return;
    }

    long x = 0;
    long y = 0;
    long resultado = 0;

    int entero_a = es_entero(a, &x);
    int entero_b = es_entero(b, &y);

    int escritos;

    /* Plegado de constantes. */
    if (entero_a && entero_b &&
        calcular(x, y, op, &resultado)) {

        escritos = snprintf(dst, (size_t)cap,
                            "%s = %ld", res, resultado);
    }

    /* Identidad: x + 0 = x. */
    else if (strcmp(op, "+") == 0 &&
             entero_b && y == 0) {

        escritos = snprintf(dst, (size_t)cap,
                            "%s = %s", res, a);
    }

    /* Identidad: x * 1 = x. */
    else if (strcmp(op, "*") == 0 &&
             entero_b && y == 1) {

        escritos = snprintf(dst, (size_t)cap,
                            "%s = %s", res, a);
    }

    /* Identidad: x * 0 = 0. */
    else if (strcmp(op, "*") == 0 &&
             entero_b && y == 0) {

        escritos = snprintf(dst, (size_t)cap,
                            "%s = 0", res);
    }

    /* Conservar la instruccion original. */
    else {

        escritos = snprintf(dst, (size_t)cap,
                            "%s = %s %s %s",
                            res, a, op, b);
    }

    /* Verificar la capacidad del buffer. */
    if (escritos < 0 || escritos >= cap)
        dst[0] = '\0';
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Simplificación de expresiones constantes

### Salida del harness
```text
=== AUDITORIA SESION 28 - Plegado de constantes e identidades ===
Caso                               esperado:              obtenido:              resultado
[ 1] 3 + 5                              esperado: _t0 = 8      obtenido: _t0 = 8      OK
[ 2] 2 * 3                              esperado: _t0 = 6      obtenido: _t0 = 6      OK
[ 3] 7 - 10                             esperado: _t0 = -3     obtenido: _t0 = -3     OK
[ 4] x + 0                              esperado: _t0 = x      obtenido: _t0 = x      OK
[ 5] x + y (sin plegar)                 esperado: _t0 = x + y  obtenido: _t0 = x + y  OK
[ 6] 10 / 0 (no plegar)                 esperado: _t0 = 10 / 0 obtenido: _t0 = 10 / 0 OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion28.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
La IA no omitió ningún caso borde de los evaluados. Sin embargo, no contempla las identidades algebraicas con la constante del lado izquierdo, como 0 + x, 1 * x y 0 * x. Como mejora, se propone incorporar estas condiciones para ampliar la optimización. Ambas implementaciones superaron correctamente las seis pruebas del harness.
> 

---
**Firma digital:** [Usuario de GitHub]
