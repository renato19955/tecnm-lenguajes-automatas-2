# Bitácora de Requerimientos y Auditoría IA — Sesión 15
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Bloques Anidados y Ciclo de Vida  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam  
**Matrícula:** 24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Operaciones Push y Pop de ámbito  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue administrar los ámbitos anidados mediante punteros, liberar correctamente la memoria al cerrar un bloque y recuperar el ámbito padre sin eliminar el ámbito raíz.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Liberación de memoria al cerrar bloque |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un sistema de administración
de ámbitos anidados en lenguaje C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa las funciones push_ambito()
y pop_ambito(), respetando las firmas
del archivo api_sesion15.h.

Utiliza malloc() para crear nuevos ámbitos
y enlázalos con su ámbito padre.

Utiliza free() para liberar la memoria
cuando se cierre un bloque.

Valida los punteros NULL y evita liberar
el ámbito raíz.

La función push_ambito() debe devolver
el nuevo ámbito o NULL ante un error.

La función pop_ambito() debe devolver
el ámbito padre o NULL si recibe la raíz
o un puntero inválido por ser NULL.

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
/* ia_sesion15_c.c
 * Sesion 15 - Fase Cientifico
 * Administracion de bloques anidados.
 */

#include "api_sesion15.h"
#include <stdlib.h>
#include <stddef.h>

/* Crea un nuevo ambito enlazado al padre. */
Ambito *push_ambito(Ambito *actual, const char *nombre) {

    if (actual == NULL || nombre == NULL)
        return NULL;

    Ambito *nuevo = malloc(sizeof(*nuevo));

    if (nuevo == NULL)
        return NULL;

    nuevo->nombre = nombre;
    nuevo->padre = actual;

    return nuevo;
}

/* Cierra el ambito actual y recupera su padre. */
Ambito *pop_ambito(Ambito *actual) {

    if (actual == NULL)
        return NULL;

    /* La raiz no se libera. */
    if (actual->padre == NULL)
        return NULL;

    Ambito *padre = actual->padre;

    free(actual);

    return padre;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Aislamiento de variables fuera de bloque

### Salida del harness
```text
=== AUDITORIA SESION 15 - Push y pop de ambitos ===
Caso                               esperado:              obtenido:              resultado
[ 1] nombre del ambito nuevo            esperado: if           obtenido: if           OK
[ 2] profundidad con dos push           esperado: 3            obtenido: 3            OK
[ 3] pop devuelve el padre              esperado: 1            obtenido: 1            OK
[ 4] pop sobre la raiz                  esperado: 1            obtenido: 1            OK
[ 5] push con padre NULL                esperado: 1            obtenido: 1            OK
[ 6] profundidad tras pop               esperado: 2            obtenido: 2            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion15.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 

---
**Firma digital:** [Usuario de GitHub]
