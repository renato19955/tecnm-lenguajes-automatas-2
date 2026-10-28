{
  "cells": [
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "l_RGf8SIKoTj"
      },
      "source": [
        "# Lenguajes y Autómatas II (SCD-1016)\n",
        "## Sesión 01 · Encuadre, Diagnóstico de Punteros y Metodología IA\n",
        "\n",
        "**Docente:** MCC. Iván Márquez Larios (`ijmarquezl`) · **Repositorio maestro:** https://github.com/ijmarquezl/tecnm-lenguajes-automatas-2\n",
        "\n",
        "---\n",
        "\n",
        "> **Pregunta guía:** ¿Por qué un compilador necesita mirar la memoria del programa que analiza?\n",
        ">\n",
        "> **Objetivo de la sesión:** Entender cómo vive el código fuente en memoria y recorrerlo con punteros sin salirse del buffer, implementando una inversión in-place y auditándola con pruebas automáticas.\n",
        "\n",
        "**Entregable:** tu función invertir() en artesano_sesion01_c.c, el harness en verde y la bitácora en auditorias/auditoria_sesion01.md\n",
        "\n",
        "**Tiempo:** 60 min · **Fases:** Artesano (manual) → Científico (IA con contrato) → Auditor (harness)\n",
        "\n",
        "> ¿Dudas de C? Consulta la [guía de C](https://github.com/ijmarquezl/tecnm-lenguajes-automatas-2/blob/main/docs/guia-c.md) antes de empezar.\n"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "dYIURhtlKoTl"
      },
      "source": [
        "## 1. ¿Qué vas a construir?\n",
        "\n",
        "Recibes una cadena mutable y debes darle vuelta sin reservar memoria nueva. Al terminar, la misma dirección de memoria contiene los caracteres al revés.\n",
        "\n",
        "Al final de la sesión tendrás: **Una función en C que invierte cualquier cadena en su mismo espacio de memoria y un harness en verde que lo demuestra con seis casos.**\n",
        "\n",
        "Los conceptos y el porqué de esta sesión están en la presentación\n",
        "([sesión 01](https://ijmarquezl.github.io/tecnm-lenguajes-automatas-2/presentaciones/sesion01.html)). Aquí solo está el trabajo: qué se\n",
        "recibe, qué debe salir y cómo se comprueba.\n"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "gKmBNIumKoTl"
      },
      "source": [
        "## 2. El contrato exacto\n",
        "\n",
        "El archivo `api_sesion01.h` declara la interfaz que el harness sabe probar.\n",
        "Tu implementación **debe** respetar esas firmas; si las cambias, el harness no compila.\n",
        "\n",
        "```c\n",
        "#include \"api_sesion01.h\"\n",
        "\n",
        "void invertir(char *s);\n",
        "```\n",
        "\n",
        "- Sin malloc ni arreglos auxiliares: solo dos índices y un char temporal.\n",
        "- La cadena se modifica en su lugar; no devuelvas una copia.\n",
        "- invertir(NULL) debe terminar sin caerse.\n",
        "\n",
        "> El harness compila con `-Wall -Wextra -Werror`: una advertencia del compilador cuenta como error.\n",
        ">\n",
        "> El harness compila `harness_sesion01_c.c` junto con `artesano_sesion01_c.c`\n",
        "> (o `ia_sesion01_c.c`). Por eso tu archivo **no debe tener `main()`**: el `main()`\n",
        "> es del harness.\n"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "UP4TJTJ5KoTm"
      },
      "source": [
        "## 3. Cómo se califica\n",
        "\n",
        "### Casos que evalúa el harness\n",
        "\n",
        "| # | Caso | Resultado esperado |\n",
        "| :-: | :--- | :--- |\n",
        "| 1 | `invertir(\"HOLA\")` | `ALOH` |\n",
        "| 2 | `invertir(\"abc\")` | `cba` |\n",
        "| 3 | `invertir(\"\")` | `(cadena vacía)` |\n",
        "| 4 | `invertir(\"AB\")` | `BA` |\n",
        "| 5 | `invertir(\"a\")` | `a` |\n",
        "| 6 | `invertir(NULL)` | `sin crash` |\n",
        "\n",
        "### Salida literal que debes obtener\n",
        "\n",
        "```text\n",
        "=== AUDITORIA SESION 01 - Inversion de buffer in-place ===\n",
        "Caso                               esperado:              obtenido:              resultado\n",
        "[ 1] invertir(\"HOLA\")                   esperado: ALOH         obtenido: ALOH         OK\n",
        "[ 2] invertir(\"abc\")                    esperado: cba          obtenido: cba          OK\n",
        "[ 3] invertir(\"\")                       esperado:              obtenido:              OK\n",
        "[ 4] invertir(\"AB\")                     esperado: BA           obtenido: BA           OK\n",
        "[ 5] invertir(\"a\")                      esperado: a            obtenido: a            OK\n",
        "[ 6] invertir(NULL)                     esperado: sin crash    obtenido: sin crash    OK\n",
        "\n",
        "RESULTADO: 6/6 pruebas superadas\n",
        "TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion01.md\n",
        "```\n",
        "\n",
        "Los números entre corchetes son el contador de casos. Mientras algo falle verás\n",
        "`FALLA` en esa línea, el total será menor y el programa terminará con código de\n",
        "salida 1.\n",
        "\n",
        "### Criterios de aceptación\n",
        "\n",
        "- [ ] El buffer original queda invertido: no hay copia ni malloc.\n",
        "- [ ] Los seis casos del harness salen OK con código de salida 0.\n",
        "- [ ] invertir(NULL) y la cadena vacía no provocan fallo de segmentación.\n"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "RYvHwKPCKoTm"
      },
      "source": [
        "## Paso 0 · Entorno\n"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": 1,
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "cVrgbWTlKoTm",
        "outputId": "c6d5af28-5c34-45e9-c963-df11d4bee4ed"
      },
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Entorno listo:\n",
            "gcc (Ubuntu 13.3.0-6ubuntu2~24.04.1) 13.3.0\n"
          ]
        }
      ],
      "source": [
        "import os\n",
        "\n",
        "if os.system('which gcc > /dev/null 2>&1') != 0:\n",
        "    print('Instalando compilador...')\n",
        "    !apt-get update -qq && apt-get install -y -qq gcc\n",
        "\n",
        "print('Entorno listo:')\n",
        "!gcc --version | head -n 1\n"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "yNw-5qOwKoTn"
      },
      "source": [
        "## Fase 1 · El Artesano (15 min, sin IA)\n",
        "\n",
        "Escribe a mano la implementación en `artesano_sesion01_c.c`. El archivo ya trae la\n",
        "firma y un `TODO`: edita el contenido de la celda `%%writefile` que está abajo (o el\n",
        "archivo en el panel **Archivos** de Colab) y vuelve a ejecutarla para guardar los\n",
        "cambios. Hazlo sin consultar a la IA. Si te atoras, anota en tu bitácora exactamente\n",
        "en qué te atoraste; eso también es evidencia.\n"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": 2,
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "JYosdvEhKoTn",
        "outputId": "6cb7e536-515a-40c8-8ba3-ead99df1d4a7"
      },
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Writing api_sesion01.h\n"
          ]
        }
      ],
      "source": [
        "\n",
        "%%writefile api_sesion01.h\n",
        "#ifndef API_SESION01_H\n",
        "#define API_SESION01_H\n",
        "\n",
        "void invertir(char *s);\n",
        "\n",
        "#endif"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": 3,
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "sARfDr3pKoTo",
        "outputId": "4ff2617d-5d6e-4e6c-e800-32bbff682c22"
      },
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Writing artesano_sesion01_c.c\n"
          ]
        }
      ],
      "source": [
        "\n",
        "%%writefile artesano_sesion01_c.c\n",
        "#include \"api_sesion01.h\"\n",
        "#include <string.h>\n",
        "\n",
        "void invertir(char *s) {\n",
        "    if (s == NULL) {\n",
        "        return;\n",
        "    }\n",
        "    int inicio = 0;\n",
        "    int fin = (int)strlen(s) - 1;\n",
        "    while (inicio < fin) {\n",
        "        char temporal = s[inicio];\n",
        "        s[inicio] = s[fin];\n",
        "        s[fin] = temporal;\n",
        "        inicio++;\n",
        "        fin--;\n",
        "    }\n",
        "}"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "biulvHMxKoTo"
      },
      "source": [
        "## Fase 2 · El Científico (10 min)\n",
        "\n",
        "Redacta el contrato **C-R-E-O** (Contexto, Rol, Especificación, Output) para pedirle\n",
        "la misma función a la IA, y pega el resultado en la celda de `ia_sesion01_c.c`. La\n",
        "firma debe ser idéntica a la del contrato; si la IA la cambia, corrígela antes de\n",
        "compilar.\n"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": 4,
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "bqfqGe8vKoTo",
        "outputId": "10dde0d4-e40c-4a4f-aff5-69863ba9e2e9"
      },
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Writing ia_sesion01_c.c\n"
          ]
        }
      ],
      "source": [
        "\n",
        "%%writefile ia_sesion01_c.c\n",
        "#include \"api_sesion01.h\"\n",
        "#include <string.h>\n",
        "\n",
        "void invertir(char *s) {\n",
        "    if (s == NULL) {\n",
        "        return;\n",
        "    }\n",
        "    size_t longitud = strlen(s);\n",
        "    if (longitud < 2) {\n",
        "        return;\n",
        "    }\n",
        "    char *izquierda = s;\n",
        "    char *derecha = s + longitud - 1;\n",
        "    while (izquierda < derecha) {\n",
        "        char auxiliar = *izquierda;\n",
        "        *izquierda = *derecha;\n",
        "        *derecha = auxiliar;\n",
        "\n",
        "        izquierda++;\n",
        "        derecha--;\n",
        "    }\n",
        "}"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "R17C5UJpKoTo"
      },
      "source": [
        "## Fase 3 · El Auditor (10 min)\n",
        "\n",
        "El harness de abajo **no aprueba nada por sí solo**: ejecuta tu función caso por caso,\n",
        "la compara contra el valor esperado e imprime `OK` o `FALLA`. Léelo antes de ejecutarlo:\n",
        "los casos te dicen qué se está midiendo.\n"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": 5,
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "0JGWEWMmKoTo",
        "outputId": "bdae8780-63b2-4bab-8648-3eb5c2e24825"
      },
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Writing harness_sesion01_c.c\n"
          ]
        }
      ],
      "source": [
        "%%writefile harness_sesion01_c.c\n",
        "/* harness_sesion01_c.c - auditoria automatica de la Sesion 01\n",
        " *\n",
        " * Compilacion:\n",
        " *   gcc -std=c11 -Wall -Wextra -Werror harness_sesion01_c.c artesano_sesion01_c.c -o test_artesano\n",
        " *\n",
        " * El harness NO aprueba nada por si solo: cada linea se calcula\n",
        " * ejecutando tu implementacion y comparandola contra el valor esperado.\n",
        " * Si algun caso falla, el programa termina con codigo de salida 1.\n",
        " */\n",
        "#include <stdio.h>\n",
        "#include <string.h>\n",
        "#include \"api_sesion01.h\"\n",
        "\n",
        "static int probadas = 0;\n",
        "static int fallidas = 0;\n",
        "\n",
        "static void verificar_str(const char *caso, const char *esperado,\n",
        "                          const char *obtenido) {\n",
        "    int ok = (obtenido != NULL) && (strcmp(esperado, obtenido) == 0);\n",
        "    probadas++;\n",
        "    if (!ok) fallidas++;\n",
        "    printf(\"[%2d] %-34s esperado: %-12s obtenido: %-12s %s\\n\",\n",
        "           probadas, caso, esperado, obtenido ? obtenido : \"(NULL)\",\n",
        "           ok ? \"OK\" : \"FALLA\");\n",
        "}\n",
        "\n",
        "int main(void) {\n",
        "    printf(\"=== AUDITORIA SESION 01 - Inversion de buffer in-place ===\\n\");\n",
        "    printf(\"Caso                               esperado:              obtenido:              resultado\\n\");\n",
        "\n",
        "    { char a[] = \"HOLA\"; invertir(a); verificar_str(\"invertir(\\\"HOLA\\\")\", \"ALOH\", a); }\n",
        "    { char b[] = \"abc\"; invertir(b); verificar_str(\"invertir(\\\"abc\\\")\", \"cba\", b); }\n",
        "    { char c[] = \"\"; invertir(c); verificar_str(\"invertir(\\\"\\\")\", \"\", c); }\n",
        "    { char d[] = \"AB\"; invertir(d); verificar_str(\"invertir(\\\"AB\\\")\", \"BA\", d); }\n",
        "    { char e[] = \"a\"; invertir(e); verificar_str(\"invertir(\\\"a\\\")\", \"a\", e); }\n",
        "    { invertir(NULL); verificar_str(\"invertir(NULL)\", \"sin crash\", \"sin crash\"); }\n",
        "\n",
        "    printf(\"\\nRESULTADO: %d/%d pruebas superadas\\n\",\n",
        "           probadas - fallidas, probadas);\n",
        "    if (fallidas != 0) {\n",
        "        printf(\"Aun hay %d caso(s) sin resolver. Revisa tu implementacion.\\n\",\n",
        "               fallidas);\n",
        "        return 1;\n",
        "    }\n",
        "    printf(\"TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion01.md\" \"\\n\");\n",
        "    return 0;\n",
        "}\n"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": 6,
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "lP2SNZTLKoTo",
        "outputId": "0e90c320-56aa-4007-c5f1-df7ab2a164cc"
      },
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "=== AUDITORIA SESION 01 - Inversion de buffer in-place ===\n",
            "Caso                               esperado:              obtenido:              resultado\n",
            "[ 1] invertir(\"HOLA\")                   esperado: ALOH         obtenido: ALOH         OK\n",
            "[ 2] invertir(\"abc\")                    esperado: cba          obtenido: cba          OK\n",
            "[ 3] invertir(\"\")                       esperado:              obtenido:              OK\n",
            "[ 4] invertir(\"AB\")                     esperado: BA           obtenido: BA           OK\n",
            "[ 5] invertir(\"a\")                      esperado: a            obtenido: a            OK\n",
            "[ 6] invertir(NULL)                     esperado: sin crash    obtenido: sin crash    OK\n",
            "\n",
            "RESULTADO: 6/6 pruebas superadas\n",
            "TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion01.md\n"
          ]
        }
      ],
      "source": [
        "!gcc -std=c11 -Wall -Wextra -Werror harness_sesion01_c.c artesano_sesion01_c.c -o test_artesano && ./test_artesano"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": 7,
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "q_64GtNQKoTo",
        "outputId": "68db6502-c781-4715-b082-bd3f1fca3bc7"
      },
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "=== AUDITORIA SESION 01 - Inversion de buffer in-place ===\n",
            "Caso                               esperado:              obtenido:              resultado\n",
            "[ 1] invertir(\"HOLA\")                   esperado: ALOH         obtenido: ALOH         OK\n",
            "[ 2] invertir(\"abc\")                    esperado: cba          obtenido: cba          OK\n",
            "[ 3] invertir(\"\")                       esperado:              obtenido:              OK\n",
            "[ 4] invertir(\"AB\")                     esperado: BA           obtenido: BA           OK\n",
            "[ 5] invertir(\"a\")                      esperado: a            obtenido: a            OK\n",
            "[ 6] invertir(NULL)                     esperado: sin crash    obtenido: sin crash    OK\n",
            "\n",
            "RESULTADO: 6/6 pruebas superadas\n",
            "TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion01.md\n"
          ]
        }
      ],
      "source": [
        "!gcc -std=c11 -Wall -Wextra -Werror harness_sesion01_c.c ia_sesion01_c.c -o test_ia && ./test_ia"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "qfybIXYrKoTp"
      },
      "source": [
        "## Resultado esperado\n",
        "\n",
        "Si tu implementación es correcta, la última línea del harness será:\n",
        "\n",
        "```text\n",
        "RESULTADO: 6/6 pruebas superadas\n",
        "```\n",
        "\n",
        "y la celda terminará sin error (código de salida 0). Compara tu salida completa\n",
        "contra la de la sección 3: debe ser idéntica, línea por línea.\n"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "Od9wf08IKoTp"
      },
      "source": [
        "## Entrega\n",
        "\n",
        "1. Copia `plantillas/auditoria_sesion01.md` a `auditorias/auditoria_sesion01.md` en tu fork.\n",
        "2. Adjunta la salida del harness (las 6 líneas con `OK`).\n",
        "3. Explica en el reporte crítico qué caso borde omitió la IA y cómo lo detectaste.\n",
        "4. Haz commit y sincroniza tu fork.\n",
        "\n",
        "Este laboratorio alimenta el 40% de laboratorios y es la base del Hito correspondiente\n",
        "del Proyecto Integrador.\n"
      ]
    }
  ],
  "metadata": {
    "language_info": {
      "name": "python"
    },
    "kernelspec": {
      "display_name": "Python 3",
      "name": "python3"
    },
    "colab": {
      "provenance": []
    }
  },
  "nbformat": 4,
  "nbformat_minor": 0
}
