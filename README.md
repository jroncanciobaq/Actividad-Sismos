🌎 Análisis Estadístico de Eventos Sísmicos
📌 Descripción

Este proyecto corresponde al Taller 5 de Programación para Analítica de Datos de la Maestría en Analítica de Datos de la Universidad Central.

El objetivo es realizar un proceso de exploración, limpieza, transformación, consulta y análisis estadístico de una base de datos de eventos sísmicos registrados durante un periodo aproximado de seis años.

El análisis combina herramientas de Python, Pandas, DuckDB, NumPy, Matplotlib, Seaborn y SciPy para estudiar la distribución de determinados eventos sísmicos según su magnitud, intensidad máxima estimada y nivel de amenaza.

🎯 Objetivos
Explorar inicialmente la base de datos de eventos sísmicos.
Identificar valores faltantes y características generales de los datos.
Realizar procesos de limpieza y transformación.
Normalizar coordenadas geográficas.
Clasificar los eventos según profundidad y magnitud.
Establecer un nivel de amenaza sísmica a partir del departamento asociado al evento.
Calcular una variable de intensidad máxima estimada.
Realizar consultas analíticas utilizando DuckDB.
Analizar la distribución de los eventos sísmicos mediante pruebas estadísticas.
Aplicar pruebas paramétricas y no paramétricas para comparar grupos.
Visualizar la distribución de las variables mediante histogramas y boxplots.
Identificar las principales limitaciones del análisis.
🗂️ Datos

La base de datos utilizada contiene 646 registros y 25 variables inicialmente.

Entre las variables originales se encuentran:

time: fecha y hora del evento.
latitude: latitud.
longitude: longitud.
depth: profundidad del evento.
mag: magnitud.
magType: tipo de magnitud.
nst: número de estaciones sísmicas.
gap: área sin estaciones.
dmin: distancia al epicentro.
rms: error asociado al tiempo de arribo.
horizontalError: error horizontal.
depthError: error de profundidad.
magError: error de magnitud.
Details: descripción del evento.
City: ciudad o municipio.
state: departamento o región.
Place: país.

Durante la exploración inicial se identificaron valores faltantes principalmente en:

nst: 36,4 %
City: 0,3 %

Después del proceso de limpieza, los registros faltantes de City fueron completados utilizando información disponible en Details.

🛠️ Tecnologías utilizadas
Herramienta	Uso
🐍 Python	Lenguaje principal
🐼 Pandas	Manipulación y transformación de datos
🔢 NumPy	Operaciones numéricas y clasificación de variables
🦆 DuckDB	Consultas SQL sobre DataFrames
📊 Matplotlib	Visualización de datos
🎨 Seaborn	Gráficos estadísticos
📐 SciPy	Pruebas estadísticas
📈 Scikit-posthocs	Herramientas para análisis estadístico post-hoc
📦 Instalación de librerías

Las principales librerías utilizadas en el notebook son:

pip install pandas
pip install numpy
pip install duckdb
pip install seaborn
pip install matplotlib
pip install scipy
pip install scikit-posthocs

También pueden instalarse conjuntamente:

pip install pandas numpy duckdb seaborn matplotlib scipy scikit-posthocs
🔎 1. Exploración inicial

La primera etapa consiste en cargar la base de datos mediante Pandas y realizar una revisión general de su estructura.

Se utiliza:

df_sismos.info()

para identificar:

número de registros;
número de variables;
tipos de datos;
valores no nulos;
memoria utilizada.

También se calcula el porcentaje de valores faltantes para cada variable:

df_sismos.isna().mean() * 100

Esta exploración permitió identificar que la base contiene 646 eventos sísmicos y que la variable con mayor porcentaje de datos faltantes corresponde al número de estaciones sísmicas (nst).

🧹 2. Limpieza y transformación de datos

El proceso de limpieza contempla diferentes etapas.

📍 Normalización de coordenadas

Las coordenadas de latitud y longitud son transformadas numéricamente y posteriormente divididas por 10000 para obtener los valores utilizados en el análisis.

Coordenadas = ['latitude', 'longitude']

for col in Coordenadas:
    df_sismos[col] = (
        df_sismos[col]
        .astype(str)
        .str.replace(',', '', regex=False)
        .astype(float)
    )

df_sismos[Coordenadas] = df_sismos[Coordenadas] / 10000
🔢 Conversión de variables

Las variables de profundidad y magnitud son convertidas a formato numérico:

df_sismos['depth'] = pd.to_numeric(
    df_sismos['depth'],
    errors='coerce'
)

df_sismos['mag'] = pd.to_numeric(
    df_sismos['mag'],
    errors='coerce'
)

También se eliminan espacios innecesarios en la descripción del evento.

🏙️ Tratamiento de valores faltantes

Los registros sin información en City son completados utilizando la variable Details.

City_na = df_sismos['City'].isna()

df_sismos.loc[City_na, 'City'] = (
    df_sismos.loc[City_na, 'Details']
)
🌊 3. Clasificación de profundidad

Se crea una nueva variable denominada Nivel_profundidad.

Los eventos son clasificados en tres categorías:

Profundidad	Clasificación
≤ 30 km	Terremotos superficiales
> 30 km y ≤ 120 km	Terremotos intermedios
> 120 km	Terremotos profundos

La clasificación se implementa mediante NumPy.

📏 4. Clasificación de magnitud

Se crea la variable Clasificacion_Magnitud.

Las categorías utilizadas en el notebook son:

Magnitud	Clasificación
≤ 2,9	Micro
3,0 – 3,9	Menor
4,0 – 4,9	Ligero
5,0 – 5,9	Moderado
6,0 – 6,9	Fuerte
7,0 – 7,9	Muy fuerte
≥ 8,0	Grave
⚠️ 5. Clasificación del nivel de amenaza

Se crea una variable denominada Nivel_amenaza utilizando el departamento o región asociado a cada evento.

Los departamentos/regiones son agrupados en tres categorías:

Alto
Intermedio
Bajo

Esta clasificación se utiliza posteriormente como uno de los filtros principales de las consultas estadísticas.

📐 6. Cálculo de la intensidad máxima

El notebook calcula una variable denominada Intensidad_maxima utilizando una expresión empírica basada en los parámetros definidos en el código:

a = 2
b = 2
c = 0.7

df_sismos['Intensidad_maxima'] = (
    a * df_sismos['mag']
    - b * np.log10(df_sismos['depth'])
    - c
)

Esta variable se utiliza posteriormente para filtrar los eventos y realizar las comparaciones estadísticas entre departamentos.

🗃️ 7. Consultas SQL con DuckDB

Una de las principales herramientas utilizadas en el proyecto es DuckDB, que permite ejecutar consultas SQL directamente sobre el DataFrame.

Consulta 1 — Filtrado de eventos

Se seleccionan eventos que cumplen simultáneamente:

Magnitud entre 4,0 y 4,9.
Intensidad máxima entre 4 y 6.
Nivel de amenaza Intermedio o Alto.
SELECT *
FROM df_sismos
WHERE Magnitud BETWEEN 4.0 AND 4.9
AND Intensidad_maxima BETWEEN 4 AND 6
AND Nivel_amenaza IN ('Intermedio', 'Alto')
Consulta 2 — Conteo por departamento

Posteriormente se calcula el número de eventos por departamento:

SELECT Departamento,
       COUNT(Departamento) AS Conteo
FROM df_sismos
WHERE Magnitud BETWEEN 4.0 AND 4.9
AND Intensidad_maxima BETWEEN 4 AND 6
AND Nivel_amenaza IN ('Intermedio', 'Alto')
GROUP BY Departamento
ORDER BY Conteo DESC

Los cuatro departamentos con mayor cantidad de eventos dentro de estos criterios fueron:

Departamento	Eventos
Santander	54
Chocó	44
Antioquia	28
Valle del Cauca	15
Consulta 3 — Selección de departamentos

Finalmente, se seleccionan los departamentos que presentan más de 10 eventos:

SELECT Departamento,
       Conteo
FROM (
    SELECT Departamento,
           COUNT(Departamento) AS Conteo
    FROM df_sismos
    WHERE Magnitud BETWEEN 4.0 AND 4.9
    AND Intensidad_maxima BETWEEN 4 AND 6
    AND Nivel_amenaza IN ('Intermedio', 'Alto')
    GROUP BY Departamento
)
WHERE Conteo > 10
ORDER BY Conteo DESC

El resultado se utiliza como base para el análisis de hipótesis.

📊 8. Análisis estadístico
8.1. Prueba Chi-cuadrado

Se utiliza una prueba de Chi-cuadrado de bondad de ajuste para analizar si la frecuencia de eventos entre los cuatro departamentos seleccionados puede considerarse uniforme.

Hipótesis

H₀: Los eventos sísmicos se distribuyen de forma equitativa entre los departamentos.

H₁: Los eventos sísmicos no se distribuyen de forma equitativa entre los departamentos.

Los conteos observados fueron:

Santander       54
Chocó           44
Antioquia       28
Valle del Cauca 15

El resultado obtenido fue:

Chi² = 25,2695
p = 1,3561 × 10⁻⁵

Con un nivel de significancia de α = 0,05, el notebook rechaza la hipótesis nula de distribución uniforme.

📈 9. ANOVA de una vía

Posteriormente se utiliza un ANOVA de una vía para comparar la intensidad máxima entre los cuatro departamentos:

Santander
Chocó
Antioquia
Valle del Cauca

El resultado obtenido fue:

F = 28,2411
p = 2,7927 × 10⁻¹⁴

El análisis del notebook utiliza este resultado como evidencia de diferencias entre los grupos.

También se genera un boxplot para visualizar la distribución de la intensidad máxima por departamento.

⚖️ 10. Prueba de Levene

Antes de interpretar el ANOVA, se realiza una prueba de Levene para evaluar la homogeneidad de las varianzas.

Resultado:

Estadístico de Levene = 8,7736
p = 2,3106 × 10⁻⁵

El resultado obtenido indica varianzas heterogéneas entre los grupos bajo el criterio utilizado en el notebook.

Por esta razón, el análisis continúa mediante una prueba no paramétrica.

📊 11. Prueba de Kruskal-Wallis

Debido a la heterogeneidad de las varianzas, se aplica la prueba de Kruskal-Wallis.

Resultado:

H = 55,5466
p = 5,2492 × 10⁻¹²

El resultado permite identificar diferencias estadísticamente significativas entre las distribuciones de los grupos analizados.

🔬 12. Comparaciones post-hoc

Después de Kruskal-Wallis se realizan comparaciones por pares mediante la prueba de Mann-Whitney U, aplicando una corrección de Bonferroni por las seis comparaciones posibles.

Los resultados obtenidos fueron:

Comparación	p corregido	Resultado en el notebook
Santander vs Chocó	8,9082 × 10⁻¹⁰	Diferencia significativa
Santander vs Antioquia	1,1605 × 10⁻⁷	Diferencia significativa
Santander vs Valle del Cauca	1,0000	No significativa
Chocó vs Antioquia	4,3471 × 10⁻¹	No significativa
Chocó vs Valle del Cauca	7,4399 × 10⁻³	Diferencia significativa
Antioquia vs Valle del Cauca	2,2950 × 10⁻²	Diferencia significativa

El nivel de significancia utilizado es:

α = 0,05
📉 13. Visualizaciones

El análisis incorpora diferentes representaciones gráficas para facilitar la interpretación de los datos.

Histograma de magnitudes

Permite observar la distribución de los valores de magnitud presentes en la base.

Boxplot de magnitud

Permite visualizar la distribución de la magnitud de los eventos entre los diferentes departamentos.

Boxplot de intensidad máxima

Permite comparar visualmente la distribución de la variable Intensidad_maxima entre:

Santander
Chocó
Antioquia
Valle del Cauca
🧪 14. Flujo general del análisis

El proyecto sigue el siguiente flujo:

Carga de datos
      ↓
Exploración inicial
      ↓
Identificación de valores faltantes
      ↓
Limpieza y transformación
      ↓
Clasificación de profundidad
      ↓
Clasificación de magnitud
      ↓
Clasificación del nivel de amenaza
      ↓
Cálculo de intensidad máxima
      ↓
Consultas SQL con DuckDB
      ↓
Selección de departamentos
      ↓
Chi-cuadrado
      ↓
ANOVA
      ↓
Prueba de Levene
      ↓
Kruskal-Wallis
      ↓
Mann-Whitney + Bonferroni
      ↓
Visualización e interpretación
⚠️ 15. Limitaciones

El propio análisis identifica dos limitaciones principales:

Restricción de los valores analizados:
El análisis estadístico se concentra en determinados rangos de magnitud, intensidad máxima y nivel de amenaza. Esto puede limitar la generalización de los resultados.
Alcance temporal reducido:
La base utilizada comprende aproximadamente seis años, por lo que el periodo analizado es relativamente corto para estudiar la recurrencia de eventos sísmicos.

Estas limitaciones deben considerarse al interpretar los resultados estadísticos.

📁 Estructura sugerida del repositorio
Taller-5/
│
├── README.md
├── Taller_5.ipynb
│
└── data/
    ├── Sismos2.csv
    └── Sismos_limpio2.csv

Los archivos CSV no se incluyen directamente en el notebook como rutas relativas; actualmente el código utiliza rutas locales de Windows. Para facilitar la reproducción del proyecto en GitHub, se recomienda modificar estas rutas para utilizar una estructura relativa dentro del repositorio.

Por ejemplo:

df_sismos = pd.read_csv(
    "data/Sismos2.csv",
    sep=";",
    decimal=","
)
▶️ Cómo ejecutar el proyecto
1. Clonar el repositorio
git clone <URL_DEL_REPOSITORIO>
2. Acceder a la carpeta
cd Taller-5
3. Instalar las dependencias
pip install pandas numpy duckdb seaborn matplotlib scipy scikit-posthocs
4. Abrir el notebook

El análisis puede ejecutarse utilizando:

Jupyter Notebook
JupyterLab
Visual Studio Code
Google Colab, realizando previamente los ajustes necesarios para la ubicación de los archivos.
5. Ejecutar Taller_5.ipynb

Se recomienda ejecutar las celdas en orden para reproducir:

la carga de datos;
la limpieza;
las transformaciones;
las consultas SQL;
las pruebas estadísticas;
las visualizaciones.
📚 Aprendizajes

Este ejercicio permite integrar diferentes herramientas y conceptos de analítica de datos:

Manipulación de datos con Pandas.
Operaciones numéricas con NumPy.
Consultas SQL mediante DuckDB.
Exploración y visualización de datos.
Manejo de valores faltantes.
Transformación y clasificación de variables.
Pruebas de hipótesis.
Análisis paramétrico y no paramétrico.
Comparaciones múltiples mediante corrección de Bonferroni.
Interpretación estadística de resultados.

👨‍💻 Autores

Adriana Sofía Pinzón Burgos y 
Juan Sebastian Roncancio Baquero

Maestría en Analítica de Datos
Universidad Central — Colombia

📌 Nota

Este repositorio tiene fines académicos y presenta el desarrollo del Taller 5 de la asignatura Programación para Analítica de Datos.

El análisis y sus conclusiones corresponden específicamente a los datos, filtros, transformaciones y procedimientos estadísticos implementados en el notebook.
