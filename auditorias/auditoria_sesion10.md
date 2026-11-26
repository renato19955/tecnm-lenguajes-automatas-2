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
Actúa como un especialista en compiladores. Desarrolla
el mini-evaluador de la sesión 10 respetando las firmas
de api_sesion10.h. Evalúa correctamente las expresiones,
controla los errores y respeta la precedencia de las
operaciones. Genera código compatible con el harness.


## 3. Código Devuelto por el Modelo

#include "api_sesion10.h"
#include <ctype.h>
#include <stddef.h>

typedef struct {
    const char *cursor;
    int estado;
} Contexto;

static void omitir(Contexto *c) {
    while (*c->cursor == ' ')
        c->cursor++;
}

static long suma(Contexto *c);

static long factor(Contexto *c) {
    omitir(c);

    if (*c->cursor == '(') {
        c->cursor++;
        long resultado = suma(c);

        if (c->estado)
            return 0;

        omitir(c);

        if (*c->cursor != ')') {
            c->estado = 2;
            return 0;
        }

        c->cursor++;
        return resultado;
    }

    if (!isdigit((unsigned char)*c->cursor)) {
        c->estado = 2;
        return 0;
    }

    long resultado = 0;

    while (isdigit((unsigned char)*c->cursor)) {
        resultado = 10 * resultado + (*c->cursor - '0');
        c->cursor++;
    }

    return resultado;
}

static long producto(Contexto *c) {
    long acumulado = factor(c);

    for (;;) {
        if (c->estado)
            return 0;

        omitir(c);

        if (*c->cursor != '*' && *c->cursor != '/')
            return acumulado;

        char signo = *c->cursor++;
        long derecho = factor(c);

        if (c->estado)
            return 0;

        switch (signo) {
            case '*':
                acumulado = acumulado * derecho;
                break;

            case '/':
                if (derecho == 0) {
                    c->estado = 1;
                    return 0;
                }

                acumulado = acumulado / derecho;
                break;
        }
    }
}

static long suma(Contexto *c) {
    long acumulado = producto(c);

    for (;;) {
        if (c->estado)
            return 0;

        omitir(c);

        if (*c->cursor != '+' && *c->cursor != '-')
            return acumulado;

        char signo = *c->cursor++;
        long derecho = producto(c);

        if (c->estado)
            return 0;

        switch (signo) {
            case '+':
                acumulado += derecho;
                break;

            case '-':
                acumulado -= derecho;
                break;
        }
    }
}

long evaluar_expresion(const char *fuente, int *error) {
    int local = 0;

    if (error == NULL)
        error = &local;

    *error = 0;

    if (fuente == NULL) {
        *error = 2;
        return 0;
    }

    Contexto c = {fuente, 0};

    long valor = suma(&c);

    omitir(&c);

    if (c.estado == 0 && *c.cursor != '\0')
        c.estado = 2;

    *error = c.estado;

    return c.estado == 0 ? valor : 0;
}


---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Batería de 10 expresiones mixtas complejas

### Salida del harness
```text
Pega aquí la salida de la celda del harness. Debe terminar con:
RESULTADO: n/n pruebas superadas
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 

---
**Firma digital:** [Usuario de GitHub]
