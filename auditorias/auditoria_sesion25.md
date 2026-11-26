# Bitácora de Requerimientos y Auditoría IA — Sesión 25
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Control de Flujo: Bucles while y for  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam 
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Estructura de ciclo en instrucciones TAC  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue organizar correctamente las etiquetas de inicio y fin, colocar la actualización del iterador antes del salto de regreso y controlar la capacidad del arreglo para evitar desbordamientos de memoria.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Garantía de actualización de iterador |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un generador de código
de tres direcciones (TAC) para ciclos
while y for en lenguaje C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa las funciones emitir_while()
y emitir_for(), respetando las firmas
del archivo api_sesion25.h.

El ciclo while debe generar cuatro instrucciones:
marca de inicio, salto condicional,
GOTO de regreso y marca de salida.

El ciclo for debe generar la inicialización,
marca de inicio, condición, actualización
del iterador, GOTO y marca de salida.

Si inicio o post son NULL,
omite las instrucciones correspondientes.

Genera dos etiquetas únicas por cada ciclo.

Coloca la actualización del iterador
antes del GOTO de regreso.

Valida punteros NULL, capacidad del arreglo
y desbordamientos del contador de etiquetas.

Utiliza snprintf() y un arreglo auxiliar
para evitar instrucciones incompletas.

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
/* ia_sesion25_c.c
 * Sesion 25 - Fase Cientifico
 * Generacion de TAC para while y for.
 */

#include "api_sesion25.h"
#include <stdio.h>
#include <string.h>
#include <limits.h>

/* Funcion auxiliar para generar ciclos. */
static void generar_ciclo(char lineas[][64], int max,
                           int *n, const char *inicio,
                           const char *cond,
                           const char *post,
                           int *etiquetas) {

    if (lineas == NULL || n == NULL ||
        cond == NULL || etiquetas == NULL)
        return;

    if (max <= 0 || *n < 0 || *n > max)
        return;

    if (*etiquetas < 0 || *etiquetas > INT_MAX - 2)
        return;

    int necesarias = 4;

    if (inicio != NULL)
        necesarias++;

    if (post != NULL)
        necesarias++;

    if (max - *n < necesarias)
        return;

    int li = *etiquetas;
    int lf = li + 1;

    /* Almacenamiento temporal de instrucciones. */
    char aux[6][64];

    int k = 0;
    int r;

    /* Inicializacion opcional. */
    if (inicio != NULL) {

        r = snprintf(aux[k], 64, "%s", inicio);

        if (r < 0 || r >= 64)
            return;

        k++;
    }

    /* Marca de inicio. */
    snprintf(aux[k++], 64, "L%d:", li);

    /* Salto condicional. */
    r = snprintf(aux[k], 64,
                 "IFFALSE %s GOTO L%d", cond, lf);

    if (r < 0 || r >= 64)
        return;

    k++;

    /* Actualizacion opcional. */
    if (post != NULL) {

        r = snprintf(aux[k], 64, "%s", post);

        if (r < 0 || r >= 64)
            return;

        k++;
    }

    /* Salto de regreso. */
    snprintf(aux[k++], 64, "GOTO L%d", li);

    /* Marca de salida. */
    snprintf(aux[k++], 64, "L%d:", lf);

    /* Agregar las instrucciones al arreglo. */
    for (int i = 0; i < k; i++) {

        strcpy(lineas[*n], aux[i]);

        (*n)++;
    }

    *etiquetas += 2;
}

/* Generar ciclo while. */
void emitir_while(char lineas[][64], int max,
                  int *n, const char *cond,
                  int *etiquetas) {

    generar_ciclo(lineas, max, n, NULL,
                  cond, NULL, etiquetas);
}

/* Generar ciclo for. */
void emitir_for(char lineas[][64], int max,
                int *n, const char *inicio,
                const char *cond, const char *post,
                int *etiquetas) {

    generar_ciclo(lineas, max, n, inicio,
                  cond, post, etiquetas);
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Bucles anidados y condiciones de parada

### Salida del harness
```text
=== AUDITORIA SESION 25 - Esqueleto TAC de while y for ===
Caso                               esperado:              obtenido:              resultado
[ 1] while: marca de inicio             esperado: L0:          obtenido: L0:          OK
[ 2] while: condicion
     esperado: IFFALSE a < b GOTO L1
     obtenido: IFFALSE a < b GOTO L1
     -> OK
[ 3] dos ciclos sin colision            esperado: L2:          obtenido: L2:          OK
[ 4] for: inicializacion                esperado: i = 0        obtenido: i = 0        OK
[ 5] for: actualizacion                 esperado: i = i + 1    obtenido: i = i + 1    OK
[ 6] for sin post (NULL)                esperado: 5            obtenido: 5            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion25.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
La IA no omitió ningún caso borde de los evaluados. Sin embargo, cuando existe espacio insuficiente, la función termina sin proporcionar un diagnóstico del error. Como mejora, se propone incorporar un mecanismo que notifique esta situación. Ambas implementaciones superaron correctamente las seis pruebas del arnés.
> 

---
**Firma digital:** [Usuario de GitHub]
