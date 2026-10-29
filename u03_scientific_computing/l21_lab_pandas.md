---
title: "Lab: Pandas"
unit: "3"
lesson: 21
type: lab
tags: [python, pandas, dataframe, read_csv, url, indice, to_csv, read_json, read_html, datos-reales]
difficulty: introductory
duration: "60 mins"
---

# Lab: Pandas

**Meta:** cargar datos **reales** desde internet con Pandas y darles un **primer vistazo**. Empezamos con la misma `muestras.csv` de la L19 (la receta de la L19 en una línea) y pasamos a una tabla real de 344 pingüinos: leerla desde una **URL**, inspeccionarla (`.shape`, `.head`, `.tail`, `.dtypes`, `.describe()`), notar los datos que **faltan**, entender el **índice**, seleccionar columnas y filtrar filas, **guardar** una copia con `to_csv` y leer otros formatos (JSON, una tabla de Wikipedia, una consulta a PubChem). Cierras con datos químicos de vino tinto. Acompaña a la nota de concepto [Pandas](l21_concept_pandas.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`, empezando por las dos celdas de preparación. Este lab **necesita internet** (Colab ya lo tiene): varias celdas leen datos desde una URL. Donde diga **Predice**, piensa primero qué pasaría y después ejecuta la celda; la respuesta está en el bloque desplegable que le sigue (en el Paso 5 la línea daría un error a propósito, por eso está comentada con `#` -- déjala así). En **Tu turno**, el Ejercicio 1 trae un bloque comentado con partes marcadas con `____` que debes **completar** al quitar los `#`; el Ejercicio 2 trae solo la lista de tareas y lo escribes tú; el Ejercicio 3 (opcional) lo escribes desde cero. Compara con la respuesta esperada de cada bloque desplegable.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u03_scientific_computing/l21_lab_pandas.ipynb)

## Preparación

Pandas no es parte de Python estándar: hay que **instalarlo** (en Colab) e **importarlo**. Corre estas **dos** celdas primero. La primera instala Pandas y `lxml` (la herramienta que usa Pandas para leer páginas web en el Paso 7); en Colab ya están, así que no hace nada. La segunda importa Pandas con el alias `pd`.

```python
%pip install -q pandas lxml
```

```python
import pandas as pd

print("Pandas listo para trabajar")
```

~~~text
Pandas listo para trabajar
~~~

## Paso 1: calentamiento -- la misma muestras.csv de la L19

Primero escribimos `muestras.csv` (la misma de la L19). Donde antes recorrías el archivo a mano, `pd.read_csv` lo lee entero a un **DataFrame** en una línea:

```python
texto = """muestra,sitio,ph,temperatura
1,rio_Norte,7.2,18.0
2,rio_Sur,6.8,21.0
3,rio_Norte,7.4,17.0
4,rio_Sur,6.5,22.0
5,rio_Norte,7.0,19.0
6,rio_Sur,6.8,20.0
"""

with open('muestras.csv', 'w') as f:
    f.write(texto)

agua = pd.read_csv('muestras.csv')
print(agua)
print(round(agua['ph'].mean(), 2))
```

~~~text
   muestra      sitio   ph  temperatura
0        1  rio_Norte  7.2         18.0
1        2    rio_Sur  6.8         21.0
2        3  rio_Norte  7.4         17.0
3        4    rio_Sur  6.5         22.0
4        5  rio_Norte  7.0         19.0
5        6    rio_Sur  6.8         20.0
6.95
~~~

`agua` es la tabla completa, con los nombres de columna y un índice de filas (`0` a `5`). Y `agua['ph'].mean()` da `6.95`: el **mismo** promedio que en la L19 sacaste con un `for`. Una línea reemplazó todo el bucle. Ahora, datos de verdad.

## Paso 2: datos reales desde una URL

Los datos *Palmer Penguins* son mediciones de 344 pingüinos de tres especies (Adelie, Chinstrap y Gentoo) en tres islas de la Antártida: largo y alto del pico, largo de la aleta, masa corporal y sexo. Viven en el **mismo** repositorio de GitHub que el `iris.csv` que en la L19 leíste con `urllib.request.urlopen(url)`, `.decode()` y la receta línea por línea. Con Pandas, la URL va directo a `read_csv`:

```python
url = ('https://raw.githubusercontent.com/mwaskom/'
       'seaborn-data/master/penguins.csv')
df = pd.read_csv(url)
print(df.shape)
```

~~~text
(344, 7)
~~~

344 filas y 7 columnas, descargadas y convertidas en una sola línea. *(Fuente: Gorman, Williams y Fraser, 2014; paquete palmerpenguins de A. Horst et al., licencia CC0.)*

## Paso 3: primer vistazo (predice primero)

Con 344 filas ya no conviene imprimir la tabla entera. `.head()` muestra las primeras 5 filas:

```python
print(df.head())
```

~~~text
  species     island  bill_length_mm  bill_depth_mm  flipper_length_mm  \
0  Adelie  Torgersen            39.1           18.7              181.0
1  Adelie  Torgersen            39.5           17.4              186.0
2  Adelie  Torgersen            40.3           18.0              195.0
3  Adelie  Torgersen             NaN            NaN                NaN
4  Adelie  Torgersen            36.7           19.3              193.0

   body_mass_g     sex
0       3750.0    MALE
1       3800.0  FEMALE
2       3250.0  FEMALE
3          NaN     NaN
4       3450.0  FEMALE
~~~

La tabla no cabe a lo ancho, así que Pandas la parte en **dos bloques**: la `\` al final de una línea indica que la tabla **sigue** abajo con las columnas restantes. Fíjate en la fila `3`: dice `NaN` en casi todo. `.dtypes` dice de qué **tipo** es cada columna (también es una forma rápida de ver sus nombres):

```python
print(df.dtypes)
```

~~~text
species               object
island                object
bill_length_mm       float64
bill_depth_mm        float64
flipper_length_mm    float64
body_mass_g          float64
sex                   object
dtype: object
~~~

`float64` son números con decimales; `object` es **texto** (`species`, `island`, `sex`). `.describe()` resume las columnas **numéricas**:

```python
print(df.describe())
```

~~~text
       bill_length_mm  bill_depth_mm  flipper_length_mm  body_mass_g
count      342.000000     342.000000         342.000000   342.000000
mean        43.921930      17.151170         200.915205  4201.754386
std          5.459584       1.974793          14.061714   801.954536
min         32.100000      13.100000         172.000000  2700.000000
25%         39.225000      15.600000         190.000000  3550.000000
50%         44.450000      17.300000         197.000000  4050.000000
75%         48.500000      18.700000         213.000000  4750.000000
max         59.600000      21.500000         231.000000  6300.000000
~~~

**Predice:** la tabla tiene 344 pingüinos, pero la fila `count` dice `342`. ¿Por qué? Piensa en la fila `3` de `.head()` y luego ejecuta la celda:

```python
print(df.isna().sum())
```

<details><summary>Respuesta esperada</summary>

~~~text
species               0
island                0
bill_length_mm        2
bill_depth_mm         2
flipper_length_mm     2
body_mass_g           2
sex                  11
dtype: int64
~~~

`NaN` (*Not a Number*) es como Pandas marca un **dato que falta**: una celda vacía en el CSV. `df.isna()` marca cada celda con `True` si falta y `False` si no; `.sum()` cuenta los `True` de cada columna (cada `True` vale 1). A 2 pingüinos les faltan todas las mediciones y a 11 les falta el `sex`. Por eso `describe` cuenta `342` en las columnas numéricas: **ignora** los `NaN` (y también los ignora al calcular la media). Los datos reales casi siempre tienen huecos; hoy solo los **observamos**.
</details>

## Paso 4: el índice

Los números de la izquierda (`0`, `1`, `2`, ...) son el **índice**: una **etiqueta** para cada fila. `.tail(3)` muestra las últimas 3 filas:

```python
print(df.tail(3))
```

~~~text
    species  island  bill_length_mm  bill_depth_mm  flipper_length_mm  \
341  Gentoo  Biscoe            50.4           15.7              222.0
342  Gentoo  Biscoe            45.2           14.8              212.0
343  Gentoo  Biscoe            49.9           16.1              213.0

     body_mass_g     sex
341       5750.0    MALE
342       5200.0  FEMALE
343       5400.0    MALE
~~~

Las etiquetas van de `0` a `343` (344 filas). Ojo: el índice es una **etiqueta** pegada a cada fila, no su posición en la tabla. Mientras la tabla está completa coinciden; al filtrar (Paso 5) verás que dejan de coincidir.

## Paso 5: seleccionar columnas y filtrar filas (predice primero)

Una columna se toma con su nombre; el resultado es una **Series** (una columna con su índice). Sus valores se guardan en un arreglo de NumPy, así que tiene los agregados de la L20:

```python
print(df['body_mass_g'].head(3))
print(round(df['body_mass_g'].mean(), 2))
print(df['body_mass_g'].min(), df['body_mass_g'].max())
```

~~~text
0    3750.0
1    3800.0
2    3250.0
Name: body_mass_g, dtype: float64
4201.75
2700.0 6300.0
~~~

La masa promedio es `4201.75` gramos (de los 342 pingüinos con dato): el mismo número que `describe` mostró en la fila `mean` (`4201.754386`), ahora redondeado. El más liviano pesa 2700 g y el más pesado 6300 g, las filas `min` y `max` de `describe`. Para **varias** columnas se pasa una **lista** de nombres (corchetes dobles), y el resultado es un DataFrame:

```python
print(df[['species', 'body_mass_g']].head(3))
```

~~~text
  species  body_mass_g
0  Adelie       3750.0
1  Adelie       3800.0
2  Adelie       3250.0
~~~

Para quedarte con las filas que cumplen una condición se usa la **máscara booleana** de la L20. ¿Qué pingüinos pesan más de 6000 g?

```python
pesados = df[df['body_mass_g'] > 6000]
print(pesados.shape)
print(pesados[['species', 'island', 'body_mass_g']])
```

~~~text
(2, 7)
    species  island  body_mass_g
237  Gentoo  Biscoe       6300.0
253  Gentoo  Biscoe       6050.0
~~~

Solo 2, y los dos son Gentoo de la isla Biscoe. Mira el **índice**: `237` y `253`. Las filas **conservan su etiqueta** de la tabla completa, aunque ahora son la primera y la segunda. Por eso el índice es una etiqueta y no una posición. (En la L22 verás cómo renumerarlo.)

**Predice:** ¿qué pasaría si pides una columna con el nombre mal escrito? La línea está comentada a propósito: piénsalo y luego mira la respuesta.

```python
# Predice: pedir una columna que no existe (mayuscula)
# print(df['Species'])

# Pista: el operador in (L13) pregunta si un nombre esta
print('Species' in df.columns)
print('species' in df.columns)
```

~~~text
False
True
~~~

<details><summary>Respuesta esperada</summary>

Si quitaras los `#`, pedir `df['Species']` (con mayúscula) daría un error, porque la columna se llama `species`:

~~~text
KeyError: 'Species'
~~~

Los nombres de columna **distinguen mayúsculas de minúsculas**. Pandas no encuentra una columna llamada `Species` y lanza un `KeyError`. (La dejamos comentada porque un error vivo detendría el resto del notebook.) En el Paso 7 verás una versión de estos mismos datos donde la columna **sí** se llama `Species`.
</details>

## Paso 6: guardar una copia con to_csv (predice primero)

Los datos vienen de internet; si mañana la URL cambia o no hay conexión, conviene tener una **copia local**. `df.to_csv('archivo.csv')` escribe el DataFrame a un CSV.

**Predice:** guardamos `df` y lo volvemos a leer. ¿La copia tendrá la misma forma `(344, 7)`?

```python
df.to_csv('pinguinos.csv')
copia = pd.read_csv('pinguinos.csv')
print(copia.shape)
print(copia.columns.tolist()[:3])
```

<details><summary>Respuesta esperada</summary>

~~~text
(344, 8)
['Unnamed: 0', 'species', 'island']
~~~

¡Una columna de más! Por defecto, `to_csv` guarda **también el índice** como una primera columna sin nombre. Al leer el archivo, Pandas no sabe que era el índice: lo trata como una columna de datos más y la llama `Unnamed: 0` ("sin nombre"). La solución es pedirle que **no** guarde el índice, con `index=False`.
</details>

```python
df.to_csv('pinguinos.csv', index=False)
copia = pd.read_csv('pinguinos.csv')
print(copia.shape)
print(copia.columns.tolist()[:3])
```

~~~text
(344, 7)
['species', 'island', 'bill_length_mm']
~~~

Ahora la copia es idéntica a la original: `(344, 7)`, sin `Unnamed: 0`. Regla práctica: guarda con `index=False` salvo que el índice tenga información propia.

## Paso 7: otros formatos

`read_csv` es solo uno de los lectores de Pandas. Todos devuelven un DataFrame, así que todo lo anterior sirve igual.

**JSON** (un formato de texto muy usado por los servicios web): los **mismos** 344 pingüinos, publicados por otro proyecto (*vega-datasets*) con otros nombres de columna:

```python
url_json = ('https://cdn.jsdelivr.net/npm/'
            'vega-datasets@2.8.0/data/penguins.json')
pj = pd.read_json(url_json)
print(pj.shape)
for nombre in pj.columns:
    print(nombre)
```

~~~text
(344, 7)
Species
Island
Beak Length (mm)
Beak Depth (mm)
Flipper Length (mm)
Body Mass (g)
Sex
~~~

Los mismos datos, `(344, 7)`, pero aquí la columna se llama `Species` (con mayúscula): `pj['Species']` funciona y `df['Species']` no. Antes de seleccionar, revisa siempre cómo se llaman las columnas **de esa tabla**. (Casi los mismos: en esta versión, a un pingüino el sexo que falta está escrito como un punto, `'.'`, en vez de quedar vacío. Pandas no reconoce `'.'` como dato que falta, así que `pj.isna().sum()` cuenta 10 en `Sex`, no 11. Así de sucios pueden ser los datos reales.) *(Fuente: vega-datasets 2.8.0, con los mismos datos Palmer Penguins, CC0.)*

**HTML** (una tabla de una página web): `pd.read_html` busca **todas** las tablas de una página. Leemos el anexo de Wikipedia con los 78 municipios de Puerto Rico (la URL apunta a una versión fija de la página, para que los resultados no cambien si alguien la edita):

```python
url_wiki = ('https://es.wikipedia.org/w/index.php'
            '?title=Anexo:Municipios_de_Puerto_Rico'
            '&oldid=174878338')
navegador = {'User-Agent': 'Mozilla/5.0'}
tablas = pd.read_html(url_wiki, storage_options=navegador)
print(len(tablas))
m = tablas[1]
print(m.shape)
for nombre in m.columns:
    print(nombre)
```

~~~text
3
(78, 7)
Municipio
Localización
Bandera
Escudo
Código FIPS[2]
Población (2020)[3]
Superficie (km²)[4]
~~~

Tres detalles de los datos reales de la web:

- **El `User-Agent`.** Wikipedia rechaza los programas que no se identifican (da un error `HTTP Error 403: Forbidden`). `storage_options=navegador` envía un `User-Agent`, el nombre con que un navegador se presenta.
- **Una lista de tablas.** `read_html` devuelve una **lista** con todas las tablas de la página (aquí 3); la de los municipios es la de la posición `1`.
- **Nombres sucios.** Los nombres traen las notas al pie de Wikipedia (`[2]`, `[3]`, `[4]`) y acentos, y las columnas de imágenes (mapa, bandera, escudo) llegan vacías (`NaN`). Lo práctico es **reemplazar** los nombres por unos cortos asignando una lista a `m.columns`, en el mismo orden:

```python
m.columns = ['municipio', 'mapa', 'bandera', 'escudo',
             'fips', 'poblacion', 'superficie']
print(m[['municipio', 'poblacion', 'superficie']].head(3))
grandes = m[m['poblacion'] > 100000]
print(grandes[['municipio', 'poblacion']])
```

~~~text
   municipio  poblacion  superficie
0   Adjuntas      18020      172.73
1     Aguada      38136       79.90
2  Aguadilla      55101       94.61
   municipio  poblacion
10   Bayamón     185187
12    Caguas     127244
15  Carolina     154815
57     Ponce     137491
64  San Juan     342259
~~~

Con nombres limpios, el filtro es el de siempre: los municipios con más de 100,000 habitantes en el censo 2020. *(Fuente: Wikipedia en español, licencia CC BY-SA 4.0.)*

**Bonus -- una URL que es una consulta (PubChem).** Una URL no tiene que ser un archivo guardado: puede ser una **pregunta** a una base de datos. PubChem (la base de datos química pública de los NIH) arma un CSV en el momento con las propiedades que le pidas. Aquí pedimos la cafeína, la aspirina, la glucosa y el etanol por su número de compuesto (CID):

```python
url_pc = ('https://pubchem.ncbi.nlm.nih.gov/rest/pug/'
          'compound/cid/2519,2244,5793,702/property/'
          'Title,MolecularFormula,MolecularWeight,XLogP/CSV')
compuestos = pd.read_csv(url_pc)
print(compuestos)
```

~~~text
    CID      Title MolecularFormula  MolecularWeight  XLogP
0  2519   Caffeine        C8H10N4O2           194.19   -0.1
1  2244    Aspirin           C9H8O4           180.16    1.2
2  5793  D-Glucose          C6H12O6           180.16   -2.6
3   702    Ethanol            C2H6O            46.07   -0.1
~~~

La URL dice **qué** compuestos (`cid/2519,2244,...`), **qué** propiedades (`Title,MolecularFormula,...`) y en **qué formato** (`CSV`). `MolecularWeight` es la masa molar en g/mol y `XLogP` estima si la molécula prefiere el agua (negativo) o la grasa (positivo). Cambia los números de la URL y tendrás otros compuestos. *(Fuente: PubChem, NCBI/NIH, datos de dominio público.)*

## Tu turno (programa completo)

Un estudio de vinos tintos portugueses midió la química de 1599 vinos -- acidez, azúcar, cloruros, sulfitos, densidad, **pH**, alcohol -- junto con una nota de **calidad** de 0 a 10 dada por catadores. *(Fuente: Cortez et al., 2009, UCI Machine Learning Repository, licencia CC BY 4.0.)*

**Predice:** este archivo **no** separa sus campos con comas sino con **punto y coma** (`;`). ¿Qué forma tendrá si lo lees con `read_csv` sin decirle nada?

```python
url_vino = ('https://archive.ics.uci.edu/ml/'
            'machine-learning-databases/wine-quality/'
            'winequality-red.csv')
malo = pd.read_csv(url_vino)
print(malo.shape)
```

<details><summary>Respuesta esperada</summary>

~~~text
(1599, 1)
~~~

¡Una sola columna! `read_csv` busca comas por defecto; como no encuentra ninguna, deja cada línea entera en **una** columna, cuyo nombre es la línea de encabezado completa (`fixed acidity;"volatile acidity";...`). La solución es decirle el separador con `sep=';'`. Es el error más común al leer un CSV real: si ves una sola columna, revisa el separador.
</details>

### Ejercicio 1: cargar y dar un primer vistazo

Lee el vino con el separador correcto y dale un primer vistazo. Completa las dos partes marcadas con `____`: el separador y el nombre de la columna de acidez fija (`fixed acidity`, con un **espacio** en medio: va entre comillas igual que cualquier otro nombre).

```python
# TODO: quita los # y completa las partes que faltan (____).
# v = pd.read_csv(url_vino, sep=____)
# print(v.shape)
# print(v.dtypes)
# print(v.isna().sum().sum())
# print(round(v['pH'].mean(), 2), v['pH'].min(), v['pH'].max())
# print(round(v[____].mean(), 2))
```

<details><summary>Respuesta esperada</summary>

~~~text
(1599, 12)
fixed acidity           float64
volatile acidity        float64
citric acid             float64
residual sugar          float64
chlorides               float64
free sulfur dioxide     float64
total sulfur dioxide    float64
density                 float64
pH                      float64
sulphates               float64
alcohol                 float64
quality                   int64
dtype: object
0
3.31 2.74 4.01
8.32
~~~

Las partes que faltan son `sep=';'` y `v['fixed acidity']`. La tabla tiene 1599 vinos y 12 columnas, **todas numéricas** (11 `float64` y la `quality`, entera, `int64`). `v.isna().sum()` cuenta los faltantes de cada columna y el segundo `.sum()` los suma todos: `0`. A diferencia de los pingüinos, esta tabla no tiene huecos. El pH promedio es `3.31` (entre `2.74` y `4.01`): el vino es **ácido**, muy lejos del `6.95` del agua de río del Paso 1. La acidez fija promedio es `8.32` g/L (de ácido tartárico).
</details>

### Ejercicio 2: los mejores vinos

Escribe tú el programa, en la celda vacía de abajo (usa la `v` del Ejercicio 1):

1. Filtra los vinos con `quality` **mayor o igual que 8** (`>=`) y guárdalos en `mejores`. ¿Cuántos son? (usa `.shape`).
2. Muestra solo sus columnas `alcohol`, `pH` y `quality`. Mira el índice: ¿por qué no va de `0` a `17`?
3. Calcula el alcohol promedio de `mejores` y el de **todos** los vinos (con `round` a 2 decimales). ¿Cuál es mayor?
4. Guarda `mejores` en `vinos_top.csv` **sin** el índice, léelo de nuevo y comprueba con `.shape` que no apareció un `Unnamed: 0`.

```python
# Ejercicio 2: escribe tu programa aqui

```

<details><summary>Respuesta esperada</summary>

Una solución:

~~~text
mejores = v[v['quality'] >= 8]
print(mejores.shape)
print(mejores[['alcohol', 'pH', 'quality']])
print(round(mejores['alcohol'].mean(), 2))
print(round(v['alcohol'].mean(), 2))
mejores.to_csv('vinos_top.csv', index=False)
print(pd.read_csv('vinos_top.csv').shape)
~~~

~~~text
(18, 12)
      alcohol    pH  quality
267      12.8  3.35        8
278      12.6  3.23        8
390      12.9  3.56        8
440       9.8  2.88        8
455      13.4  3.22        8
481      11.7  3.15        8
495      11.0  3.15        8
498      11.0  3.15        8
588      14.0  3.72        8
828      12.7  3.46        8
1061     12.5  3.21        8
1090     11.8  2.98        8
1120     13.1  3.20        8
1202     11.7  3.35        8
1269     14.0  3.50        8
1403     10.0  3.23        8
1449     11.3  3.23        8
1549     11.4  3.24        8
12.09
10.42
(18, 12)
~~~

Solo 18 de 1599 vinos tienen nota 8 o más (ninguno llega a 9). El índice salta (`267`, `278`, `390`, ...) porque las filas **conservan su etiqueta** de la tabla completa (Paso 5). Los mejores vinos tienen en promedio `12.09` % de alcohol, contra `10.42` % de todos: en estos datos, los vinos mejor calificados tienden a tener más alcohol. La copia leída de nuevo tiene `(18, 12)`, las mismas 12 columnas, sin `Unnamed: 0`, gracias a `index=False`.
</details>

### Ejercicio 3 (opcional): desafío -- los vinos más ácidos

Sin plantilla: escribe tú el programa completo. Un pH **más bajo** significa una solución **más ácida**. Filtra los vinos con `pH` **menor que 3.0**: ¿cuántos son? ¿Cuál es su acidez fija promedio (`fixed acidity`, con `round` a 2 decimales), comparada con la de todos los vinos?

```python
# Ejercicio 3 (opcional): escribe tu programa aqui

```

<details><summary>Respuesta esperada</summary>

Una solución:

~~~text
acidos = v[v['pH'] < 3.0]
print(acidos.shape)
print(round(acidos['fixed acidity'].mean(), 2))
print(round(v['fixed acidity'].mean(), 2))
~~~

~~~text
(29, 12)
11.29
8.32
~~~

Hay 29 vinos con pH menor que 3.0, y su acidez fija promedio es `11.29` g/L, contra `8.32` g/L de todos. Tiene sentido químico: más ácido disuelto, más iones de hidrógeno, **menor** pH.
</details>

## Resumen

- Se **importa** con `import pandas as pd`; la estructura central es el **DataFrame** (una tabla con columnas con nombre y un índice de filas).
- `pd.read_csv(...)` lee un CSV en **una línea**, desde un archivo **o una URL**; si el separador no es la coma, se indica con `sep=';'`.
- Primer vistazo: `df.shape`, `df.columns`, `df.head()` / `df.tail()`, `df.dtypes` (números o texto) y `df.describe()`.
- Los datos que faltan aparecen como `NaN`; `df.isna().sum()` los cuenta por columna, y `describe` y `mean` los ignoran.
- El **índice** es una **etiqueta** de fila, no una posición: al filtrar, las filas conservan su etiqueta.
- Una columna es `df['col']` (una **Series**); varias, `df[['a', 'b']]`; filtrar filas, `df[df['col'] > valor]` (la máscara booleana de la L20).
- `df.to_csv('archivo.csv', index=False)` guarda una copia sin la columna extra `Unnamed: 0`.
- Otros lectores: `pd.read_json(...)` y `pd.read_html(...)` (una **lista** de tablas); una URL también puede ser una **consulta** (PubChem).
- **Sigue en la L22** con más Pandas: contar valores, columnas calculadas y agrupar por categoría.
