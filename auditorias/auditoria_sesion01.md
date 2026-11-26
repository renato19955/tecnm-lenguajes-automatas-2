# Bitácora de Requerimientos y Auditoría IA — Sesión 01
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Encuadre, Diagnóstico de Punteros y Metodología IA  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Cesar Renato Rivero Balam
**Matrícula:**  24531401
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Inversión de buffer in-place (char*)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue controlar los índices de la cadena para evitar accesos fuera de los límites de memoria. También fue necesario validar los punteros NULL e intercambiar los caracteres sin utilizar memoria adicional ni modificar el carácter de terminación de la cadena.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Punteros, O(n) tiempo, O(1) memoria |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
Contexto:
Estoy desarrollando una práctica de Lenguajes y
Autómatas II utilizando C99 y el compilador GCC.

Rol:
Actúa como ingeniero de software especializado
en sistemas, compiladores y manejo de memoria.

Especificación:
Implementa la función void invertir(char *s)
para invertir una cadena en su propia memoria.

Utiliza dos punteros y una variable char temporal.
No utilices malloc ni arreglos auxiliares.
Valida punteros NULL, cadenas vacías y cadenas
de un solo carácter.

La complejidad temporal debe ser O(n) y la
complejidad espacial O(1).

Salida:
Devuelve únicamente el código modular en C y
su análisis de complejidad. Respeta la firma
de api_sesion01.h y no incluyas main().
El código debe compilar con -Wall -Wextra -Werror.
---

## 3. Código Devuelto por el Modelo
#include "api_sesion01.h"
#include <string.h>

void invertir(char *s) {
    if (s == NULL) {
        return;
    }

    size_t longitud = strlen(s);

    if (longitud < 2) {
        return;
    }

    char *izquierda = s;
    char *derecha = s + longitud - 1;

    while (izquierda < derecha) {
        char auxiliar = *izquierda;

        *izquierda = *derecha;
        *derecha = auxiliar;

        izquierda++;
        derecha--;
    }
}

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
=== AUDITORIA SESION 01 - Inversion de buffer in-place ===
Caso                               esperado:              obtenido:              resultado
[ 1] invertir("HOLA")                   esperado: ALOH         obtenido: ALOH         OK
[ 2] invertir("abc")                    esperado: cba          obtenido: cba          OK
[ 3] invertir("")                       esperado:              obtenido:              OK
[ 4] invertir("AB")                     esperado: BA           obtenido: BA           OK
[ 5] invertir("a")                      esperado: a            obtenido: a            OK
[ 6] invertir(NULL)                     esperado: sin crash    obtenido: sin crash    OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion01.md

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion01.md

Caso base y límites superados sin Crash.
[x] No se identificaron accesos fuera de límites ni Memory Leaks mediante la revisión del código.
[x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 

---
**Firma digital:** [Usuario de GitHub]
