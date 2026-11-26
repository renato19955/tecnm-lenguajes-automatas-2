# Bitácora de Requerimientos y Auditoría IA — Sesión 18
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Arreglos y Verificación de Límites  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam 
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Cálculo de tamaño de arreglo en memoria  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue calcular el espacio ocupado por un arreglo según su tipo de dato, validar índices negativos y evitar desbordamientos numéricos o accesos fuera de los límites establecidos.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Indexación entera no negativa |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un sistema de cálculo
de memoria y verificación de límites
de arreglos en lenguaje C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa las funciones bytes_arreglo()
e indice_en_rango(), respetando las firmas
del archivo api_sesion18.h.

Utiliza los siguientes tamaños en bytes:
char = 1
short = 2
int = 4
float = 4
double = 8
puntero = 8

Calcula el tamaño total multiplicando
el ancho del tipo por el número de elementos.

Rechaza tipos desconocidos, punteros NULL
y dimensiones negativas devolviendo -1.

Un arreglo de cero elementos ocupa cero bytes.

Evita desbordamientos utilizando LONG_MAX.

Valida que los índices sean mayores o iguales
a cero y menores al número de elementos.

Devuelve 1 si el índice es válido
y 0 si está fuera de rango.

Salida:
Entrega únicamente código modular en C
y su análisis de complejidad.
No incluyas la función main().
El código debe compilar con
-Wall -Wextra -Werror.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion18_c.c
 * Sesion 18 - Fase Cientifico
 * Calculo de memoria y verificacion de indices.
 */

#include "api_sesion18.h"
#include <string.h>
#include <stddef.h>
#include <limits.h>

/* Determina el ancho del tipo de dato. */
static int ancho_tipo(const char *tipo) {

    if (tipo == NULL)
        return -1;

    if (strcmp(tipo, "char") == 0)
        return 1;

    if (strcmp(tipo, "short") == 0)
        return 2;

    if (strcmp(tipo, "int") == 0)
        return 4;

    if (strcmp(tipo, "float") == 0)
        return 4;

    if (strcmp(tipo, "double") == 0)
        return 8;

    if (strcmp(tipo, "puntero") == 0)
        return 8;

    return -1;
}

/* Calcula el espacio total de un arreglo. */
long bytes_arreglo(const char *tipo, int elementos) {

    if (tipo == NULL || elementos < 0)
        return -1;

    int ancho = ancho_tipo(tipo);

    if (ancho == -1)
        return -1;

    /* Evitar desbordamiento del resultado. */
    if ((long)elementos > LONG_MAX / ancho)
        return -1;

    return (long)elementos * ancho;
}

/* Verifica si el indice pertenece al arreglo. */
int indice_en_rango(int indice, int elementos) {

    if (elementos <= 0)
        return 0;

    if (indice < 0)
        return 0;

    if (indice >= elementos)
        return 0;

    return 1;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Límites estáticos y dimensiones inválidas

### Salida del harness
```text
=== AUDITORIA SESION 18 - Tamano de arreglo y limites ===
Caso                               esperado:              obtenido:              resultado
[ 1] int[10]                            esperado: 40           obtenido: 40           OK
[ 2] double[3]                          esperado: 24           obtenido: 24           OK
[ 3] int[-4]                            esperado: -1           obtenido: -1           OK
[ 4] indice 0 de 5                      esperado: 1            obtenido: 1            OK
[ 5] indice 5 de 5                      esperado: 0            obtenido: 0            OK
[ 6] indice negativo                    esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion18.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 

---
**Firma digital:** [Usuario de GitHub]
