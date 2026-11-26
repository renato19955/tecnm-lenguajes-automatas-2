# Bitácora de Requerimientos y Auditoría IA — Sesión 27
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Indexación de Arreglos en TAC  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Nombre Completo]  
**Matrícula:** [No. de Control]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Cálculo de desplazamientos base + offset  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue calcular correctamente los desplazamientos de arreglos unidimensionales y bidimensionales, controlar la generación de temporales y evitar desbordamientos en el arreglo de instrucciones TAC
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Aplanado de direcciones unidimensionales |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un generador de código
de tres direcciones (TAC) para la indexación
de arreglos en lenguaje C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa las funciones emitir_acceso()
y emitir_acceso_2d(), respetando las firmas
del archivo api_sesion27.h.

Para arreglos unidimensionales, multiplica
el índice por el ancho del elemento
y genera la lectura indexada.

Para arreglos bidimensionales, calcula
el índice lineal mediante i * filas + j,
según el contrato de la sesión.

Multiplica el índice lineal por el ancho
del elemento para obtener el desplazamiento.

Genera temporales únicos y consecutivos.

Emite dos instrucciones para arreglos 1D
y cuatro instrucciones para arreglos 2D.

Valida punteros NULL, dimensiones inválidas,
capacidad del arreglo y desbordamientos
del contador de temporales.

Utiliza snprintf() para evitar desbordamientos
y arreglos auxiliares para impedir que se
generen instrucciones incompletas.

Devuelve la cantidad de instrucciones emitidas
o 0 cuando los datos sean inválidos.

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
#include "api_sesion27.h"
#include <stdio.h>
#include <string.h>
#include <limits.h>

/* Validar los datos recibidos. */
static int validar(char lineas[][64], int max,
                   const char *base, int ancho,
                   int *temporales, int necesarias) {

    if (lineas == NULL || base == NULL ||
        temporales == NULL)
        return 0;

    if (max < necesarias || ancho <= 0)
        return 0;

    if (*temporales < 0 ||
        *temporales > INT_MAX - necesarias)
        return 0;

    return 1;
}

/* Generar acceso a un arreglo unidimensional. */
int emitir_acceso(char lineas[][64], int max,
                  const char *base, const char *indice,
                  int ancho, int *temporales) {

    if (!validar(lineas, max, base, ancho,
                 temporales, 2) || indice == NULL)
        return 0;

    int t = *temporales;

    char aux[2][64];
    int r[2];

    /* Calcular desplazamiento. */
    r[0] = snprintf(aux[0], 64,
                    "_t%d = %s * %d",
                    t, indice, ancho);

    /* Generar lectura indexada. */
    r[1] = snprintf(aux[1], 64,
                    "_t%d = %s[_t%d]",
                    t + 1, base, t);

    /* Validar las instrucciones. */
    for (int k = 0; k < 2; k++) {

        if (r[k] < 0 || r[k] >= 64)
            return 0;
    }

    /* Copiar las instrucciones generadas. */
    for (int k = 0; k < 2; k++)
        strcpy(lineas[k], aux[k]);

    *temporales += 2;

    return 2;
}

/* Generar acceso a un arreglo bidimensional. */
int emitir_acceso_2d(char lineas[][64], int max,
                     const char *base, const char *i,
                     const char *j, int filas,
                     int ancho, int *temporales) {

    if (!validar(lineas, max, base, ancho,
                 temporales, 4) ||
        i == NULL || j == NULL || filas <= 0)
        return 0;

    int t = *temporales;

    char aux[4][64];
    int r[4];

    /* Multiplicar el primer indice. */
    r[0] = snprintf(aux[0], 64,
                    "_t%d = %s * %d",
                    t, i, filas);

    /* Sumar el segundo indice. */
    r[1] = snprintf(aux[1], 64,
                    "_t%d = _t%d + %s",
                    t + 1, t, j);

    /* Calcular el desplazamiento por ancho. */
    r[2] = snprintf(aux[2], 64,
                    "_t%d = _t%d * %d",
                    t + 2, t + 1, ancho);

    /* Generar lectura indexada. */
    r[3] = snprintf(aux[3], 64,
                    "_t%d = %s[_t%d]",
                    t + 3, base, t + 2);

    /* Validar las instrucciones. */
    for (int k = 0; k < 4; k++) {

        if (r[k] < 0 || r[k] >= 64)
            return 0;
    }

    /* Copiar las instrucciones generadas. */
    for (int k = 0; k < 4; k++)
        strcpy(lineas[k], aux[k]);

    *temporales += 4;

    return 4;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Arreglos multidimensionales en TAC

### Salida del harness
```text
=== AUDITORIA SESION 27 - Desplazamientos para arreglos 1D y 2D ===
Caso                               esperado:              obtenido:              resultado
[ 1] 1D: i * 4                          esperado: _t0 = i * 4  obtenido: _t0 = i * 4  OK
[ 2] 1D: A[_t0]                         esperado: _t1 = A[_t0] obtenido: _t1 = A[_t0] OK
[ 3] 2D: i * 3                          esperado: _t0 = i * 3  obtenido: _t0 = i * 3  OK
[ 4] 2D: offset por ancho
     esperado: _t2 = _t1 * 4
     obtenido: _t2 = _t1 * 4
     -> OK
[ 5] 2D: M[_t2]                         esperado: _t3 = M[_t2] obtenido: _t3 = M[_t2] OK
[ 6] base NULL (sin acceso)             esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion27.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
La IA no omitió ningún caso borde de los evaluados. Sin embargo, el código no comprueba si los índices representan posiciones válidas dentro del arreglo original. Como mejora, se propone incorporar instrucciones TAC de verificación de límites antes de realizar la lectura. Ambas implementaciones superaron correctamente las seis pruebas del arnés
> 

---
**Firma digital:** [Usuario de GitHub]
