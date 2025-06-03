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
│   ├── database/ # Imagenes que conforman la base de datos, extraidas del ortomosaico
│   │   ├── databaste_atributes.xlsx # Archivo de excel con datos adicionales sobre las imágenes que conforman del database
│   │   ├── test/ # Imagenes de prueba a diferentes resoluciones y formatos, en RGB y modelo de elevación
│   │   │   ├── png_1x res/
│   │   │   ├── png_4x_res/
│   │   │   ├── tif_1x_res/
│   │   │   ├── tif_elev/
│   │   │   └── tif_x4_res/
│   │   ├── train/ # Imagenes de entrenamiento a diferentes resoluciones y formatos, en RGB y modelo de elevación
│   │   │   ├── png_high_res/
│   │   │   ├── tif_elev/
│   │   │   ├── tif_high_res/
│   │   │   └── tif_x4_res/
│   ├── labels/ # Etiquetas rectangulares hechas en el conjunto de prueba en formato csv y json
│   ├── results/ # Algunos resultados como segmentaciones, identificación de bounding boxes y filtrado de copas por mapa de elvación
│   └── samples/ # Imágenes de ejemplo (.tif)
├── models/ 
│   └── modelo_rgb.joblib # Modelo k-NN en espacio de color RGB entrenado
├── notebooks/
│   ├── Tree_Detection_Validation.ipynb # Ajuste del modelo y métricas
│   ├── Tree_Detection_Train.ipynb # Entrenamiento del modelo
│   └── Tree_Detection_Test.ipynb # Métricas de desempeño
│       └── Testing/ # Notebooks adicionales
│           ├── EDA_Citrus Elevation.ipynb # Pruebas iniciales de división de parches y exploración del espacio RGB y de elevación
│           ├── Knn_Segmentation.ipynb # Exploración de las caractaerísticas de color por parches de imágenees de prueba
│           ├── labeling.ipynb # Tareas asociadas al etiquetado y anpalisis de etiquetas
│           └── Testing_KNN.ipynb # Pruebas de k-NN con imágenes de prueba en RGB
├── requirements.txt        # Dependencias
└── README.md               # Este documento

