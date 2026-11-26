# Bitácora de Requerimientos y Auditoría IA — Sesión 21
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Fundamentos de Código de 3 Direcciones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam  
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Representación en Cuádruplos (op,a,b,r)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue generar temporales únicos, incrementar correctamente el contador y construir cuádruplos con el formato establecido, respetando la capacidad del buffer y evitando desbordamientos de memoria.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Generación de temporales únicos (_t0..) |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un generador de código
de tres direcciones (TAC) en lenguaje C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa las funciones nuevo_temporal()
y emitir_cuadruplo(), respetando las firmas
del archivo api_sesion21.h.

Genera temporales únicos con el formato
_t0, _t1, _t2 y así sucesivamente.

Incrementa el contador después de generar
correctamente cada temporal.

Representa las instrucciones con el formato:
(op, a1, a2, res).

Sustituye los argumentos NULL por "_".

Utiliza snprintf() para respetar la capacidad
del buffer y evitar desbordamientos.

Valida punteros NULL, capacidades inválidas
y desbordamientos del contador con INT_MAX.

No utilices memoria dinámica ni variables globales.

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
/* ia_sesion21_c.c
 * Sesion 21 - Fase Cientifico
 * Generacion de temporales y cuadruplos.
 */

#include "api_sesion21.h"
#include <stdio.h>
#include <stddef.h>
#include <limits.h>

/* Sustituye un argumento NULL por "_". */
static const char *argumento_seguro(const char *valor) {

    if (valor == NULL)
        return "_";

    return valor;
}

/* Genera el siguiente temporal. */
void nuevo_temporal(char *dst, int cap, int *contador) {

    if (dst == NULL || cap <= 0)
        return;

    dst[0] = '\0';

    if (contador == NULL)
        return;

    if (*contador < 0 || *contador == INT_MAX)
        return;

    int escritos = snprintf(dst, (size_t)cap,
                            "_t%d", *contador);

    if (escritos < 0 || escritos >= cap) {
        dst[0] = '\0';
        return;
    }

    (*contador)++;
}

/* Genera un cuadruplo con formato fijo. */
void emitir_cuadruplo(char *dst, int cap,
                      const char *op,
                      const char *a1,
                      const char *a2,
                      const char *res) {

    if (dst == NULL || cap <= 0)
        return;

    dst[0] = '\0';

    const char *operador = argumento_seguro(op);
    const char *arg1 = argumento_seguro(a1);
    const char *arg2 = argumento_seguro(a2);
    const char *resultado = argumento_seguro(res);

    int escritos = snprintf(dst, (size_t)cap,
                            "(%s, %s, %s, %s)",
                            operador, arg1, arg2, resultado);

    if (escritos < 0 || escritos >= cap)
        dst[0] = '\0';
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Expresiones aritméticas compuestas

### Salida del harness
```text
=== AUDITORIA SESION 21 - Cuadruplos y temporales unicos ===
Caso                               esperado:              obtenido:              resultado
[ 1] nuevo_temporal(_t0)                esperado: _t0          obtenido: _t0          OK
[ 2] nuevo_temporal(_t7)                esperado: _t7          obtenido: _t7          OK
[ 3] cuadruplo de a + b
     esperado: (+, a, b, _t0)
     obtenido: (+, a, b, _t0)
     -> OK
[ 4] cuadruplo de _t0 * c
     esperado: (*, _t0, c, _t1)
     obtenido: (*, _t0, c, _t1)
     -> OK
[ 5] asignacion simple                  esperado: (=, a, _, x) obtenido: (=, a, _, x) OK
[ 6] goto sin operandos
     esperado: (goto, L1, _, _)
     obtenido: (goto, L1, _, _)
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion21.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
La IA no omitió ningún caso borde de los evaluados e incorporó validaciones para buffers insuficientes y desbordamientos del contador. Sin embargo, el arnés no comprueba directamente el incremento consecutivo de temporales ni las capacidades insuficientes. Como mejora, se propone agregar pruebas para estos casos. Ambas implementaciones superaron correctamente las seis pruebas del arnés.
> 

---
**Firma digital:** [Usuario de GitHub]
