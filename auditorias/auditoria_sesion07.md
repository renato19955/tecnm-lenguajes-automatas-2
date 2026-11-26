# Bitácora de Requerimientos y Auditoría IA — Sesión 07
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Evaluación de Expresiones Aritméticas  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam
**Matrícula:** 24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Evaluador recursivo de operaciones  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue controlar las operaciones recursivas, detectar divisiones entre cero y propagar correctamente los errores de los nodos hijos hacia el nodo padre.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Manejo estricto de división entre cero |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un evaluador de expresiones
aritméticas en lenguaje C utilizando GCC.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa la función evaluar(ASTNode *n, int *error)
respetando la firma de api_sesion07.h.

Evalúa recursivamente las operaciones de suma,
resta, multiplicación, división y módulo.

Devuelve error 1 cuando exista una división
o módulo entre cero y error 2 cuando el nodo
o la operación sean inválidos.

Propaga los errores de los nodos hijos sin
continuar evaluando los nodos padres.
Valida los punteros NULL.

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
/* ia_sesion07_c.c - Fase 2: El Cientifico
 * Evaluador recursivo de expresiones aritmeticas.
 */

#include "api_sesion07.h"
#include <stddef.h>

int evaluar(ASTNode *n, int *error) {
    int izquierda;
    int derecha;

    if (error == NULL) {
        return 0;
    }

    if (*error != 0) {
        return 0;
    }

    if (n == NULL) {
        *error = 2;
        return 0;
    }

    if (n->tipo == NODO_NUM) {
        return n->valor;
    }

    if (n->tipo != NODO_BINOP) {
        *error = 2;
        return 0;
    }

    izquierda = evaluar(n->izq, error);

    if (*error != 0) {
        return 0;
    }

    derecha = evaluar(n->der, error);

    if (*error != 0) {
        return 0;
    }

    switch (n->op) {
        case '+':
            return izquierda + derecha;

        case '-':
            return izquierda - derecha;

        case '*':
            return izquierda * derecha;

        case '/':
            if (derecha == 0) {
                *error = 1;
                return 0;
            }
            return izquierda / derecha;

        case '%':
            if (derecha == 0) {
                *error = 1;
                return 0;
            }
            return izquierda % derecha;

        default:
            *error = 2;
            return 0;
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Expresiones complejas y división por 0

### Salida del harness
```text
=== AUDITORIA SESION 07 - Evaluador recursivo con deteccion de errores ===
Caso                               esperado:              obtenido:              resultado
[ 1] (2+3)*4                            esperado: 20           obtenido: 20           OK
[ 2] 7/0 -> error 1                     esperado: 1            obtenido: 1            OK
[ 3] 7%0 -> error 1                     esperado: 1            obtenido: 1            OK
[ 4] arbol NULL -> error 2              esperado: 2            obtenido: 2            OK
[ 5] 10-2-3                             esperado: 5            obtenido: 5            OK
[ 6] (8/2)+1                            esperado: 5            obtenido: 5            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion07.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 

---
**Firma digital:** [Usuario de GitHub]
