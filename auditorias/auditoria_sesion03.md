# Bitácora de Requerimientos y Auditoría IA — Sesión 03
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Google Colab, Mágicos y Flujo de Entrega  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Nombre Completo]  
**Matrícula:** [No. de Control]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Manipulación de archivos y llamadas OS  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
La principal dificultad fue controlar la apertura y cierre de archivos, detectar rutas inexistentes y contar correctamente las líneas cuando el archivo no contiene un salto de línea al final.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Salida estándar y códigos de retorno ABI |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:
Estoy desarrollando una práctica en lenguaje C
utilizando GCC y Google Colab.

Rol:
Actúa como ingeniero de software especializado
en sistemas y compiladores.

Especificación:
Implementa las funciones escribir_texto()
y contar_lineas() respetando las firmas
del archivo api_sesion03.h.

Utiliza fopen, fputs, fgetc y fclose.
Controla archivos vacíos, rutas inexistentes
y archivos sin salto de línea final.

Devuelve -1 cuando ocurra un error y 0 cuando
la escritura sea exitosa.

Salida:
Entrega únicamente código modular en C y
su análisis de complejidad.
No incluyas la función main().
El código debe compilar con
-Wall -Wextra -Werror.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion03.h"
#include <stdio.h>

int escribir_texto(const char *ruta, const char *texto) {
    FILE *archivo = fopen(ruta, "w");

    if (archivo == NULL) {
        return -1;
    }

    if (fputs(texto, archivo) == EOF) {
        fclose(archivo);
        return -1;
    }

    if (fclose(archivo) == EOF) {
        return -1;
    }

    return 0;
}

int contar_lineas(const char *ruta) {
    FILE *archivo = fopen(ruta, "r");

    if (archivo == NULL) {
        return -1;
    }

    int lineas = 0;
    int c;
    int anterior = '\n';

    while ((c = fgetc(archivo)) != EOF) {
        if (c == '\n') {
            lineas++;
        }

        anterior = c;
    }

    if (ferror(archivo)) {
        fclose(archivo);
        return -1;
    }

    if (anterior != '\n') {
        lineas++;
    }

    if (fclose(archivo) != 0) {
        return -1;
    }

    return lineas;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Detección de errores de compilación GCC

### Salida del harness
```text
=== AUDITORIA SESION 03 - Archivos: escribir y contar lineas ===
Caso                               esperado:              obtenido:              resultado
[ 1] escribir_texto(...)==0             esperado: 0            obtenido: 0            OK
[ 2] contar_lineas(3 lineas)            esperado: 3            obtenido: 3            OK
[ 3] contar_lineas(sin salto final)     esperado: 2            obtenido: 2            OK
[ 4] contar_lineas(archivo vacio)       esperado: 0            obtenido: 0            OK
[ 5] contar_lineas(inexistente)         esperado: -1           obtenido: -1           OK
[ 6] escribir_texto(ruta invalida)      esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion03.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 

---
**Firma digital:** [Usuario de GitHub]
