# Bitácora de Requerimientos y Auditoría IA — Sesión 12
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Atributos de Símbolos y Memoria  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam  
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Estructura de registro de símbolos  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue calcular correctamente el tamaño de cada tipo de dato, determinar los desplazamientos de memoria y respetar la alineación de los símbolos sin provocar desbordamientos numéricos
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Cálculo de desplazamientos y alineación |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un sistema de cálculo
de memoria para una tabla de símbolos en C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa las funciones ancho_de_tipo()
y asignar_offset(), respetando las firmas
del archivo api_sesion12.h.

Utiliza los siguientes tamaños en bytes:
char = 1
short = 2
int = 4
float = 4
double = 8
puntero = 8

Calcula el desplazamiento de cada símbolo
respetando la alineación correspondiente.

Devuelve -1 cuando exista un tipo desconocido,
un registro NULL o un cursor negativo.

Evita desbordamientos utilizando INT_MAX.

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
/* ia_sesion12_c.c
 * Sesion 12 - Fase Cientifico
 * Calculo de memoria y alineacion de simbolos.
 */

#include "api_sesion12.h"
#include <string.h>
#include <stddef.h>
#include <limits.h>

typedef struct {
    const char *nombre;
    int bytes;
} TipoDato;

static const TipoDato tipos_validos[] = {
    {"char", 1},
    {"short", 2},
    {"int", 4},
    {"float", 4},
    {"double", 8},
    {"puntero", 8}
};

int ancho_de_tipo(const char *tipo) {
    if (tipo == NULL)
        return -1;

    size_t cantidad = sizeof(tipos_validos) /
                      sizeof(tipos_validos[0]);

    for (size_t i = 0; i < cantidad; i++) {
        if (strcmp(tipo, tipos_validos[i].nombre) == 0)
            return tipos_validos[i].bytes;
    }

    return -1;
}

static int alinear_cursor(int cursor, int alineacion) {
    if (cursor < 0 || alineacion <= 0)
        return -1;

    int residuo = cursor % alineacion;

    if (residuo == 0)
        return cursor;

    int ajuste = alineacion - residuo;

    if (cursor > INT_MAX - ajuste)
        return -1;

    return cursor + ajuste;
}

int asignar_offset(Simbolo *simbolo, int cursor) {
    if (simbolo == NULL || cursor < 0)
        return -1;

    int ancho = ancho_de_tipo(simbolo->tipo);

    if (ancho == -1)
        return -1;

    int posicion = alinear_cursor(cursor, ancho);

    if (posicion == -1)
        return -1;

    if (posicion > INT_MAX - ancho)
        return -1;

    simbolo->offset = posicion;

    return posicion + ancho;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Alineación de tipos y tamaños en bytes

### Salida del harness
```text
=== AUDITORIA SESION 12 - Registro de simbolo y offsets alineados ===
Caso                               esperado:              obtenido:              resultado
[ 1] ancho_de_tipo("int")               esperado: 4            obtenido: 4            OK
[ 2] ancho_de_tipo("double")            esperado: 8            obtenido: 8            OK
[ 3] int en cursor 0                    esperado: 4            obtenido: 4            OK
[ 4] char e int: offset de b            esperado: 4            obtenido: 4            OK
[ 5] double desde cursor 4              esperado: 8            obtenido: 8            OK
[ 6] registro NULL                      esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion12.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 

---
**Firma digital:** [Usuario de GitHub]
