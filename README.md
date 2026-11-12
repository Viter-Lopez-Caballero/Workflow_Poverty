# Detectar la pobreza sin medir el ingreso

### Aprendizaje automático y redes neuronales aplicados a la ENIGH 2016–2020

**Autores:** Lucía Morales-Morales, Vitervo López-Caballero\* y Alan Hernández-Solano  
\*Autor de correspondencia

Este repositorio contiene los datos, el diccionario de datos y los cuadernos de Google Colab necesarios para reproducir los resultados del artículo *«Detectar la pobreza sin medir el ingreso: aprendizaje automático y redes neuronales aplicados a la ENIGH 2016–2020»*, preparado para el *International Journal of Combinatorial Optimization Problems and Informatics* (IJCOPI).

**Pregunta de investigación:** ¿es posible identificar a los hogares mexicanos en pobreza por ingresos **sin observar su ingreso**, solo a partir de sus características sociodemográficas, de seguridad alimentaria, de vivienda y de equipamiento?

---

## Contenido

- [Estructura del repositorio](#estructura-del-repositorio)
- [Datos](#datos)
- [Flujo de trabajo](#flujo-de-trabajo)
- [Cómo reproducir los resultados](#cómo-reproducir-los-resultados)
- [Resultados esperados](#resultados-esperados)
- [Entorno y versiones](#entorno-y-versiones)
- [Notas sobre la reproducibilidad](#notas-sobre-la-reproducibilidad)
- [Cómo citar](#cómo-citar)
- [Licencia y uso de los datos](#licencia-y-uso-de-los-datos)
- [Autores y contacto](#autores-y-contacto)

---

## Estructura del repositorio

```
├── 1. Dataset/
│   ├── base2016.xlsx
│   ├── base2018.xlsx
│   └── base2020.xlsx
├── 2. Diccionario de datos/
│   └── (descripción de cada atributo de las bases)
├── 3. Cuadernos de trabajo/
│   ├── Analisis_pobreza_ML_mejorado.ipynb
│   └── Analisis_pobreza_ML_v2_Optuna.ipynb
└── README.md
```

| Carpeta | Contenido |
|---|---|
| [`1. Dataset`](1.%20Dataset/) | Bases de hogares de la ENIGH 2016, 2018 y 2020 en formato Excel (una fila por hogar, 34 columnas, hoja `Sheet1`). |
| [`2. Diccionario de datos`](2.%20Diccionario%20de%20datos/) | Descripción de cada atributo: nombre, tipo, valores posibles e interpretación. |
| [`3. Cuadernos de trabajo`](3.%20Cuadernos%20de%20trabajo/) | Cuadernos de Google Colab con todo el proceso de aprendizaje automático. |

| Cuaderno | Propósito | Sección del artículo |
|---|---|---|
| `Analisis_pobreza_ML_mejorado.ipynb` | **Flujo principal**: preprocesamiento, búsqueda aleatoria de hiperparámetros, entrenamiento y evaluación de los modelos, pruebas estadísticas, ablación, importancia de variables y análisis de sensibilidad. | Secciones 2, 3.1–3.3, 3.5 |
| `Analisis_pobreza_ML_v2_Optuna.ipynb` | **Análisis de robustez**: optimización bayesiana de hiperparámetros (TPE + poda Hyperband), ensamble ponderado y calibración del umbral de decisión. | Sección 3.4 |

---

## Datos

Los datos provienen de la **Encuesta Nacional de Ingresos y Gastos de los Hogares (ENIGH)**, nueva serie, del Instituto Nacional de Estadística y Geografía (INEGI), ediciones 2016, 2018 y 2020.

| Archivo | Hogares |
|---|---|
| `base2016.xlsx` | 70 311 |
| `base2018.xlsx` | 74 647 |
| `base2020.xlsx` | 89 006 |
| **Total** | **233 964** |

**Variable objetivo:** `plp` (1 = hogar con ingreso corriente per cápita por debajo de la línea de pobreza por ingresos de CONEVAL; 43.33 % de los hogares).

**Variables excluidas de los predictores:**

| Variable | Motivo |
|---|---|
| `folioviv`, `foliohog` | Identificadores sin valor predictivo. |
| `ictpc` | **Fuga de información**: `plp` se obtiene directamente al comparar `ictpc` con la línea de pobreza. |
| `plp_e` | **Fuga de información**: pobreza extrema por ingresos, derivada del mismo ingreso. |
| `escala_elcsa` | Redundante: se reproduce exactamente a partir de las 8 variables `acc_alim_*`. |

La descripción completa de cada atributo se encuentra en la carpeta [`2. Diccionario de datos`](2.%20Diccionario%20de%20datos/).

> **Advertencia sobre `prog_soc_porcent`:** esta variable es la proporción del ingreso que proviene de programas sociales, por lo que contiene información *indirecta* del ingreso. Sin ella, la exactitud de XGBoost baja de 0.7779 a 0.7571 (sección 3.5 del artículo).

---

## Flujo de trabajo

```
1. Recolección e integración  →  2. Preprocesamiento  →  3. Modelado  →  4. Evaluación e interpretación
```

1. **Integración:** unión de las tres ediciones, verificación de columnas y duplicados, y creación de la variable `anio`.
2. **Preprocesamiento** (ajustado solo con datos de entrenamiento):
   - Exclusión de identificadores, variables con fuga de información y variables redundantes.
   - Imputación (mediana / moda) e indicador de dato faltante para `bano_comp`.
   - Ingeniería de características: tasa de dependencia, ocupados por integrante, hacinamiento, integrantes por baño, suma de inseguridad alimentaria, índice de bienes, hogar sin ocupados.
   - *One-Hot Encoding* de `entidad` y `anio`; escolaridad como variable ordinal; estandarización de variables numéricas.
   - División estratificada 80/20 (entrenamiento / prueba) y 10 % del entrenamiento como validación.
3. **Modelado:** ocho clasificadores:
   - Red neuronal totalmente conectada (FCNN) 128-64-32-16-1 (TensorFlow/Keras)
   - Random Forest
   - Regresión logística
   - Perceptrón multicapa (MLP)
   - Perceptrón simple (clásico) y perceptrón promediado
   - XGBoost
   - Gradient Boosting (`HistGradientBoostingClassifier`)
4. **Evaluación:** exactitud, precisión, *recall*, F1-score, ROC-AUC, MCC, intervalos de confianza al 95 % por *bootstrap*, prueba de McNemar, estudio de ablación e importancia de variables por permutación.

---

## Cómo reproducir los resultados

### Opción A: Google Drive (recomendada)

1. Descargue o clone este repositorio.
2. Suba la carpeta del repositorio a su Google Drive.
3. Abra el cuaderno deseado de `3. Cuadernos de trabajo` con Google Colab.
4. En la **sección 2 (Carga de datos)**, cambie la ruta para que apunte a la carpeta `1. Dataset` de su Drive, por ejemplo:

   ```python
   RUTA_DATOS = "/content/drive/MyDrive/<nombre-del-repositorio>/1. Dataset/"
   ```

5. Ejecute todas las celdas: `Entorno de ejecución → Ejecutar todas`. Colab solicitará permiso para acceder a Google Drive.

### Opción B: clonar el repositorio directamente en Colab

1. Abra el cuaderno en Colab desde GitHub (`Archivo → Abrir cuaderno → GitHub`).
2. Agregue una celda al inicio y ejecútela:

   ```python
   !git clone https://github.com/<usuario>/<repositorio>.git
   ```

3. En la **sección 2**, después de la celda que monta Google Drive, reemplace la ruta por:

   ```python
   RUTA_DATOS = "/content/<repositorio>/1. Dataset/"
   ```

   El cuaderno monta Google Drive automáticamente al ejecutarse en Colab; con esta línea la lectura se hace desde la copia clonada.

4. Ejecute todas las celdas.

### Dependencias adicionales

Google Colab ya incluye pandas, scikit-learn, XGBoost, TensorFlow, matplotlib y seaborn. Solo el cuaderno de robustez requiere **Optuna**: descomente la línea de instalación al inicio de la sección 10:

```python
!pip install -q optuna
```

### Tiempos de ejecución aproximados

| Cuaderno | Tiempo aproximado |
|---|---|
| `Analisis_pobreza_ML_mejorado.ipynb` | 30–60 min |
| `Analisis_pobreza_ML_v2_Optuna.ipynb` | 60–90 min |

Los tiempos dependen del hardware asignado por Colab. Para una prueba rápida, ponga `MODO_RAPIDO = True` en la sección 1 (reduce el número de pruebas de ajuste de hiperparámetros; los resultados no coincidirán con los del artículo).

### Archivos generados

Cada cuaderno guarda sus resultados en una carpeta del entorno de Colab:

| Cuaderno | Carpeta de salida | Archivos principales |
|---|---|---|
| Flujo principal | `resultados_pobreza/` | `metricas_modelos.csv`, `ic_y_mcnemar.csv`, `ablacion.csv`, `importancia_permutacion.csv`, `sensibilidad_prog_soc.csv`, `mejores_hiperparametros.json` y figuras en PNG (300 dpi) |
| Robustez | `resultados_pobreza_v2/` | Los anteriores, más `comparacion_v1_v2.csv`, `umbral_calibrado.csv`, `mejores_hiperparametros_optuna.json`, `fig_historial_optuna.png` y la base `optuna_pobreza.db` |

> Las carpetas de salida se crean en el almacenamiento temporal de Colab. Descárguelas o cópielas a su Drive antes de cerrar la sesión.

---

## Resultados esperados

### Flujo principal (conjunto de prueba, 46 793 hogares)

| Modelo | Exactitud | ROC-AUC | MCC |
|---|---|---|---|
| **XGBoost** | **0.7779** | **0.8584** | **0.5451** |
| Gradient Boosting | 0.7772 | 0.8575 | 0.5437 |
| Perceptrón multicapa | 0.7753 | 0.8541 | 0.5396 |
| FCNN 128-64-32-16-1 | 0.7750 | 0.8557 | 0.5383 |
| Random Forest | 0.7664 | 0.8445 | 0.5211 |
| Regresión logística | 0.7576 | 0.8343 | 0.5028 |
| Perceptrón promediado | 0.7545 | 0.8319 | 0.4971 |
| Perceptrón (clásico) | 0.5804 | 0.6709 | 0.2245 |

XGBoost y Gradient Boosting son estadísticamente equivalentes (prueba de McNemar, p = 0.42).

### Análisis de robustez (optimización bayesiana)

| Modelo | Exactitud con búsqueda aleatoria | Exactitud con TPE (Optuna) |
|---|---|---|
| Gradient Boosting | 0.7772 | 0.7764 |
| XGBoost | 0.7779 | 0.7758 |
| Perceptrón multicapa | 0.7753 | 0.7732 |
| FCNN 128-64-32-16-1 | 0.7750 | 0.7719 |
| Random Forest | 0.7664 | 0.7668 |
| Regresión logística | 0.7576 | 0.7576 |
| Perceptrón promediado | 0.7545 | 0.7573 |
| Perceptrón (clásico) | 0.5804 | 0.7202 |
| Ensamble ponderado | — | 0.7762 |

La optimización bayesiana no mejora a los mejores modelos: con las variables disponibles, el desempeño se satura alrededor de 0.776–0.778 de exactitud.

---

## Entorno y versiones

Los resultados del artículo se obtuvieron con:

| Componente | Versión |
|---|---|
| Python | 3.12.3 |
| scikit-learn | 1.9.1 |
| XGBoost | 3.4.1 |
| TensorFlow / Keras | 2.21.0 |
| Optuna | 5.0.0 |
| pandas | 3.0.6 |
| NumPy | 2.5.4 |
| SciPy | 1.18.1 |
| matplotlib | 3.11.2 |
| seaborn | 0.13.2 |

Hardware: entorno Linux con un núcleo de CPU y 3 GB de RAM (no se requiere GPU).

---

## Notas sobre la reproducibilidad

- Todos los procesos aleatorios usan la semilla fija `SEMILLA = 42` (particiones, modelos, búsquedas y TensorFlow).
- Con versiones de librerías distintas a las indicadas, las métricas pueden variar en milésimas, sobre todo en las redes neuronales y en las búsquedas de hiperparámetros.
- El cuaderno de robustez guarda los estudios de Optuna en `optuna_pobreza.db`. Si Colab se desconecta, al volver a ejecutar las celdas la optimización continúa donde se quedó. **Para repetir la optimización desde cero, borre ese archivo.**
- Las métricas reportadas corresponden al conjunto de prueba, que no se usa en ninguna decisión de entrenamiento ni de ajuste.
- No se usaron los factores de expansión de la ENIGH: las métricas describen el desempeño sobre la muestra, no estimaciones poblacionales.

---

## Cómo citar

Si utiliza este código o estos resultados, cite el artículo:

**Formato APA:**

```
Morales-Morales, L., López-Caballero, V., & Hernández-Solano, A. (2027). Detectar la
pobreza sin medir el ingreso: aprendizaje automático y redes neuronales aplicados a la
ENIGH 2016–2020. International Journal of Combinatorial Optimization Problems and
Informatics, [volumen]([número]), [páginas]. [DOI]
```

**BibTeX:**

```bibtex
@article{MoralesMorales2027,
  author  = {Morales-Morales, Luc{\'i}a and L{\'o}pez-Caballero, Vitervo and Hern{\'a}ndez-Solano, Alan},
  title   = {Detectar la pobreza sin medir el ingreso: aprendizaje autom{\'a}tico y redes neuronales aplicados a la {ENIGH} 2016--2020},
  journal = {International Journal of Combinatorial Optimization Problems and Informatics},
  year    = {2027},
  volume  = {[volumen]},
  number  = {[número]},
  pages   = {[páginas]},
  doi     = {[DOI]}
}
```

---

## Licencia y uso de los datos

- **Código:** [indique aquí la licencia elegida para el código, por ejemplo MIT].
- **Datos:** los microdatos originales de la ENIGH son publicados por el INEGI y están sujetos a sus *Términos de libre uso de la información del INEGI*. Las bases de la carpeta `1. Dataset` son una selección de variables a nivel hogar derivada de esos microdatos. Fuente: INEGI, Encuesta Nacional de Ingresos y Gastos de los Hogares (ENIGH) 2016, 2018 y 2020, nueva serie.

---

## Autores y contacto

| Autor | Rol |
|---|---|
| Lucía Morales-Morales | Autora |
| Vitervo López-Caballero | Autor de correspondencia |
| Alan Hernández-Solano | Autor |

**Correspondencia:** Vitervo López-Caballero — [correo electrónico] — [institución]
