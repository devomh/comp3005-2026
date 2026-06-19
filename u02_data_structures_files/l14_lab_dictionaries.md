---
title: "Lab: Diccionarios"
unit: "2"
lesson: 14
type: lab
tags: [python, diccionarios, dict, clave-valor, keys, values, items, get, conteo]
difficulty: introductory
duration: "60 mins"
---

# Lab: Diccionarios

**Meta:** crear un diccionario y acceder por **clave**; modificar y agregar pares; usar sus métodos (`len`, `.keys()`, `.values()`, `.items()`, `.get()`) y el operador `in`; recorrerlo con un `for`; y armar una **tabla de frecuencias** contando las bases de una cadena de ADN. Acompaña a la nota de concepto [Diccionarios](l14_concept_dictionaries.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`. Donde diga **Predice**, las líneas vienen comentadas con `#` pero están completas: quita el `#` y ejecútalas para comprobar tu predicción. Donde diga **Tu turno**, además hay una parte marcada con `____` dentro del bloque comentado que debes **completar** al quitar los `#`. Compara con la respuesta esperada de cada bloque desplegable.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u02_data_structures_files/l14_lab_dictionaries.ipynb)

## Preparación

Solo Python estándar: no hay que instalar nada. Corre esta celda primero.

```python
print("Listo para los diccionarios")
```

~~~text
Listo para los diccionarios
~~~

## Paso 1: crear, acceder, modificar y agregar

Un diccionario va entre llaves `{...}` con pares `clave: valor`. Se accede a un valor por su **clave** con `d[clave]`. La instrucción `d[clave] = valor` **modifica** el valor si la clave ya existe, o **crea** el par si no existe.

```python
masas = {'H': 1.008, 'C': 12.011, 'O': 15.999}
print(masas)          # todo el diccionario
print(masas['C'])     # el valor asociado a la clave 'C'
```

~~~text
{'H': 1.008, 'C': 12.011, 'O': 15.999}
12.011
~~~

```python
masas['N'] = 14.007     # 'N' no existia -> agrega el par nuevo
masas['H'] = 1.0079     # 'H' ya existia -> modifica su valor
print(masas)
```

~~~text
{'H': 1.0079, 'C': 12.011, 'O': 15.999, 'N': 14.007}
~~~

Se agregó `'N'` y cambió el valor de `'H'` (de `1.008` a `1.0079`). La misma línea `d[clave] = valor` sirve para ambas cosas: el que `'N'` no existiera la convirtió en "agregar"; el que `'H'` ya existiera la convirtió en "modificar".

## Paso 2: métodos y el operador `in`

`len` cuenta los pares; `.keys()`, `.values()` e `.items()` dan las claves, los valores y los pares. `.get(clave, default)` lee **sin error**: devuelve el `default` si la clave no está. El operador `in` pregunta si una **clave** existe.

```python
print(len(masas))
print(masas.keys())
print(masas.values())
print(masas.items())
print(masas.get('Na', 0.0))   # clave ausente -> default
print(masas.get('C', 0.0))    # clave presente -> su valor
print('C' in masas)
print('Na' in masas)
```

~~~text
4
dict_keys(['H', 'C', 'O', 'N'])
dict_values([1.0079, 12.011, 15.999, 14.007])
dict_items([('H', 1.0079), ('C', 12.011), ('O', 15.999), ('N', 14.007)])
0.0
12.011
True
False
~~~

`.keys()`, `.values()` e `.items()` te dan las tres "vistas" del diccionario (las claves, los valores y los pares). `get('Na', 0.0)` devolvió `0.0` porque `'Na'` no es una clave (con `masas['Na']` habrías tenido un `KeyError`). `'C' in masas` es `True` y `'Na' in masas` es `False`: `in` mira las **claves**.

## Paso 3: recorrer un diccionario con `for`

Un `for k in d` recorre las **claves** (una por vuelta); con la clave accedes al valor. Para obtener clave y valor a la vez, recorre `d.items()`.

```python
for sym in masas:
    print(sym, masas[sym])
```

~~~text
H 1.0079
C 12.011
O 15.999
N 14.007
~~~

```python
for sym, m in masas.items():
    print('Elemento: {0:>2} | Masa: {1}'.format(sym, m))
```

~~~text
Elemento:  H | Masa: 1.0079
Elemento:  C | Masa: 12.011
Elemento:  O | Masa: 15.999
Elemento:  N | Masa: 14.007
~~~

El primer `for` usa la clave `sym` para leer `masas[sym]`. El segundo desempaca cada par de `.items()` en `sym` y `m`, y los alinea con `.format()`.

## Paso 4: `.get()` con default (predice primero)

`.get(clave, default)` nunca falla. **Predice** qué imprime cada línea antes de ejecutar; luego quita los `#` y comprueba.

```python
# Predice la salida, luego quita los # y comprueba.
# print(masas.get('Na', 0.0))   # 'Na' NO es una clave
# print(masas.get('O', 0.0))    # 'O' SI es una clave
```

<details><summary>Respuesta esperada</summary>

~~~text
0.0
15.999
~~~

`masas.get('Na', 0.0)` devuelve el default `0.0` porque `'Na'` no está en el diccionario; `masas.get('O', 0.0)` devuelve `15.999`, el valor real de la clave `'O'`. La diferencia con `masas['Na']` es que `.get()` no lanza `KeyError`.
</details>

## Tu turno (programa completo)

### Ejercicio: tabla de frecuencias de las bases de una cadena de ADN

Tienes una cadena de ADN. Cuenta cuántas veces aparece cada base (`A`, `T`, `G`, `C`) y guarda el resultado en un diccionario, usando el idioma `conteo[base] = conteo.get(base, 0) + 1`. Completa la parte que falta.

```python
adn = 'ATGCATGCAT'
conteo = {}
# TODO: quita los # y completa con el idioma get(base, 0) + 1.
# for base in adn:
#     conteo[base] = conteo.____(base, 0) + 1
# print(conteo)
```

<details><summary>Respuesta esperada</summary>

~~~text
{'A': 3, 'T': 3, 'G': 2, 'C': 2}
~~~

La parte que falta es `conteo.get(base, 0)`: por cada `base` de la cadena, toma el conteo que llevas (o `0` la primera vez), súmale `1` y guárdalo en `conteo[base]`. Al recorrer `'ATGCATGCAT'` quedan `A` y `T` tres veces, y `G` y `C` dos veces. Esta "tabla de frecuencias" es justo lo que hará `value_counts` de Pandas en el Módulo 3.
</details>

## Resumen

- Un diccionario `{clave: valor}` guarda datos por **clave** (con sentido), no por posición; se accede con `d[clave]`.
- `d[clave] = valor` **modifica** si la clave existe y **agrega** si no. Leer una clave ausente con `d[clave]` da `KeyError`; `.get(clave, default)` lo evita.
- Métodos: `len`, `.keys()`, `.values()`, `.items()`, `.get()`. Operador: `clave in d` (mira las **claves**).
- `for k in d` recorre las claves (sin orden garantizado); `for k, v in d.items()` da clave y valor.
- El idioma `conteo[k] = conteo.get(k, 0) + 1` arma una **tabla de frecuencias**.
- **Sigue con la L15** (Módulo 2): las **funciones** (`def`), para empaquetar y reutilizar tu código.
