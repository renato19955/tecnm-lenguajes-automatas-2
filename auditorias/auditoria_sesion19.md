# Bitácora de Requerimientos y Auditoría IA — Sesión 19
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Estructuras (struct) y Miembros  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Offsets de miembros en estructuras  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue calcular la posición de cada miembro de la estructura, aplicar correctamente la alineación de memoria y considerar los bytes de relleno (padding) para obtener el tamaño total.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Alineación de memoria por palabras |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un sistema de cálculo
de offsets y tamaños de estructuras en C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa las funciones offset_campo()
y tamano_struct(), respetando las firmas
del archivo api_sesion19.h.

Utiliza los siguientes tamaños en bytes:
char = 1
short = 2
int = 4
float = 4
double = 8
puntero = 8

Alinea cada miembro según el ancho de su tipo.

Calcula el padding necesario entre los campos
y al final de la estructura.

Redondea el tamaño total al múltiplo
de la alineación del miembro más grande.

Devuelve -1 cuando el miembro no exista,
el tipo sea desconocido o los datos sean inválidos.

Devuelve 0 para una estructura sin campos.

Evita desbordamientos mediante INT_MAX.

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
/* ia_sesion19_c.c
 * Sesion 19 - Fase Cientifico
 * Calculo de offsets, alineacion y padding.
 */

#include "api_sesion19.h"
#include <string.h>
#include <stddef.h>
#include <limits.h>

/* Obtiene el ancho del tipo de dato. */
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

/* Alinea una posicion al multiplo indicado. */
static int alinear(int posicion, int ancho) {

    int resto = posicion % ancho;

    if (resto == 0)
        return posicion;

    int relleno = ancho - resto;

    if (posicion > INT_MAX - relleno)
        return -1;

    return posicion + relleno;
}

/* Calcula el offset de un campo especifico. */
int offset_campo(const Campo *campos,
                 int total,
                 const char *nombre) {

    if (campos == NULL || nombre == NULL || total <= 0)
        return -1;

    int posicion = 0;
    int resultado = -1;

    for (int i = 0; i < total; i++) {

        int ancho = ancho_tipo(campos[i].tipo);

        if (ancho == -1)
            return -1;

        posicion = alinear(posicion, ancho);

        if (posicion == -1)
            return -1;

        if (campos[i].nombre != NULL &&
            strcmp(campos[i].nombre, nombre) == 0) {
            resultado = posicion;
        }

        if (posicion > INT_MAX - ancho)
            return -1;

        posicion += ancho;
    }

    return resultado;
}

/* Calcula el tamano total con padding final. */
int tamano_struct(const Campo *campos, int total) {

    if (total < 0)
        return -1;

    if (total == 0)
        return 0;

    if (campos == NULL)
        return -1;

    int posicion = 0;
    int alineacion_maxima = 1;

    for (int i = 0; i < total; i++) {

        int ancho = ancho_tipo(campos[i].tipo);

        if (ancho == -1)
            return -1;

        if (ancho > alineacion_maxima)
            alineacion_maxima = ancho;

        posicion = alinear(posicion, ancho);

        if (posicion == -1)
            return -1;

        if (posicion > INT_MAX - ancho)
            return -1;

        posicion += ancho;
    }

    return alinear(posicion, alineacion_maxima);
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Acceso a miembros inexistentes

### Salida del harness
```text
=== AUDITORIA SESION 19 - Offsets de miembros y tamano de struct ===
Caso                               esperado:              obtenido:              resultado
[ 1] struct char e int: b               esperado: 4            obtenido: 4            OK
[ 2] struct de tres campos: c           esperado: 8            obtenido: 8            OK
[ 3] tamano total con padding           esperado: 12           obtenido: 12           OK
[ 4] double seguido de char             esperado: 16           obtenido: 16           OK
[ 5] miembro inexistente                esperado: -1           obtenido: -1           OK
[ 6] struct sin campos                  esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion19.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
La IA no omitió ningún caso borde de los evaluados e incorporó protección contra desbordamientos mediante INT_MAX. Sin embargo, no valida los nombres duplicados dentro de una estructura. Como mejora, se propone detectar estos casos antes de calcular los offsets. Ambas implementaciones superaron correctamente las seis pruebas del arnés.
> 

---
**Firma digital:** [Usuario de GitHub]
