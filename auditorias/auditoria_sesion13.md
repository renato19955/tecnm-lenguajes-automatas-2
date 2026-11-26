# Bitácora de Requerimientos y Auditoría IA — Sesión 13
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Manejo de Ámbitos (Scope Global/Local)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero BAlam  
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Pila de tablas de símbolos (Stack Scopes)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue realizar la búsqueda de identificadores entre ámbitos locales y globales, respetar el ocultamiento de variables y evitar declaraciones duplicadas dentro del mismo ámbito, sin superar la capacidad del arreglo.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Ocultamiento de variables (Shadowing) |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un sistema de manejo
de ámbitos globales y locales en lenguaje C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa las funciones declarar() y buscar(),
respetando las firmas de api_sesion13.h.

Utiliza una estructura Ambito con un arreglo
de nombres, un contador y un puntero al padre.

La capacidad máxima de cada ámbito
debe ser AMBITO_MAX, equivalente a 8.

La función declarar() debe devolver:
1 si el identificador fue registrado.
0 si ya existe en el ámbito actual.
-1 si el ámbito está lleno o los datos
recibidos son inválidos.

La función buscar() debe recorrer los ámbitos
desde el bloque actual hasta la raíz.

Respeta el ocultamiento de variables,
dando prioridad al ámbito más cercano.

Devuelve el nombre del ámbito encontrado
o NULL si el identificador no existe.

Valida punteros NULL, nombres vacíos
y límites del arreglo.

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
#include "api_sesion13.h"
#include <string.h>
#include <stddef.h>

/* Verifica si un nombre existe en el ambito. */
static int existe(const Ambito *ambito,
                  const char *nombre) {

    for (int i = 0;
         i < ambito->total && i < AMBITO_MAX;
         i++) {

        if (ambito->nombres[i] != NULL &&
            strcmp(ambito->nombres[i], nombre) == 0) {

            return 1;
        }
    }

    return 0;
}

/* Declara un identificador en el ambito actual. */
int declarar(Ambito *ambito, const char *nombre) {

    if (ambito == NULL || nombre == NULL ||
        nombre[0] == '\0')
        return -1;

    if (ambito->total < 0 ||
        ambito->total > AMBITO_MAX)
        return -1;

    if (existe(ambito, nombre))
        return 0;

    if (ambito->total == AMBITO_MAX)
        return -1;

    ambito->nombres[ambito->total] = nombre;

    ambito->total++;

    return 1;
}

/* Busca un identificador desde el ambito actual. */
const char *buscar(const Ambito *ambito,
                   const char *nombre) {

    if (nombre == NULL || nombre[0] == '\0')
        return NULL;

    while (ambito != NULL) {

        if (existe(ambito, nombre))
            return ambito->nombre;

        ambito = ambito->padre;
    }

    return NULL;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Búsqueda jerárquica de identificadores

### Salida del harness
```text
=== AUDITORIA SESION 13 - Pila de ambitos y busqueda jerarquica ===
Caso                               esperado:              obtenido:              resultado
[ 1] declarar(&g, "x")                  esperado: 1            obtenido: 1            OK
[ 2] buscar en el mismo ambito          esperado: global       obtenido: global       OK
[ 3] buscar subiendo al padre           esperado: global       obtenido: global       OK
[ 4] shadowing en el hijo               esperado: bloque       obtenido: bloque       OK
[ 5] tres niveles hasta la raiz         esperado: raiz         obtenido: raiz         OK
[ 6] nombre no declarado                esperado: (nulo)       obtenido: (nulo)       OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion13.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
La IA no omitió ningún caso borde de los evaluados. Sin embargo, no contempla referencias circulares entre los ámbitos padre, lo que podría provocar un ciclo infinito durante la búsqueda. Como mejora, se propone incorporar una validación para detectar referencias circulares. Ambas implementaciones superaron correctamente las seis pruebas del arnés.
> 

---
**Firma digital:** [Usuario de GitHub]
