---
title: "Lab: Introducción a Python I"
unit: "1"
lesson: 4
type: lab
tags: [python, calculadora, variables, tipos-de-datos, colab]
difficulty: introductory
duration: "60 mins"
---

# Lab: Introducción a Python I

**Meta:** escribir y ejecutar tus primeros programas en Python: la calculadora, `print`, variables y tipos. Acompaña a la nota de concepto [Introducción a Python I](l04_concept_python_intro_1.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`. Donde diga **Tu turno**, escribe tú el código (las líneas vienen comentadas con `#`: quítalo y complétalas). Compara con la respuesta esperada que aparece en cada bloque desplegable.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u01_fundamentals/l04_lab_python_intro_1.ipynb)

## Preparación

Esta lab usa solo Python estándar: no hay que instalar nada ni importar librerías. Corre esta celda primero para confirmar que el notebook responde.

```python
# Corre esta celda primero (Colab reinicia el entorno al abrir).
print("Listo para programar en Python")
```

~~~text
Listo para programar en Python
~~~

## Paso 1: Python como calculadora

Una celda de código se comporta como una calculadora. Ejecuta los siete operadores y observa cada resultado.

```python
print(7 + 3)
print(7 - 3)
print(7 * 3)
print(7 / 3)
print(7 ** 3)
print(7 // 3)
print(7 % 3)
```

~~~text
10
4
21
2.3333333333333335
343
2
1
~~~

Fíjate en los tres que suelen confundir: `/` da la división real (`2.3333333333333335`), `//` da solo el cociente entero (`2`) y `%` da el residuo (`1`).

Un cálculo científico es igual de simple. La concentración de `0.5` moles disueltos en `2.0` litros:

```python
# concentracion = moles / litros
print(0.5 / 2.0)
```

~~~text
0.25
~~~

## Paso 2: variables y `print()`

Una **variable** le pone nombre a un dato con la asignación `nombre = dato`. Ejecuta este ejemplo: calcula el área de un rectángulo.

```python
ancho = 5
largo = 6
area = ancho * largo
print("area =", area)
```

~~~text
area = 30
~~~

Recuerda: la línea `area = ancho * largo` **no imprime nada** (solo guarda el dato); el `print()` es el que muestra el resultado.

**Completa tú.** Quita los `#` y completa la línea marcada para calcular la **densidad** (`masa / volumen`).

```python
# Quita los # y completa la linea marcada.
# masa = 18.0       # gramos
# volumen = 20.0    # mililitros
# densidad = ____   # <-- escribe aqui: masa / volumen
# print("La densidad es", densidad, "g/mL")
```

<details><summary>Respuesta esperada</summary>

~~~text
La densidad es 0.9 g/mL
~~~
</details>

## Paso 3: tipos de datos

La función `type()` te dice el **tipo** de un dato. Ejecuta y observa.

```python
print(type(7))
print(type(7 / 3))
print(type("hola"))
print(type(True))
```

~~~text
<class 'int'>
<class 'float'>
<class 'str'>
<class 'bool'>
~~~

Un entero es `int`; un número con decimales es `float`; el texto entre comillas es `str`; `True`/`False` son `bool`.

**Completa tú.** Antes de ejecutar, **predice** el tipo de cada expresión. Luego quita los `#` y comprueba. Pista: `"5+1/2"` (con comillas) es solo texto.

```python
# Predice primero, luego quita los # y comprueba.
# print(type(5 + 1 / 2))    # int o float?
# print(type("5+1/2"))      # int, float o str?
```

<details><summary>Respuesta esperada</summary>

~~~text
<class 'float'>
<class 'str'>
~~~

`5 + 1 / 2` es una operación con números, y como `1 / 2` tiene decimales, el resultado es `float`. En cambio `"5+1/2"` está entre comillas: es una cadena (`str`), texto que Python no calcula.
</details>

## Tu turno (programas completos)

### Ejercicio 1: índice de masa corporal (BMI)

Calcula el **BMI** a partir de los valores dados y muéstralo redondeado a 1 decimal con `round(valor, 1)`.
Fórmula: `BMI = (peso / estatura**2) * 703` (peso en libras, estatura en pulgadas).

```python
peso = 165.0       # libras
estatura = 68.0    # pulgadas (5 pies 8 pulgadas)
# TODO: quita los # y completa las dos lineas.
# bmi = ____
# print("El BMI es", round(bmi, 1))
```

<details><summary>Respuesta esperada</summary>

~~~text
El BMI es 25.1
~~~
</details>

### Ejercicio 2: temperatura de Celsius a Fahrenheit

Convierte una temperatura de grados Celsius a Fahrenheit con la fórmula `F = C * 9/5 + 32`, y muéstrala redondeada a 1 decimal con `round(valor, 1)`. (37 grados Celsius es la temperatura corporal normal.)

```python
celsius = 37.0     # grados Celsius
# TODO: quita los # y completa las dos lineas.
# fahrenheit = ____
# print("La temperatura es", round(fahrenheit, 1), "F")
```

<details><summary>Respuesta esperada</summary>

~~~text
La temperatura es 98.6 F
~~~
</details>

## Resumen

- Una celda de código corre Python: `+ - * / ** // %` como calculadora (`/` real, `//` cociente, `%` residuo).
- `print()` muestra resultados; una **asignación** (`nombre = dato`) guarda un dato pero no imprime.
- `type()` revela el tipo: `int`, `float`, `str`, `bool` (y `complex`, `None`).
- Un cálculo científico es un algoritmo de pocos pasos escrito en Python.
- **Sigue con la L05:** harás que el programa **pida los datos al usuario** (`input()`) y **convierta tipos**.
