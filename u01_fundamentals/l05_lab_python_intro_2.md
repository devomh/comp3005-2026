---
title: "Lab: Introducción a Python II"
unit: "1"
lesson: 5
type: lab
tags: [python, conversion-de-tipos, input, operadores, depuracion, estilo]
difficulty: introductory
duration: "60 mins"
---

# Lab: Introducción a Python II

**Meta:** convertir tipos, leer datos, entender por qué un operador cambia según el tipo, leer un reporte de error y escribir código legible. Acompaña a la nota de concepto [Introducción a Python II](l05_concept_python_intro_2.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`. Donde diga **Completa tú** o **Tu turno**, las líneas vienen comentadas con `#`: quítalo y complétalas. Compara con la respuesta esperada de cada bloque desplegable.

## Preparación

Solo Python estándar: no hay que instalar nada. Corre esta celda primero.

```python
print("Listo para seguir con Python")
```

~~~text
Listo para seguir con Python
~~~

## Paso 1: conversión de tipos

Los nombres de los tipos funcionan como funciones de conversión. Ejecuta y observa cómo `+` cambia de comportamiento según el tipo.

```python
print(int("15"))               # cadena -> entero
print(float("15"))             # cadena -> flotante
print(str(15))                 # numero -> cadena
print("15" + "27")             # cadenas: concatena (pega)
print(int("15") + int("27"))   # enteros: suma
```

~~~text
15
15.0
15
1527
42
~~~

Las dos últimas líneas son el corazón de la lección: `"15" + "27"` **pega** (`1527`), pero `int("15") + int("27")` **suma** (`42`). Convertir primero decide cuál de las dos ocurre.

## Paso 2: leer datos con `input()`

En Colab, para pedir un dato al usuario escribirías algo como esto (NO lo ejecutes aquí; solo obsérvalo):

~~~python
masa = float(input("Ingrese la masa (g): "))
~~~

Como este notebook se ejecuta sin detenerse a pedir datos, usamos un **valor fijo** que simula lo que devolvería `input()` (siempre una cadena) y luego lo convertimos.

```python
lectura = "15"             # input() SIEMPRE devuelve una cadena
print(type(lectura))       # confirma que es str
masa = float(lectura)      # convertimos a numero
print(type(masa), masa)    # ahora es float
```

~~~text
<class 'str'>
<class 'float'> 15.0
~~~

## Paso 3: un operador, varios comportamientos

**Completa tú.** Antes de ejecutar, **predice** cada resultado. Luego quita los `#` y comprueba. Recuerda: `+` suma números pero concatena cadenas.

```python
# Predice primero, luego quita los # y comprueba.
# print(3 + 5)           # int + int
# print(3.2 + 5.1)       # float + float
# print("3.2" + "5.1")   # str + str
```

<details><summary>Respuesta esperada</summary>

~~~text
8
8.3
3.25.1
~~~

`3 + 5` suma enteros (`8`); `3.2 + 5.1` suma flotantes (`8.3`); `"3.2" + "5.1"` concatena las cadenas y da `3.25.1` (no es una suma).
</details>

## Paso 4: leer un reporte de error (depuración)

Este programa intenta sumar un número con un texto. Obsérvalo (NO hace falta ejecutarlo):

~~~python
base = 5.0
altura = "3"                       # se quedo como cadena (sin convertir)
perimetro = 2 * base + 2 * altura  # aqui falla
print("El perimetro es:", perimetro)
~~~

Al ejecutarlo, Python imprime este reporte de error (*traceback*):

~~~text
Traceback (most recent call last):
  File "<ipython-input>", line 3, in <module>
    perimetro = 2 * base + 2 * altura
                ~~~~~~~~~^~~~~~~~~~~~
TypeError: unsupported operand type(s) for +: 'float' and 'str'
~~~

Para leerlo: la **última línea** da el **tipo** (`TypeError`) y el **mensaje** (se usó `+` entre un `float` y un `str`); arriba, la **línea** donde ocurrió. El arreglo es convertir `altura` a número. Ejecuta la versión corregida:

```python
base = 5.0
altura = 3.0          # corregido: ahora es un numero (antes era "3")
perimetro = 2 * base + 2 * altura
print("El perimetro es:", perimetro)
```

~~~text
El perimetro es: 16.0
~~~

## Paso 5: escribir código legible

Este programa funciona, pero es difícil de leer (todo junto, nombres pobres). Ejecútalo: imprime el resultado.

```python
v=60.0;t=2.0;d=v*t;print(d)
```

~~~text
120.0
~~~

**Completa tú.** Reescríbelo de forma legible: nombres con significado, líneas en blanco y un mensaje claro al imprimir.

```python
# Quita los # y reescribe el programa de arriba de forma legible.
# velocidad = 60.0   # millas por hora
# tiempo = 2.0       # horas
# distancia = ____
# print("Se recorren", distancia, "millas")
```

<details><summary>Respuesta esperada</summary>

~~~text
Se recorren 120.0 millas
~~~

La versión legible separa entradas, cálculo y salida, usa nombres claros (`velocidad`, `tiempo`, `distancia`) e imprime un mensaje entendible. Hace lo mismo que `v=60.0;t=2.0;d=v*t;print(d)`, pero se entiende de un vistazo.
</details>

## Tu turno (programa completo)

### Ejercicio: promedio de dos mediciones

Dos lecturas de pH llegan como **texto** (como si vinieran de `input()`): `"7.2"` y `"7.8"`. Conviértelas a número, calcula el **promedio** y muéstralo redondeado a 2 decimales con `round(valor, 2)`.

```python
lectura1 = "7.2"       # texto, como lo devolveria input()
lectura2 = "7.8"
# TODO: quita los # y completa las tres lineas.
# ph1 = ____
# ph2 = ____
# promedio = (ph1 + ph2) / 2
# print("El pH promedio es", round(promedio, 2))
```

<details><summary>Respuesta esperada</summary>

~~~text
El pH promedio es 7.5
~~~

Hay que convertir cada lectura con `float()` antes de promediar. Si olvidas convertir, `"7.2" + "7.8"` concatenaría los textos (`"7.27.8"`) en vez de sumarlos.
</details>

## Resumen

- `int()`, `float()`, `str()` convierten entre tipos; `input()` siempre devuelve una cadena, así que conviene convertir lo que lees.
- El operador `+` suma números pero **concatena** cadenas: por eso convertir primero importa.
- Un *traceback* se lee de abajo hacia arriba: la última línea da el **tipo** y el **mensaje** del error.
- Código legible: nombres con significado, líneas en blanco, comentarios útiles.
- **Sigue con la L06:** aprenderás **lógica booleana** para que tus programas tomen decisiones.
