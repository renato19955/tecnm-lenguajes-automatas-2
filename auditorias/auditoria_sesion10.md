# Bitácora de Requerimientos y Auditoría IA — Sesión 10
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Hito 1: Mini-Evaluador de Expresiones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Integración AST y Tipado Estático  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue respetar la precedencia de los operadores, controlar los paréntesis mediante funciones recursivas y detectar errores de sintaxis y divisiones entre cero sin interrumpir inesperadamente el programa.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Contrato formal de Frontend Semántico |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando un mini-evaluador de
expresiones aritméticas en C utilizando GCC.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa la función evaluar_expresion()
respetando la firma de api_sesion10.h.

Evalúa expresiones con números enteros,
suma, resta, multiplicación y división.

Respeta la precedencia de los operadores,
los paréntesis y los espacios.

Utiliza funciones recursivas para analizar
los números, productos y expresiones.

Devuelve los siguientes códigos de error:
0: Expresión válida.
1: División entre cero.
2: Sintaxis inválida.

No permitas variables ni operadores unarios.

Salida:
Entrega únicamente código modular en C
y su análisis de complejidad.
No incluyas la función main().
El código debe compilar con
-Wall -Wextra -Werror.

```


## 3. Código Devuelto por el Modelo
```text
#include "api_sesion10.h"
#include <ctype.h>
#include <stddef.h>

typedef struct {
    const char *entrada;
    int error;
} Analizador;

static void omitir_espacios(Analizador *a) {
    while (isspace((unsigned char)*a->entrada))
        a->entrada++;
}

static long analizar_expresion(Analizador *a);

static long analizar_numero(Analizador *a) {
    omitir_espacios(a);

    if (*a->entrada == '(') {
        a->entrada++;

        long resultado = analizar_expresion(a);

        if (a->error)
            return 0;

        omitir_espacios(a);

        if (*a->entrada != ')') {
            a->error = 2;
            return 0;
        }

        a->entrada++;
        return resultado;
    }

    if (!isdigit((unsigned char)*a->entrada)) {
        a->error = 2;
        return 0;
    }

    long numero = 0;

    while (isdigit((unsigned char)*a->entrada)) {
        numero = numero * 10 + (*a->entrada - '0');
        a->entrada++;
    }

    return numero;
}

static long analizar_producto(Analizador *a) {
    long resultado = analizar_numero(a);

    while (!a->error) {
        omitir_espacios(a);

        char operador = *a->entrada;

        if (operador != '*' && operador != '/')
            break;

        a->entrada++;

        long valor = analizar_numero(a);

        if (a->error)
            return 0;

        if (operador == '*') {
            resultado *= valor;
        } else {
            if (valor == 0) {
                a->error = 1;
                return 0;
            }

            resultado /= valor;
        }
    }

    return resultado;
}

static long analizar_expresion(Analizador *a) {
    long resultado = analizar_producto(a);

    while (!a->error) {
        omitir_espacios(a);

        char operador = *a->entrada;

        if (operador != '+' && operador != '-')
            break;

        a->entrada++;

        long valor = analizar_producto(a);

        if (a->error)
            return 0;

        if (operador == '+')
            resultado += valor;
        else
            resultado -= valor;
    }

    return resultado;
}

long evaluar_expresion(const char *fuente, int *error) {
    if (error != NULL)
        *error = 0;

    if (fuente == NULL) {
        if (error != NULL)
            *error = 2;

        return 0;
    }

    Analizador a = {fuente, 0};

    long resultado = analizar_expresion(&a);

    omitir_espacios(&a);

    if (!a.error && *a.entrada != '\0')
        a.error = 2;

    if (error != NULL)
        *error = a.error;

    return a.error ? 0 : resultado;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Batería de 10 expresiones mixtas complejas

### Salida del harness
```text
=== AUDITORIA SESION 10 - Mini-evaluador de expresiones (Hito 1) ===
Caso                               esperado:              obtenido:              resultado
[ 1] "2+3*4"                            esperado: 14           obtenido: 14           OK
[ 2] "(2+3)*4"                          esperado: 20           obtenido: 20           OK
[ 3] "10/0" -> error 1                  esperado: 1            obtenido: 1            OK
[ 4] "2+" -> error 2                    esperado: 2            obtenido: 2            OK
[ 5] "  7 "                             esperado: 7            obtenido: 7            OK
[ 6] "5-10"                             esperado: -5           obtenido: -5           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion10.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 

---
**Firma digital:** [Usuario de GitHub]
