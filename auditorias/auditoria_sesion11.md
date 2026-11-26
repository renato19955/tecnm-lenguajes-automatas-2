# Bitácora de Requerimientos y Auditoría IA — Sesión 11
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Arquitectura de la Tabla de Símbolos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam 
**Matrícula:** 24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Tabla Hash con encadenamiento  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue manejar las colisiones mediante listas enlazadas, controlar la asignación de memoria dinámica y evitar la inserción de símbolos duplicados sin perder los datos almacenados.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Función de dispersión y colisiones O(1) |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando una tabla de símbolos
en lenguaje C utilizando GCC.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa las funciones hash_djb2()
e insertar_simbolo(), respetando las firmas
del archivo api_sesion11.h.

Utiliza el algoritmo de dispersión djb2
y listas enlazadas para resolver colisiones.

Evita la inserción de nombres duplicados
y conserva el tipo y offset de cada símbolo.

Valida los punteros NULL, la cantidad de
cubetas y la longitud de los nombres.

Devuelve los siguientes valores al insertar:
1: Símbolo insertado correctamente.
0: Símbolo duplicado.
-1: Argumentos inválidos o error de memoria.

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
//* Sesion 11 - Implementacion del Cientifico
 * Tabla de simbolos mediante hashing djb2.
 */

#include "api_sesion11.h"
#include <stdint.h>
#include <stdlib.h>
#include <string.h>

#define MAX_NOMBRE 16

unsigned int hash_djb2(const char *clave) {
    uint32_t hash = 5381u;

    if (clave == NULL)
        return 0u;

    while (*clave != '\0') {
        hash = ((hash << 5) + hash)
             + (unsigned char)*clave;

        clave++;
    }

    return (unsigned int)hash;
}

static Simbolo *crear_simbolo(const char *nombre,
                            int tipo,
                            int offset) {

    Simbolo *nuevo = malloc(sizeof(*nuevo));

    if (nuevo == NULL)
        return NULL;

    strcpy(nuevo->nombre, nombre);

    nuevo->tipo = tipo;
    nuevo->offset = offset;
    nuevo->sig = NULL;

    return nuevo;
}

int insertar_simbolo(Simbolo *tabla[],
                    unsigned int cubetas,
                    const char *nombre,
                    int tipo,
                    int offset) {

    if (tabla == NULL || nombre == NULL || cubetas == 0)
        return -1;

    size_t longitud = strlen(nombre);

    if (longitud == 0 || longitud >= MAX_NOMBRE)
        return -1;

    unsigned int indice = hash_djb2(nombre) % cubetas;

    Simbolo **actual = &tabla[indice];

    while (*actual != NULL) {
        if (strcmp((*actual)->nombre, nombre) == 0)
            return 0;

        actual = &(*actual)->sig;
    }

    Simbolo *nuevo = crear_simbolo(nombre, tipo, offset);

    if (nuevo == NULL)
        return -1;

    *actual = nuevo;

    return 1;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Manejo de claves duplicadas y colisiones

### Salida del harness
```text
=== AUDITORIA SESION 11 - Tabla hash con encadenamiento ===
Caso                               esperado:              obtenido:              resultado
[ 1] hash_djb2("")                      esperado: 5381         obtenido: 5381         OK
[ 2] hash_djb2("abc")                   esperado: 193485963    obtenido: 193485963    OK
[ 3] insertar x dos veces               esperado: 0            obtenido: 0            OK
[ 4] dos claves, una cubeta             esperado: 2            obtenido: 2            OK
[ 5] offset de beta en cadena           esperado: 4            obtenido: 4            OK
[ 6] insertar en tabla NULL             esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion11.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 

---
**Firma digital:** [Usuario de GitHub]
