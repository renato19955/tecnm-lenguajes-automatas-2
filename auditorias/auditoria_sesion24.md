# Bitácora de Requerimientos y Auditoría IA — Sesión 24
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Control de Flujo: Sentencia if-else  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam  
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Saltos condicionales y evaluación booleana  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue diferenciar los saltos condicionales AND y OR, respetar la evaluación por cortocircuito y controlar la capacidad del arreglo para evitar desbordamientos de memoria.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Cortocircuito lógico en expresiones |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un generador de saltos
condicionales para expresiones booleanas en C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa la función emitir_condicion(),
respetando la firma de api_sesion24.h.

Para el operador AND (&), genera instrucciones
con el formato:
IFFALSE a GOTO L9

Para el operador OR (|), genera instrucciones
con el formato:
IFTRUE a GOTO L9

Respeta la evaluación por cortocircuito.

Genera un salto independiente por cada operando,
utilizando la misma etiqueta de destino.

Si el segundo operando es NULL,
genera únicamente una instrucción.

Valida punteros NULL, operadores desconocidos
y capacidades inválidas.

Utiliza snprintf() para evitar desbordamientos.

No superes el máximo de líneas permitido
ni los 63 caracteres por instrucción.

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
/* ia_sesion24_c.c
 * Sesion 24 - Fase Cientifico
 * Saltos condicionales con cortocircuito.
 */

#include "api_sesion24.h"
#include <stdio.h>

/* Funcion auxiliar para generar un salto. */
static int emitir_salto(char destino[64],
                        const char *instruccion,
                        const char *operando,
                        const char *etiqueta) {

    int n = snprintf(destino, 64, "%s %s GOTO %s",
                     instruccion, operando, etiqueta);

    if (n < 0 || n >= 64) {
        destino[0] = '\0';
        return 0;
    }

    return 1;
}

int emitir_condicion(char lineas[][64], int max,
                     const char *a, const char *b,
                     char op, const char *etiqueta) {

    if (lineas == NULL || max <= 0 ||
        a == NULL || etiqueta == NULL)
        return 0;

    const char *instruccion;

    switch (op) {

        case '&':
            instruccion = "IFFALSE";
            break;

        case '|':
            instruccion = "IFTRUE";
            break;

        default:
            return 0;
    }

    int emitidas = 0;

    const char *operandos[2] = {a, b};

    for (int i = 0; i < 2 && emitidas < max; i++) {

        if (operandos[i] == NULL)
            break;

        if (!emitir_salto(lineas[emitidas],
                          instruccion,
                          operandos[i],
                          etiqueta))
            break;

        emitidas++;
    }

    return emitidas;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Condiciones compuestas con AND y OR

### Salida del harness
```text
=== AUDITORIA SESION 24 - Saltos condicionales con cortocircuito ===
Caso                               esperado:              obtenido:              resultado
[ 1] a && b (primer salto)
     esperado: IFFALSE a GOTO L9
     obtenido: IFFALSE a GOTO L9
     -> OK
[ 2] a && b (segundo salto)
     esperado: IFFALSE b GOTO L9
     obtenido: IFFALSE b GOTO L9
     -> OK
[ 3] a && b (dos saltos)                esperado: 2            obtenido: 2            OK
[ 4] a || b (primer salto)
     esperado: IFTRUE a GOTO L9
     obtenido: IFTRUE a GOTO L9
     -> OK
[ 5] a || b (segundo salto)
     esperado: IFTRUE b GOTO L9
     obtenido: IFTRUE b GOTO L9
     -> OK
[ 6] condicion simple (b NULL)
     esperado: IFFALSE a GOTO L0
     obtenido: IFFALSE a GOTO L0
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion24.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
La IA no omitió ningún caso borde de los evaluados. Sin embargo, cuando el arreglo tiene capacidad para una sola instrucción y existen dos operandos, puede generar una condición incompleta. Como mejora, se propone comprobar previamente que exista espacio suficiente para ambas instrucciones. Las dos implementaciones superaron correctamente las seis pruebas del arnés.
> 

---
**Firma digital:** [Usuario de GitHub]
