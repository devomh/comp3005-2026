---
title: "Lab: Condicionales"
unit: "1"
lesson: 7
type: lab
tags: [python, condicionales, if-else, elif, anidados, cortocircuito]
difficulty: introductory
duration: "60 mins"
---

# Lab: Condicionales

**Meta:** escribir decisiones completas con `if`/`else`, encadenar casos con `elif`, anidar condicionales y entender el cortocircuito de `and`/`or`. Acompaña a la nota de concepto [Condicionales](l07_concept_conditionals.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`. Donde diga **Predice** o **Tu turno**, las líneas vienen comentadas con `#`: quita el `#` **y completa lo que falte** (las condiciones marcadas con `____`). Compara con la respuesta esperada de cada bloque desplegable.

## Preparación

Solo Python estándar: no hay que instalar nada. Corre esta celda primero.

```python
print("Listo para los condicionales")
```

~~~text
Listo para los condicionales
~~~

## Paso 1: dos caminos con `if ... else`

El `else` le da una alternativa al `if`: su bloque se ejecuta cuando la condición del `if` es `False`. Este programa decide si un número es par o impar (`nro % 2` es el residuo de dividir entre 2).

En Colab pedirías el número al usuario (NO ejecutes esto aquí; solo obsérvalo):

~~~python
nro = int(input("Ingrese un numero entero: "))
~~~

Aquí usamos un valor fijo para que el notebook corra completo. Ejecuta:

```python
nro = 8       # en Colab lo pedirias con input()
if nro % 2 == 0:
    print("Es par")
else:
    print("No es par")
```

~~~text
Es par
~~~

Como `8 % 2` es `0`, la condición `nro % 2 == 0` es `True` y corre el bloque del `if`. Cambia `nro` a `7` y vuelve a ejecutar: ahora la condición es `False` y corre el bloque del `else` (`No es par`).

## Paso 2: varios casos con `if ... elif ... else`

Cuando hay más de dos posibilidades, encadenas con `elif`. Python evalúa las condiciones en orden y ejecuta el bloque del **primer** `True`; descarta el resto. Este programa clasifica el signo de un número.

```python
n = -3.0      # en Colab lo pedirias con input()
if n > 0:
    print("Positivo")
elif n == 0:
    print("Cero")
else:
    print("Negativo")
```

~~~text
Negativo
~~~

Con `n = -3.0`, la primera condición (`n > 0`) es `False`, la segunda (`n == 0`) también, así que se ejecuta el `else`. Prueba con `5.0` y con `0.0` para recorrer las otras dos ramas.

## Paso 3: una decisión dentro de otra (anidados)

**Predice primero.** Este programa resuelve `a*x + b = 0` con el análisis completo: si `a` no es cero hay solución; si `a` es cero, un segundo `if` distingue dos casos según `b`. Antes de ejecutar, **predice** qué imprime con `a = 0.0` y `b = 2.0`. Luego quita los `#` y comprueba.

```python
# Predice que imprime, luego quita los # y comprueba.
# a = 0.0
# b = 2.0
# if a != 0:
#     x = -b / a
#     print("Solucion x =", x)
# else:
#     if b != 0:
#         print("La ecuacion no tiene solucion")
#     else:
#         print("La ecuacion tiene infinitas soluciones")
```

<details><summary>Respuesta esperada</summary>

~~~text
La ecuacion no tiene solucion
~~~

Como `a` vale `0.0`, la condición externa `a != 0` es `False` y entramos al `else` externo. Allí, el segundo `if` evalúa `b != 0`: como `b` vale `2.0` (distinto de cero), imprime `La ecuacion no tiene solucion`. Si `b` también fuera `0.0`, imprimiría `La ecuacion tiene infinitas soluciones`.
</details>

## Paso 4: cortocircuito de `and` y `or`

Python deja de evaluar una expresión booleana en cuanto el resultado está decidido: `and` para en el primer `False`, `or` en el primer `True`. Eso permite **proteger** una operación riesgosa. Ejecuta y fíjate en que **no hay error** aunque `a` valga `0`:

```python
a = 0
b = 2

if a != 0 and 1/a > 0:
    print("El reciproco de a es positivo.")

if (b > 0) or (b/a > 0):
    print("b es positivo o b y a tienen el mismo signo.")
```

~~~text
b es positivo o b y a tienen el mismo signo.
~~~

En el primer `if`, `a != 0` es `False`; como es un `and`, Python no llega a evaluar `1/a > 0` (que habría fallado por división entre cero). En el segundo, `b > 0` es `True`; como es un `or`, no evalúa `b/a > 0`. Por eso solo se imprime la segunda línea. Poner la condición protectora antes del `and` es un patrón muy útil.

## Tu turno (programa completo)

### Ejercicio: clasificar un pH

Una medición de pH se clasifica como `acida` si es menor que 7, `neutra` si es exactamente 7, y `basica` si es mayor que 7. Completa la cadena `if/elif/else` para clasificar `ph = 8.2`.

```python
ph = 8.2
# TODO: quita los # y completa las condiciones.
# if ____:
#     print("La muestra es acida")
# elif ____:
#     print("La muestra es neutra")
# else:
#     print("La muestra es basica")
```

<details><summary>Respuesta esperada</summary>

~~~text
La muestra es basica
~~~

Las condiciones son `ph < 7` (acida) y `ph == 7` (neutra); el `else` cubre el resto (basica). Como `8.2 < 7` es `False` y `8.2 == 7` es `False`, se ejecuta el `else` y se imprime `La muestra es basica`. Con `ph = 6.5` imprimiría `La muestra es acida`, y con `ph = 7.0`, `La muestra es neutra`.
</details>

## Resumen

- `if ... else`: dos caminos; el bloque del `else` corre cuando la condición del `if` es `False`.
- `if ... elif ... else`: varios casos en orden; gana el **primer** `True`; el `else` recoge el resto.
- **Condicionales anidados:** una decisión dentro del bloque de otra; cada nivel añade su indentación.
- **Cortocircuito:** `and` para en el primer `False`, `or` en el primer `True`; sirve para proteger una operación (validar antes de dividir).
- **Sigue con la L08:** con `while` pasarás de decidir **si** se ejecuta un bloque a decidir **cuántas veces** se repite.
