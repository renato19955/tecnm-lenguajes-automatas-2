# Bitácora de Requerimientos y Auditoría IA — Sesión 17
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Comprobación Semántica de Retornos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam  
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Validación de tipo de sentencia return  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue comprobar la compatibilidad entre los tipos de retorno, identificar las rutas de ejecución sin retorno y evitar que las funciones de tipo void devolvieran valores.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Rutas de ejecución con retorno garantizado |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un sistema de comprobación
semántica de retornos en lenguaje C.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa la función diagnosticar_retorno(),
respetando la firma de api_sesion17.h.

Verifica la compatibilidad entre el tipo
de la función y el tipo de la expresión.

Permite conversiones hacia tipos de mayor
amplitud según la jerarquía establecida.

Evita que las funciones void devuelvan valores.

Comprueba que todas las rutas de una función
no-void tengan una sentencia return.

Valida punteros NULL, tipos desconocidos
y cantidades incoherentes de rutas.

Devuelve los siguientes valores:
0: Retorno correcto.
1: Tipo de retorno incompatible.
2: Existe una ruta sin retorno.
-1: Argumentos inválidos.

Prioriza la incompatibilidad de tipos
antes de comprobar las rutas de retorno.

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
/* ia_sesion17_c.c
 * Sesion 17 - Fase Cientifico
 * Comprobacion semantica de retornos.
 */

#include "api_sesion17.h"
#include <string.h>
#include <stddef.h>

/* Determina el nivel de amplitud de cada tipo. */
static int nivel_tipo(const char *tipo) {

    if (tipo == NULL)
        return -1;

    if (strcmp(tipo, "void") == 0)
        return 0;

    if (strcmp(tipo, "char") == 0)
        return 1;

    if (strcmp(tipo, "short") == 0)
        return 2;

    if (strcmp(tipo, "int") == 0)
        return 3;

    if (strcmp(tipo, "long") == 0)
        return 4;

    if (strcmp(tipo, "float") == 0)
        return 5;

    if (strcmp(tipo, "double") == 0)
        return 6;

    return -1;
}

/* Comprueba los tipos y las rutas de retorno. */
int diagnosticar_retorno(const char *tipo_funcion,
                         const char *tipo_expresion,
                         int rutas_totales,
                         int rutas_con_return) {

    /* Validacion de argumentos. */
    if (tipo_funcion == NULL || tipo_expresion == NULL)
        return -1;

    if (rutas_totales <= 0 ||
        rutas_con_return < 0 ||
        rutas_con_return > rutas_totales)
        return -1;

    int destino = nivel_tipo(tipo_funcion);
    int origen = nivel_tipo(tipo_expresion);

    /* Tipos desconocidos. */
    if (destino == -1 || origen == -1)
        return -1;

    /* Las funciones void no pueden devolver valores. */
    if (destino == 0) {
        if (origen != 0)
            return 1;

        return 0;
    }

    /* Una funcion no-void debe devolver un valor. */
    if (origen == 0)
        return 1;

    /* Comprobar compatibilidad de tipos. */
    if (origen > destino)
        return 1;

    /* Comprobar que todas las rutas tengan return. */
    if (rutas_con_return < rutas_totales)
        return 2;

    return 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Funciones sin retorno o retornos mixtos

### Salida del harness
```text
=== AUDITORIA SESION 17 - Validacion de retornos y rutas ===
Caso                               esperado:              obtenido:              resultado
[ 1] int con int y dos rutas            esperado: 0            obtenido: 0            OK
[ 2] void devuelve valor                esperado: 1            obtenido: 1            OK
[ 3] ruta sin retorno                   esperado: 2            obtenido: 2            OK
[ 4] double acepta int                  esperado: 0            obtenido: 0            OK
[ 5] int no acepta double               esperado: 1            obtenido: 1            OK
[ 6] cero rutas de ejecucion            esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion17.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 

---
**Firma digital:** [Usuario de GitHub]
