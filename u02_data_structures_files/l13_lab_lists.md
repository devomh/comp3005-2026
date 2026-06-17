---
title: "Lab: Listas"
unit: "2"
lesson: 13
type: lab
tags: [python, listas, lists, mutabilidad, append, split, join, comprension-de-listas]
difficulty: introductory
duration: "60 mins"
---

# Lab: Listas

**Meta:** crear, indizar y **mutar** listas; usar sus operadores y métodos; recorrerlas con un `for`; convertir entre texto y lista con `split` y `join`; y construir listas con una **comprensión**, aplicado a una lista de mediciones. Acompaña a la nota de concepto [Listas](l13_concept_lists.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`. Donde diga **Predice**, las líneas vienen comentadas con `#` pero están completas: quita el `#` y ejecútalas para comprobar tu predicción. Donde diga **Tu turno**, además hay una parte marcada con `____` que debes **completar** antes de ejecutar. Compara con la respuesta esperada de cada bloque desplegable.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u02_data_structures_files/l13_lab_lists.ipynb)

## Preparación

Solo Python estándar: no hay que instalar nada. Corre esta celda primero.

```python
print("Listo para las listas")
```

~~~text
Listo para las listas
~~~

## Paso 1: crear, indizar y mutar

Una lista va entre corchetes `[...]` con los elementos separados por comas. Se indiza con `lista[i]` (desde `0`). A diferencia de una cadena, una lista es **mutable**: puedes cambiar un elemento, agregar con `.append()` y quitar con `del`.

```python
notas = [90, 80, 75, 100]
print(notas)        # toda la lista
print(notas[0])     # primer elemento
```

~~~text
[90, 80, 75, 100]
90
~~~

```python
notas[0] = 88         # cambiar el primer elemento
print(notas)
notas.append(65)      # agregar al final
print(notas)
del notas[1]          # eliminar el de la posicion 1 (el 80)
print(notas)
```

~~~text
[88, 80, 75, 100]
[88, 80, 75, 100, 65]
[88, 75, 100, 65]
~~~

La misma lista fue cambiando: `notas[0] = 88` reemplazó el primer valor, `.append(65)` agregó al final y `del notas[1]` quitó el elemento de la posición `1` (el `80`). En una cadena nada de esto sería posible (es inmutable).

## Paso 2: operadores y métodos

`+` concatena dos listas, `*` repite una lista e `in` pregunta si un valor está. `len` cuenta elementos, `.count()` cuenta apariciones y `.index()` da la posición de la primera.

```python
print([1, 2, 3] + [4, 5])
print([0] * 3)
print(90 in [90, 80, 75])
print(50 in [90, 80, 75])
```

~~~text
[1, 2, 3, 4, 5]
[0, 0, 0]
True
False
~~~

```python
base = [90, 80, 75, 100, 90]
print(len(base))
print(base.count(90))
print(base.index(75))
```

~~~text
5
2
2
~~~

## Paso 3: recorrer una lista con `for`

Una lista es un iterable, así que un `for` la recorre elemento por elemento. Con el patrón acumulador de la L08, sumamos sus valores:

```python
edades = [45, 33, 55, 30]
suma = 0
for e in edades:
    suma += e
print(suma)
```

~~~text
163
~~~

`e` toma cada valor por turno (`45`, `33`, `55`, `30`) y el acumulador `suma` los va agregando: `45 + 33 + 55 + 30 = 163`.

## Paso 4: de cadena a lista y de vuelta (`split` y `join`)

`str.split(sep)` parte una cadena en una **lista** de subcadenas; `sep.join(lista)` une una lista de cadenas en una **cadena**.

```python
print('7.1, 6.8, 7.4'.split(','))
print('-'.join(['A', 'T', 'G']))
```

~~~text
['7.1', ' 6.8', ' 7.4']
A-T-G
~~~

`split(',')` cortó por las comas (como había un espacio tras cada coma, quedó pegado: `' 6.8'`). `'-'.join([...])` unió los tres elementos con un guion en medio. `split` devuelve una lista; `join` devuelve una cadena.

## Paso 5: comprensión de listas (predice primero)

Una **comprensión** `[<expr> for <var> in <iterable> if <condicion>]` construye una lista en una línea. **Predice** qué imprime antes de ejecutar; luego quita los `#` y comprueba.

```python
# Predice la salida, luego quita los # y comprueba.
# edades = [45, 33, 55, 30, 25, 33, 25, 40]
# intervalo = [e for e in edades if e >= 35 and e <= 45]
# print(intervalo)
```

<details><summary>Respuesta esperada</summary>

~~~text
[45, 40]
~~~

La comprensión recorre `edades` y, con el filtro `if e >= 35 and e <= 45`, solo agrega los elementos **entre 35 y 45**: de la lista, esos son `45` y `40`, así que el resultado es `[45, 40]`. Equivale a un `for` con un `if` y un `.append()`, pero en una sola línea.
</details>

## Tu turno (programa completo)

### Ejercicio: promedio de una lista de mediciones

Tienes varias lecturas de pH como un solo texto, separadas por comas. Conviértelas en una lista de números con `split` + una comprensión que aplique `float()` a cada parte, y calcula el promedio. Completa las partes que faltan.

```python
texto = '7.1,6.8,7.4,7.0,6.9'
# TODO: quita los # y completa el split y la conversion a float.
# lecturas = [float(x) for x in texto.____(',')]
# promedio = sum(lecturas) / len(lecturas)
# print('Lecturas:', lecturas)
# print('Promedio: {:.2f}'.format(promedio))
```

<details><summary>Respuesta esperada</summary>

~~~text
Lecturas: [7.1, 6.8, 7.4, 7.0, 6.9]
Promedio: 7.04
~~~

La parte que falta es `texto.split(',')`, que parte el texto en `['7.1', '6.8', '7.4', '7.0', '6.9']`; la comprensión `[float(x) for x in ...]` convierte cada subcadena en un número. Luego `sum(lecturas) / len(lecturas)` da el promedio: `35.2 / 5 = 7.04`. Este "texto separado por comas -> lista de números" es justo lo que harás con archivos CSV en el Módulo 3.
</details>

## Resumen

- Una lista `[a, b, c]` guarda muchos valores en orden; se indiza con `lista[i]` y es **mutable** (`lista[i]=x`, `.append`, `del`), a diferencia de la cadena.
- Operadores: `+` concatena, `*` repite, `in` pregunta pertenencia. Funciones/métodos: `len`, `.append`, `.count`, `.index`, `del`.
- `for e in lista` recorre la lista elemento por elemento (es un iterable).
- `split` parte una cadena en una **lista**; `join` une una lista de cadenas en una **cadena**.
- Una **comprensión** `[expr for x in it (if cond)]` construye una lista en una línea (transforma y filtra).
- **Sigue con la L14** (Módulo 2): los **diccionarios**, que guardan datos por **nombre** (clave) en vez de por posición.
