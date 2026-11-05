---
title: "Lab: Visualización científica"
unit: "3"
lesson: 23
type: lab
tags: [python, matplotlib, pandas, seaborn, visualizacion, graficas, barras, pastel, histograma, caja, dispersion, pairplot, heatmap, radar, subplots, savefig, simpson, penguins]
difficulty: introductory
duration: "90 mins"
---

# Lab: Visualización científica

**Meta:** darle ojos a la tabla de pingüinos de la L21 y la L22. Vas a construir una gráfica para cada tipo de pregunta: **relación** entre dos mediciones, **comparar** categorías, **composición** de un todo, **distribución** de una medición y una **tabla como color**. Por el camino aprenderás a rotular (título, ejes con unidades, leyenda), a elegir colores, a girar las etiquetas de los ejes, a cambiar el tamaño de una figura, a poner dos gráficas lado a lado y a **guardar** una figura en un archivo. Acompaña a la nota de concepto [Visualización científica](l23_concept_visualization.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`, empezando por las dos celdas de preparación. Cada gráfica aparece debajo de su celda. Cada paso empieza desde la tabla `df`, así que puedes saltar un paso sin romper los siguientes. Donde diga **Tu turno**, completa el código y compara con la respuesta del bloque desplegable.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u03_scientific_computing/l23_lab_visualization.ipynb)

## Preparación

Usaremos tres librerías de gráficas que trabajan juntas: **matplotlib** (la base; su módulo `pyplot` se importa como `plt`), **Pandas** (sus tablas traen el método `.plot`, que dibuja con matplotlib) y **seaborn** (se importa como `sns`; hace en una llamada gráficas que en matplotlib tomarían varias líneas). Corre estas **dos** celdas primero. La primera instala las librerías (en Colab; si ya están, no hace nada). La segunda las importa y lee la tabla de pingüinos desde su URL (**necesita internet**; Colab ya lo tiene).

```python
%pip install -q matplotlib pandas seaborn
```

```python
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import seaborn as sns

url = ('https://raw.githubusercontent.com/mwaskom/'
       'seaborn-data/master/penguins.csv')
df = pd.read_csv(url)
print(df.shape)
```

~~~text
(344, 7)
~~~

Es la misma tabla de la L21 y la L22: 344 pingüinos, 7 columnas (`species`, `island`, cuatro mediciones en mm y g, y `sex`).

**Tres formas de graficar.** En este lab las verás mezcladas, cada una donde es más corta:

| Forma | Ejemplo | Úsala cuando |
|---|---|---|
| Pandas | `serie.plot.bar()` | ya tienes la Series o la tabla (un `groupby`, un `value_counts`, un `crosstab`) |
| pyplot | `plt.plot(x, y)`, `plt.title(...)` | quieres dibujar `x` contra `y` directamente, o ajustar detalles: título, ejes, tamaño, guardar |
| seaborn | `sns.boxplot(data=df, x=..., y=..., hue=...)` | quieres colorear o separar por grupo (`hue=`) en una sola llamada |

Las tres dibujan sobre matplotlib, así que `plt.title`, `plt.xlabel` y `plt.show()` funcionan con todas.

## Paso 1: relación entre dos mediciones (línea o marcadores)

¿Los pingüinos con la aleta más larga pesan más? Son **dos columnas numéricas**: ponemos una en cada eje. La función básica de pyplot es `plt.plot(x, y)`:

```python
plt.plot(df['flipper_length_mm'], df['body_mass_g'])
plt.show()
```

*(Al ejecutar: una **maraña** de líneas. `plt.plot` une cada punto con el de la **fila siguiente** de la tabla, y las filas no están ordenadas por largo de aleta, así que las líneas saltan de un lado a otro. Además no tiene título ni rótulos: no sabemos qué es cada eje.)*

Una **línea** solo tiene sentido cuando el eje x tiene un **orden** (el tiempo, por ejemplo) y la tabla está en ese orden. Aquí no lo hay: queremos ver **los puntos sueltos**. Basta un tercer argumento, `'o'` (solo marcadores, sin línea). Y rotulamos con pyplot, siempre con **unidades**:

```python
plt.plot(df['flipper_length_mm'], df['body_mass_g'], 'o')
plt.title('Masa vs largo de la aleta')
plt.xlabel('Largo de la aleta (mm)')
plt.ylabel('Masa corporal (g)')
plt.show()
```

*(Al ejecutar: una nube de puntos que sube de izquierda a derecha -- a más aleta, más masa --, con título y ejes rotulados.)*

**¿Y con Pandas?** En Pandas, `.plot()` **sin tipo** también dibuja una **línea**. En una tabla, `df.plot(x='flipper_length_mm', y='body_mass_g')` elige qué columna va en cada eje y dibuja esta misma maraña; con `marker='o'` agrega además un marcador en cada punto (línea y marcadores). Úsala cuando el eje x tiene un orden, por ejemplo el tiempo.

## Paso 2: comparar categorías (barras)

Una **barra** compara un valor entre categorías. Es la forma natural de graficar un resumen de la L22. Primero, la masa promedio por especie, en barras **verticales**. `rot=0` deja los nombres horizontales y `color=` elige el color:

```python
masa = df.groupby('species')['body_mass_g'].mean().round(1)
print(masa)
masa.plot.bar(title='Masa promedio por especie',
              xlabel='Especie', ylabel='Masa promedio (g)',
              rot=0, color='seagreen')
plt.show()
```

~~~text
species
Adelie       3700.7
Chinstrap    3733.1
Gentoo       5076.0
Name: body_mass_g, dtype: float64
~~~

*(Al ejecutar: tres barras verdes; la de Gentoo es claramente la más alta, y Adelie y Chinstrap casi iguales.)*

Las barras **horizontales** (`.plot.barh()`) leen mejor cuando las categorías son muchas o tienen nombres largos. Ordenamos antes con `sort_values()`: `barh` dibuja la primera fila abajo, así que la isla con más pingüinos queda arriba. Aquí rotulamos con pyplot, que siempre sabe cuál es el eje x y cuál el y:

```python
islas = df['island'].value_counts().sort_values()
print(islas)
islas.plot.barh(title='Pinguinos por isla')
plt.xlabel('Numero de pinguinos')
plt.ylabel('Isla')
plt.show()
```

~~~text
island
Torgersen     52
Dream        124
Biscoe       168
Name: count, dtype: int64
~~~

*(Al ejecutar: tres barras horizontales; Biscoe, la más larga, arriba.)*

Una barra **apilada** muestra dos categorías a la vez. Partimos del `crosstab` de la L22 (especie por isla): cada especie es una barra y cada isla un trozo de color. `colormap=` elige la paleta de colores y `plt.legend(title=...)` le pone título a la leyenda:

```python
tabla = pd.crosstab(df['species'], df['island'])
print(tabla)
tabla.plot.bar(stacked=True, rot=0, colormap='Set2',
               title='Pinguinos por especie e isla',
               xlabel='Especie', ylabel='Numero de pinguinos')
plt.legend(title='Isla')
plt.show()
```

~~~text
island     Biscoe  Dream  Torgersen
species                            
Adelie         44     56         52
Chinstrap       0     68          0
Gentoo        124      0          0
~~~

*(Al ejecutar: la barra de Adelie tiene tres colores -- vive en las tres islas --; la de Chinstrap es de un solo color (solo Dream) y la de Gentoo también (solo Biscoe). Los ceros de la tabla se ven como colores que faltan.)*

## Paso 3: composición de un todo (pastel)

Un **pastel** muestra qué parte del total es cada categoría. `autopct='%1.0f%%'` escribe el porcentaje en cada trozo (sin decimales) y `ylabel=''` quita el rótulo automático del lado:

```python
conteo = df['species'].value_counts()
print(conteo)
conteo.plot.pie(autopct='%1.0f%%', ylabel='',
                title='Proporcion de cada especie')
plt.show()
```

~~~text
species
Adelie       152
Gentoo       124
Chinstrap     68
Name: count, dtype: int64
~~~

*(Al ejecutar: un círculo en tres trozos con 44%, 36% y 20%.)*

El pastel funciona con **pocas** partes de un todo. Para **comparar** valores, la barra es mejor: el ojo compara largos mucho mejor que ángulos. ¿Podrías decir, solo por el pastel, cuánto más grande es el trozo de Adelie que el de Gentoo?

## Paso 4: distribución de una medición (histograma y caja)

Un **histograma** reparte los valores de una columna en intervalos (`bins`) y cuenta cuántos caen en cada uno: muestra la **forma** de los datos. `edgecolor='black'` dibuja el borde de cada barra:

```python
df['body_mass_g'].plot.hist(bins=20, edgecolor='black',
                            title='Distribucion de la masa',
                            xlabel='Masa corporal (g)',
                            ylabel='Frecuencia')
plt.show()
```

*(Al ejecutar: la mayoría de los pingüinos pesa entre 3000 y 4500 g, pero hay una "cola" larga hacia los 6000 g, con varias jorobas.)*

¿Por qué esa forma rara? Porque mezcla tres especies. Con el `groupby` de la L22 dibujamos un histograma **por especie** en la misma figura. `bins=range(2700, 6400, 150)` (el `range` de la L11) da a las tres especies los **mismos** intervalos, de 150 g cada uno, para que sus barras se puedan comparar; `alpha=0.5` hace las barras semitransparentes para que se vean las que quedan detrás, y `legend=True` dice cuál es cuál:

```python
df.groupby('species')['body_mass_g'].plot.hist(
    bins=range(2700, 6400, 150), alpha=0.5, legend=True)
plt.title('Distribucion de la masa por especie')
plt.xlabel('Masa corporal (g)')
plt.ylabel('Frecuencia')
plt.show()
```

*(Al ejecutar: Adelie y Chinstrap se montan una sobre otra, sobre todo entre 3000 y 4500 g; Gentoo forma su propia montaña a la derecha, de unos 4000 a 6300 g. La "cola" del histograma anterior eran los Gentoo.)*

Un **diagrama de caja** (box plot) resume una distribución en cinco números: la línea del medio es la **mediana**, la caja va del primer al tercer **cuartil** (la mitad central de los datos), los bigotes llegan hasta los valores más lejanos que no son raros, y los puntos sueltos son **valores atípicos**. Pandas dibuja una caja por cada columna numérica. Como los nombres de las columnas son largos, los giramos con `plt.xticks(rotation=45)`:

```python
df.plot.box(title='Todas las mediciones juntas')
plt.xticks(rotation=45)
plt.show()
```

*(Al ejecutar: la caja de `body_mass_g` (miles de gramos) ocupa toda la figura y las de las medidas en milímetros quedan **aplastadas** cerca del cero: no se ve nada de ellas. Las columnas tienen escalas muy distintas.)*

La solución: una caja por panel, cada una con **su propia escala**, con `subplots=True`. Para que quepan cuatro paneles agrandamos la figura con `figsize=(ancho, alto)`, en pulgadas:

```python
df.plot.box(subplots=True, figsize=(14, 4),
            title='Cada medicion con su propia escala')
plt.show()
```

*(Al ejecutar: cuatro cajas lado a lado, cada una con su eje; ahora sí se ve la forma de cada medición. El nombre de cada columna lleva su unidad: `_mm` o `_g`.)*

Para comparar **una** medición **entre grupos**, seaborn lo hace en una llamada: `x=` es el grupo, `y=` la medición y `hue=` colorea por grupo con la paleta `palette=`. `legend=False` quita la leyenda, que aquí repetiría los nombres del eje:

```python
sns.boxplot(data=df, x='species', y='body_mass_g',
            hue='species', palette='Set2', legend=False)
plt.title('Masa por especie')
plt.xlabel('Especie')
plt.ylabel('Masa corporal (g)')
plt.show()
```

*(Al ejecutar: tres cajas; la de Gentoo está mucho más arriba. Adelie y Chinstrap tienen medianas casi iguales, y Chinstrap tiene un par de valores atípicos.)*

## Paso 5: relación entre dos mediciones, por grupo (dispersión, Simpson y pairplot)

Volvamos a la nube del Paso 1, ahora con seaborn. `hue='species'` colorea cada punto según su especie y agrega la leyenda sola:

```python
sns.scatterplot(data=df, x='flipper_length_mm', y='body_mass_g',
                hue='species')
plt.title('Masa vs largo de la aleta, por especie')
plt.xlabel('Largo de la aleta (mm)')
plt.ylabel('Masa corporal (g)')
plt.show()
```

*(Al ejecutar: la nube del Paso 1, en tres colores. Los Gentoo forman su propio grupo arriba a la derecha; Adelie y Chinstrap se mezclan abajo a la izquierda.)*

**Una advertencia: la paradoja de Simpson.** Ahora el pico: ¿los pingüinos con el pico más **largo** lo tienen también más **profundo** (más alto)? `sns.regplot` dibuja los puntos y una **recta de tendencia** (la recta que mejor sigue a la nube; `ci=None` quita la banda sombreada de incertidumbre):

```python
sns.regplot(data=df, x='bill_length_mm', y='bill_depth_mm',
            ci=None)
plt.title('Todos los pinguinos juntos')
plt.xlabel('Largo del pico (mm)')
plt.ylabel('Profundidad del pico (mm)')
plt.show()
```

*(Al ejecutar: la recta **baja**. Parece que un pico más largo es menos profundo.)*

Pero esa nube mezcla tres especies. `sns.lmplot` hace lo mismo **por grupo**, con `hue=`: una nube y una recta por especie:

```python
sns.lmplot(data=df, x='bill_length_mm', y='bill_depth_mm',
           hue='species', ci=None)
plt.title('Cada especie por separado')
plt.xlabel('Largo del pico (mm)')
plt.ylabel('Profundidad del pico (mm)')
plt.show()
```

*(Al ejecutar: las **tres** rectas **suben**. Dentro de cada especie, un pico más largo es también más profundo: lo contrario de la gráfica anterior.)*

Eso es la **paradoja de Simpson**: una tendencia de todos los datos juntos puede **invertirse** dentro de cada grupo. Aquí la recta de todos baja solo porque los Gentoo tienen picos largos pero poco profundos y los Adelie picos cortos pero profundos. La lección para un científico: antes de concluir, **separa o colorea por grupo**.

¿Y si quieres ver **todas** las parejas de mediciones a la vez? `sns.pairplot` dibuja una cuadrícula: cada panel es la dispersión de dos columnas, y en la diagonal va la distribución de cada columna por especie:

```python
sns.pairplot(df, hue='species')
plt.show()
```

*(Al ejecutar: una cuadrícula de 4 x 4 paneles. En la diagonal, la distribución aparece como una **curva suavizada** (una curva de densidad, como un histograma sin escalones). Es una herramienta para **explorar** -- buscar qué parejas tienen una relación o separan bien a las especies --, no una figura para un informe: no lleva título y es demasiado grande para leer de un vistazo.)*

## Paso 6: una tabla como color (heatmap)

Un **heatmap** (mapa de calor) pinta cada celda de una tabla con un color según su valor. Volvamos al `crosstab` de especie por isla. `annot=True` escribe el número en cada celda, `fmt='d'` lo escribe como entero, y `cmap='Blues'` es un mapa de color (**colormap**) **secuencial**: de blanco (poco) a azul oscuro (mucho):

```python
tabla = pd.crosstab(df['species'], df['island'])
sns.heatmap(tabla, annot=True, fmt='d', cmap='Blues')
plt.title('Pinguinos por especie e isla')
plt.xlabel('Isla')
plt.ylabel('Especie')
plt.show()
```

*(Al ejecutar: la tabla del Paso 2 como cuadrícula de colores; la celda Gentoo-Biscoe (124) es la más oscura y los ceros son blancos.)*

El uso más común del heatmap en ciencia es la **matriz de correlación**. La **correlación** `r` es un número entre -1 y 1 que mide qué tan bien dos columnas numéricas siguen una línea recta: cerca de 1, suben juntas; cerca de -1, una sube cuando la otra baja; cerca de 0, no hay relación lineal. `df.corr(numeric_only=True)` calcula `r` para todas las parejas de columnas numéricas. Como `r` va de -1 a 1 con el 0 en el medio, usamos un colormap **divergente** (`coolwarm`: azul negativo, blanco cero, rojo positivo), y `vmin=-1, vmax=1` fija la escala completa. Esta vez el tamaño se fija con `plt.figure(figsize=...)` **antes** de dibujar (`sns.heatmap` no recibe `figsize=`), y `fmt='.2f'` escribe dos decimales:

```python
corr = df.corr(numeric_only=True)
plt.figure(figsize=(7, 5))
sns.heatmap(corr, annot=True, fmt='.2f', cmap='coolwarm',
            vmin=-1, vmax=1)
plt.title('Correlacion entre las mediciones')
plt.show()
```

*(Al ejecutar: la diagonal vale 1.00 (cada columna consigo misma). La correlación más fuerte es aleta-masa, `0.87` (la nube del Paso 1). La de largo del pico con profundidad del pico es `-0.24`: la recta que bajaba en el Paso 5.)*

Pero el Paso 5 nos advirtió: ese `-0.24` mezcla tres especies. Calculemos `r` **dentro** de cada especie:

```python
for especie in ['Adelie', 'Chinstrap', 'Gentoo']:
    grupo = df[df['species'] == especie]
    r = grupo['bill_length_mm'].corr(grupo['bill_depth_mm'])
    print(especie, round(r, 2))
```

~~~text
Adelie 0.39
Chinstrap 0.65
Gentoo 0.64
~~~

Las tres son **positivas**: Simpson en números. Un heatmap de correlación de todos los datos es un buen primer vistazo, no una conclusión.

## Paso 7: el perfil de cada especie (radar)

Un **radar** (gráfica polar) dibuja varias mediciones en ejes que salen del centro, como los rayos de una rueda; cada especie es un polígono. Primero, el promedio de cada medición por especie:

```python
cols = ['bill_length_mm', 'bill_depth_mm',
        'flipper_length_mm', 'body_mass_g']
prom = df.groupby('species')[cols].mean()
print(prom.round(1))
```

~~~text
           bill_length_mm  bill_depth_mm  flipper_length_mm  body_mass_g
species                                                                 
Adelie               38.8           18.3              190.0       3700.7
Chinstrap            48.8           18.4              195.8       3733.1
Gentoo               47.5           15.0              217.2       5076.0
~~~

El código del radar es más largo que los otros, así que lo guardamos en una **función** (L15) para usarlo dos veces. Úsala tal cual; lo que importa es la idea. `np.linspace(0, 2 * np.pi, 4, endpoint=False)` (con 4 columnas) reparte **4 ángulos iguales** en la vuelta completa (uno por medición), `plt.subplot(polar=True)` crea los ejes circulares, se repite el primer valor al final para **cerrar** cada polígono, y `plt.xticks(posiciones, nombres)` pone el nombre de cada medición en su rayo:

```python
def radar(tabla, titulo):
    angulos = list(np.linspace(0, 2 * np.pi, len(tabla.columns),
                               endpoint=False))
    angulos = angulos + angulos[:1]
    plt.figure(figsize=(6, 6))
    plt.subplot(polar=True)
    for especie in tabla.index:
        valores = list(tabla.loc[especie])
        valores = valores + valores[:1]
        plt.plot(angulos, valores, label=especie)
        plt.fill(angulos, valores, alpha=0.1)
    plt.xticks(angulos[:-1], tabla.columns)
    plt.title(titulo)
    plt.legend(loc='lower left')
    plt.show()

radar(prom, 'Promedios sin normalizar')
```

*(Al ejecutar: un radar inútil. `body_mass_g` (unos 4000) es tan grande que los tres polígonos se aplastan en una aguja hacia ese rayo; las medidas en mm (15 a 220) quedan pegadas al centro.)*

Es el mismo problema de **escala** de las cajas del Paso 4. La solución: **normalizar** cada columna a una escala común de 0 a 1, restando el mínimo y dividiendo entre el rango (máximo menos mínimo) de **todos** los pingüinos. Es una operación vectorizada (L20) sobre la tabla entera:

```python
minimo = df[cols].min()
maximo = df[cols].max()
norm = (prom - minimo) / (maximo - minimo)
print(norm.round(2))
radar(norm, 'Promedios normalizados (0 a 1)')
```

~~~text
           bill_length_mm  bill_depth_mm  flipper_length_mm  body_mass_g
species                                                                 
Adelie               0.24           0.62               0.30         0.28
Chinstrap            0.61           0.63               0.40         0.29
Gentoo               0.56           0.22               0.77         0.66
~~~

*(Al ejecutar: ahora cada especie tiene su **perfil**. Gentoo: aleta larga y mucha masa, pero pico poco profundo. Adelie: pico corto. Chinstrap: pico largo y profundo. Un 0 es el pingüino más pequeño en esa medición y un 1 el más grande.)*

## Paso 8: dos gráficas lado a lado, un zoom y guardar la figura

Para terminar, una figura con **dos paneles**. `plt.subplots(1, 2, figsize=(12, 5))` crea una figura con 1 fila y 2 columnas de ejes y devuelve dos cosas: la figura (`fig`) y el par de paneles `(izq, der)`, que se desempaquetan como en la L19; cada función de seaborn recibe `ax=` para saber en qué panel dibujar. En los paneles, los rótulos se ponen con `set_title`, `set_xlabel` y `set_ylabel`. A la derecha hacemos **zoom**: `set_xlim` y `set_ylim` fijan los **límites** de los ejes a una región, como una lupa. Y `fig.savefig(...)` guarda la figura en un archivo PNG; `dpi=150` es la resolución (puntos por pulgada). Guarda **antes** de `plt.show()`: `show` muestra la figura y luego la borra, así que guardar después daría una imagen en blanco.

```python
fig, (izq, der) = plt.subplots(1, 2, figsize=(12, 5))
sns.scatterplot(data=df, x='flipper_length_mm', y='body_mass_g',
                hue='species', ax=izq)
sns.scatterplot(data=df, x='flipper_length_mm', y='body_mass_g',
                hue='species', ax=der, legend=False)
der.set_xlim(200, 215)
der.set_ylim(3500, 5000)
izq.set_title('Todos los pinguinos')
der.set_title('Zoom: donde se juntan las especies')
for panel in [izq, der]:
    panel.set_xlabel('Largo de la aleta (mm)')
    panel.set_ylabel('Masa corporal (g)')
fig.savefig('pinguinos_zoom.png', dpi=150)
plt.show()
```

*(Al ejecutar: a la izquierda, la nube completa; a la derecha, la región de 200 a 215 mm y 3500 a 5000 g, donde las tres especies se tocan. En el zoom los puntos forman **columnas** verticales: la aleta se midió en milímetros enteros. En Colab, el archivo `pinguinos_zoom.png` aparece en el panel de archivos, a la izquierda.)*

El zoom revela algo que la vista completa esconde: los datos tienen la **resolución** del instrumento. Comprobémoslo: ¿cuántas masas son múltiplos de 25 g, y cuántas masas hay?

```python
print((df['body_mass_g'] % 25 == 0).sum(), df['body_mass_g'].count())
```

~~~text
342 342
~~~

Las 342 masas medidas son múltiplos de 25 g: las masas se registraron de 25 en 25 gramos (la resolución de la medición en el campo).

## Tu turno

### Ejercicio 1 (guiado): sexo por isla

¿Cuántos pingüinos de cada sexo hay en cada isla? Haz una barra **apilada** de `pd.crosstab(df['island'], df['sex'])`. Completa las dos partes marcadas con `____` al quitar los `#`:

```python
# TODO: quita los # y completa las dos partes con ____.
# sexo = pd.crosstab(df['island'], df['sex'])
# print(sexo)
# sexo.plot.bar(stacked=____, rot=0, title='Sexo por isla',
#               xlabel='Isla', ylabel=____)
# plt.legend(title='Sexo')
# plt.show()
```

<details><summary>Respuesta esperada</summary>

Las partes que faltan son `stacked=True` y un rótulo con lo que mide el eje y, por ejemplo `ylabel='Numero de pinguinos'`:

~~~python
sexo = pd.crosstab(df['island'], df['sex'])
print(sexo)
sexo.plot.bar(stacked=True, rot=0, title='Sexo por isla',
              xlabel='Isla', ylabel='Numero de pinguinos')
plt.legend(title='Sexo')
plt.show()
~~~

~~~text
sex        FEMALE  MALE
island                 
Biscoe         80    83
Dream          61    62
Torgersen      24    23
~~~

En cada isla hay casi tantas hembras como machos: cada barra está partida casi por la mitad. Fíjate en que la tabla suma 333 pingüinos, no 344: `crosstab` deja fuera las 11 filas donde falta el sexo (los `NaN` de la L21).
</details>

### Ejercicio 2 (elige la gráfica): la profundidad del pico

¿Qué especie tiene el pico más **profundo**, y cuánto **varía** la profundidad dentro de cada especie? Elige la gráfica que mejor responde (piensa en el Paso 4), dibújala con título y ejes rotulados con unidades, y escribe en una línea por qué elegiste esa.

```python
# Tu codigo aqui
```

<details><summary>Respuesta esperada</summary>

Es una medición comparada **entre grupos** (y se pregunta cuánto varía), así que la mejor opción es un **diagrama de caja por especie** (un histograma por especie también sirve):

~~~python
sns.boxplot(data=df, x='species', y='bill_depth_mm',
            hue='species', palette='Set2', legend=False)
plt.title('Profundidad del pico por especie')
plt.xlabel('Especie')
plt.ylabel('Profundidad del pico (mm)')
plt.show()
~~~

Las cajas responden las dos preguntas a la vez. **Cuál es más profundo:** Adelie y Chinstrap están prácticamente empatados (medianas cerca de 18.4 mm), y Gentoo tiene el pico claramente menos profundo (cerca de 15 mm). **Cuánto varía:** la altura de cada caja (la mitad central de los datos) es parecida en las tres especies, de 1.5 a 2 mm. Un promedio solo no habría mostrado ni el empate ni la variación.
</details>

### Ejercicio 3 (arregla esta figura): la masa por isla

Un compañero quiere comparar la masa promedio de los pingüinos de cada isla y escribió esto. Córrelo y mira qué falla:

```python
df.groupby('island')['body_mass_g'].mean().plot(figsize=(3, 2))
plt.show()
```

Escribe una versión que (1) use el **tipo** de gráfica correcto, (2) tenga título y ejes rotulados con unidades, (3) tenga un tamaño legible con `figsize` y (4) se guarde en el archivo `mi_figura.png`.

```python
# Tu codigo aqui
```

<details><summary>Respuesta esperada</summary>

Fallaba en tres cosas: `.plot()` sin tipo dibuja una **línea**, y una línea une categorías que no tienen orden (Biscoe, Dream, Torgersen), no tiene rótulos y es diminuta. Las islas son categorías, así que va una **barra**:

~~~python
masa_isla = df.groupby('island')['body_mass_g'].mean().round(1)
print(masa_isla)
masa_isla.plot.bar(figsize=(7, 4), rot=0,
                   title='Masa promedio por isla',
                   xlabel='Isla', ylabel='Masa promedio (g)')
plt.savefig('mi_figura.png', dpi=150)
plt.show()
~~~

~~~text
island
Biscoe       4716.0
Dream        3712.9
Torgersen    3706.4
Name: body_mass_g, dtype: float64
~~~

Los pingüinos de Biscoe pesan unos 1000 g más. Pero, ¿es por la isla? Recuerda la barra apilada del Paso 2: **los Gentoo, la especie más pesada, viven solo en Biscoe**. La diferencia viene de la especie, no de la isla: otra vez, separa por grupo antes de concluir.
</details>

## Resumen

- Tres formas de graficar sobre matplotlib: **Pandas** (`serie.plot.<tipo>()`, cuando ya tienes la tabla o el resumen), **pyplot** (`plt.plot`, `plt.title`, `plt.xlabel`, `plt.figure`, `plt.savefig`, `plt.show`) y **seaborn** (`sns.<grafica>(data=df, x=, y=, hue=)`, para colorear o separar por grupo).
- **Relación** entre dos mediciones: `plt.plot(x, y, 'o')` o `sns.scatterplot(..., hue=)`; una línea (`plt.plot(x, y)` o, en Pandas, `df.plot(x=, y=)`) solo si el eje x tiene orden. `sns.pairplot` para explorar todas las parejas.
- **Comparar** categorías: barras verticales (`.plot.bar()`), horizontales (`.plot.barh()`) y apiladas (`stacked=True`); un radar de promedios **normalizados**.
- **Composición**: pastel (`.plot.pie(autopct=...)`), solo con pocas partes de un todo.
- **Distribución**: histograma (`.plot.hist(bins=)`, también por grupo) y caja (`df.plot.box()`, `sns.boxplot`); cuidado con las escalas distintas (`subplots=True`).
- **Tabla como color**: `sns.heatmap(tabla, annot=True, cmap=...)`; colormap secuencial para conteos, divergente (`vmin=-1, vmax=1`) para correlaciones.
- **Detalles**: título, ejes con **unidades**, leyenda, `color=` / `colormap=` / `palette=`, `rot=` / `plt.xticks(rotation=)`, `figsize=`, paneles con `plt.subplots` y límites con `set_xlim` / `set_ylim`, y `savefig` **antes** de `show`.
- **Honestidad**: una tendencia de todos los datos puede invertirse dentro de cada grupo (Simpson). Separa por grupo antes de concluir.
- **Sigue en la L24:** las imágenes como arreglos de NumPy, que también se muestran con matplotlib.
