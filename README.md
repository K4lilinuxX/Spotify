# Clasificador Musical y Análisis de Popularidad (Spotify, YouTube & TikTok)

Este proyecto realiza un pipeline completo de Ciencia de Datos para limpiar, enriquecer y analizar un conjunto de datos musical. Utilizando la API de Last.fm, el sistema identifica los géneros nativos de los artistas y los clasifica mediante un motor de reglas local en dos grandes bloques culturales: **Latino** y **Anglosajón**, con el objetivo de contrastar su rendimiento y viralidad en plataformas de streaming.

---

## 🚀 Características Clave
* **Extracción Automatizada:** Conexión e interacción con la API de Last.fm usando control de tasa (*Rate Limiting*).
* **Ingeniería de Características:** Clasificación cultural vectorizada basada en expresiones regulares y máscaras booleanas con NumPy.
* **Pipeline Robusto (Idempotente):** Limpieza avanzada de cadenas con Regex para asegurar que el script se ejecute de inicio a fin ("Run All") sin errores de tipo de dato.
* **Análisis Estadístico:** Visualización de tendencias centrales (promedios), acumulados y comportamiento en Spotify, YouTube y TikTok.

---

## 🛠️ Arquitectura y Metodología de Clasificación

El proceso de segmentación analítica se divide en dos fases consecutivas independientes:

### 1. Identificación de Género Musical (El Rol de la API)
La clasificación por género musical aprovecha el etiquetado colaborativo (*folksonomía*) global de la **API de Last.fm**.
* A través del método `artist.gettoptags` y la librería `requests`, el script consulta cada uno de los 2,000 artistas únicos del dataset.
* Para optimizar la respuesta y eliminar ruido, se extraen únicamente los **5 tags más relevantes** (de mayor peso) en formato JSON, mapeándolos al DataFrame en la columna `Generos`.

### 2. Segmentación Cultural (El Rol del Script)
La API de Last.fm no agrupa por regiones ni bloques culturales. La clasificación final en los grupos **'Latino'** y **'Anglosajón'** es ejecutada al 100% en el entorno local mediante la función `np.select()` aplicando las siguientes reglas de negocio:

* **Bloque Latino:** Filtra coincidencias con términos de la cultura hispanohablante y ritmos regionales:
  $$\texttt{latin, reggaeton, mexicano, urbano, español, ranchera, corrido, salsa, bachata}$$
* **Bloque Anglosajón:** Requiere una **regla de doble validación condicional**:
  1. Contener géneros raíz o de mercados globales: `pop, rap, hip hop, rock, country, uk, english, indie`.
  2. **Exclusión Estricta (`~es_latino`):** No debe contener ninguna etiqueta del bloque latino. Esto evita falsos positivos en fusiones musicales urbanas (ej. un tema de reggaetón etiquetado también como *pop*).
* **Remanentes:** Los casos no coincidentes o con tags vacíos se catalogan como `'Otro'` y se descartan en el subset `df_analisis` para mantener la pureza estadística.

---

## 🧹 Pipeline de Limpieza de Datos

Para garantizar la estabilidad del script al ejecutar todas las celdas en cadena (*Run All*), las métricas numéricas con formato de texto (que incluyen comas como separadores de miles y millones) pasan por un proceso de sanitización con expresiones regulares antes de su conversión de tipo:

```python
# Ejemplo de sanitización aplicado a columnas de vistas/streams
df_analisis["Spotify Streams"] = df_analisis["Spotify Streams"].astype(str).str.replace(r',', '', regex=True)
df_analisis["Spotify Streams"] = df_analisis["Spotify Streams"].replace(['nan', ''], '0').fillna('0').astype(int)
```

## 📊 Hallazgos Estadísticos y Conclusiones

Tras limpiar el set de datos y aislar una muestra final de 2,792 registros anglosajones y 687 registros latinos, se obtuvieron las siguientes métricas clave de rendimiento comercial:

1. Popularidad vs. Reproducciones Activas
* Popularidad Promedio (Spotify): El bloque Latino lidera con un 67.10 frente a un 64.78 del bloque Anglosajón. Esto demuestra el alto impacto, vigencia y velocidad de consumo que tienen los lanzamientos en español en las listas de éxitos actuales.

* Streams Promedio por Canción: El bloque Anglosajón se impone en volumen histórico acumulado por pista.

2. Margen de Distribución de Streams
El margen exacto por el cual aventaja el bloque Anglosajón al Latino en reproducciones promedio por canción se define mediante la siguiente relación matemática:

$$\text{Margen} = 505.87\text{ millones} - 400.19\text{ millones}$$
$$\text{Margen} = 105.68\text{ millones de streams}$$

Esto significa que una canción promedio de raíz angloparlante cuenta con un 26.4% más de reproducciones individuales que una canción del bloque latino dentro del dataset estudiado, explicado principalmente por el alcance histórico y global de dicho mercado.  

## 🗂️ Requisitos e Instalación

Para ejecutar este entorno de desarrollo, asegúrate de contar con Python 3.x e instalar las dependencias requeridas:

```bash
pip install pandas numpy matplotlib seaborn requests
```