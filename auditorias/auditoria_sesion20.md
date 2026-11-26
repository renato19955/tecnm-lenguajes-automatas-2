# Bitácora de Requerimientos y Auditoría IA — Sesión 20
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Hito 2: Analizador Semántico y Scopes  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam 
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Integración de Tabla de Símbolos y AST  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue recorrer los ámbitos anidados, identificar las variables visibles, respetar el sombreado de identificadores y evitar contar símbolos repetidos o acceder a posiciones inválidas de memoria.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Contrato de Validación Semántica Integral |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un analizador semántico
con ámbitos anidados en lenguaje C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa las funciones profundidad_de()
y contar_visibles(), respetando las firmas
del archivo api_sesion20.h.

Recorre los ámbitos desde el bloque actual
hasta el ámbito raíz.

Respeta el sombreado de variables, dando
prioridad al ámbito más interno.

La función profundidad_de() debe devolver:
0: Símbolo encontrado en el ámbito actual.
n: Nivel de profundidad del símbolo.
-1: Símbolo inexistente.
-2: Argumentos NULL.

La función contar_visibles() debe contar
los símbolos visibles sin repetir nombres
y devolver -2 si el ámbito es NULL.

Respeta el límite AMBITO_MAX de 8 símbolos
por ámbito y evita accesos fuera de rango.

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
/* Sesion 20 - Fase Cientifico */

#include "api_sesion20.h"
#include <string.h>
#include <stddef.h>

/* Busca un simbolo dentro de un ambito. */
static int existe_en_ambito(const Ambito *a,
                            const char *nombre) {

    if (a == NULL || nombre == NULL)
        return 0;

    for (int i = 0; i < a->total && i < AMBITO_MAX; i++) {

        if (a->simbolos[i] != NULL &&
            strcmp(a->simbolos[i], nombre) == 0)
            return 1;
    }

    return 0;
}

/* Determina la profundidad de un simbolo. */
int profundidad_de(const Ambito *actual,
                   const char *nombre) {

    if (actual == NULL || nombre == NULL)
        return -2;

    int nivel = 0;

    while (actual != NULL) {

        if (existe_en_ambito(actual, nombre))
            return nivel;

        actual = actual->padre;
        nivel++;
    }

    return -1;
}

/* Cuenta los simbolos visibles sin repetir. */
int contar_visibles(const Ambito *actual) {

    if (actual == NULL)
        return -2;

    int contador = 0;

    for (const Ambito *a = actual; a != NULL; a = a->padre) {

        for (int i = 0; i < a->total && i < AMBITO_MAX; i++) {

            const char *nombre = a->simbolos[i];

            if (nombre == NULL)
                continue;

            int repetido = 0;

            /* Buscar en ambitos mas internos. */
            for (const Ambito *b = actual; b != a; b = b->padre) {

                if (existe_en_ambito(b, nombre)) {
                    repetido = 1;
                    break;
                }
            }

            /* Buscar repetidos en el mismo ambito. */
            for (int j = 0; j < i && !repetido; j++) {

                if (a->simbolos[j] != NULL &&
                    strcmp(a->simbolos[j], nombre) == 0)
                    repetido = 1;
            }

            if (!repetido)
                contador++;
        }
    }

    return contador;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Casos complejos de shadowing y ámbito

### Salida del harness
```text
=== AUDITORIA SESION 20 - Resolucion de nombres y visibilidad ===
Caso                               esperado:              obtenido:              resultado
[ 1] nombre en el ambito actual         esperado: 0            obtenido: 0            OK
[ 2] nombre dos niveles arriba          esperado: 2            obtenido: 2            OK
[ 3] shadowing gana el interno          esperado: 0            obtenido: 0            OK
[ 4] nombre no visible                  esperado: -1           obtenido: -1           OK
[ 5] visibles sin repetir               esperado: 3            obtenido: 3            OK
[ 6] ambito NULL                        esperado: -2           obtenido: -2           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion20.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
La IA no omitió ningún caso borde de los evaluados. Sin embargo, no contempla cadenas de ámbitos con referencias circulares, lo que podría generar un ciclo infinito. Como mejora, se propone detectar referencias circulares antes de recorrer los ámbitos. Ambas implementaciones superaron correctamente las seis pruebas del arnés.
> 

---
**Firma digital:** [Usuario de GitHub]
