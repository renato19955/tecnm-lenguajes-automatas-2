# Bitácora de Requerimientos y Auditoría IA — Sesión 05
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Recorridos del AST y Notación Polaca  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam 
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Impresión en Postorden / RPN  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue controlar el recorrido recursivo del árbol, respetar el orden izquierdo, derecho y raíz, y evitar que los caracteres escritos superaran la capacidad del buffer.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Invariantes de recorrido recursivo |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando una práctica de recorridos
de árboles AST en lenguaje C utilizando GCC.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa la función postorden() respetando
la firma del archivo api_sesion05.h.

Recorre el árbol en el orden izquierdo,
derecho y raíz para generar la notación
polaca inversa (RPN).

Escribe el resultado en un buffer, separando
los elementos por un espacio y respetando
su capacidad máxima.

Controla árboles NULL y garantiza que
la cadena termine con '\0'.

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
#include "api_sesion05.h"
#include <stdio.h>
#include <string.h>

static void generar(ASTNode *n, char *buf, int cap, int *pos) {
    if (n == NULL || *pos >= cap - 1) {
        return;
    }

    if (n->tipo == NODO_BINOP) {
        generar(n->izq, buf, cap, pos);
        generar(n->der, buf, cap, pos);
    }

    char token[32];

    if (n->tipo == NODO_NUM) {
        snprintf(token, sizeof(token), "%d", n->valor);
    } else {
        snprintf(token, sizeof(token), "%c", n->op);
    }

    int longitud = (int)strlen(token);

    if (*pos > 0 && *pos < cap - 1) {
        buf[(*pos)++] = ' ';
    }

    for (int i = 0; i < longitud && *pos < cap - 1; i++) {
        buf[(*pos)++] = token[i];
    }

    buf[*pos] = '\0';
}

void postorden(ASTNode *n, char *buf, int cap) {
    if (buf == NULL || cap <= 0) {
        return;
    }

    buf[0] = '\0';

    int pos = 0;

    generar(n, buf, cap, &pos);
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Validación contra desbordamiento de pila

### Salida del harness
```text
=== AUDITORIA SESION 05 - Recorrido postorden y RPN ===
Caso                               esperado:              obtenido:              resultado
[ 1] hoja 42                            esperado: 42           obtenido: 42           OK
[ 2] 3+(5*2)                            esperado: 3 5 2 * +    obtenido: 3 5 2 * +    OK
[ 3] arbol NULL                         esperado:              obtenido:              OK
[ 4] (1+2)*(3+4)
     esperado: 1 2 + 3 4 + *
     obtenido: 1 2 + 3 4 + *
     -> OK
[ 5] 8-3                                esperado: 8 3 -        obtenido: 8 3 -        OK

RESULTADO: 5/5 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion05.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 

---
**Firma digital:** [Usuario de GitHub]
