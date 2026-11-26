# Bitácora de Requerimientos y Auditoría IA — Sesión 26
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** TAC para Llamadas a Funciones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam 
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Instrucciones param, call y return  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue organizar los parámetros en el orden correcto, contabilizar los argumentos de cada función y conservar los temporales de las llamadas anidadas, evitando desbordamientos en el arreglo de instrucciones.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Convención de paso de argumentos TAC |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un generador de código
de tres direcciones (TAC) para llamadas
a funciones y retornos en lenguaje C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa las funciones emitir_llamada()
y emitir_return(), respetando las firmas
del archivo api_sesion26.h.

Genera una instrucción PARAM por cada
argumento distinto de NULL.

Respeta el orden de los argumentos.

Genera llamadas con el formato:
_t0 = CALL f, 2

Si resultado es NULL, genera únicamente:
CALL f, 2

Permite llamadas anidadas conservando
los temporales proporcionados.

Genera instrucciones de retorno:
RETURN _t0

Si el valor es NULL, genera solo RETURN.

Valida punteros NULL y la capacidad
del arreglo de instrucciones.

Utiliza snprintf() para evitar desbordamientos.

Salida:
Entrega únicamente código modular en C
y su análisis de complejidad.
No incluyas la función main().
El código debe compilar con
-Wall -Wextra -Werror.
```

---

## 3. Código Devuelto por el Modelo
```
/* ia_sesion26_c.c
 * Sesion 26 - Fase Cientifico
 * Llamadas, parametros y retorno.
 */

#include "api_sesion26.h"
#include <stdio.h>
#include <string.h>

int emitir_llamada(char lineas[][64], int max,
                    int *n, const char *funcion,
                    const char *a1, const char *a2,
                    const char *resultado) {

    if (lineas == NULL || n == NULL ||
        funcion == NULL || max <= 0)
        return 0;

    if (*n < 0 || *n > max)
        return 0;

    int argumentos = 0;

    if (a1 != NULL)
        argumentos++;

    if (a2 != NULL)
        argumentos++;

    int necesarias = argumentos + 1;

    if (max - *n < necesarias)
        return 0;

    char aux[3][64];

    int k = 0;
    int r;

    /* Primer argumento. */
    if (a1 != NULL) {

        r = snprintf(aux[k], 64, "PARAM %s", a1);

        if (r < 0 || r >= 64)
            return 0;

        k++;
    }

    /* Segundo argumento. */
    if (a2 != NULL) {

        r = snprintf(aux[k], 64, "PARAM %s", a2);

        if (r < 0 || r >= 64)
            return 0;

        k++;
    }

    /* Llamada con o sin resultado. */
    if (resultado != NULL) {

        r = snprintf(aux[k], 64,
                     "%s = CALL %s, %d",
                     resultado, funcion, argumentos);

    } else {

        r = snprintf(aux[k], 64,
                     "CALL %s, %d",
                     funcion, argumentos);
    }

    if (r < 0 || r >= 64)
        return 0;

    k++;

    /* Agregar las instrucciones al arreglo. */
    for (int i = 0; i < k; i++) {

        strcpy(lineas[*n], aux[i]);

        (*n)++;
    }

    return k;
}

/* Generar retorno con o sin valor. */
void emitir_return(char *dst, int cap,
                   const char *valor) {

    if (dst == NULL || cap <= 0)
        return;

    int r;

    if (valor != NULL) {

        r = snprintf(dst, (size_t)cap,
                     "RETURN %s", valor);

    } else {

        r = snprintf(dst, (size_t)cap,
                     "RETURN");
    }

    if (r < 0 || r >= cap)
        dst[0] = '\0';
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Llamadas anidadas f(g(x))

### Salida del harness
```text
=== AUDITORIA SESION 26 - Llamadas, parametros y retorno ===
Caso                               esperado:              obtenido:              resultado
[ 1] PARAM x                            esperado: PARAM x      obtenido: PARAM x      OK
[ 2] PARAM y                            esperado: PARAM y      obtenido: PARAM y      OK
[ 3] CALL con dos argumentos
     esperado: _t0 = CALL f, 2
     obtenido: _t0 = CALL f, 2
     -> OK
[ 4] llamada anidada f(g(x))
     esperado: _t1 = CALL f, 1
     obtenido: _t1 = CALL f, 1
     -> OK
[ 5] CALL sin argumentos
     esperado: _t0 = CALL p, 0
     obtenido: _t0 = CALL p, 0
     -> OK
[ 6] RETURN con valor                   esperado: RETURN _t0   obtenido: RETURN _t0   OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion26.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
Un posible problema era generar llamadas incompletas cuando el arreglo no tuviera espacio suficiente. Para prevenirlo, se incorporó una validación previa de capacidad y un arreglo auxiliar que almacena las instrucciones antes de agregarlas. También se contemplaron llamadas sin argumentos y retornos sin valor.
> 

---
**Firma digital:** [Usuario de GitHub]
