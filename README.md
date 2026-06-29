# Low-Resource Amazonian NMT: Estudio Comparativo

Estudio comparativo de modelos de traducción automática neuronal (NMT) y métricas de evaluación para lenguas amazónicas de escasos recursos utilizando **mBART50**, **mT5** y **NLLB**.

Este repositorio contiene los pipelines de preprocesamiento, extensión de tokenizadores, entrenamiento, reentrenamiento y evaluación desarrollados como parte de la tesis de maestría:
> *Estudio comparativo de modelos de traducción automática neuronal y métricas de evaluación en lenguas amazónicas de escasos recursos*

---

## Lenguas y Modelos Evaluados

*   **Lenguas Amazónicas**:
    *   **Shipibo-Konibo** (Familia Pano)
    *   **Ashaninka** (Familia Arawak)
    *   **Yine** (Familia Arawak)
*   **Modelos Base**:
    *   **mBART50** (`facebook/mbart-large-50-many-to-many-mmt`)
    *   **mT5** (`google/mt5-base` / `google/mt5-small`)
    *   **NLLB** (`facebook/nllb-200-distilled-600M`)
*   **Métricas de Calidad**:
    *   **BLEU** (Evaluación basada en n-gramas a nivel de palabra)
    *   **ChrF** (Evaluación basada en n-gramas a nivel de caracteres, ideal para lenguas aglutinantes)
    *   **COMET** (Métrica semántica basada en embeddings neuronales de referencia/fuente)
    *   **METEOR** y **BERTScore** (Métricas de soporte para robustez semántica)

---

## Estructura del Repositorio

El repositorio se organiza de forma secuencial de acuerdo al pipeline de experimentación:

```plaintext
comparative-study-amazonian-nmt/
├── 0_datasets/                      # Conjuntos de datos estructurados por lengua
│   ├── ashaninka/                   # CSVs de corpus completo, train, val y test (Ashaninka-Español)
│   ├── shipibo_konibo/              # CSVs correspondientes para Shipibo-Konibo
│   └── yine/                        # CSVs correspondientes para Yine
├── 1_preprocessing/                 # Scripts de curaduría y limpieza de datos
│   ├── Ashaninka/
│   │   └── 1_limpieza_de_datos.py   # Remoción de ruido, filtrado por longitud y duplicados
│   ├── Shipibo-Konibo/
│   │   └── 1_limpieza_de_datos.py
│   └── Yine/
│       └── 1_limpieza_de_datos.py
├── 2_tokenizer_extension/            # Estrategias de aumento de tokenizadores (Vocabulario)
│   └── Shipibo-Konibo/
│       ├── 2_tokenizacion_mbart50_v1.py  # Estrategia de segmentación y extensión v1
│       ├── 2_tokenizacion_mbart50_v2.py  # Estrategia v2
│       └── 2_tokenizacion_mbart50_v3.py  # Estrategia v3
└── 3_training/                      # Cuadernos de entrenamiento y evaluación en Google Colab
    ├── Ashaninka/
    │   ├── 3_Entrenamiento_mBART50_v4.ipynb
    │   ├── 3_Entrenamiento_mT5_v4.ipynb
    │   └── 3_Entrenamiento_NLLB_v4.ipynb
    ├── Shipibo-Konibo/
    │   ├── 3_Entrenamiento_mBART50_v4.ipynb
    │   ├── 3_Entrenamiento_mT5_v4.ipynb
    │   └── 3_Entrenamiento_NLLB_v4.ipynb
    └── Yine/
        ├── 2_Entrenamiento_mBART50_v4.ipynb
        ├── 2_Entrenamiento_mT5_v4 .ipynb
        └── 2_Entrenamiento_NLLB_v4.ipynb
```

---

## Pipeline de Trabajo

### 1. Preprocesamiento ([1_preprocessing/](file:///c:/Users/Miguel/Desktop/comparative-study-amazonian-nmt/1_preprocessing/))
Para cada lengua, se procesan los datos crudos y se realiza:
*   Conversión a minúsculas y normalización de espacios.
*   Remoción de palabras sueltas (ruido de alineación con longitud de palabras <= 1).
*   Filtrado por longitud mínima de caracteres (longitud > 2).
*   Filtro de desbalance de longitud entre fuente y destino (umbral < 150 caracteres para Shipibo-Konibo, < 100 para Yine y Ashaninka).
*   Eliminación de pares idénticos y duplicados de traducción.
*   División estratificada según el origen del dataset (p. ej. relatos, educativo, religioso) en: **80% Entrenamiento** ([train.csv](file:///c:/Users/Miguel/Desktop/comparative-study-amazonian-nmt/0_datasets/shipibo_konibo/train.csv)), **10% Validación** ([val.csv](file:///c:/Users/Miguel/Desktop/comparative-study-amazonian-nmt/0_datasets/shipibo_konibo/val.csv)) y **10% Prueba** ([test.csv](file:///c:/Users/Miguel/Desktop/comparative-study-amazonian-nmt/0_datasets/shipibo_konibo/test.csv)).

### 2. Extensión del Tokenizador ([2_tokenizer_extension/](file:///c:/Users/Miguel/Desktop/comparative-study-amazonian-nmt/2_tokenizer_extension/))
Se implementan estrategias de segmentación y silabeo (v1, v2 y v3) específicas para el Shipibo-Konibo con el fin de ampliar el vocabulario original del tokenizador preentrenado (de mBART50). Esto permite reducir la fragmentación excesiva y mejorar la representación de unidades léxicas en lenguas polisintéticas con morfología compleja.

*Nota: Esta carpeta y sus archivos no se ejecutan como parte del flujo de trabajo de entrenamiento habitual; su propósito es evidenciar los pasos y sustento experimental de la sección 3.1 de la tesis. Para ejecutar el pipeline principal, solo deben correrse los archivos de preprocesamiento (`1_preprocessing`) y posteriormente los cuadernos de entrenamiento (`3_training`).*


### 3. Entrenamiento y Evaluación ([3_training/](file:///c:/Users/Miguel/Desktop/comparative-study-amazonian-nmt/3_training/))
La metodología de experimentación en los cuadernos consta de **dos fases**:
1.  **Fase 1**: Ajuste fino (Fine-Tuning) inicial del modelo utilizando el dataset limpio. Se evalúa el modelo guardado en el conjunto de validación (`val.csv`).
2.  **Fase 2 (Reentrenamiento)**: Limpieza del entorno de GPU (`torch.cuda.empty_cache()`), recarga de los pesos del modelo de Fase 1, reentrenamiento ajustando hiperparámetros adicionales y evaluación final. Las métricas finales del estudio se evalúan sobre el conjunto de validación final y sobre el conjunto de prueba independiente (`test.csv`).

---

## Resultados y Métricas Finales (Conjunto de Prueba)

A continuación se resumen los resultados obtenidos en el conjunto de test independiente (`test.csv`):

| Lengua | Modelo NMT | BLEU (%) ↑ | ChrF ↑ | COMET (wmt22) ↑ | METEOR ↑ | BERTScore (F1) ↑ |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| **Ashaninka** | **mBART50** | **15.56%** | **41.59** | **0.7527** | 0.3196 | 0.8282 |
| | mT5 | 0.00% | 15.73 | 0.5667 | 0.0734 | 0.7313 |
| | NLLB | 14.36% | 39.09 | 0.7406 | 0.3073 | 0.8264 |
| **Shipibo-Konibo** | mBART50 | 4.48% | 22.51 | 0.4444 | 0.1985 | 0.6943 |
| | mT5 | 3.41% | 21.30 | 0.4736 | 0.1508 | 0.7050 |
| | **NLLB** | **12.74%** | **34.86** | **0.6181** | **0.3156** | **0.7806** |
| **Yine** | **mBART50** | **17.40%** | **36.02** | **0.6515** | **0.3898** | **0.7996** |
| | mT5 | 12.58% | 29.74 | 0.5971 | 0.3192 | 0.7892 |
| | NLLB | 5.44% | 23.81 | 0.5580 | 0.2685 | 0.7549 |



### Conclusiones del Estudio:
1.  **mBART50** demuestra el mejor desempeño en Ashaninka (BLEU: 15.56%, ChrF: 41.59) y Yine (BLEU: 17.40%, ChrF: 36.02).
2.  **NLLB** sobresale significativamente en Shipibo-Konibo, superando a los otros modelos con un BLEU de 12.74% y ChrF de 34.86.
3.  **mT5** presenta claras dificultades en conjuntos de datos extremadamente pequeños (como Ashaninka, donde obtiene BLEU 0.00% e inestabilidad general), demostrando ser altamente sensible a la escasez crítica de recursos si no se aplican hiperparámetros fuertemente regularizados o vocabularios adaptados.
4.  **COMET**: El uso de la métrica `wmt22-comet-da` en la fase de evaluación final proporciona puntajes más estables y legibles en comparación con las métricas anteriores.

---

## Guía de Reproducción para Investigadores

Si desea reproducir, validar o extender estos experimentos, siga estas instrucciones:

### 1. Requisitos de Entorno
Instale las librerías requeridas de Python (preferiblemente en un entorno con GPU habilitada, como Google Colab o servidores dedicados):
```bash
pip install transformers[torch] datasets evaluate sacrebleu unbabel-comet bert-score sentencepiece pandas scikit-learn
```

### 2. Flujo de Ejecución Paso a Paso
1.  **Limpieza de Datos**:
    *   Navegue al directorio de preprocesamiento del idioma deseado (p. ej., [1_preprocessing/Shipibo-Konibo/](file:///c:/Users/Miguel/Desktop/comparative-study-amazonian-nmt/1_preprocessing/Shipibo-Konibo/)).
    *   Ejecute el script `1_limpieza_de_datos.py` para generar los archivos `corpus.csv`, `train.csv`, `val.csv` y `test.csv` a partir del corpus crudo.
2.  **Extensión del Tokenizador (Evidencia)**:
    *   Esta etapa no es ejecutable dentro del flujo ordinario de entrenamiento. Su propósito es evidenciar de manera teórica e ilustrativa las pruebas de silabeo y vocabulario presentadas en la sección 3.1 de la tesis. Para la reproducción del pipeline de traducción, puede omitirse y pasar directamente al entrenamiento.
3.  **Fine-Tuning y Reentrenamiento**:
    *   Abra el cuaderno Jupyter correspondiente de la carpeta [3_training/](file:///c:/Users/Miguel/Desktop/comparative-study-amazonian-nmt/3_training/) (por ejemplo, [3_Entrenamiento_NLLB_v4.ipynb](file:///c:/Users/Miguel/Desktop/comparative-study-amazonian-nmt/3_training/Shipibo-Konibo/3_Entrenamiento_NLLB_v4.ipynb)) en Google Colab o su entorno local.
    *   Configure la variable `base_path` apuntando al almacenamiento correspondiente.
    *   Ejecute secuencialmente las celdas para Fine-Tuning (Fase 1), Limpieza de memoria RAM/GPU, Reentrenamiento (Fase 2) y Evaluación.
4.  **Evaluación**:
    *   Las celdas finales de cada cuaderno cargarán automáticamente los conjuntos de validación/prueba y calcularán las métricas utilizando la API de `evaluate`.
