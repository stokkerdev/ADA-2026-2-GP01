# Plan de trabajo — Primera entrega ADAG1

**Equipo:** 1  
**Proyecto:** Proyecto Oráculo  
**Fecha límite informada:** 2 de noviembre de 2026  
**Estado:** preparación de la primera entrega

## Objetivo de la entrega

Presentar una primera versión coherente y reproducible de los cuatro productos exigidos:

1. Informe de formalización y prueba de NP-dureza.
2. Repositorio con la solución base.
3. Manual técnico.
4. Archivos específicos del Proyecto Oráculo: `entrega.json`, notebook de búsqueda y notebook de evaluación.

## Prioridad 1 — Documentación académica

- Completar nombres de integrantes y datos del equipo.
- Revisar la formalización de las dos partes del problema.
- Revisar la reducción desde 3-SAT y su demostración en ambos sentidos.
- Explicar que la NP-dureza corresponde a la configuración con oráculo exacto.
- Separar claramente el análisis de muestreo y presupuesto de la Parte 2.
- Incorporar los resultados reales de búsqueda y validación cuando se obtengan.

## Prioridad 2 — Notebook de búsqueda

Debe documentar:

- Modelo utilizado.
- Datos y particiones.
- Semilla.
- Configuración inicial.
- Método de búsqueda.
- Candidatos evaluados.
- Precisión de cada candidato.
- Tiempo o cantidad de consultas.
- Trazas de fallos.
- Decisiones tomadas a partir de esas trazas.
- Configuración final seleccionada.
- Llamada explícita a `oraculo.validar`.

La primera estrategia recomendada es búsqueda local por coordenadas, porque es reproducible y fácil de justificar. Se puede complementar con una etapa de comparación aleatoria o con una mejora local desde varias semillas si el presupuesto lo permite.

## Prioridad 3 — Notebook de evaluación

Debe:

- Cargar `entrega.json` automáticamente.
- Mostrar el grupo, la semana y la configuración cargada.
- No depender de editar manualmente la configuración.
- Quedar preparado para que la profesora lo ejecute sobre sus datos reservados.
- No incluir ni requerir datos privados durante la preparación.

## Prioridad 4 — `entrega.json`

Debe generarse únicamente después de obtener la configuración mediante una ejecución real del oráculo. El archivo debe tener esta estructura:

```json
{
  "grupo": "G01",
  "config": {
    "rol": 0,
    "estrategia": 0,
    "formato": 0,
    "estilo": 0,
    "cierre": 0,
    "temperatura": 0.0
  },
  "semana": 0
}
```

Los índices anteriores son solo un ejemplo de formato y no representan la solución final. El valor de `semana` debe confirmarse con el cronograma académico.

## Ejecución necesaria

La ejecución completa necesita Google Colab con GPU CUDA, preferiblemente T4. El entorno local de revisión no tiene instaladas las dependencias pesadas ni una GPU para ejecutar el modelo.

Antes de la entrega se debe comprobar:

```text
[ ] El notebook de búsqueda abre y ejecuta sus celdas en orden.
[ ] El notebook de evaluación carga entrega.json sin editar código.
[ ] La configuración de entrega.json coincide con la reportada en el notebook.
[ ] La precisión de búsqueda está registrada.
[ ] La validación está registrada y no fue usada para seleccionar la configuración.
[ ] Las trazas sustentan las decisiones de descarte.
[ ] El informe no presenta una heurística como óptimo global probado.
[ ] README, manual, dependencias y archivos están dentro del repositorio.
[ ] Se eliminaron archivos temporales y datos de caché del repositorio.
[ ] Se completaron integrantes, grupo y semana académica.
```

## Riesgos y decisiones

- No se debe inventar una precisión ni una configuración final.
- Si no se logra ejecutar el modelo antes de la entrega, se debe presentar la solución base, los notebooks reproducibles y dejar explícitamente documentado el resultado experimental pendiente.
- La fecha `2 de noviembre` no determina por sí sola el número de semana académica requerido por `entrega.json`; ese dato debe salir del cronograma del curso o de la indicación de la profesora.
