---
title: "Lab: Cadenas de caracteres"
unit: "2"
lesson: 12
type: lab
tags: [python, cadenas, strings, indexacion, slicing, metodos, escape, format, recorrido]
difficulty: introductory
duration: "60 mins"
---

# Lab: Cadenas de caracteres

**Meta:** crear y combinar cadenas, indizarlas y rebanarlas (*slicing*), transformarlas con métodos, dar formato con `.format()` y recorrerlas con un `for` para contar las bases de una secuencia de ADN. Acompaña a la nota de concepto [Cadenas de caracteres](l12_concept_strings.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`. Donde diga **Predice**, las líneas vienen comentadas con `#` pero están completas: quita el `#` y ejecútalas para comprobar tu predicción. Donde diga **Tu turno**, además hay una parte marcada con `____` que debes **completar** antes de ejecutar. Compara con la respuesta esperada de cada bloque desplegable.

## Preparación

Solo Python estándar: no hay que instalar nada. Corre esta celda primero.

```python
print("Listo para las cadenas")
```

~~~text
Listo para las cadenas
~~~

## Paso 1: crear y combinar cadenas

Una cadena va entre comillas simples o dobles. Para incluir una comilla, usa el otro tipo o escápala con `\'`. El operador `+` concatena (une) y `*` repite. Las cadenas son **inmutables**: los métodos devuelven una cadena **nueva** y no cambian la original.

```python
print("Usando 'comilla simple' en la cadena")
print('Hola' + ' ' + 'mundo')
print('ADN' * 3)
```

~~~text
Usando 'comilla simple' en la cadena
Hola mundo
ADNADNADN
~~~

Comprueba la inmutabilidad: `.upper()` produce una cadena nueva, pero la original no cambia.

```python
adn = 'atgcat'
print(adn.upper())   # cadena NUEVA en mayusculas
print(adn)           # la original NO cambio
```

~~~text
ATGCAT
atgcat
~~~

## Paso 2: indización y subcadenas (*slicing*)

Cada carácter tiene un índice (desde `0`); los negativos cuentan desde el final. Una porción `str[a:b]` llega hasta `b-1` (igual que `range`); `str[::-1]` invierte la cadena.

```python
adn = 'ATGCAT'
print(adn[0])      # primer caracter
print(adn[-1])     # ultimo caracter
print(adn[0:3])    # subcadena: indices 0,1,2
print(adn[2:])     # desde el indice 2 hasta el final
print(adn[::-1])   # invertida
```

~~~text
A
T
ATG
GCAT
TACGTA
~~~

`adn[0]` es `'A'` y `adn[-1]` es `'T'`; `adn[0:3]` es `'ATG'` (el índice `3` no se incluye) y `adn[::-1]` da la cadena al revés.

## Paso 3: métodos de cadena

Los métodos se llaman con un punto: `cadena.metodo(...)`. `len(s)` cuenta caracteres; `.count(sub)` cuenta apariciones; `.find(sub)` da la posición o **`-1`** si no está; `.strip()` quita espacios de los extremos; `.replace(a, b)` reemplaza; `.upper()` pasa a mayúsculas.

```python
print(len('ATGCAT'))
print('ATGCAT'.count('A'))
print('ATGCAT'.find('GC'))
print('ATGCAT'.find('Z'))
print('  hola  '.strip())
print('ATGCAT'.replace('A', 'a'))
```

~~~text
6
2
2
-1
hola
aTGCaT
~~~

Fíjate en `find('Z')`: devuelve `-1` porque no hay ninguna `Z`. Y `replace('A', 'a')` devuelve una cadena nueva con las `A` cambiadas; la original seguiría intacta.

## Paso 4: escape y formato (predice primero)

`\n` es un salto de línea y `\t` un tabulador. `.format()` rellena las marcas `{}` (o `{0}`, `{1}`) de una cadena, y `{:.2f}` da formato a un decimal. **Predice** la salida antes de ejecutar; luego quita los `#` y comprueba.

```python
# Predice la salida, luego quita los # y comprueba.
# print('linea1\nlinea2')
# print('col1\tcol2')
# print('{} mide {:.2f} cm'.format('muestra', 3.14159))
# print('Muestra {0}: {1} mg/mL'.format('A', 25))
```

<details><summary>Respuesta esperada</summary>

~~~text
linea1
linea2
col1	col2
muestra mide 3.14 cm
Muestra A: 25 mg/mL
~~~

`\n` parte la salida en dos líneas y `\t` deja un espacio de tabulación entre `col1` y `col2`. En el `.format()`, `{:.2f}` muestra `3.14159` con dos decimales (`3.14`), y `{0}`/`{1}` insertan los argumentos por posición.
</details>

## Tu turno (programa completo)

### Ejercicio: contenido GC de una secuencia de ADN

Una secuencia de ADN es una cadena de bases (`A`, `T`, `G`, `C`). El **contenido GC** es el porcentaje de bases que son `G` o `C`. Recorre la secuencia con un `for`, cuenta las bases `G` y `C` con un contador, y reporta el porcentaje con `.format()`.

```python
secuencia = 'ATGCGCGATTAC'
gc = 0
# TODO: quita los # y completa el for y la condicion.
# for base in secuencia:
#     if base == 'G' or base ____ 'C':
#         gc ____
# total = len(secuencia)
# print('Bases G+C:', gc, 'de', total)
# print('Contenido GC: {:.1f}%'.format(gc / total * 100))
```

<details><summary>Respuesta esperada</summary>

~~~text
Bases G+C: 6 de 12
Contenido GC: 50.0%
~~~

Las partes que faltan son `base == 'C'` (en la condición `if base == 'G' or base == 'C':`) y `gc += 1` (el contador). La secuencia `'ATGCGCGATTAC'` tiene `12` bases; `6` de ellas son `G` o `C`, así que el contenido GC es `6 / 12 * 100 = 50.0%`. Junta un contador (L08), un `for` que recorre la cadena (L11), un `if` con `or` (L06) y `.format()` para el reporte.
</details>

## Resumen

- Las cadenas se crean con comillas `'...'`/`"..."`; `+` concatena, `*` repite; son **inmutables** (los métodos devuelven una cadena nueva).
- `str[i]` indiza (desde `0`; `str[-1]` es el último); `str[a:b]` rebana hasta `b-1`; `str[::-1]` invierte.
- Métodos útiles: `len`, `.upper`, `.lower`, `.strip`, `.replace`, `.count`, `.find` (devuelve `-1` si no encuentra).
- `\n` salta de línea y `\t` tabula; `.format()` inserta valores en las marcas `{}` / `{0}` y formatea decimales con `{:.2f}`.
- Una cadena es un **iterable**: `for c in cadena` la recorre carácter por carácter (como contar el contenido GC de un ADN).
- **Sigue con la L13** (Módulo 2): las **listas**, la estructura para guardar muchos valores (y `cadena.split()`, que parte un texto en una lista).
