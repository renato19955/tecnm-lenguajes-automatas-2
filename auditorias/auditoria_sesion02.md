# Bitácora de Requerimientos y Auditoría IA — Sesión 02
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Gestión en GitHub Web y Nodos Dinámicos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam  
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Lista enlazada de tokens en Heap  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue reservar memoria dinámica para cada nodo, enlazar correctamente los tokens y liberar la memoria sin perder las referencias a los siguientes nodos también fue necesario controlar la longitud de los lexemas y validar los punteros NULL
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Gestión de memoria dinámica C99/Rust |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando una lista enlazada
de tokens utilizando memoria dinámica
en lenguaje C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa las funciones agregar_token(),
contar_tokens() y liberar_lista(), respetando
las firmas del archivo api_sesion02.h.

Utiliza la estructura NodoToken con un
arreglo lexema de 16 caracteres y un
puntero al siguiente nodo.

Reserva memoria para cada nodo con malloc().

Agrega los nuevos tokens al final de la lista,
conservando el orden de inserción.

Limita los lexemas a 15 caracteres y garantiza
su terminación con el carácter nulo.

Si malloc() falla o el lexema es NULL,
devuelve la cabeza original.

La función contar_tokens() debe devolver
la cantidad de nodos existentes.

Libera todos los nodos mediante free(),
guardando previamente el siguiente nodo
para evitar accesos a memoria liberada.

Controla correctamente los punteros NULL.

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
#include "api_sesion02.h"
#include <stdio.h>
#include <stdlib.h>

/* Agregar un token al final de la lista. */
NodoToken *agregar_token(NodoToken *cabeza,
                         const char *lexema) {

    if (lexema == NULL)
        return cabeza;

    NodoToken *nuevo = malloc(sizeof *nuevo);

    if (nuevo == NULL)
        return cabeza;

    snprintf(nuevo->lexema,
             sizeof nuevo->lexema,
             "%.15s", lexema);

    nuevo->siguiente = NULL;

    if (cabeza == NULL)
        return nuevo;

    NodoToken *actual = cabeza;

    while (actual->siguiente != NULL) {

        actual = actual->siguiente;
    }

    actual->siguiente = nuevo;

    return cabeza;
}

/* Contar los tokens existentes. */
int contar_tokens(NodoToken *cabeza) {

    int total = 0;

    for (NodoToken *p = cabeza;
         p != NULL;
         p = p->siguiente) {

        total++;
    }

    return total;
}

/* Liberar la memoria de todos los nodos. */
void liberar_lista(NodoToken *cabeza) {

    while (cabeza != NULL) {

        NodoToken *sig = cabeza->siguiente;

        free(cabeza);

        cabeza = sig;
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Prevención de Use-After-Free y cadenas nulas

### Salida del harness
```text
=== AUDITORIA SESION 02 - Lista enlazada de tokens en heap ===
Caso                               esperado:              obtenido:              resultado
[ 1] agregar_token(NULL,"if")           esperado: if           obtenido: if           OK
[ 2] contar_tokens(3 tokens)            esperado: 3            obtenido: 3            OK
[ 3] orden: segundo token               esperado: dos          obtenido: dos          OK
[ 4] ultimo->siguiente == NULL          esperado: 1            obtenido: 1            OK
[ 5] contar_tokens(NULL)                esperado: 0            obtenido: 0            OK
[ 6] liberar_lista(NULL)                esperado: sin crash    obtenido: sin crash    OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion02.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
La IA no omitió ningún caso borde de los evaluados. Sin embargo, después de liberar la lista, el puntero original puede conservar una referencia a memoria liberada. Como medida preventiva, se recomienda asignar NULL a la cabeza después de ejecutar liberar_lista(). Ambas implementaciones superaron correctamente las seis pruebas del arnés.
> 

---
**Firma digital:** [Usuario de GitHub]
