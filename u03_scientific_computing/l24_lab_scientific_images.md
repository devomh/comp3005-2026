---
title: "Lab: Análisis de imágenes científicas"
unit: "3"
lesson: 24
type: lab
tags: [python, imagenes, numpy, pillow, histograma, umbralizacion, color, filtros, microscopia, geles, ia]
difficulty: introductory
duration: "75 mins"
---

# Lab: Análisis de imágenes científicas

**Meta:** analizar **imágenes científicas reales** como arreglos de NumPy. Empiezas con una imagen sintética (un calentamiento rápido) y luego trabajas tres casos reales: una **micrografía** de núcleos celulares (en grises), una **hoja** con clorosis (a color) y un **gel** de ADN. Usarás el **histograma** para elegir un umbral y lo calcularás de forma **automática**, medirás **área**, combinarás **canales de color**, limpiarás una máscara con un **filtro**, sacarás un **perfil** robusto para contar bandas y, al final, usarás **Gemini** en Colab para ir más allá y **verificarás** su respuesta con tus propias medidas. En la L20 ya armaste una imagen a color como un arreglo 3D; hoy una imagen en grises es el caso sencillo: un solo canal. Acompaña a la nota de concepto [Análisis de imágenes científicas](l24_concept_scientific_images.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`, empezando por las dos celdas de preparación (la segunda descarga las tres imágenes reales). Las celdas de gráfica no imprimen texto: abren una figura. Donde diga **Predice**, piensa primero qué pasaría y después ejecuta la celda; la respuesta está en el bloque desplegable que le sigue. El Paso 6 usa **Gemini**, el asistente de IA de Colab: es práctica **no calificada**, y lo importante es **verificar** lo que responde. En **Tu turno**, los Ejercicios 1 y 2 traen un bloque comentado con una parte marcada con `____` que debes **completar** al quitar los `#`; los Ejercicios 3 y 4 (opcionales) los escribes tú. Compara con la respuesta esperada de cada bloque desplegable. **Tiempo sugerido (75 min):** Preparación y Paso 1, 12 min; Pasos 2 y 3, 15 min; Paso 4, 15 min; Paso 5, 10 min; Paso 6, 13 min; Ejercicios 1 y 2, 10 min. Lo marcado **(opcional)** puede quedar para casa.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u03_scientific_computing/l24_lab_scientific_images.ipynb)

## Preparación

Usaremos **Pillow** (la librería de imágenes, se importa como `PIL`), **NumPy** y **matplotlib**. Corre estas **dos** celdas primero. La primera instala las librerías (en Colab; si ya están, no hace nada). La segunda las importa y descarga las tres imágenes reales del repositorio del curso a una carpeta `imagenes/` (solo si todavía no están).

```python
%pip install -q pillow numpy matplotlib
```

```python
from pathlib import Path
from urllib.request import urlretrieve
from PIL import Image, ImageFilter
import numpy as np
import matplotlib.pyplot as plt

# Las tres imagenes reales viven en el repositorio del curso.
IMAGENES = {
    "l24_nucleos.tif": "https://raw.githubusercontent.com/devomh/comp3005-2026/main/u03_scientific_computing/imagenes/l24_nucleos.tif",
    "l24_hoja.jpg": "https://raw.githubusercontent.com/devomh/comp3005-2026/main/u03_scientific_computing/imagenes/l24_hoja.jpg",
    "l24_gel.png": "https://raw.githubusercontent.com/devomh/comp3005-2026/main/u03_scientific_computing/imagenes/l24_gel.png",
}
Path("imagenes").mkdir(exist_ok=True)
for nombre, url in IMAGENES.items():
    if not (Path("imagenes") / nombre).exists():   # en Colab: descargar
        urlretrieve(url, Path("imagenes") / nombre)

print("Pillow, NumPy y matplotlib listos;", len(IMAGENES), "imagenes en la carpeta imagenes/")
```

~~~text
Pillow, NumPy y matplotlib listos; 3 imagenes en la carpeta imagenes/
~~~

## Paso 1: calentamiento -- una imagen sintética

Antes de las imágenes reales, una **fabricada** con NumPy: un campo oscuro (intensidad 40) con siete "células" cuadradas y brillantes (180 a 220). Es tan limpia que todo sale obvio, y por eso sirve para ver las ideas. Esta celda solo **fabrica** el archivo; en la práctica la imagen vendría de un microscopio.

```python
# Esta celda FABRICA una imagen de ejemplo.
campo = np.full((80, 100), 40, dtype=np.uint8)   # fondo oscuro: 80 filas x 100 columnas
celulas = [(8, 12, 200), (10, 60, 220), (38, 28, 180), (52, 74, 210),
           (60, 18, 200), (28, 84, 190), (66, 48, 210)]
for (fila, col, brillo) in celulas:
    campo[fila:fila+8, col:col+8] = brillo       # cada celula = un cuadro brillante 8x8
Image.fromarray(campo).save('microscopia.png')

img = Image.open('microscopia.png')
arr = np.array(img)
print("formato:", img.format, "| modo:", img.mode, "| tamano:", img.size)
print("shape (alto, ancho):", arr.shape, "| dtype:", arr.dtype)
print("minimo:", arr.min(), "| maximo:", arr.max(), "| promedio:", round(float(arr.mean()), 2))
```

~~~text
formato: PNG | modo: L | tamano: (100, 80)
shape (alto, ancho): (80, 100) | dtype: uint8
minimo: 40 | maximo: 220 | promedio: 49.04
~~~

El modo `L` significa **un solo canal** (escala de grises): por eso el arreglo es 2D, `(80, 100)`, y no 3D como la bandera de la L20. Ojo con el orden: Pillow da el tamaño como `(ancho, alto)` = `(100, 80)`, pero NumPy da el `shape` como `(filas, columnas)` = `(80, 100)`. Para **verlo**, `plt.imshow` con el mapa de grises; `vmin=0, vmax=255` fija la escala completa (0 negro, 255 blanco), así el gris de cada píxel corresponde a su número real y no se "estira" el contraste:

```python
plt.imshow(arr, cmap='gray', vmin=0, vmax=255)
plt.title('Campo sintetico')
plt.colorbar(label='Intensidad')
plt.show()
```

*(Al ejecutar se abre la figura: el campo oscuro con siete cuadros brillantes y una barra que traduce número a gris.)*

Ahora la **umbralización** (la máscara booleana de la L20, en 2D) y el **área**, y de paso los **formatos**: guardamos el mismo campo como PNG y como JPEG y medimos sobre cada uno.

```python
mascara = arr > 120
print("pixeles de senal:", mascara.sum(), "| area:", round(float(mascara.mean()) * 100, 2), "%")

Image.fromarray(campo).save('campo.png')
Image.fromarray(campo).save('campo.jpg', quality=85)
png = np.array(Image.open('campo.png'))
jpg = np.array(Image.open('campo.jpg'))
print("PNG identico?:", np.array_equal(campo, png), "| JPEG identico?:", np.array_equal(campo, jpg))
print("diferencia maxima JPEG:", np.abs(campo.astype(int) - jpg.astype(int)).max())
print("area en el JPEG:", round(float((jpg > 120).mean()) * 100, 2), "%")
print("promedio PNG:", round(float(png.mean()), 2), "| promedio JPEG:", round(float(jpg.mean()), 2))
```

~~~text
pixeles de senal: 448 | area: 5.6 %
PNG identico?: True | JPEG identico?: False
diferencia maxima JPEG: 15
area en el JPEG: 5.6 %
promedio PNG: 49.04 | promedio JPEG: 49.01
~~~

El umbral 120 cae entre el fondo (40) y las células (180-220): 448 píxeles, 5.6 % del campo. El PNG es **sin pérdida** (idéntico); el JPEG es **con pérdida** (algunos píxeles cambian hasta en 15). Fíjate en *cuándo importa*: el **área** del JPEG es la misma (los cambios son pequeños frente a la distancia entre fondo y célula), pero las **intensidades** cambian (el promedio baja de 49.04 a 49.01). Para medir intensidades, PNG o TIFF; una foto JPEG puede servir para medir área.

En esta imagen todo fue fácil porque solo hay dos valores. Las imágenes reales no son así.

## Paso 2: Caso A -- núcleos reales y el histograma

La imagen `l24_nucleos.tif` es una **micrografía de fluorescencia**: núcleos de células humanas teñidos con un colorante de ADN (cada mancha clara es un núcleo). Viene de la colección de ejemplos de scikit-image (David Root, licencia CC0). Es un **TIFF**, el formato sin pérdida típico de los microscopios.

```python
nuc_img = Image.open('imagenes/l24_nucleos.tif')
nuc = np.array(nuc_img)
print("formato:", nuc_img.format, "| modo:", nuc_img.mode, "| shape:", nuc.shape, "| dtype:", nuc.dtype)
print("minimo:", nuc.min(), "| maximo:", nuc.max(), "| promedio:", round(float(nuc.mean()), 2))
print("pixeles con intensidad <= 20:", round(float((nuc <= 20).mean()) * 100, 1), "%")

plt.imshow(nuc, cmap='gray', vmin=0, vmax=255)
plt.title('Nucleos (fluorescencia)')
plt.show()
```

~~~text
formato: TIFF | modo: L | shape: (512, 512) | dtype: uint8
minimo: 7 | maximo: 255 | promedio: 17.29
pixeles con intensidad <= 20: 85.1 %
~~~

*(Al ejecutar se abre la figura: un campo casi negro con cientos de núcleos grises; los más claros son células en división.)*

El promedio es apenas 17.29 y el 85.1 % de los píxeles vale 20 o menos: casi todo es **fondo**. ¿Dónde poner el umbral? Lo dice el **histograma** de los píxeles. `nuc.ravel()` aplana el arreglo 2D en una sola fila de números, y `plt.hist` cuenta cuántos píxeles hay de cada intensidad. Como el fondo domina, usamos `log=True` (eje vertical en escala logarítmica) para ver también el grupo pequeño de los núcleos:

```python
plt.hist(nuc.ravel(), bins=256, range=(0, 256), log=True)
plt.title('Histograma de los nucleos')
plt.xlabel('Intensidad')
plt.ylabel('Numero de pixeles (escala log)')
plt.show()
```

*(Al ejecutar se abre la figura: un pico enorme cerca de 10 -- el fondo -- y una segunda joroba alrededor de 50-70 -- los núcleos. Entre ambos hay un **valle** cerca de 38.)*

**Predice:** en el calentamiento el umbral 120 funcionó perfecto. ¿Qué porcentaje de área daría 120 en esta imagen real? ¿Y un umbral muy bajo, como 10?

```python
for u in [10, 20, 40, 80, 120]:
    print("umbral", u, "-> area", round(float((nuc > u).mean()) * 100, 2), "%")
```

~~~text
umbral 10 -> area 45.98 %
umbral 20 -> area 14.94 %
umbral 40 -> area 11.01 %
umbral 80 -> area 1.74 %
umbral 120 -> area 0.37 %
~~~

<details><summary>Respuesta</summary>

Con 120 solo pasan los núcleos más brillantes: 0.37 % del área, casi nada. Con 10 el umbral cae **dentro** del pico del fondo y se cuela fondo como si fuera señal: 45.98 %. El resultado depende muchísimo del umbral, y el umbral "obvio" de una imagen sintética no sirve en una real. El histograma dice dónde ponerlo: en el **valle** entre los dos grupos.
</details>

Podemos encontrar ese valle **automáticamente** con una idea sencilla (el método *isodata*): empieza con el promedio como umbral; separa los píxeles en dos grupos (fondo y señal); pon el nuevo umbral en el punto medio entre el promedio de cada grupo; y **repite mientras** el umbral siga cambiando. Es un `while` (L08) con dos máscaras (L20) dentro de una función (L15):

```python
def umbral_automatico(valores):
    t = float(valores.mean())          # empezar en el promedio
    anterior = -1.0
    while abs(t - anterior) > 0.5:     # abs = valor absoluto: repetir mientras siga cambiando
        anterior = t
        fondo = valores[valores <= t].mean()
        senal = valores[valores > t].mean()
        t = (fondo + senal) / 2        # punto medio entre los dos grupos
    return t

umbral_nuc = int(umbral_automatico(nuc))
mascara_nuc = nuc > umbral_nuc
print("umbral automatico:", umbral_nuc)
print("area cubierta por nucleos:", round(float(mascara_nuc.mean()) * 100, 2), "%")
```

~~~text
umbral automatico: 38
area cubierta por nucleos: 11.36 %
~~~

El algoritmo cae en 38, justo en el valle del histograma: los núcleos cubren el **11.36 %** del campo. Veamos la máscara:

```python
plt.imshow(mascara_nuc, cmap='gray')
plt.title('Mascara: nucleos (blanco) vs fondo (negro)')
plt.show()
```

*(Al ejecutar se abre la figura: cientos de núcleos blancos sobre negro, separados limpiamente del fondo.)*

## Paso 3: realzar y recortar

La micrografía se ve muy oscura. Para **ver** mejor los núcleos tenues podemos **multiplicar** o **sumar** a las intensidades. Como en la L20, primero `astype(int)` (para que 255 + algo no dé la vuelta) y luego `np.clip` para quedarnos en 0-255:

```python
brillante = np.clip(nuc.astype(int) * 4, 0, 255).astype(np.uint8)   # contraste x4
claro = np.clip(nuc.astype(int) + 60, 0, 255).astype(np.uint8)      # +60 a todo

print("promedio original:", round(float(nuc.mean()), 2), "| x4:", round(float(brillante.mean()), 2))
print("misma mascara con x4 y umbral 4*38?:", np.array_equal(brillante > 4 * umbral_nuc, mascara_nuc))
print("misma mascara con +60 y umbral 38+60?:", np.array_equal(claro > umbral_nuc + 60, mascara_nuc))

plt.imshow(brillante, cmap='gray', vmin=0, vmax=255)
plt.title('Nucleos con contraste x4')
plt.show()
```

~~~text
promedio original: 17.29 | x4: 65.45
misma mascara con x4 y umbral 4*38?: True
misma mascara con +60 y umbral 38+60?: True
~~~

*(Al ejecutar se abre la figura: los mismos núcleos, ahora claramente visibles; los más brillantes quedan blancos.)*

Realzar cambia lo que **ves**, no lo que **mides**: si el umbral se ajusta igual (por 4, o más 60), la máscara es exactamente la misma. Otra forma de quitar lo que no interesa es **recortar** con un rebanado. Acerquémonos a una zona densa del campo:

```python
zona = brillante[320:420, 180:280]     # filas 320-419, columnas 180-279
plt.imshow(zona, cmap='gray', vmin=0, vmax=255)
plt.title('Zona densa (acercamiento)')
plt.show()
```

*(Al ejecutar se abre la figura: un acercamiento donde varios núcleos se **tocan** entre sí. Medir su área es fácil; contarlos uno por uno no, porque dos núcleos pegados forman una sola mancha. Volveremos a esto en el Paso 6.)*

## Paso 4: Caso B -- una hoja con clorosis (color)

La imagen `l24_hoja.jpg` es una hoja de liquidámbar con **clorosis intervenal**: las venas siguen verdes, pero el tejido entre ellas se pone amarillo (la planta no produce suficiente clorofila, a menudo por falta de hierro). Es una foto **JPEG** sobre grava (Jim Conrad, dominio público). Es a **color**: un arreglo 3D, como en la L20.

```python
hoja_img = Image.open('imagenes/l24_hoja.jpg')
foto = np.array(hoja_img)
print("formato:", hoja_img.format, "| modo:", hoja_img.mode, "| shape:", foto.shape)

R = foto[:, :, 0].astype(int)    # canal rojo
G = foto[:, :, 1].astype(int)    # canal verde
B = foto[:, :, 2].astype(int)    # canal azul
print("promedio R:", round(float(R.mean()), 1), "| G:", round(float(G.mean()), 1), "| B:", round(float(B.mean()), 1))

plt.imshow(foto)
plt.title('Hoja con clorosis')
plt.show()
```

~~~text
formato: JPEG | modo: RGB | shape: (603, 600, 3)
promedio R: 140.4 | G: 139.3 | B: 93.7
~~~

*(Al ejecutar se abre la figura: la hoja verde-amarilla en forma de estrella sobre grava gris y café.)*

¿Cómo separar la hoja de la grava? Una idea: la hoja (verde **o** amarilla) tiene **mucho más verde que azul**, y la grava gris tiene verde y azul parecidos. Así que la resta `G - B` es grande en la hoja y pequeña en el fondo.

**Predice:** ¿por qué convertimos los canales con `astype(int)` antes de restar? Mira un píxel de grava donde el azul es mayor que el verde (`.tolist()` convierte el píxel en una lista para imprimirlo limpio).

```python
resta_uint8 = foto[:, :, 1] - foto[:, :, 2]    # restar los canales SIN convertir
print("pixel [R, G, B]:", foto[0, 266].tolist())
print("G - B con uint8:", resta_uint8[0, 266])
print("G - B con int:  ", G[0, 266] - B[0, 266])
print("area de 'hoja' sin convertir:", round(float((resta_uint8 > 71).mean()) * 100, 2), "%")
```

~~~text
pixel [R, G, B]: [152, 139, 149]
G - B con uint8: 246
G - B con int:   -10
area de 'hoja' sin convertir: 25.7 %
~~~

<details><summary>Respuesta</summary>

En `uint8` no existen los negativos: la resta "da la vuelta" desde 0 hasta 255 (el mismo desbordamiento de la L20, ahora hacia abajo), y un píxel de grava con más azul que verde parece tener un `G - B` enorme (246), como si fuera hoja. Con `int` el resultado es el correcto, -10. Con el umbral que usaremos enseguida (71), esos píxeles falsos inflan el área de la hoja de 24.3 % a 25.7 %. Por eso `R`, `G` y `B` se convirtieron a `int` al sacarlos.
</details>

Ahora el histograma de `G - B`, y el mismo `umbral_automatico` del Paso 2 aplicado a esa resta:

```python
dif = G - B
plt.hist(dif.ravel(), bins=100)
plt.title('Histograma de G - B')
plt.xlabel('G - B')
plt.ylabel('Numero de pixeles')
plt.show()

umbral_hoja = int(umbral_automatico(dif))
mascara_hoja = dif > umbral_hoja
print("umbral automatico para G - B:", umbral_hoja)
print("area de la hoja:", round(float(mascara_hoja.mean()) * 100, 2), "% de la foto")
```

~~~text
umbral automatico para G - B: 71
area de la hoja: 24.3 % de la foto
~~~

*(Al ejecutar se abre la figura: un pico alto cerca de 25 -- la grava -- y otro más bajo cerca de 115 -- la hoja --, con un valle entre ambos.)*

El histograma vuelve a tener dos grupos, y el umbral automático (71) cae en su valle: la hoja ocupa el 24.3 % de la foto (el pecíolo, más pálido, queda fuera). La máscara todavía tiene motas sueltas de grava verdosa y huequitos. Un **filtro de mediana** los limpia: cambia cada píxel por el valor "del medio" de sus vecinos (aquí una ventana de 7 x 7), así que una mota aislada desaparece y un hueco pequeño se rellena. Pillow trabaja con imágenes, así que convertimos la máscara (`True`/`False`) a una imagen 0/255, filtramos y volvemos a booleanos. Después, `FIND_EDGES` encuentra el **borde** de la hoja:

```python
img_mascara = Image.fromarray(mascara_hoja.astype(np.uint8) * 255)
limpia = np.array(img_mascara.filter(ImageFilter.MedianFilter(7))) > 127
borde = np.array(Image.fromarray(limpia.astype(np.uint8) * 255).filter(ImageFilter.FIND_EDGES)) > 0

print("area antes del filtro:", round(float(mascara_hoja.mean()) * 100, 2), "%")
print("area despues del filtro:", round(float(limpia.mean()) * 100, 2), "%")
print("pixeles de borde:", borde.sum())

contorno = foto.copy()
contorno[borde] = [255, 0, 0]          # pintar el borde de rojo
plt.imshow(contorno)
plt.title('Borde de la hoja')
plt.show()
```

~~~text
area antes del filtro: 24.3 %
area despues del filtro: 24.28 %
pixeles de borde: 3448
~~~

*(Al ejecutar se abre la figura: la foto original con el contorno de la hoja trazado en rojo, siguiendo sus cinco puntas.)*

El área casi no cambia (24.3 % a 24.28 %): el filtro quitó unas motas y rellenó unos huecos en cantidades parecidas. Su valor está en que la máscara queda **limpia**, y así el borde sigue la hoja y no cada mota de grava.

Ya tenemos la hoja. ¿Qué tan **amarilla** está? El amarillo tiene rojo **y** verde altos; el verde de la clorofila tiene poco rojo. Así que el cociente **R/G** dentro de la hoja mide el amarillamiento: cuanto más bajo, más verde; cerca de 1 (rojo y verde parecidos), amarillo. Veamos su histograma:

```python
indice_rg = R[limpia] / G[limpia]      # solo los pixeles de la hoja
plt.hist(indice_rg, bins=60, range=(0.3, 1.5))
plt.title('R/G dentro de la hoja')
plt.xlabel('R/G')
plt.ylabel('Numero de pixeles')
plt.show()

print("pixeles de hoja:", len(indice_rg))
print("indice de amarillamiento (promedio de R/G):", round(float(indice_rg.mean()), 2))
```

~~~text
pixeles de hoja: 87861
indice de amarillamiento (promedio de R/G): 0.85
~~~

*(Al ejecutar se abre la figura: un **solo** pico, alrededor de 0.86, sin valle.)*

Este histograma tiene **un solo pico**: el amarillamiento es un **continuo** (de verde a amarillo pasando por todos los tonos), no dos grupos separados. Cualquier corte "amarillo sí / no" sería arbitrario. Por eso no umbralizamos: reportamos un **índice continuo**, el promedio de R/G dentro de la hoja (0.85), que sirve para **comparar** hojas o seguir una misma planta en el tiempo. ¿Y no dijimos que un JPEG no sirve para medir intensidades? Para valores finos, no; pero un **promedio** sobre casi 88 mil píxeles apenas cambia con la compresión (recuerda el calentamiento: 49.04 frente a 49.01). El índice sirve para comparar hojas fotografiadas igual; para medir color absoluto haría falta una foto sin pérdida y una tarjeta de color de referencia.

**(Opcional)** Un mapa de color del cociente muestra dónde está el amarillo:

```python
mapa = np.full(R.shape, np.nan)        # np.nan = "sin dato": fuera de la hoja se ve en blanco
mapa[limpia] = indice_rg
plt.imshow(mapa, cmap='RdYlGn_r', vmin=0.6, vmax=1.0)
plt.title('Mapa de R/G (verde = clorofila, rojo = amarillento)')
plt.colorbar(label='R/G')
plt.show()
```

*(Al ejecutar se abre la figura: sobre fondo blanco, las venas en verde y el tejido entre ellas en amarillo, naranja y rojo: el patrón de la clorosis intervenal.)*

## Paso 5: Caso C -- un gel de ADN y el perfil robusto

La imagen `l24_gel.png` es un gel de agarosa teñido con bromuro de etidio (Shinryuu, dominio público). Tiene tres **carriles**: a la izquierda, ADN de células en **apoptosis** (muerte celular programada: el ADN se corta entre los nucleosomas, en fragmentos que difieren en unos 170-200 pares de bases, el tramo de un nucleosoma, y forma una "escalera"); al centro, un **marcador** de tamaños conocidos (2000, 1500, 1000, 750, 500 y 250 pares de bases, rotulados a la derecha); a la derecha, ADN de células **control** (intacto y muy grande: casi no avanza y se queda arriba). Está guardada en **modo `P`** (paleta): hay que pasarla a grises con `convert('L')`.

```python
gel_img = Image.open('imagenes/l24_gel.png')
print("modo original:", gel_img.mode)
g = np.array(gel_img.convert('L'))
print("modo convertido: L | shape:", g.shape)

plt.imshow(g, cmap='gray', vmin=0, vmax=255)
plt.title('Gel de ADN')
plt.xlabel('Columna (pixel)')
plt.ylabel('Fila (pixel)')
plt.show()
```

~~~text
modo original: P
modo convertido: L | shape: (193, 178)
~~~

*(Al ejecutar se abre la figura: tres carriles verticales con bandas claras; arriba, los pozos y, justo debajo, una franja muy brillante; a la derecha, los rótulos de tamaño.)*

Antes de medir, **recortamos** lo que no es dato: arriba, los **pozos** donde se carga la muestra y, justo debajo, el ADN grande que casi no avanzó (en el control, todo su ADN intacto; ahí todo satura), hasta la fila 49; y a la derecha, los **rótulos** (desde la columna 115). Después miramos el **fondo**: el gel no es negro. `np.median` da la **mediana**, el valor del medio (la línea central de la caja de la L23).

```python
gel = g[50:, :115]                     # sin pozos ni ADN atascado (filas 0-49), sin rotulos (columnas 115+)
print("shape recortado:", gel.shape)
print("fondo (mediana de intensidad):", int(np.median(gel)))

plt.hist(gel.ravel(), bins=128, range=(0, 256))
plt.title('Histograma del gel recortado')
plt.xlabel('Intensidad')
plt.ylabel('Numero de pixeles')
plt.show()
```

~~~text
shape recortado: (143, 115)
fondo (mediana de intensidad): 114
~~~

*(Al ejecutar se abre la figura: casi todos los píxeles en un pico entre 105 y 115 -- el fondo gris del gel -- y una cola hacia los valores altos -- las bandas.)*

El fondo está entre 105 y 115 (mediana 114), no en 0: cualquier umbral debe quedar **bastante por encima**. Ahora el perfil de cada carril. Una **sola** columna de un gel real es ruidosa. Más robusto es **promediar una franja** de columnas del carril: `gel[:, c0:c1].mean(axis=1)` promedia las columnas de la franja y deja **un valor por fila**. Comparemos ambas en el marcador (con un umbral de 150) y en el control (cerca del fondo, con 120), lado a lado con `plt.subplots` (L23):

```python
carriles = {"apoptosis": (12, 28), "marcador": (50, 66), "control": (88, 104)}

fig, (izq, der) = plt.subplots(1, 2, figsize=(11, 4))
for eje, nombre, umbral in [(izq, "marcador", 150), (der, "control", 120)]:
    c0, c1 = carriles[nombre]
    eje.plot(gel[:, (c0 + c1) // 2], label='una columna', alpha=0.6)
    eje.plot(gel[:, c0:c1].mean(axis=1), label='promedio de la franja')
    eje.axhline(umbral, color='red', linestyle='--', label='umbral ' + str(umbral))
    eje.set_title('Carril ' + nombre)
    eje.set_xlabel('Fila (desde el borde del recorte)')
    eje.legend()
izq.set_ylabel('Intensidad')
plt.show()
```

*(Al ejecutar se abre la figura: a la izquierda, el marcador con seis picos sobre el umbral 150, casi igual con una columna o con la franja; a la derecha, el control bajo y sin picos: empieza cerca de 118 justo bajo el recorte (el resplandor del ADN atascado de arriba) y baja a unos 105; la curva de una sola columna tiembla más y salta a 122 en la fila 2, por encima del umbral 120, mientras la franja se queda abajo.)*

Para **contar** las bandas usamos una función de estado (un `for`, un `if` y un contador, L08-L15): suma 1 cada vez que el perfil cruza de fondo a señal, con un centinela `dentro` para no contar dos veces la misma banda.

```python
def contar_bandas(perfil, umbral):
    n = 0
    dentro = False
    for valor in perfil:
        if valor > umbral and not dentro:
            n += 1              # empieza una banda nueva
            dentro = True
        elif valor <= umbral:
            dentro = False      # volvimos al fondo
    return n

for nombre, (c0, c1) in carriles.items():
    una = gel[:, (c0 + c1) // 2]
    fr = gel[:, c0:c1].mean(axis=1)
    print(nombre, "| umbral 120: una columna", contar_bandas(una, 120), "franja", contar_bandas(fr, 120),
          "| umbral 150: una columna", contar_bandas(una, 150), "franja", contar_bandas(fr, 150))
```

~~~text
apoptosis | umbral 120: una columna 4 franja 2 | umbral 150: una columna 3 franja 3
marcador | umbral 120: una columna 5 franja 5 | umbral 150: una columna 6 franja 6
control | umbral 120: una columna 1 franja 0 | umbral 150: una columna 0 franja 0
~~~

Con el umbral 150, bien por encima del fondo, el **marcador** da 6 bandas (justo los seis tamaños rotulados) y el **control** da 0 (su ADN intacto se quedó arriba, en la zona que recortamos). Cerca del fondo (120) los conteos fallan: una sola columna del control "ve" una banda que no existe (un salto de una columna en el borde del recorte) y la de apoptosis cuenta 4 donde la franja cuenta 2; en el marcador, dos bandas se funden y salen 5. El carril de **apoptosis** es otra historia: su escalera va montada sobre una "mancha" de fondo que sube y baja, así que su conteo cambia con el umbral:

```python
c0, c1 = carriles["apoptosis"]
franja_apo = gel[:, c0:c1].mean(axis=1)
for u in [130, 150, 170, 190]:
    print("umbral", u, "-> bandas en apoptosis:", contar_bandas(franja_apo, u))
```

~~~text
umbral 130 -> bandas en apoptosis: 2
umbral 150 -> bandas en apoptosis: 3
umbral 170 -> bandas en apoptosis: 3
umbral 190 -> bandas en apoptosis: 5
~~~

Ningún umbral único sirve: hay que **quitar la mancha de fondo** antes de contar los picos. Eso está más allá de nuestras herramientas... y es un buen trabajo para pedirle a la IA en el Paso 6.

## Paso 6: IA con responsabilidad (Gemini en Colab)

Colab trae **Gemini**, un asistente de IA: abre el panel de chat con el botón de Gemini de Colab y escríbele. Puede proponer código con técnicas que no vimos en el curso (por ejemplo, la librería **scikit-image**, que Colab ya trae instalada). Ese código **no entra en el examen**; **cómo verificarlo, sí**. Tres reglas:

- **Primero tú, después la IA.** Ya resolviste los casos a mano; por eso puedes juzgar lo que responda.
- **Verifica siempre** con tus propias medidas (el histograma, la máscara, el área, un caso donde sabes la respuesta). La IA puede equivocarse con total seguridad.
- **Nunca** en trabajo calificado, y no le subas datos sensibles o sin publicar.

Un **prompt técnico** tiene cuatro partes: **contexto** (qué variable, qué `shape` y `dtype`, qué muestra), **meta medible** (qué número quieres), **restricciones** (qué librerías, qué no tocar) y **verificación** (pide que imprima o muestre algo que puedas comprobar).

| Prompt débil | Prompt técnico |
|---|---|
| "cuenta las células de mi imagen" | contexto + meta + restricciones + verificación (abajo) |
| no dice cuál imagen ni cómo es | `nuc` es un arreglo 2D `uint8` `(512, 512)` de núcleos fluorescentes |
| no dice qué número quiere | "imprime cuántos núcleos hay" |
| no pide nada comprobable | "imprime el umbral y el % de área; muestra la segmentación" |

### Prompt 1: contar núcleos que se tocan

Copia este prompt en el chat de Gemini. Fíjate en que **no** le damos nuestro umbral ni nuestra área: así, si su respuesta coincide con la nuestra, es una comprobación de verdad.

~~~text
Tengo en Colab una variable nuc: un arreglo NumPy 2D, dtype uint8,
shape (512, 512), con una micrografia de fluorescencia de nucleos
celulares (claros sobre fondo oscuro). Algunos nucleos se tocan y
forman una sola mancha. Quiero CONTAR los nucleos, separando los
que se tocan. Puedes usar scikit-image y scipy. No modifiques nuc.
Imprime: el umbral que usas, el % de area de los nucleos, el numero
de manchas antes de separar y el numero de nucleos despues de
separar. Explica cada paso en una linea.
~~~

Gemini podría proponer algo como esto (es la **respuesta de referencia** del curso, ya probada; si no tienes Gemini a mano, ejecútala tal cual). No necesitas entender cada función: lo que te toca es **verificarla**.

```python
# Respuesta de referencia (codigo sugerido por una IA y revisado; fuera del curso).
from skimage import filters, measure, segmentation, feature
from scipy import ndimage

umbral_otsu = filters.threshold_otsu(nuc)            # umbral automatico (metodo de Otsu)
mascara_ia = nuc > umbral_otsu
manchas = measure.label(mascara_ia)                    # numera cada mancha conectada
distancia = ndimage.distance_transform_edt(mascara_ia) # distancia de cada pixel al fondo

def separar_nucleos(min_dist):
    # una marca en el centro de cada nucleo (picos de distancia separados por min_dist)
    picos = feature.peak_local_max(distancia, min_distance=min_dist, labels=manchas)
    marcas = np.zeros_like(manchas)
    marcas[tuple(picos.T)] = np.arange(1, len(picos) + 1)
    return segmentation.watershed(-distancia, marcas, mask=mascara_ia)  # separa los que se tocan

nucleos = separar_nucleos(7)
print("umbral (Otsu):", umbral_otsu)
print("area:", round(float(mascara_ia.mean()) * 100, 2), "%")
print("manchas antes de separar:", manchas.max())
print("nucleos despues de separar:", nucleos.max())
```

~~~text
umbral (Otsu): 38
area: 11.36 %
manchas antes de separar: 293
nucleos despues de separar: 299
~~~

**Verifica** con lo que tú sabes. Las dos primeras comprobaciones comparan con tu trabajo del Paso 2; la última prueba si el conteo depende de un número que **eligió la IA** (`min_distance`, la distancia mínima entre dos centros de núcleo):

```python
print("1. Otsu da el mismo umbral que tu isodata?:", umbral_otsu == umbral_nuc)
print("2. misma mascara que la tuya?:", np.array_equal(mascara_ia, mascara_nuc))
print("3. area promedio por nucleo:", round(float(mascara_nuc.sum() / nucleos.max()), 1), "pixeles")
for d in [5, 7, 9]:
    print("4. con min_distance =", d, "->", separar_nucleos(d).max(), "nucleos")
```

~~~text
1. Otsu da el mismo umbral que tu isodata?: True
2. misma mascara que la tuya?: True
3. area promedio por nucleo: 99.6 pixeles
4. con min_distance = 5 -> 315 nucleos
4. con min_distance = 7 -> 299 nucleos
4. con min_distance = 9 -> 284 nucleos
~~~

Lo que **sí** se sostiene: dos métodos independientes (tu `while` y el de Otsu, que no conocía tu resultado) llegan al **mismo** umbral y a la **misma** máscara, y unos 100 píxeles por núcleo (un círculo de unos 11 píxeles de diámetro) concuerda con lo que se ve en el acercamiento del Paso 3. Lo que **no**: el número exacto de núcleos depende de `min_distance` (de 284 a 315). Lo honesto es reportar **unos 300 núcleos (entre 280 y 320)**, no "299" como si fuera exacto. Un número de la IA que cambia con un parámetro que nadie justificó es una estimación, no un dato.

### Prompt 2 (opcional): quitar el fondo del carril de apoptosis

~~~text
Tengo un perfil 1D franja_apo (arreglo NumPy de 143 valores, la
intensidad por fila) de un carril de un gel de ADN de celulas en
apoptosis: una escalera de bandas montada sobre una mancha de fondo
que sube y baja, asi que un umbral fijo no sirve para contar las
bandas. Quiero contar los picos (bandas). Puedes usar scipy. Imprime
el numero de picos y sus posiciones. Aplica EXACTAMENTE el mismo
metodo a un carril marcador con 6 bandas conocidas,
gel[:, 50:66].mean(axis=1), y a un carril control sin bandas,
gel[:, 88:104].mean(axis=1), y prueba que tan sensible es a sus
parametros.
~~~

Fíjate en la última línea: pedimos que **pruebe el método donde ya sabemos la respuesta** y que diga **qué tan sensible** es. Una respuesta de referencia:

```python
# Respuesta de referencia (codigo sugerido por una IA y revisado; fuera del curso).
from scipy.signal import find_peaks

def contar_picos(perfil, prominencia):
    # prominence = cuanto sobresale un pico sobre lo que lo rodea (ignora la mancha lenta)
    posiciones, _ = find_peaks(perfil, prominence=prominencia)
    return posiciones

for p in [10, 15, 20]:
    conteos = [len(contar_picos(gel[:, c0:c1].mean(axis=1), p)) for (c0, c1) in carriles.values()]
    print("prominence", p, "-> apoptosis, marcador, control:", conteos)
```

~~~text
prominence 10 -> apoptosis, marcador, control: [7, 6, 0]
prominence 15 -> apoptosis, marcador, control: [6, 6, 0]
prominence 20 -> apoptosis, marcador, control: [6, 6, 0]
~~~

El marcador da 6 y el control 0 con los tres valores, pero eso **no basta**: el marcador tiene bandas tan altas que cualquier valor las encuentra, así que no sirve para elegir el parámetro. En la apoptosis, en cambio, el conteo cambia (7 con 10; 6 con 15 y 20). Falta una comprobación **científica**: los peldaños deberían estar **separados** por unos 160-200 pares de bases (un nucleosoma). Usemos el marcador como **regla** para estimar el tamaño de cada peldaño. Lo que cuenta es esa idea, no las funciones: `np.interp` interpola entre las bandas conocidas y `np.log10` usa escala logarítmica, como se leen los geles (las dos quedan fuera del examen).

```python
pos_marcador = contar_picos(gel[:, 50:66].mean(axis=1), 15)
pos_escalera = contar_picos(franja_apo, 15)
tamanos = [2000, 1500, 1000, 750, 500, 250]       # pares de bases del marcador
aprox = 10 ** np.interp(pos_escalera, pos_marcador, np.log10(tamanos))
print("filas del marcador:", pos_marcador.tolist())
print("filas de la escalera:", pos_escalera.tolist())
print("tamano aproximado (pb):", np.round(aprox).astype(int).tolist())
```

~~~text
filas del marcador: [3, 14, 33, 47, 69, 103]
filas de la escalera: [31, 41, 53, 68, 89, 124]
tamano aproximado (pb): [1044, 848, 671, 509, 333, 250]
~~~

Los peldaños quedan a unos 1040, 850, 670, 510 y 330 pares de bases, separados por 160-200: el paso de un nucleosoma, justo lo que predice la biología (con una regla tan aproximada como un marcador de seis bandas, no se espera más precisión). El último cae en la banda de 250 o por debajo (`np.interp` no estima por debajo de la banda más pequeña del marcador, así que imprime 250). Conclusión honesta: **unos 6 peldaños, estable con `prominence` entre 15 y 20, y con tamaños coherentes con la apoptosis**. Sin esas dos pruebas, un número de la IA es solo un número.

## Tu turno

### Ejercicio 1: revisa la respuesta de la IA (hoja)

Un compañero le preguntó a Gemini y pegó esto en el grupo: *"La hoja cubre el 28 % de la foto."* Nosotros medimos 24.3 %. Sin preguntarle a la IA, averigua **qué umbral** de `G - B` pudo dar ese 28 % y **por qué** es peor que el nuestro. Pista: prueba umbrales por debajo de 71 y recuerda el histograma de `G - B` del Paso 4.

```python
# TODO: completa la lista con umbrales menores que 71 y busca el que da cerca de 28 %.
# for u in [____]:
#     print("umbral", u, "-> area", round(float((dif > u).mean()) * 100, 2), "%")
```

<details><summary>Respuesta esperada</summary>

~~~python
for u in [30, 40, 50]:
    print("umbral", u, "-> area", round(float((dif > u).mean()) * 100, 2), "%")
~~~

Imprime:

~~~text
umbral 30 -> area 35.56 %
umbral 40 -> area 28.33 %
umbral 50 -> area 26.4 %
~~~

El umbral 40 da 28.33 %: ese es el 28 %. Es peor porque 40 queda en la **falda del pico de la grava** del histograma, no en el valle (cerca de 71): se cuelan muchas motas de grava verdosa como si fueran hoja. Verificaste a la IA con un histograma y un bucle.
</details>

### Ejercicio 2: ¿cuántas bandas ves con cada umbral?

Cuenta las bandas del **marcador** con umbrales de 150, 180 y 200 usando el promedio de su franja. Completa el `____` con las columnas del marcador (míralas en el diccionario `carriles`) y explica por qué el conteo baja.

```python
# TODO: completa las columnas de la franja del marcador.
# franja_marcador = gel[:, ____].mean(axis=1)
# for u in [150, 180, 200]:
#     print("umbral", u, "-> bandas del marcador:", contar_bandas(franja_marcador, u))
```

<details><summary>Respuesta esperada</summary>

~~~python
franja_marcador = gel[:, 50:66].mean(axis=1)
for u in [150, 180, 200]:
    print("umbral", u, "-> bandas del marcador:", contar_bandas(franja_marcador, u))
~~~

Imprime:

~~~text
umbral 150 -> bandas del marcador: 6
umbral 180 -> bandas del marcador: 5
umbral 200 -> bandas del marcador: 4
~~~

En el perfil del Paso 5, las bandas **pequeñas** (abajo en el gel) son las más tenues: la de 250 pares de bases apenas llega a 160 y la de 500 a 193. Con el umbral 180 se pierde la de 250; con 200, también la de 500. Un umbral demasiado alto pierde señal tenue, igual que en la micrografía.
</details>

### Ejercicio 3 (opcional): quita lo que no es hoja

La máscara limpia de la hoja (`limpia`) todavía incluye unas manchitas arriba de la hoja: un brote verde y motas de grava. La hoja empieza más abajo, cerca de la fila 189. Escribe tú el código: haz una **copia** de `limpia`, pon en `False` las filas de arriba (por ejemplo, las primeras 150) con un rebanado, y calcula de nuevo el área y el índice R/G.

```python
# TODO: escribe aqui tu codigo (Ejercicio 3).
```

<details><summary>Respuesta esperada</summary>

~~~python
hoja_sola = limpia.copy()
hoja_sola[:150] = False          # las filas 0-149 no son la hoja
print("area de la hoja sola:", round(float(hoja_sola.mean()) * 100, 2), "%")
print("indice R/G:", round(float((R[hoja_sola] / G[hoja_sola]).mean()), 2))
~~~

Imprime `area de la hoja sola: 24.13 %` (antes, 24.28 %) e `indice R/G: 0.85` (igual). Recortar quitó 552 píxeles que no eran hoja; el índice no cambia porque eran muy pocos frente a los casi 88 mil de la hoja.
</details>

### Ejercicio 4 (opcional, desafío): los núcleos en división

En la micrografía, los núcleos **más brillantes** son células en **división** (mitosis): su ADN está condensado. Sin andamiaje: mira la **cola derecha** del histograma del Paso 2, elige un umbral para separarlos del resto de los núcleos, calcula qué porcentaje del campo cubren y muestra la máscara. Di qué umbral usaste y por qué.

```python
# TODO: escribe aqui tu codigo (Ejercicio 4).
```

<details><summary>Respuesta esperada</summary>

No hay un único umbral correcto: en la cola no hay un valle tan claro como el de núcleo/fondo. Uno razonable queda donde termina la joroba de los núcleos normales, entre 80 y 120. Por ejemplo:

~~~python
umbral_division = 110
division = nuc > umbral_division
print("area de nucleos en division:", round(float(division.mean()) * 100, 2), "%")
plt.imshow(division, cmap='gray')
plt.show()
~~~

Con 110 sale `area de nucleos en division: 0.48 %` (con 80, 1.74 %; con 100, 0.65 %; con 120, 0.37 %). La máscara muestra unas cincuenta manchitas, muchas solo el **centro** brillante de un núcleo y no el núcleo entero: un umbral alto es una aproximación. El ejemplo de scikit-image con esta misma imagen usa un método automático de tres clases que corta en 82. Lo importante: **dices qué umbral usaste y por qué**, porque el resultado depende de él.
</details>

## Resumen

- Una imagen en grises es **un canal**: un arreglo 2D `(alto, ancho)` de `uint8`; a color son tres (L20). `convert('L')` pasa una imagen de color o de paleta a grises.
- **Formatos:** TIFF/PNG sin pérdida (para medir intensidades); un JPEG altera los valores, aunque el área umbralizada pueda no cambiar.
- **Histograma** (`plt.hist(arr.ravel(), ...)`, `log=True` si domina el fondo): el umbral va en el **valle**; `umbral_automatico` (un `while` + máscaras) lo encuentra solo y coincide con Otsu.
- **Color:** combina canales (`G - B`, siempre con `astype(int)`); limpia la máscara con `MedianFilter` y traza el borde con `FIND_EDGES`. Si el histograma no tiene valle, reporta un **índice continuo** (R/G), no un corte.
- **Realzar** (`* k`, `+ c`, `np.clip`) cambia lo que ves, no lo que mides; **recorta** lo que no es dato.
- **Perfil robusto:** promedia una franja (`mean(axis=1)`) y pon el umbral por encima del fondo; cuenta con `contar_bandas`.
- **IA:** prompt técnico (contexto, meta, restricciones, verificación) y **verifica** con tus medidas o en casos donde sabes la respuesta.
- **Sigue en la L25:** otro tipo de dato científico, las **secuencias de ADN**, con la librería **Biopython**.
