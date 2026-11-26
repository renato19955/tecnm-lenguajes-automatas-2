# Bitácora de Requerimientos y Auditoría IA — Sesión 23
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Etiquetas y Saltos Incondicionales  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam 
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Generador de etiquetas e instrucciones goto  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue generar etiquetas únicas, controlar el incremento del contador y diferenciar las instrucciones de salto de las marcas de destino, evitando desbordamientos en el buffer.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Trazabilidad del flujo de control |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un generador de etiquetas
y saltos incondicionales en lenguaje C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa las funciones nueva_etiqueta()
y emitir_flujo(), respetando las firmas
del archivo api_sesion23.h.

Genera etiquetas únicas y consecutivas
con el formato L0, L1, L2...

Incrementa el contador después de generar
correctamente cada etiqueta.

Si es_salto es distinto de cero,
genera una instrucción con el formato:
GOTO L2

Si es_salto es igual a cero,
genera una marca con el formato:
L2:

Cuando la etiqueta sea NULL,
devuelve una cadena vacía en el buffer.

Utiliza snprintf() para respetar la capacidad
del buffer y evitar desbordamientos.

Valida punteros NULL, capacidades inválidas
y el límite del contador mediante INT_MAX.

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
#include "api_sesion23.h"
#include <stdio.h>
#include <limits.h>

/* Genera una etiqueta unica. */
void nueva_etiqueta(char *dst, int cap, int *contador) {

    if (dst == NULL || cap <= 0)
        return;

    dst[0] = '\0';

    if (contador == NULL || *contador == INT_MAX)
        return;

    int n = snprintf(dst, (size_t)cap,
                     "L%d", *contador);

    if (n < 0 || n >= cap) {
        dst[0] = '\0';
        return;
    }

    (*contador)++;
}

/* Genera un salto o una marca de destino. */
void emitir_flujo(char *dst, int cap,
                  const char *etiqueta, int es_salto) {

    if (dst == NULL || cap <= 0)
        return;

    if (etiqueta == NULL) {
        dst[0] = '\0';
        return;
    }

    int n;

    if (es_salto != 0)
        n = snprintf(dst, (size_t)cap,
                     "GOTO %s", etiqueta);
    else
        n = snprintf(dst, (size_t)cap,
                     "%s:", etiqueta);

    if (n < 0 || n >= cap)
        dst[0] = '\0';
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Saltos adelante y atrás estructurados

### Salida del harness
```text
=== AUDITORIA SESION 23 - Etiquetas y saltos incondicionales ===
Caso                               esperado:              obtenido:              resultado
[ 1] nueva_etiqueta(L0)                 esperado: L0           obtenido: L0           OK
[ 2] nueva_etiqueta(L5)                 esperado: L5           obtenido: L5           OK
[ 3] emitir_flujo(GOTO L2)              esperado: GOTO L2      obtenido: GOTO L2      OK
[ 4] emitir_flujo(marca L2)             esperado: L2:          obtenido: L2:          OK
[ 5] emitir_flujo(NULL)                 esperado:              obtenido:              OK
[ 6] nueva_etiqueta(L10)                esperado: L10          obtenido: L10          OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion23.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
La IA no contempló los contadores negativos, lo que podría generar etiquetas como L-1. Como mejora, se propone validar que el contador sea mayor o igual a cero antes de generar una etiqueta. Ambas implementaciones superaron correctamente las seis pruebas del arnés.
> 

---
**Firma digital:** [Usuario de GitHub]
