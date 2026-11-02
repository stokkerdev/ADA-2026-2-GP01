# Manual técnico — Proyecto Oráculo

**Curso:** Análisis y Diseño de Algoritmos, 2026-B
**Gaxprupo:** ADAG1 — completar identificador del equipo
**Subproyecto:** Configuración óptima de prompts bajo presupuesto de consultas
**Estado:** Solución base de la primera entrega

## 1. Propósito

El proyecto busca una configuración de instrucciones que maximice la precisión con la que un modelo de lenguaje cumple restricciones verificables incluidas en las instancias de prueba. Una configuración selecciona un índice para cada ranura (`rol`, `estrategia`, `formato`, `estilo` y `cierre`) y un valor de temperatura.

La solución base separa dos actividades:

1. **Búsqueda:** evaluación de configuraciones sobre el lote público `busqueda`.

1. **Validación:** evaluación posterior de la configuración seleccionada sobre el lote público reservado `validacion`, sin usarlo para decidir la configuración.

La evaluación final del curso usa datos privados que no están incluidos en este repositorio.

## 2. Entorno y dependencias

- Python 3.10 o superior.

- GPU NVIDIA compatible con CUDA; el flujo está diseñado para ejecutarse en Google Colab con una T4.

- PyTorch con soporte CUDA.

- `transformers`, `accelerate`, `bitsandbytes`, `tqdm`, `matplotlib`, `nltk`, `spacy`, `emoji`, `langdetect` e `immutabledict`.

- El repositorio de verificadores `allenai/open-instruct`, ubicado por defecto en `/content/open-instruct` cuando se ejecuta en Colab.

Instalación orientativa en Colab:

```bash
pip install -U "bitsandbytes>=0.46.1" transformers accelerate \
  nltk spacy emoji langdetect immutabledict tqdm matplotlib
python -m spacy download en_core_web_sm

git clone https://github.com/allenai/open-instruct /content/open-instruct
```

El notebook principal contiene las instrucciones de instalación y descarga de los archivos. Después de instalar `transformers` y los verificadores, se debe reiniciar la sesión de Colab antes de cargar el modelo.

## 3. Estructura del repositorio

```
.
├── ayudas.py                    # Carga de modelo, datos, particiones y entrega
├── datos_visibles.json           # Instancias públicas y marcas de split
├── entrega.json                 # Configuración final reportada, cuando se complete
├── MANUAL_TECNICO.md            # Este manual
├── README.md                    # Descripción rápida del proyecto
├── INFORME_PRIMERA_ENTREGA.md   # Formalización y reducción desde 3-SAT
├── oraculo.py                   # Evaluador y caja negra del proyecto
├── proyecto_oraculo.ipynb       # Notebook base del curso
├── Primera_entrega.ipynb        # Notebook final organizado de la entrega
├── notebook_busqueda.ipynb      # Búsqueda heurística reproducible
├── notebook_evaluacion.ipynb    # Evaluación de entrega.json
└── requirements.txt             # Dependencias base
```

El notebook de búsqueda y el notebook de evaluación deben quedar separados antes de la entrega final, o documentarse claramente si se mantienen dentro de un único notebook. La configuración de `entrega.json` debe coincidir con la que se justifique en el notebook de búsqueda.

## 4. Modelo de datos

`datos_visibles.json` es una lista de objetos. Cada instancia contiene, entre otros campos:

- `id`: identificador de la instancia.

- `prompt`: petición completa enviada al modelo.

- `pregunta`: parte principal de la petición.

- `restriccion`: restricciones verificables separadas por tabuladores.

- `ids`: identificadores de los verificadores aplicables.

- `kwargs`: parámetros de los verificadores.

- `split`: uno de `busqueda`, `validacion` o `descartada`.

El archivo actual contiene 450 instancias: 100 de búsqueda, 87 de validación y 263 descartadas. Las descartadas no deben entrar en la selección de la configuración.

Carga de datos:

```python
from ayudas import cargar_datos, dividir

datos = cargar_datos("datos_visibles.json")
busqueda, validacion = dividir(datos)
```

## 5. Catálogo y configuración

El catálogo se define en `oraculo.py`. Cada configuración es un diccionario con esta forma:

```python
{
    "rol": 5,
    "estrategia": 2,
    "formato": 0,
    "estilo": 6,
    "cierre": 3,
    "temperatura": 0.0
}
```

La función `espacio()` genera el producto cartesiano de las opciones con temperatura fija en `0.0`. El espacio completo con las temperaturas disponibles se genera con `espacio(TEMPERATURAS)`.

La función objetivo observada es:

```
precision(c) = número de instancias resueltas por c / número de instancias evaluadas
```

Una instancia vale 1 únicamente si la respuesta cumple todas sus restricciones verificables; si falla al menos una, vale 0.

## 6. Flujo de ejecución

### 6.1 Cargar el modelo

```python
from ayudas import cargar_modelo
modelo = cargar_modelo("qwen17b")
```

Los alias disponibles se encuentran en `ayudas.py`. El modelo requiere GPU.

### 6.2 Crear el oráculo

```python
from oraculo import Oraculo, espacio

oraculo = Oraculo(
    modelo,
    busqueda,
    validacion,
    lote=16,
    presupuesto=32_000_000,
)
configs = espacio()
```

El oráculo guarda las generaciones en una caché para no recalcular una misma consulta. La ruta de caché puede configurarse para Google Drive si se requiere persistencia entre sesiones de Colab.

### 6.3 Evaluar una configuración

```python
resultado = oraculo.evaluar(configs[0], busqueda, semilla=1)
print(resultado.precision)
print(resultado.trazas[:3])
```

`resultado.precision` es la métrica de la configuración. Cada elemento de `resultado.trazas` contiene el identificador de la instancia, la primera restricción incumplida y la salida producida.

### 6.4 Validar la configuración elegida

```python
resultado_validacion = oraculo.validar(configuracion_elegida, n=30)
print(resultado_validacion.precision)
```

Esta operación debe aparecer explícitamente en el notebook de búsqueda. La validación no debe utilizarse para elegir una configuración, porque deja de funcionar como conjunto reservado.

## 7. Reproducibilidad

- Usar semillas explícitas en las llamadas a `evaluar`.

- Registrar cada configuración evaluada y su precisión.

- Conservar la salida o una tabla equivalente del proceso de búsqueda.

- Registrar el modelo, temperatura, lote, tamaño del conjunto y fecha de ejecución.

- Mantener la caché cuando una sesión de Colab se interrumpa.

- Reportar por separado el mejor resultado observado y cualquier afirmación de optimalidad. Una búsqueda heurística no prueba por sí sola que la configuración sea óptima globalmente.

## 8. Prueba básica de consistencia

Antes de ejecutar el modelo, se puede verificar la estructura del dataset:

```python
import json
from collections import Counter

data = json.load(open("datos_visibles.json"))
assert Counter(x["split"] for x in data)["busqueda"] == 100
assert Counter(x["split"] for x in data)["validacion"] == 87
assert all("id" in x and "ids" in x for x in data)
```

La prueba de corrección funcional completa requiere cargar el modelo y los verificadores. No debe presentarse como ejecutada si el entorno no tiene GPU.

## 9. Limitaciones conocidas

- `oraculo.py` depende de PyTorch, Transformers y `open-instruct`; no es ejecutable en un entorno Python mínimo sin instalar esas dependencias.

- El resultado depende del modelo elegido, la temperatura, la semilla y el lote evaluado.

- El conjunto público no permite anticipar exactamente la calificación sobre el conjunto privado.

- La validación pública mide generalización a instancias reservadas, no garantiza rendimiento sobre todas las familias ocultas.

- La búsqueda aleatoria del notebook base es únicamente un ejemplo y no constituye una justificación suficiente del método de optimización. Debe reemplazarse o complementarse con el método documentado por el grupo.

- La caché `cache_oraculo.json` se guarda por lotes. Si una ejecución se interrumpe, se puede volver a crear `Oraculo` con la misma caché para reutilizar las respuestas ya generadas.
