# Bitácora de Requerimientos y Auditoría IA — Sesión 22
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Generación de TAC para Expresiones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam 
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Traductor de nodos AST a TAC  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue recorrer el árbol sintáctico en postorden, conservar la precedencia de los operadores, administrar los temporales y evitar superar la capacidad del arreglo de instrucciones TAC.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Conservación de precedencia de operadores |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un traductor de árboles
sintácticos AST a código de tres direcciones
(TAC) en lenguaje C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa la función generar_tac(),
respetando la firma de api_sesion22.h.

Realiza un recorrido recursivo en postorden.

Procesa primero los hijos izquierdo y derecho
y posteriormente el operador.

Las hojas no deben generar instrucciones.

Utiliza nuevo_temporal() para generar
identificadores únicos: _t0, _t1, _t2...

Genera instrucciones con el formato:
_t0 = a + b

Conserva la precedencia de los operadores
establecida en el árbol sintáctico.

Respeta el máximo de líneas permitido
y el límite de 63 caracteres por línea.

Valida punteros NULL y evita accesos
fuera de los límites del arreglo.

Devuelve la cantidad de instrucciones emitidas.

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
/* ia_sesion22_c.c
 * Sesion 22 - Fase Cientifico
 * Generacion de TAC mediante postorden.
 */

#include "api_sesion22.h"
#include <stdio.h>
#include <stddef.h>

typedef struct {
    char nombre[64][16];
    int usados;
} Pool;

/* Generador de temporales proporcionado. */
const char *nuevo_temporal(Pool *p, int *contador) {

    if (p->usados >= 64)
        return "_t?";

    snprintf(p->nombre[p->usados],
             sizeof p->nombre[0],
             "_t%d", (*contador)++);

    return p->nombre[p->usados++];
}

typedef struct {
    Pool pool;
    int contador;
    int emitidas;
    int max;
    char (*lineas)[64];
} Contexto;

/* Recorre el arbol en postorden. */
static const char *traducir(const Nodo *nodo,
                            Contexto *ctx) {

    if (nodo == NULL)
        return NULL;

    /* Las hojas no generan instrucciones. */
    if (nodo->tipo == 'H')
        return nodo->valor;

    if (nodo->tipo != 'B')
        return NULL;

    /* Primero el hijo izquierdo. */
    const char *izq = traducir(nodo->izq, ctx);

    if (izq == NULL)
        return NULL;

    /* Despues el hijo derecho. */
    const char *der = traducir(nodo->der, ctx);

    if (der == NULL)
        return NULL;

    /* Verificar la capacidad disponible. */
    if (ctx->emitidas >= ctx->max ||
        ctx->pool.usados >= 64)
        return NULL;

    /* Generar un temporal unico. */
    const char *temp = nuevo_temporal(
        &ctx->pool, &ctx->contador);

    /* Emitir la instruccion TAC. */
    snprintf(ctx->lineas[ctx->emitidas], 64,
             "%.15s = %.15s %c %.15s",
             temp, izq, nodo->op, der);

    ctx->emitidas++;

    return temp;
}

int generar_tac(const Nodo *raiz,
                char lineas[][64], int max) {

    if (raiz == NULL || lineas == NULL || max <= 0)
        return 0;

    Contexto ctx = {0};

    ctx.lineas = lineas;
    ctx.max = max;

    traducir(raiz, &ctx);

    return ctx.emitidas;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Expresiones anidadas con paréntesis

### Salida del harness
```text
=== AUDITORIA SESION 22 - Traductor de AST a TAC con precedencia ===
Caso                               esperado:              obtenido:              resultado
[ 1] a + b * c                          esperado: _t0 = b * c  obtenido: _t0 = b * c  OK
[ 2] a + b * c (2a linea)
     esperado: _t1 = a + _t0
     obtenido: _t1 = a + _t0
     -> OK
[ 3] (a + b) * c
     esperado: _t1 = _t0 * c
     obtenido: _t1 = _t0 * c
     -> OK
[ 4] x * (y - z) + w
     esperado: _t2 = _t1 + w
     obtenido: _t2 = _t1 + w
     -> OK
[ 5] hoja sin operaciones               esperado: 0            obtenido: 0            OK
[ 6] (a + b) * (c - d)
     esperado: _t2 = _t0 * _t1
     obtenido: _t2 = _t0 * _t1
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion22.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
La IA no omitió ningún caso borde de los evaluados. Sin embargo, cuando se agota la capacidad de los temporales o del arreglo de instrucciones, puede devolver una traducción incompleta sin indicar explícitamente el error. Como mejora, se propone incorporar un mecanismo de diagnóstico. Ambas implementaciones superaron correctamente las seis pruebas del arnés.
> 

---
**Firma digital:** [Usuario de GitHub]
