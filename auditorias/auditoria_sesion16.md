# Bitácora de Requerimientos y Auditoría IA — Sesión 16
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Firmas de Función y Parámetros  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Registro de prototipos y aridad  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue verificar que la cantidad de argumentos coincidiera con los parámetros declarados, localizar correctamente las funciones registradas y distinguir entre una función inexistente y una llamada con aridad incorrecta.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Coincidencia estricta de parámetros |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un verificador de firmas
de funciones y parámetros en lenguaje C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa la función verificar_llamada(),
respetando la firma de api_sesion16.h.

Realiza una búsqueda exacta del nombre
de la función mediante strcmp().

Compara la cantidad de argumentos recibidos
con la aridad registrada en el catálogo.

Permite funciones con cero parámetros.

Valida punteros NULL, catálogos vacíos
y cantidades negativas de argumentos.

Devuelve los siguientes valores:
1: Llamada válida.
0: Cantidad incorrecta de argumentos.
-1: Función inexistente o argumentos inválidos.

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
/* ia_sesion16_c.c
 * Sesion 16 - Fase Cientifico
 * Verificacion de firmas y parametros formales.
 */

#include "api_sesion16.h"
#include <string.h>
#include <stddef.h>

/* Busca una funcion por su nombre exacto. */
static const Funcion *buscar_funcion(
    const Funcion *catalogo,
    int total,
    const char *nombre
) {
    for (int i = 0; i < total; i++) {
        if (catalogo[i].nombre != NULL &&
            strcmp(catalogo[i].nombre, nombre) == 0) {
            return &catalogo[i];
        }
    }

    return NULL;
}

/* Comprueba que la llamada sea valida. */
int verificar_llamada(const Funcion *catalogo,
                      int total,
                      const char *nombre,
                      int nargs) {

    /* Validacion de argumentos. */
    if (catalogo == NULL || nombre == NULL)
        return -1;

    if (total <= 0 || nargs < 0)
        return -1;

    /* Localizar la firma registrada. */
    const Funcion *funcion =
        buscar_funcion(catalogo, total, nombre);

    /* La funcion no existe. */
    if (funcion == NULL)
        return -1;

    /* Comparar la cantidad de argumentos. */
    if (funcion->aridad == nargs)
        return 1;

    /* La funcion existe, pero la aridad es incorrecta. */
    return 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Llamadas con número incorrecto de args

### Salida del harness
```text
=== AUDITORIA SESION 16 - Registro de prototipos y aridad ===
Caso                               esperado:              obtenido:              resultado
[ 1] suma con 3 argumentos              esperado: 1            obtenido: 1            OK
[ 2] suma con 2 argumentos              esperado: 0            obtenido: 0            OK
[ 3] suma con 4 argumentos              esperado: 0            obtenido: 0            OK
[ 4] funcion no registrada              esperado: -1           obtenido: -1           OK
[ 5] main sin argumentos                esperado: 1            obtenido: 1            OK
[ 6] nombre NULL                        esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion16.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 

---
**Firma digital:** [Usuario de GitHub]
