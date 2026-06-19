---
title: "Lab: Archivos CSV"
unit: "3"
lesson: 19
type: lab
tags: [python, csv, archivos, split, tabular, datos, read_csv, pandas-puente]
difficulty: introductory
duration: "60 mins"
---

# Lab: Archivos CSV

**Meta:** leer un archivo **CSV** a mano: abrirlo con `with open(...)`, saltar el encabezado con `next(f)`, separar cada línea en campos con `.split(',')` y desempaquetarla en variables, convertir los campos a número y aplicar la **receta tabular** (contar, promediar, máximo, filtrar por grupo, contar por categoría). Abre el Módulo 3 y conecta con todo el Módulo 2. Acompaña a la nota de concepto [Archivos CSV](l19_concept_csv_files.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`. Donde diga **Predice**, hay líneas comentadas con `#`: piensa primero qué pasaría (la del Paso 4 daría un error a propósito -- déjala comentada). Donde diga **Tu turno**, hay un bloque comentado con una parte marcada con `____` que debes **completar** al quitar los `#`. Compara con la respuesta esperada de cada bloque desplegable.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u03_scientific_computing/l19_lab_csv_files.ipynb)

## Preparación

Solo Python estándar: no hay que instalar nada. El archivo CSV que vas a leer lo crea este mismo lab al escribirlo. Corre esta celda primero.

```python
print("Listo para trabajar con archivos CSV")
```

~~~text
Listo para trabajar con archivos CSV
~~~

## Paso 1: escribir y ver el CSV

Primero creamos `muestras.csv` (un monitoreo de calidad de agua) escribiéndolo con el modo `'w'` de la L16. Cada línea es una fila; los campos van separados por comas; la primera línea es el **encabezado**.

```python
muestras = """muestra,sitio,ph,temperatura
1,rio_Norte,7.2,18.0
2,rio_Sur,6.8,21.0
3,rio_Norte,7.4,17.0
4,rio_Sur,6.5,22.0
5,rio_Norte,7.0,19.0
6,rio_Sur,6.8,20.0
"""

with open('muestras.csv', 'w') as f:
    f.write(muestras)

print('Archivo muestras.csv creado')
```

~~~text
Archivo muestras.csv creado
~~~

Veamos solo la **primera línea** (el encabezado) leyendo el archivo e interrumpiendo con `break`:

```python
with open('muestras.csv', 'r') as f:
    for linea in f:
        print(linea.strip())
        break
```

~~~text
muestra,sitio,ph,temperatura
~~~

Esos son los nombres de las columnas: `muestra`, `sitio`, `ph`, `temperatura`. No son datos -- por eso los saltaremos al procesar.

## Paso 2: parsear las líneas

Para procesar los datos, saltamos el encabezado con `next(f)` y separamos cada línea en campos con `.split(',')`, repartiéndolos en variables con una **asignación múltiple**. Imprimamos solo la columna `ph`:

```python
with open('muestras.csv', 'r') as f:
    next(f)                      # salta el encabezado
    for linea in f:
        muestra, sitio, ph, temp = linea.strip().split(',')
        print(ph)
```

~~~text
7.2
6.8
7.4
6.5
7.0
6.8
~~~

`.strip()` quita el `\n` (L16), `.split(',')` devuelve la lista de campos (L13) y la asignación múltiple reparte los cuatro valores en cuatro variables. Ojo: `ph` aquí es la **cadena** `'7.2'`, no el número. Para calcular hay que convertirlo con `float()`.

## Paso 3: la receta tabular

La receta de casi todo ejercicio: recorrer las líneas acumulando lo que interesa en una lista y luego calcular. Primero, **contar las filas** (sin el encabezado):

```python
with open('muestras.csv', 'r') as f:
    next(f)
    n = 0
    for linea in f:
        n += 1

print('Filas:', n)
```

~~~text
Filas: 6
~~~

El **promedio de pH**: acumular cada `ph` (convertido a `float`) en una lista y dividir la suma entre la cantidad.

```python
with open('muestras.csv', 'r') as f:
    next(f)
    phs = []
    for linea in f:
        muestra, sitio, ph, temp = linea.strip().split(',')
        phs.append(float(ph))

print('Promedio de pH:', sum(phs) / len(phs))
```

~~~text
Promedio de pH: 6.95
~~~

La **temperatura máxima**, sin usar `max()`, con el patrón del acumulador de la L08: guardar la mayor vista hasta ahora.

```python
with open('muestras.csv', 'r') as f:
    next(f)
    tmax = 0.0
    for linea in f:
        muestra, sitio, ph, temp = linea.strip().split(',')
        t = float(temp)
        if t > tmax:
            tmax = t

print(tmax)
```

~~~text
22.0
~~~

## Paso 4: filtrar y agregar (predice primero)

Muchas preguntas son sobre un **grupo**, no sobre toda la tabla. Filtramos por una columna categórica con un `if`. Imprimamos los registros con `ph` mayor que 7.0:

```python
with open('muestras.csv', 'r') as f:
    next(f)
    for linea in f:
        muestra, sitio, ph, temp = linea.strip().split(',')
        if float(ph) > 7.0:
            print(muestra, sitio, ph)
```

~~~text
1 rio_Norte 7.2
3 rio_Norte 7.4
~~~

Y el **promedio de temperatura solo del `rio_Norte`**: acumular solo las filas que cumplen la condición.

```python
with open('muestras.csv', 'r') as f:
    next(f)
    temps = []
    for linea in f:
        muestra, sitio, ph, temp = linea.strip().split(',')
        if sitio == 'rio_Norte':
            temps.append(float(temp))

print('Promedio temp rio_Norte:', sum(temps) / len(temps))
```

~~~text
Promedio temp rio_Norte: 18.0
~~~

¿Cuántos registros hay de cada sitio? El idioma de conteo con diccionario de la L14 funciona igual sobre un CSV:

```python
conteo = {}
with open('muestras.csv', 'r') as f:
    next(f)
    for linea in f:
        muestra, sitio, ph, temp = linea.strip().split(',')
        conteo[sitio] = conteo.get(sitio, 0) + 1

print(conteo)
```

~~~text
{'rio_Norte': 3, 'rio_Sur': 3}
~~~

**Predice:** ¿qué pasaría si **olvidas** `next(f)` y tratas de convertir el encabezado a número? Las líneas están comentadas a propósito: piénsalo y luego mira la respuesta.

```python
# Predice: que pasa si OLVIDAS next(f) y conviertes el encabezado
# with open('muestras.csv', 'r') as f:
#     for linea in f:
#         muestra, sitio, ph, temp = linea.strip().split(',')
#         print(float(temp))   # la 1a linea es el encabezado: float('temperatura')
#         break

print('Recuerda: sin next(f), la primera linea es el encabezado (texto, no numeros)')
```

~~~text
Recuerda: sin next(f), la primera linea es el encabezado (texto, no numeros)
~~~

<details><summary>Respuesta esperada</summary>

Si quitaras los `#`, la primera vuelta del `for` tomaría el encabezado `muestra,sitio,ph,temperatura`. Al desempaquetarlo, `temp` valdría la cadena `'temperatura'`, y `float('temperatura')` daría un error:

~~~text
ValueError: could not convert string to float: 'temperatura'
~~~

Por eso siempre se salta el encabezado con `next(f)` antes de procesar: los nombres de las columnas son texto, no números. (La dejamos comentada porque un error vivo detendría el resto del notebook.)
</details>

## Tu turno (programa completo)

### Ejercicio: total de individuos en un conteo de especies

Un muestreo de biodiversidad guardó, en `especies.csv`, una fila por observación con las columnas `muestra,sitio,especie,conteo`. Escribe una función `total_conteo(nombre)` que lea el archivo, sume la columna `conteo` (conviértela a `int`) y devuelva el total de individuos. Completa la parte que falta.

```python
# TODO: quita los # y completa la parte que falta (la columna que se suma, ____).
# especies = """muestra,sitio,especie,conteo
# 1,laguna_A,tilapia,12
# 2,laguna_A,guppy,8
# 3,laguna_B,tilapia,15
# 4,laguna_B,guppy,6
# 5,laguna_A,tilapia,9
# """
# with open('especies.csv', 'w') as f:
#     f.write(especies)
#
# def total_conteo(nombre):
#     total = 0
#     with open(nombre, 'r') as f:
#         next(f)
#         for linea in f:
#             muestra, sitio, especie, conteo = linea.strip().split(',')
#             total += int(____)
#     return total
#
# print('Total de individuos:', total_conteo('especies.csv'))
```

<details><summary>Respuesta esperada</summary>

~~~text
Total de individuos: 50
~~~

La parte que falta es `conteo`: `total += int(conteo)`, la cuarta columna (el número de individuos de esa observación). La función salta el encabezado con `next(f)`, separa cada línea con `.split(',')`, convierte el campo `conteo` a `int` y lo acumula: `12 + 8 + 15 + 6 + 9 = 50`. Como cualquier función de archivo (L16), la reutilizas con otro muestreo: `total_conteo('otro_muestreo.csv')`.
</details>

## El puente a Pandas (solo para mirar)

Toda esta receta -- abrir, saltar el encabezado, separar, convertir, acumular, calcular -- la hace **Pandas** en una línea con `read_csv`. Lo verás de verdad en la S21; aquí solo míralo (no se ejecuta, porque hoy el objetivo es entenderlo a mano):

```python
# Esto es la S21 (no se ejecuta aqui): el destino del puente
# import pandas as pd
# df = pd.read_csv('muestras.csv')
# print(round(df['ph'].mean(), 2))    # -> 6.95  (el mismo promedio que calculaste a mano)
```

Hiciste a mano lo que Pandas automatiza. Por eso, cuando uses `read_csv`, sabrás exactamente qué está haciendo por dentro.

## Resumen

- Un CSV es un archivo de **texto**: cada línea es una fila; los campos van separados por comas; la primera línea suele ser el encabezado.
- Se lee con lo de la L16 (`with open(...)`, iterar `for linea in f:`) más dos piezas: **`next(f)`** salta el encabezado y `linea.strip().split(',')` separa los campos, que se reparten con una **asignación múltiple** `a, b, c = ...`.
- Todo campo sale como **texto**: hay que **convertirlo** con `float()`/`int()` antes de calcular.
- La **receta tabular**: abrir, saltar, por línea separar + convertir + (filtrar con `if`) + acumular en una lista, y calcular (promedio, máximo sin `max()`, conteo).
- **Filtrar** por una columna categórica y **agregar** una numérica da resultados por grupo; un diccionario cuenta registros por categoría (el germen de `value_counts`).
- **Sigue el Módulo 3** con **NumPy** (L20) para operar sobre arreglos de mediciones y **Pandas** (L21), que leerá el CSV con `read_csv` en una sola línea -- la receta que hoy hiciste a mano.
