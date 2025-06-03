# citrus3-detector
## Detección y Conteo automático de Árboles Cítricos a partir de Imágenes Aéreas

Este proyecto busca automatizar la identificación y conteo de **árboles de naranja y limón** en parcelas agrícolas usando imágenes adquiridas con **drones** y un **modelo de detección de objetos previamente entrenado**. Está desarrollado siguiendo la metodología **CRISP-DM**, y se enfoca en facilitar la toma de decisiones agronómicas aprovechando los desarrollos de la visión por computador.

---

## 🧠 Metodología CRISP-DM

### 1. Comprensión del Negocio

- **Objetivo**: Automatizar el conteo y localización de árboles cítricos (naranja y limón) a partir de imágenes aéreas.
- **Motivación**: Reducir tiempos y automatizar el monitoreo de cultivos. Permitir el análisis del terreno y facilitar tareas de inventario.
- **Área de análisis**: 93 hectáreas, con presencia mayoritaria de cítricos y elementos adicionales como árboles nativos, viviendas, rocas y caminos.

---

### 2. Comprensión de los Datos

- **Fuente**: Imágenes RGB obtenidas mediante un vuelo de dron, con calibración geométrica y radiométrica. 
- **Formato**: Ortomosaicos georreferenciados (formato raster, GeoTIFF).
- **Contenido**:
  - Árboles cítricos (objetivo)
  - Elementos no relevantes para el problema: árboles nativos, casas, caminos, sombras
- **Cobertura heterogénea**: Requiere filtrado cuidadoso post-inferencia.
> Consulta algunas imagenes de prueba en formato `.tif` en la carpeta [Samples](data/samples)
---

## 🧩 Flujo de Modelado Propuesto
1. **División en parches y extracción de características**
   - Cada imagen se divide en parches
   - Para cada parche se calculan estadísticas (histograma de color en RGB/HSV)
   - *Hiperparámetros*: tamaño del parche, stride (salto entre parches), espacio de color

2. **Clustering de características**
   - Se entrena un modelo KNN con todos los parches de entrenamiento
   - Se concatenan características (histogramas) para robustez
   - *Hiperparámetro*: número de clusters

3. **Selección de clusters relevantes**
   - Análisis humano de histogramas por cluster
   - Selección de clusters que representan árboles cítricos

4. **Generación de máscaras**
   - Nueva división de imágenes en parches
   - Clasificación con el KNN entrenado
   - Creación de máscaras binarias seleccionando clusters relevantes

5. **Procesamiento morfológico**
   - Erosión de máscaras con kernel cruz

6. **Integración con datos de elevación**
   - Alineación y preprocesado de imagen de elevación:
     - Suavizado con filtro gaussiano
     - Detección de elevaciones puntuales (elevación - suavizado)
     - Filtrado por umbrales (5 opciones para umbral mínimo, máximo constante)
   - Multiplicación de máscaras por elevación filtrada
   - Detección de rectángulos y centroides por capa de elevación
   - *Ventaja*: Reduce influencia de vegetación baja y separa copas superpuestas

7. **Supresión no máxima jerárquica**
   - Eliminación de duplicados por IoU
   - Prioridad a capas con mayor umbral de elevación
   - *Hiperparámetro*: threshold de IoU

8. **Optimización y evaluación**
   - Ajuste de hiperparámetros con imágenes de validación
   - Evaluación final con imágenes de test

---

### 🧠 Ventajas del Enfoque

- **Robustez**: Combinación de características visuales y de elevación
- **Flexibilidad**: Adaptable a diferentes condiciones de cultivo
- **Eficiencia**: Procesamiento por etapas para manejo de grandes áreas

---

### 🧪 Estrategia de Evaluación
- **Métricas**:
  - Precisión (Precision)
  - Exhaustividad (Recall)
  - F1-Score
  - Error absoluto en conteo
- **Evaluación espacial**:
  - Visualización sobre QGIS para validación geoespacial.

---

### ⚙️ Entorno de Desarrollo
- **Lenguaje**: Python 3.10+
- **Librerías clave**:
  - `opencv`, `scikit-image` (procesamiento de imágenes)
  - `scikit-learn` (KNN y clustering)
  - `geopandas`, `rasterio`, `shapely` (datos geoespaciales)
  - `QGIS` (visualización y análisis espacial)

---
## 📂 Estructura Actual del Proyecto

```bash
citrus3-detector/
├── data/
│   ├── samples/            # Imágenes de ejemplo (.tif)
│   ├── annotations/        # Etiquetas manuales
│   └── results/            # Resultados de detección
│
├── notebooks/
│   ├── 01_Data_Exploration.ipynb
│   ├── 02_Color_Clustering.ipynb
│   ├── 03_Elevation_Analysis.ipynb
│   └── 04_Full_Pipeline.ipynb
│
├── src/
│   ├── preprocessing.py    # Manejo de imágenes
│   ├── clustering.py       # Modelo KNN
│   ├── masking.py          # Generación de máscaras
│   ├── elevation.py        # Procesamiento DSM
│   ├── detection.py        # Pipeline completo
│   └── evaluation.py       # Métricas y visualización
│
├── docs/                   # Documentación adicional
├── requirements.txt        # Dependencias
└── README.md               # Este documento

