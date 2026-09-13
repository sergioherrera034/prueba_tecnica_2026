# 📊 Prueba Técnica - Modelo Predictivo de acepación de opción de pago

Modelo de Machine Learning para predecir la probabilidad de que un cliente acepete una opcion de pago en el contexto de cobranza financiera.

## 📋 Descripción del Proyecto

Este proyecto implementa un **torneo de modelos** con técnicas de optimización avanzada para desarrollar un clasificador binario que predice el cumplimiento de obligaciones de pago. El flujo incluye:

1. **Análisis Descriptivo** → Exploración y preparación de datos
2. **Torneo de Modelos** → Entrenamiento y comparación de 4 algoritmos (RF, LGB, GB, LR)
3. **Optimización de Hiperparámetros** → RandomizedSearchCV (15-20x más rápido)
4. **Optimización de Umbral** → Ajuste de threshold para maximizar F1-Score
5. **Reentrenamiento y Predicción** → Modelo final en datos Out-of-Time

## 🎯 Casos de Uso

- Priorización de clientes en estrategias de cobranza
- Segmentación por probabilidad acepacion
- Asignación óptima de recursos de recaudo
- Validación out-of-time de modelos en producción

## 📁 Estructura del Proyecto

```
prueba-tecnica/
├── README.md                                    # Este archivo
├── pyproject.toml                              # Configuración del proyecto
├── src/
│   ├── descriptivo/
│   │   ├── descriptivo.ipynb                   # Análisis exploratorio de datos
│   │   └── data/
│   │       ├── insumos/                        # Datos de entrada
│   │       │   ├── prueba_op_base_pivot_*.csv
│   │       │   ├── prueba_op_master_customer_*.csv
│   │       │   └── prueba_op_probabilidad_*.csv
│   │       └── sample_submission.csv           # Formato de salida esperado
│   │
│   └── torneo_modelos/
│       ├── torneo_modelos.ipynb               # Pipeline completo del torneo
│       ├── config/
│       │   └── config.yaml                    # Configuración del torneo
│       ├── data/
│       │   └── base_final.csv                 # Datos procesados
│       └── outputs/
│           └── [timestamp]/
│               ├── modelo_ganador_final.pkl   # Modelo reentrenado
│               ├── optimal_threshold.json     # Umbral óptimo
│               ├── resultados.json            # Métricas torneo
│               ├── resultado_prueba.csv       # Predicciones finales
│               ├── predicciones_train.csv
│               ├── predicciones_test.csv
│               └── *.pkl                      # Modelos intermedios
```

## 🚀 Inicio Rápido

### Requisitos Previos

- **Python:** 3.9 o superior
- **pip:** Gestor de paquetes de Python
- **Git:** Para clonar el repositorio (opcional)

### 1️⃣ Crear y Activar Ambiente Virtual

**En Windows (PowerShell):**
```bash
# Crear ambiente virtual
python -m venv venv

# Activar
.\venv\Scripts\Activate.ps1
```

**En Windows (CMD):**
```bash
python -m venv venv
venv\Scripts\activate.bat
```

**En macOS/Linux:**
```bash
python3 -m venv venv
source venv/bin/activate
```

### 2️⃣ Instalar Dependencias

```bash
# Actualizar pip
python -m pip install --upgrade pip

# Instalar desde pyproject.toml
pip install -e .
```

**Dependencias principales:**
- `jupyter` - Entorno de notebooks
- `pandas` - Manipulación de datos
- `scikit-learn` - Modelos de ML
- `lightgbm` - Gradient boosting optimizado
- `numpy` - Computación numérica
- `matplotlib` / `seaborn` - Visualizaciones
- `openpyxl` - Lectura/escritura Excel

### 3️⃣ Ejecutar los Notebooks

**Análisis Descriptivo:**
```bash
jupyter notebook src/descriptivo/descriptivo.ipynb
```

**Torneo de Modelos:**
```bash
jupyter notebook src/torneo_modelos/torneo_modelos.ipynb
```

## 📊 Pipeline del Torneo de Modelos

### Fase 1: Preparación de Datos
- Carga de múltiples fuentes de datos
- Particionamiento en TRAIN (80%), TEST (20%), EXCLUIDO_TEST
- Validación out-of-time con partición VAL

### Fase 2: Búsqueda de Hiperparámetros

| Modelo | Hiperparámetros | Iteraciones |
|--------|---|---|
| **Random Forest** | n_estimators, max_depth, min_samples_split, min_samples_leaf, max_features | 25 |
| **LightGBM** | n_estimators, num_leaves, max_depth, learning_rate, min_data_in_leaf, feature_fraction | 25 |
| **Gradient Boosting** | n_estimators, max_depth, learning_rate, subsample, min_samples_split, min_samples_leaf, max_features | 25 |
| **Logistic Regression** | C, solver, max_iter, class_weight | 25 |

**Estrategia:** RandomizedSearchCV (2-fold StratifiedKFold)  
**Métrica:** F1-Score  
**Tiempo:** ~2-5 minutos (vs 60-80 min con GridSearchCV)

### Fase 3: Selección del Ganador
- Modelo con mejor F1-Score en TEST
- Reentrenamiento con TRAIN + TEST + EXCLUIDO_TEST

### Fase 4: Optimización de Umbral
- Búsqueda de threshold óptimo en TEST + EXCLUIDO_TEST
- Iteración: 0.00 a 1.00 (paso 0.01)
- Maximización de F1-Score

### Fase 5: Predicción Final
- Modelo ganador aplicado a partición VAL (Out-of-Time)
- Umbral optimizado aplicado en predicciones
- Exportación de resultados

## 📈 Resultados Generados

Los outputs se guardan en `src/torneo_modelos/outputs/[timestamp]/`:

| Archivo | Descripción |
|---------|---|
| `modelo_ganador_final.pkl` | Modelo entrenado con TRAIN+TEST+EXCLUIDO_TEST |
| `optimal_threshold.json` | Umbral optimizado para decisiones binarias |
| `resultados.json` | Métricas completas del torneo y reentrenamiento |
| `resultado_prueba.csv` | Predicciones finales con formato estándar |
| `predicciones_train.csv` | Predicciones y probabilidades en TRAIN |
| `predicciones_test.csv` | Predicciones y probabilidades en TEST |
| `[model]_importance.csv` | Feature importance por modelo |

## 🔍 Estructura de Datos

### Variables de Entrada

**Identificación:**
- `nit_enmascarado` - ID cliente enmascarado
- `num_oblig_enmascarado` - ID obligación enmascarado

**Temporales:**
- `fecha_var_rpta_alt` - Fecha de evaluación target (YYYYMM)
- `fecha_analisis` - Fecha de análisis (derivada)
- `uso_detalle` - Partición: train, test, val, excluido_test

**Segmentación:**
- `segmento` - Segmento de negocio
- `producto` - Tipo de producto (Tarjeta, Crédito Consumo, etc.)

### Variable Target

- `var_rpta_alt` - Binaria: 1 (Cumple), 0 (Incumple)

## 📝 Configuración

Editar `src/torneo_modelos/config/config.yaml`:

```yaml
# Parámetros principales
seed: 42
cv_splits: 2
n_iter_search: 25
scoring_metric: 'f1'

# Particiones
particion_col: 'uso_detalle'
target_col: 'var_rpta_alt'

# Modelos a evaluar
models:
  - 'Random Forest'
  - 'LightGBM'
  - 'Gradient Boosting'
  - 'Logistic Regression'
```

## 📊 Métricas del Modelo

El modelo es evaluado con las siguientes métricas:

- **Accuracy** - Exactitud general
- **Precision** - Proporción de predicciones positivas correctas
- **Recall** - Cobertura de casos positivos
- **F1-Score** - Promedio armónico (métrica de optimización)
- **AUC-ROC** - Curva de características operativas
- **Matriz de Confusión** - Desglose de predicciones

## 🛠️ Troubleshooting

### Error: "ModuleNotFoundError"
```bash
# Reinstalar dependencias
pip install -e .
```

### Error: "No module named jupyter"
```bash
# Instalar jupyter
pip install jupyter
```

### Los notebooks no se cargan
```bash
# Reiniciar kernel de Jupyter
jupyter notebook --NotebookApp.iopub_data_rate_limit=1.0e10
```

## 📚 Referencias Técnicas

- **RandomizedSearchCV** - Búsqueda aleatoria de hiperparámetros (sklearn)
- **StratifiedKFold** - Validación cruzada estratificada por clases
- **LightGBM** - Gradient boosting basado en árboles
- **Feature Importance** - Análisis de importancia relativa