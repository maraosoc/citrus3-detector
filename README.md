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

## 🧩 Estrategia de Resolución del Problema
Se basa en un enfoque de tres fases (inspirado en [Weinstein, B.G.; Marconi, S.; Bohlman, S.; Zare, A.; White, E.P., 2019](https://doi.org/10.3390/rs11111309)):
- **(1) Fase no supervisada**: Se generan etiquetas suaves por medio de modelos no supervisados sobre las imágenes RGB. (A definir: RGB + elevación)
- **(2) Fase Auto-supervisada**: Se refinan las predicciones sobreponiendo las etiquetas ruidosas (por ejemplo cuando se tienen varias predicciones de un mismo árbol) y conservando solo las predicciones de alta confianza. Con esto, se entrena un modelo de detección.
- **(3) Fase supervisada**: Se reentrena el modelo de detección con etiquetas de alta calidad hechas a mano.
 ![Marco de trabajo propuesto](https://github.com/user-attachments/assets/6d3609dd-2594-4cbe-adea-6a75b289d5a6)

### 🧠 Tipo de Modelo

- **Modelo de detección**: Modelo de detección de árboles preentrenado y posteriormente ajustado para distinguir clases de árboles.
> Inicialmente se considera usar el de [DeepForest](https://github.com/weecology/DeepForest)
- **Formato de salida**: Bounding Boxes con clase y probabilidad georreferenciados. 

---

### 🏷️ Generación de Etiquetas
- **Opciones**:
  - Anotar manualmente un subconjunto representativo de parches.
  - Herramientas posibles: [LabelStudio](https://github.com/HumanSignal/label-studio/), [Qgis](https://qgis.org/).
- **Clases a identificar**:
  - `naranja`
  - `limón`
  - (Otras clases como árboles nativos se ignoran en esta etapa)

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

### ⚙️ Entorno de Entrenamiento y Desarrollo
- **Lenguaje**: Python 3.10+
- **Librerías clave**:
  - `DeepForest`, `torch`, `torchvision` (modelo base y fine-tunning)
  - `opencv`, `numpy`, `matplotlib` (procesamiento de imagen)
  - `geopandas`, `rasterio`, `shapely` (datos geoespaciales)
  - `QGIS` (visualización y análisis espacial)
---

## 📂 Estructura del Proyecto

```bash
citrus3-detector/
├── data/
│   ├── tiles/              # (A futuro) Mosaicos de la ortofoto generada por el dron para entrenamiento y validacion
│   ├── samples/            # Parches del ortomosaico como ejemplo y para pruebas
│   ├── results/            # (A futuro) Resultados de inferencia y conteo
│   ├── annotations/        # (A futuro) Etiquetas de entrenamiento
│   └── shapefiles/         # (A futuro) Capas geoespaciales para QGIS
│
├── models/
│   ├── pretrained/         # (A futuro) Modelo de detección de árboles
│   └── inference/          # (A futuro) Scripts de ejecución
│
├── scripts/
│   ├── preprocess.py       # (A futuro) Procesamiento de imágenes
│   ├── run_inference.py    # (A futuro) Ejecución del modelo
│   ├── postprocess.py      # (A futuro) Conteo y filtrado
│   └── to_qgis.py          # (A futuro) Exportación a shapefile/GeoJSON
│
├── notebooks/
│   ├── DeepForest.ipynb       # Pruebas iniciales con el paquete DeepForest
│   ├── object_detection.ipynb # Pruebas iniciales de carga de etiquetas y detección de árboles
│   └── lightning_logs         # Resultados de pytorch-lightning logger
│
├── requirements.txt (A futuro)
└── README.md               # Este documento

