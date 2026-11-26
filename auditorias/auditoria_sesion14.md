# Bitácora de Requerimientos y Auditoría IA — Sesión 14
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Declaración vs Uso de Variables  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam  
**Matrícula:** 24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Detección de variables no declaradas  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue comprobar que las variables estuvieran declaradas antes de utilizarlas, evitar nombres duplicados y controlar la capacidad máxima de la tabla de símbolos.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Invariante de definición previa |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando una tabla de símbolos
para controlar la declaración y uso de
variables en lenguaje C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa las funciones declarar() y usar(),
respetando las firmas de api_sesion14.h.

Verifica que una variable esté declarada
antes de permitir su uso.

Evita declaraciones duplicadas y realiza
comparaciones exactas mediante strcmp().

Valida punteros NULL, nombres vacíos
y la capacidad máxima de la tabla.

La función declarar() debe devolver:
1: Declaración exitosa.
0: Variable duplicada.
-1: Argumentos inválidos o tabla llena.

La función usar() debe devolver:
1: Variable declarada.
0: Variable inexistente o argumento inválido.

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
/* ia_sesion14_c.c
 * Sesion 14 - Fase Cientifico
 * Administracion de declaraciones y uso de variables.
 */

#include "api_sesion14.h"
#include <string.h>
#include <stddef.h>

/* Valida la estructura y el nombre recibido. */
static int datos_validos(const Tabla *tabla,
                         const char *nombre) {

    if (tabla == NULL || nombre == NULL)
        return 0;

    if (nombre[0] == '\0')
        return 0;

    if (tabla->total < 0 || tabla->total > TABLA_MAX)
        return 0;

    return 1;
}

/* Localiza un identificador mediante comparacion exacta. */
static int buscar_identificador(const Tabla *tabla,
                                const char *nombre) {

    for (int i = 0; i < tabla->total; i++) {
        if (tabla->nombres[i] != NULL &&
            strcmp(tabla->nombres[i], nombre) == 0) {
            return i;
        }
    }

    return -1;
}

/* Inserta una variable solamente si no existe. */
int declarar(Tabla *tabla, const char *nombre) {

    if (!datos_validos(tabla, nombre))
        return -1;

    if (buscar_identificador(tabla, nombre) >= 0)
        return 0;

    if (tabla->total >= TABLA_MAX)
        return -1;

    tabla->nombres[tabla->total] = nombre;
    tabla->total++;

    return 1;
}

/* Comprueba si una variable fue declarada. */
int usar(const Tabla *tabla, const char *nombre) {

    if (!datos_validos(tabla, nombre))
        return 0;

    return buscar_identificador(tabla, nombre) >= 0 ? 1 : 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Uso antes de declarar y redeclaración

### Salida del harness
```text
=== AUDITORIA SESION 14 - Declaracion previa y redeclaracion ===
Caso                               esperado:              obtenido:              resultado
[ 1] usar sin declarar                  esperado: 0            obtenido: 0            OK
[ 2] usar despues de declarar           esperado: 1            obtenido: 1            OK
[ 3] redeclarar el mismo nombre         esperado: 0            obtenido: 0            OK
[ 4] total tras redeclarar              esperado: 1            obtenido: 1            OK
[ 5] prefijo no es el mismo nombre      esperado: 0            obtenido: 0            OK
[ 6] declarar nombre vacio              esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion14.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 

---
**Firma digital:** [Usuario de GitHub]
