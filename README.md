# Proyecto Reconocimiento

Este proyecto se centra en el Reconocimiento de Entidades Nombradas (NER, por sus siglas en inglés) y la anonimización de textos, diseñado principalmente para textos en español, particularmente descripciones relacionadas con desapariciones. Abarca una serie de scripts de Python, mayormente en formato Jupyter Notebook, que cubren la preparación de datos, procesamiento de anotaciones, entrenamiento de modelos NER usando spaCy, validación de modelos y técnicas de anonimización de texto aprovechando tanto modelos personalizados de spaCy como la biblioteca Presidio.

## Estructura del Proyecto

El proyecto está organizado en los siguientes directorios:

-   **`0_dividir_db/`**: Contiene scripts para dividir grandes conjuntos de datos en fragmentos más pequeños y manejables, adecuados para tareas de anotación.
-   **`1_procesamiento_anotaciones/`**: Incluye scripts para validar, limpiar, fusionar, filtrar opcionalmente y visualizar anotaciones NER proporcionadas en formato JSON.
-   **`2_entrenamiento_NER/`**: Alberga los scripts y archivos de configuración necesarios para entrenar un modelo NER personalizado usando spaCy, utilizando específicamente arquitecturas basadas en transformadores.
-   **`3_prueba_modelo/`**: Proporciona scripts para probar y validar el rendimiento del modelo NER entrenado. Esto incluye la generación de visualizaciones del reconocimiento de entidades y la realización de tareas básicas de anonimización.
-   **`4_anonimizacion/`**: Presenta scripts dedicados a implementar la anonimización de texto utilizando la biblioteca Presidio, a menudo en conjunto con modelos NER entrenados a medida y patrones Regex complementarios.

## 📂 Estructura del Repositorio
```text
reconocimiento/
  0_dividir_db/
    output/                      # <- Carpeta de salida para archivos divididos (subcarpetas con fecha/hora creadas aquí) conteniendo archivos .txt divididos, assignment_log.csv, verification_log.txt
    dividir_basedatos.ipynb      # Divide el CSV de entrada (ruta codificada internamente) en partes más pequeñas para anotar.
  1_procesamiento_anotaciones/
    input_annotations/           # <- Coloca aquí los archivos JSON de anotaciones de entrada.
    logs/                        # <- Logs para validación, revisión de espacios, fusión, purga y visualización se guardan aquí.
    procesamiento_anotaciones.ipynb # Valida, limpia, fusiona, opcionalmente purga entidades y visualiza anotaciones.
    # Archivos de salida (JSON fusionado/purgado, visualizador HTML) se crean a menudo relativos a la ubicación de este script o en la raíz del proyecto.
  2_entrenamiento_NER/
    annotations_dataset/         # <- Carpeta de salida para archivos train.spacy y valid.spacy generados.
    logs/                        # <- Logs para el proceso de conversión de datos (JSON a .spacy).
    output/                      # <- Carpeta de salida donde se guardan los modelos entrenados (model-best, model-last).
    entrenamiento_NER.ipynb      # Convierte las anotaciones JSON procesadas al formato spaCy e inicia el entrenamiento del modelo.
    config.cfg                   # Archivo de configuración que define el modelo spaCy, el pipeline y los parámetros de entrenamiento.
  3_prueba_modelo/
    model-best/                  # <- Coloca aquí el modelo spaCy entrenado (salida del paso 2).
    # Archivos de salida como ner_results.html/json, anonimized_output.csv/html se guardan directamente en esta carpeta.
    validacion_modelo_NER.ipynb  # Carga el modelo entrenado para probar el rendimiento NER, visualizar resultados y realizar pruebas básicas de anonimización. Rutas de CSV de entrada codificadas.
  4_anonimizacion/
    # Archivos de salida como reporte_anonimizacion.txt/html, presidio_annotations_output.json se guardan directamente en esta carpeta.
    implementacion_presidio.ipynb # Implementa la anonimización usando Presidio, combinando modelo NER personalizado y Regex. Rutas de CSV de entrada y modelo codificadas.
  README.md                      # Este archivo de documentación (estás aquí).
  ```


## Descripción General del Flujo de Trabajo

El proyecto sigue un flujo de trabajo general:

1.  **División de Datos (`0_dividir_db/`)**:
    * El script `dividir_basedatos.ipynb` toma un archivo CSV fuente (p. ej., que contiene descripciones de desapariciones), extrae la columna de texto relevante, mezcla los datos aleatoriamente y los divide en un número predefinido de archivos `.txt` más pequeños. Gestiona la salida en carpetas únicas con marca de tiempo, registra las asignaciones de archivos para los anotadores (`assignment_log.csv`) e incluye comprobaciones para verificar la integridad de los datos durante la división (`verification_log.txt`).

2.  **Procesamiento de Anotaciones (`1_procesamiento_anotaciones/`)**:
    * Esta etapa maneja archivos de anotación JSON, típicamente ubicados en un directorio `input_annotations/`.
    * El notebook `procesamiento_anotaciones.ipynb` contiene múltiples pasos:
        * **Validación**: Comprueba el formato y la integridad de los archivos de anotación JSON.
        * **Comprobación de Espacios en Blanco**: Identifica e informa sobre problemas con espacios en blanco iniciales/finales en los tramos de entidad anotados.
        * **Fusión**: Combina anotaciones de múltiples archivos JSON en un único `merged_annotations.json`, asegurando la unicidad y filtrando las entradas sin entidades.
        * **Depuración (Opcional)**: Permite la eliminación interactiva de tipos de entidad específicos de las anotaciones fusionadas, guardando el resultado como `purged_annotations.json`.
        * **Visualización**: Genera un archivo HTML (`annotations_visualizer.html`) que muestra una muestra aleatoria de las anotaciones finales con entidades resaltadas para su revisión.
    * Los registros detallados de cada paso se guardan en el directorio `logs/`.

3.  **Entrenamiento del Modelo NER (`2_entrenamiento_NER/`)**:
    * El script `entrenamiento_NER.ipynb` prepara las anotaciones procesadas para el entrenamiento[cite: 2].
        * Convierte el archivo de anotación JSON final (p. ej., `purged_annotations.json`) al formato binario (`.spacy`) de spaCy[cite: 2].
        * Los datos se dividen en conjuntos de entrenamiento (`annotations_dataset/train.spacy`) y validación (`annotations_dataset/valid.spacy`)[cite: 2].
        * El entrenamiento se ejecuta usando `spacy train` basado en el archivo `config.cfg`[cite: 2].
        * El punto de control del modelo con mejor rendimiento se guarda en `output/model-best`[cite: 2].
    * El archivo `config.cfg` dicta el proceso de entrenamiento, especificando la arquitectura del modelo (p. ej., un transformador como `dccuchile/bert-base-spanish-wwm-cased`), los componentes del pipeline (`transformer`, `ner`), las rutas de los conjuntos de datos, los hiperparámetros de entrenamiento (como dropout, tasa de aprendizaje, paciencia, pasos máximos) y la configuración de evaluación.

4.  **Validación del Modelo (`3_prueba_modelo/`)**:
    * El script `validacion_modelo_NER.ipynb` evalúa el modelo entrenado `model-best`[cite: 1].
        * Carga el modelo y lo prueba en textos de muestra, típicamente de un archivo CSV[cite: 1].
        * Genera un informe HTML (`ner_results.html`) visualizando las entidades reconocidas en los textos de muestra[cite: 1].
        * También se crea un archivo JSON correspondiente (`ner_results.json`), detallando las entidades detectadas, sus tramos y etiquetas[cite: 1].
        * Este notebook también demuestra la anonimización básica reemplazando entidades específicas (como `NOMBRE`, `DOMICILIO`) identificadas por el modelo y guarda los resultados en `anonimized_output.csv` y `anonimized_column.html`[cite: 1].

5.  **Anonimización (`4_anonimizacion/`)**:
    * El script `implementacion_presidio.ipynb` implementa un pipeline de anonimización más robusto utilizando la biblioteca Presidio.
        * Configura el `AnalyzerEngine` de Presidio, combinando potencialmente el modelo NER personalizado de spaCy con patrones Regex predefinidos o personalizados (para entidades como RFC, CURP, números de teléfono, placas).
        * Configura el `AnonymizerEngine` de Presidio con reglas (operadores) para reemplazar la información sensible detectada con marcadores de posición (p. ej., `[NOMBRE PROTEGIDO]`, `[TELEFONO PROTEGIDO]`).
        * Procesa texto, a menudo desde una fuente CSV como `repd_vp_cedulas_principal.csv`.
        * Genera informes completos: una versión de texto (`reporte_anonimizacion.txt`) y una versión HTML interactiva (`reporte_anonimizacion_visual.html`) que muestran el texto original, resaltado (entidades detectadas) y anonimizado.
        * Opcionalmente, puede generar los hallazgos de Presidio como anotaciones JSON (`presidio_annotations_output.json`).

## Dependencias Clave

-   Python 3
-   **spaCy**: Biblioteca principal (`spacy`) e integración con transformadores (`spacy-transformers`).
-   **Presidio**: Componentes Analyzer (`presidio-analyzer`) y Anonymizer (`presidio-anonymizer`).
-   **Manejo de Datos**: `pandas`.
-   **Aprendizaje Automático**: `scikit-learn` (para división de datos), `torch` (PyTorch, requerido por transformadores).
-   **Jupyter**: Para ejecutar archivos `.ipynb` (`ipywidgets` puede usarse para interactividad).
-   **Aceleración GPU (Opcional)**: `cupy-cuda12x` (o la versión apropiada para tu configuración CUDA) puede acelerar las operaciones de spaCy/PyTorch.
-   **Modelos de Lenguaje spaCy**: Se necesitan modelos preentrenados en español como `es_dep_news_trf` o `es_core_news_lg`/`md` para la tokenización base o como parte del pipeline.

## Instrucciones de Uso

1.  **Instalar Dependencias**: Asegúrate de que todos los paquetes de Python requeridos estén instalados. Usa los comandos `!pip install` que se encuentran al principio de los notebooks relevantes. Presta atención a las versiones específicas si es necesario, especialmente para PyTorch y CUDA. Descarga los modelos de lenguaje de spaCy necesarios (p. ej., `python -m spacy download es_dep_news_trf`).
2.  **Preparar Datos**: Coloca los archivos CSV de entrada y los archivos de anotación JSON en los directorios designados (p. ej., carpeta raíz, `input_annotations/`) o actualiza las rutas de archivo dentro de los notebooks para que apunten a las ubicaciones de tus datos.
3.  **Ejecutar Notebooks**: Ejecuta los Jupyter notebooks generalmente en el orden numérico (0 -> 1 -> 2 -> 3 -> 4), ya que los pasos posteriores a menudo dependen de los resultados de los anteriores.
4.  **Revisar Resultados**: Comprueba los archivos generados en carpetas como `output/` (para modelos entrenados), `logs/` (para registros de procesamiento), `annotations_dataset/` (para archivos `.spacy`), y el directorio raíz (para informes HTML, CSVs finales, JSONs).