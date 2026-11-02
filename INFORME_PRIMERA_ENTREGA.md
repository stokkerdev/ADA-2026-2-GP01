# Primera entrega — ADAG1 Proyecto Oráculo

**Curso:** Análisis y Diseño de Algoritmos  
**Periodo:** 2026-B  
**Grupo/equipo:** Equipo 1; completar nombres  
**Repositorio:** `stokkerdev/ADA-2026-2-GP01`  
**Estado:** borrador avanzado; resultados experimentales obtenidos en GPU y pendientes de completar con nombres y semana académica.

## 1. Propósito del proyecto

El Proyecto Oráculo estudia la selección de una configuración de prompt para un modelo conversacional. La configuración combina una opción por ranura —`rol`, `estrategia`, `formato`, `estilo` y `cierre`— y un nivel de temperatura. El objetivo es maximizar la proporción de instancias cuya respuesta satisface simultáneamente todas las restricciones verificables.

El repositorio implementa un modelo experimental con ocho opciones por cada una de las cinco ranuras y temperaturas disponibles `0.0`, `0.3` y `0.7`. Con temperatura fija en `0.0` hay `8^5 = 32 768` configuraciones; al incluir tres temperaturas hay `3 · 8^5 = 98 304` configuraciones.

El proyecto se analiza en dos partes distintas:

1. **Oráculo exacto o evaluación conocida:** se conocen las reglas que cada opción satisface y se busca la configuración de mayor precisión.
2. **Oráculo ruidoso con consultas limitadas:** las reglas no se observan directamente; cada consulta al modelo devuelve una respuesta que debe verificarse y consume tiempo/costo de cómputo.

La NP-dureza corresponde a la primera parte. La segunda se estudia como un problema de asignación de consultas y complejidad de muestreo, no como una nueva reducción automática desde 3-SAT.

## 2. Formalización de la Parte 1: oráculo exacto

### 2.1 Entrada

Sean:

- `m`: número de categorías o ranuras.
- `k_j`: número de opciones de la categoría `j`.
- `K = max_j k_j`.
- `R`: universo de reglas verificables, con `rho = |R|`.
- `S_o ⊆ R`: conjunto de reglas que satisface la opción `o`.
- `n`: número de instancias de prueba.
- `R_i ⊆ R`: conjunto de reglas exigidas por la instancia `i`.
- `t`: umbral, únicamente para la versión de decisión.

El tamaño de entrada se expresa mediante `(m, K, n, rho)` y, cuando se analiza la representación explícita de reglas, también por el número total de incidencias de los conjuntos `S_o` y `R_i`. No se toma `|C|` como única medida porque el espacio de configuraciones se genera implícitamente.

### 2.2 Solución válida

Una configuración es una tupla:

```text
c = (o_1, o_2, ..., o_m),   o_j ∈ opciones(j)
```

La configuración satisface las reglas:

```text
S(c) = ⋃_{j=1}^{m} S_{o_j}
```

La instancia `i` queda resuelta si y solo si:

```text
R_i ⊆ S(c)
```

Toda solución válida selecciona exactamente una opción por categoría. La temperatura se considera un parámetro adicional de evaluación experimental; para la reducción formal de la Parte 1 se fija o se omite, pues la decisión combinatoria está en las opciones seleccionadas.

### 2.3 Función objetivo

La precisión de `c` es:

```text
Acc(c) = (1/n) · |{ i ∈ {1,...,n} : R_i ⊆ S(c) }|
```

El problema de optimización consiste en encontrar:

```text
c* ∈ C que maximiza Acc(c)
```

donde:

```text
|C| = ∏_{j=1}^{m} k_j ≤ K^m
```

### 2.4 Supuestos

- Las instancias son sintéticas y sus reglas pueden representarse como identificadores discretos.
- La evaluación de una configuración es determinista cuando se conocen `S_o` y `R_i`.
- La operación `R_i ⊆ S(c)` puede implementarse con vectores de bits en `O(rho)` o con una cota dependiente del número de reglas almacenadas.
- La temperatura no altera la prueba de NP-dureza del modelo exacto; sí afecta el experimento con el modelo real.

## 3. Formalización de la Parte 2: oráculo ruidoso

En la Parte 2 el conjunto de configuraciones candidatas se trata como una lista explícita `C`. Para una configuración `c`, una consulta sobre una instancia seleccionada aleatoriamente produce una variable aleatoria `X_c ∈ {0,1}` con media `p_c`.

La entrada está formada por:

- `|C|`: número explícito de configuraciones candidatas.
- `B`: presupuesto total de consultas.
- `n`: número de instancias disponibles para muestrear.
- El acceso al oráculo que produce observaciones ruidosas.

La salida es una configuración estimada `ĉ` y una asignación de consultas `m_c` tal que:

```text
Σ_{c∈C} m_c ≤ B
```

El objetivo es maximizar `Pr[ĉ = c*]` o, equivalentemente, minimizar el arrepentimiento simple:

```text
p_{c*} - p_{ĉ}
```

En el repositorio, cada consulta se materializa ejecutando el modelo, generando una respuesta y aplicando los verificadores de `open-instruct`. Por ello, además del costo de cómputo local, existe un presupuesto práctico de generaciones.

## 4. Versión de decisión para la NP-dureza

Definimos **Prompt-Dec** de la siguiente manera.

**Entrada:** categorías, opciones, conjuntos `S_o`, instancias `R_i` y un entero `t`.

**Pregunta:** ¿existe una configuración `c` que resuelva al menos `t` instancias?

La versión de optimización es al menos tan difícil como esta versión de decisión, porque conocer el óptimo permitiría responder si el óptimo es mayor o igual que `t`.

## 5. Reducción polinomial desde 3-SAT

Sea una fórmula 3-CNF:

```text
φ = φ_1 ∧ φ_2 ∧ ... ∧ φ_n
```

con variables `x_1, ..., x_m`, donde cada cláusula tiene tres literales.

Construimos una instancia de Prompt-Dec:

1. **Categorías:** una categoría por variable `x_j`.
2. **Opciones:** dos opciones por categoría: `True` y `False`.
3. **Reglas:** una regla `r_i` por cláusula `φ_i`.
4. **Instancias:** una instancia por cláusula, con `R_i = {r_i}`.
5. **Cobertura:**
   - La opción `True` de `x_j` satisface `r_i` si `x_j` aparece positivamente en `φ_i`.
   - La opción `False` de `x_j` satisface `r_i` si `¬x_j` aparece en `φ_i`.
6. **Umbral:** `t = n`.

La transformación se construye recorriendo las variables y las cláusulas. Su tiempo es polinomial, `O(mn)` en una implementación directa, y el tamaño de la instancia construida también es polinomial en el tamaño de `φ`.

### 5.1 Si la fórmula es satisfacible, Prompt-Dec responde sí

Supongamos que `φ` es satisfacible. Sea `a` una asignación que la satisface. Elegimos en la categoría `j` la opción `True` si `a(x_j)=True` y la opción `False` si `a(x_j)=False`.

Como `a` satisface cada cláusula `φ_i`, en cada cláusula existe al menos un literal verdadero bajo `a`. Por la forma en que construimos las coberturas, la configuración correspondiente satisface la regla `r_i`. Puesto que `R_i={r_i}`, la instancia `i` queda resuelta para todo `i`. Por tanto, resuelve `n=t` instancias y Prompt-Dec responde sí.

### 5.2 Si Prompt-Dec responde sí, la fórmula es satisfacible

Supongamos que existe una configuración que resuelve al menos `t=n` instancias. Como hay exactamente `n` instancias, debe resolverlas todas.

La configuración selecciona exactamente una de las opciones `True` o `False` por cada variable; por tanto induce una asignación de verdad `a`. Para cada cláusula `φ_i`, la instancia correspondiente se resuelve, lo que significa que `r_i ∈ S(c)`. Por la definición de la cobertura, algún literal de `φ_i` es verdadero bajo `a`. Así, todas las cláusulas se satisfacen y `φ` es satisfacible.

Concluimos:

```text
φ es satisfacible  ⇔  la instancia construida de Prompt-Dec responde sí
```

Como 3-SAT es NP-completo y la transformación es polinomial, Prompt-Dec es NP-difícil. En consecuencia, la versión de optimización de configuración óptima con oráculo exacto también es NP-difícil.

## 6. Complejidad de la solución base

Una enumeración exhaustiva visita `|C| = ∏ k_j ≤ K^m` hojas. Evaluar una configuración revisando las `n` instancias y hasta `rho` reglas cuesta `O(n·rho)` con representación por vectores de bits. Por tanto:

```text
T_exhaustivo(m,K,n,rho) = O(K^m · n · rho)
```

El backtracking con poda puede visitar menos nodos en la práctica, pero en el peor caso la poda no elimina ninguna rama. Por eso conserva la misma cota asintótica superior exponencial.

Para una búsqueda local por coordenadas, una pasada evalúa `Σ_j k_j ≤ mK` candidatos y, después de `I` pasadas:

```text
T_local(m,K,n,rho,I) = O(I · m · K · n · rho)
```

Esta cota es polinomial en los parámetros representados, pero el método es heurístico y no garantiza optimalidad en el caso general.

## 7. Evidencia experimental

La búsqueda se ejecutó con el modelo configurado como `qwen17b`, temperatura `0.0`, semilla `1`, dos reinicios y un máximo teórico de tres pasadas por reinicio. Se utilizó ascenso por coordenadas: en cada paso se variaron las ocho opciones de una ranura, manteniendo fijas las demás, y se conservó la mejor alternativa. La caché del oráculo permitió reanudar la ejecución después de una desconexión de Colab.

El máximo teórico era:

```text
2 · (1 + 3 · 5 · 8) = 242 evaluaciones de configuraciones
```

La ejecución terminó con 162 evaluaciones porque ambos reinicios se detuvieron después de dos pasadas sin una mejora adicional:

| Reinicio | Evaluaciones | Configuración final | Precisión de búsqueda |
|---:|---:|---|---:|
| 1 | 81 | `rol=1, estrategia=4, formato=0, estilo=5, cierre=0, temperatura=0.0` | 16.00% |
| 2 | 81 | `rol=0, estrategia=1, formato=4, estilo=1, cierre=0, temperatura=0.0` | 12.00% |

La mejor configuración observada fue:

```python
{
    "rol": 1,
    "estrategia": 4,
    "formato": 0,
    "estilo": 5,
    "cierre": 0,
    "temperatura": 0.0
}
```

Su precisión sobre las 100 instancias públicas de búsqueda fue **16.00%**, equivalente a 16 instancias resueltas completamente.

### 7.1 Análisis de trazas de búsqueda

El resumen de restricciones más frecuentes se obtuvo acumulando las primeras restricciones incumplidas en las trazas de las configuraciones evaluadas. Por tanto, los conteos no representan fallos de una sola configuración ni deben sumarse como si fueran instancias distintas; sirven para identificar familias de restricciones difíciles durante la exploración.

Las familias más frecuentes fueron:

| Restricción | Apariciones agregadas |
|---|---:|
| `last_word:last_word_sent` | 239 |
| `paragraphs:paragraphs` | 218 |
| `length_constraints:nth_paragraph_first_word` | 162 |
| `copy:copying_multiple` | 162 |
| `detectable_format:square_brackets` | 162 |
| `letters:letter_counting2` | 145 |
| `punctuation:punctuation_dot` | 121 |
| `length_constraints:number_words` | 108 |
| `startend:quotation` | 102 |
| `keywords:existence` | 97 |

Este patrón indica que las restricciones combinatorias y de formato —última palabra de cada oración, número y estructura de párrafos, repetición de solicitudes, corchetes y conteo de letras— fueron más difíciles para el modelo que las restricciones simples de palabras prohibidas o palabras requeridas.

### 7.2 Validación final

La configuración elegida se evaluó sobre las 87 instancias de validación reservadas:

```text
validación: 10.3% (87 instancias)
```

Esto corresponde a 9 instancias resueltas completamente de 87, pues:

```text
9 / 87 = 10.34%
```

La diferencia entre búsqueda y validación es:

```text
16.00% - 10.34% = 5.66 puntos porcentuales
```

El resultado muestra una generalización limitada: la configuración obtuvo mejor rendimiento en el conjunto utilizado para seleccionarla que en el conjunto reservado. Esto no invalida la ejecución; es una limitación esperable de una búsqueda heurística con un modelo pequeño y un conjunto de instrucciones heterogéneo. La validación debe reportarse sin volver a ajustar la configuración con base en sus errores.

En la validación, las primeras familias de fallo más frecuentes fueron `last_word:last_word_sent` (6), `keywords:no_adjacent_consecutive` (5), `copy:repeat_phrase` (5), `punctuation:punctuation_dot` (5), `first_word:first_word_sent` (4), `letters:letter_counting` (4), `keywords:word_once` (4) y `language:response_language` (3). Estos conteos corresponden a las primeras restricciones que fallaron en las instancias no resueltas y son un resumen diagnóstico, no una suma de todos los incumplimientos posibles.

### 7.3 Limitaciones

- La búsqueda por coordenadas es heurística y no demuestra optimalidad global.
- La configuración fue seleccionada usando las 100 instancias públicas de búsqueda; la validación reservada obtuvo 10.34%.
- El modelo puede producir salidas degeneradas o incumplir simultáneamente restricciones difíciles de formato, idioma y conteo.
- La conclusión final debe incluir la evaluación oficial sobre los datos privados de la profesora si la guía la solicita.
