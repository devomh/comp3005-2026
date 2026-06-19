---
title: "Lab: NumPy"
unit: "3"
lesson: 20
type: lab
tags: [python, numpy, arreglos, vectorizado, media, desviacion, import, mediciones]
difficulty: introductory
duration: "60 mins"
---

# Lab: NumPy

**Meta:** importar NumPy (`import numpy as np`); poner mediciones en un **arreglo** (`np.array`); operar de forma **vectorizada** (sobre todos los elementos a la vez); calcular agregados (`np.mean`, `np.std`, `np.sum`, `np.max`, `np.min`); indexar y manejar una **matriz** 2D; y filtrar con una **máscara booleana**. Es el mismo dato de la L19, pero ahora calculas sobre todo el arreglo de golpe. Acompaña a la nota de concepto [NumPy](l20_concept_numpy.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`, empezando por las dos celdas de preparación. Donde diga **Predice**, hay líneas comentadas con `#`: piensa primero qué pasaría (la del Paso 4 daría un error a propósito -- déjala comentada). Donde diga **Tu turno**, hay un bloque comentado con una parte marcada con `____` que debes **completar** al quitar los `#`. Compara con la respuesta esperada de cada bloque desplegable.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u03_scientific_computing/l20_lab_numpy.ipynb)

## Preparación

NumPy no es parte de Python estándar: hay que **instalarlo** (en Colab) e **importarlo**. Corre estas **dos** celdas primero. La primera instala NumPy (en Colab; si ya está, no hace nada). La segunda lo importa con el alias `np`.

```python
%pip install -q numpy
```

```python
import numpy as np

print("NumPy listo para trabajar")
```

~~~text
NumPy listo para trabajar
~~~

## Paso 1: crear un arreglo

Un **arreglo** de NumPy se crea con `np.array(...)` a partir de una lista. Pongamos las temperaturas de la L19 en un arreglo:

```python
temperaturas = np.array([18.0, 21.0, 17.0, 22.0, 19.0, 20.0])
print(temperaturas)
print(type(temperaturas))
```

~~~text
[18. 21. 17. 22. 19. 20.]
<class 'numpy.ndarray'>
~~~

Se imprime con espacios y un punto decimal (`[18. 21. ...]`), y su tipo es `numpy.ndarray`. Por dentro está optimizado para operar sobre todos sus números a la vez.

## Paso 2: operaciones vectorizadas

Un arreglo aplica una operación a **cada** elemento, sin un `for`. Sumarle 2 le suma 2 a cada medición; `* 1.8 + 32` convierte cada temperatura a Fahrenheit:

```python
print(temperaturas + 2)
print(temperaturas * 1.8 + 32)
```

~~~text
[20. 23. 19. 24. 21. 22.]
[64.4 69.8 62.6 71.6 66.2 68. ]
~~~

Compara con una **lista**: ahí `* 2` no duplica los números, **repite** la lista entera.

```python
print([18.0, 21.0] * 2)
```

~~~text
[18.0, 21.0, 18.0, 21.0]
~~~

Esa es la diferencia central: `lista * 2` repite; `arreglo * 2` opera elemento a elemento.

## Paso 3: media, desviación y otros agregados

Los resúmenes más comunes son una sola llamada (en la L19 esto era un `for`):

```python
print(np.mean(temperaturas))
print(round(np.std(temperaturas), 2))
print(np.sum(temperaturas))
print(np.max(temperaturas))
print(np.min(temperaturas))
```

~~~text
19.5
1.71
117.0
22.0
17.0
~~~

La media es `19.5`, el mismo promedio que en la L19 calculaste con un `for`. La desviación estándar (`np.std`) es un decimal largo, así que la mostramos con `round(np.std(...), 2)` -> `1.71`.

## Paso 4: indexar, rebanar y matrices (predice primero)

Un arreglo se indexa y se rebana como una lista:

```python
print(temperaturas[:3])
```

~~~text
[18. 21. 17.]
~~~

Cuando los datos son una **tabla** se usa un arreglo **2D** (matriz). `.shape` da (filas, columnas); `M[:, 1]` toma toda la columna 1; `np.mean(M)` promedia todos los números:

```python
M = np.array([[1, 2, 3],
              [4, 5, 6]])
print(M)
print(M.shape)
print(M[:, 1])
print(np.mean(M))
```

~~~text
[[1 2 3]
 [4 5 6]]
(2, 3)
[2 5]
3.5
~~~

**Predice:** ¿qué pasaría si sumas dos arreglos de **tamaños distintos**? La línea está comentada a propósito: piénsalo y luego mira la respuesta.

```python
# Predice: sumar dos arreglos de formas distintas
# print(np.array([1, 2, 3]) + np.array([1, 2]))

print('Las operaciones vectorizadas necesitan arreglos de la misma forma (o un escalar)')
```

~~~text
Las operaciones vectorizadas necesitan arreglos de la misma forma (o un escalar)
~~~

<details><summary>Respuesta esperada</summary>

Si quitaras los `#`, sumar `np.array([1, 2, 3])` (3 elementos) con `np.array([1, 2])` (2 elementos) daría un error:

~~~text
ValueError: operands could not be broadcast together with shapes (3,) (2,)
~~~

NumPy no sabe cómo emparejar 3 elementos con 2. Las operaciones elemento a elemento necesitan arreglos de la **misma** forma (o un arreglo y un escalar, como `arreglo + 2`). (La dejamos comentada porque un error vivo detendría el resto del notebook.)
</details>

## Tu turno (programa completo)

### Ejercicio: conteos celulares con un arreglo

Un microscopio contó las células de cinco muestras: `120, 135, 98, 142, 110`. Pon esos conteos en un arreglo de NumPy, calcula su **media** (con `round` a 2 decimales) y usa una **máscara booleana** para quedarte con los conteos **mayores que 120**. Completa la parte que falta.

```python
# TODO: quita los # y completa la parte que falta (el operador de comparacion, ____).
# conteos = np.array([120, 135, 98, 142, 110])
#
# print(round(np.mean(conteos), 2))
# print(conteos[conteos ____ 120])
```

<details><summary>Respuesta esperada</summary>

~~~text
121.0
[135 142]
~~~

La parte que falta es `>` (mayor que): `conteos[conteos > 120]`. La media es `round(np.mean(conteos), 2)` -> `(120 + 135 + 98 + 142 + 110) / 5 = 121.0`. La máscara `conteos > 120` marca cada conteo como `True`/`False`, y `conteos[conteos > 120]` devuelve solo los que pasan: `[135 142]` (los conteos `120`, `98` y `110` no son mayores que `120`). Es el filtro de la L19, ahora sin un `for`.
</details>

## Resumen

- Se **importa** con `import numpy as np`; a partir de ahí todo es `np.algo`.
- `np.array([...])` crea un **arreglo** de números del mismo tipo, optimizado para cálculo.
- Las operaciones son **vectorizadas**: `arreglo + 2`, `arreglo * 1.8 + 32` actúan sobre **todos** los elementos a la vez, sin un `for`. (`lista * 2` repite; `arreglo * 2` duplica cada elemento.)
- Los **agregados** son una llamada: `np.mean`, `np.std` (muéstrala con `round`), `np.sum`, `np.max`, `np.min`.
- Un arreglo se indexa/rebana como una lista; una **matriz** 2D tiene `.shape`, `M[i, j]` y `M[:, j]` (una columna).
- Una **máscara booleana** `a[a > umbral]` filtra sin `for`, y `np.sum(a > umbral)` cuenta los que cumplen.
- **Sigue en la L21** con **Pandas**: una columna de un `DataFrame` es, por dentro, un arreglo de NumPy, así que `df['col'].mean()` usa el `mean` de hoy.
