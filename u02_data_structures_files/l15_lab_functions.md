---
title: "Lab: Funciones"
unit: "2"
lesson: 15
type: lab
tags: [python, funciones, def, return, parametros, argumentos, reutilizacion, alcance-local]
difficulty: introductory
duration: "60 mins"
---

# Lab: Funciones

**Meta:** definir funciones con `def`; distinguir declararlas de llamarlas; usar parámetros, argumentos y `return`; reconocer que sin `return` una función devuelve `None` y que sus variables son **locales**; y escribir una función científica reutilizable. Acompaña a la nota de concepto [Funciones](l15_concept_functions.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`. Donde diga **Predice**, las líneas vienen comentadas con `#` pero están completas: quita el `#` y ejecútalas para comprobar tu predicción. Donde diga **Tu turno**, además hay una parte marcada con `____` dentro del bloque comentado que debes **completar** al quitar los `#`. Compara con la respuesta esperada de cada bloque desplegable.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u02_data_structures_files/l15_lab_functions.ipynb)

## Preparación

Solo Python estándar: no hay que instalar nada. Corre esta celda primero.

```python
print("Listo para las funciones")
```

~~~text
Listo para las funciones
~~~

## Paso 1: definir y llamar (reutilizar con distintos argumentos)

Una función se declara con `def nombre(parametros):` y un bloque indentado; `return` devuelve un valor. **Definir** la función no ejecuta el cuerpo: el cuerpo corre cuando la **llamas** con argumentos.

```python
def molaridad(moles, litros):
    return moles / litros

print(molaridad(0.5, 2))
print(molaridad(3, 4))
print(molaridad(1, 8))
```

~~~text
0.25
0.75
0.125
~~~

La misma función `molaridad` se llamó tres veces con argumentos distintos. Eso es lo que la hace útil: la escribes **una vez** y la **reutilizas**. Definir (con `def`) y llamar (con `molaridad(...)`) son pasos distintos: si solo la defines y nunca la llamas, no se imprime nada.

## Paso 2: parámetros, argumentos y un `for` en el cuerpo

Los **parámetros** son los nombres en la definición; los **argumentos**, los valores en la llamada (por posición). El cuerpo puede tener varias instrucciones, incluso un `for`. Aquí encapsulamos el acumulador de la L13:

```python
def promedio(notas):
    suma = 0
    for x in notas:
        suma += x
    return suma / len(notas)

print(promedio([80, 90, 100]))
```

~~~text
90.0
~~~

Al llamar `promedio([80, 90, 100])`, el parámetro `notas` toma la lista; el cuerpo la recorre, acumula `suma` y devuelve `(80 + 90 + 100) / 3 = 90.0`. La función sirve para **cualquier** lista de notas.

## Paso 3: `return` vs. imprimir (y el `None`)

Una función puede **imprimir** (un efecto) sin **devolver** nada. Si no tiene `return`, devuelve `None`.

```python
def gc(adn):
    return (adn.count('G') + adn.count('C')) / len(adn) * 100

print(gc('ATGCATGCAT'))
print(gc('GCGCGC'))
```

~~~text
40.0
100.0
~~~

`gc` **devuelve** el porcentaje de G+C: para `'ATGCATGCAT'` (2 `G` y 2 `C` de 10 bases) da `40.0`, el mismo dato que contaste a mano en la L14. Ahora compara con una función que **imprime** pero no devuelve:

```python
def encabezado(texto):
    print('===', texto, '===')

r = encabezado('Reporte')
print(r)
```

~~~text
=== Reporte ===
None
~~~

`encabezado` imprimió su línea pero no tiene `return`, así que devolvió `None`: por eso `r` quedó en `None`. **Imprimir** (mostrar algo) no es lo mismo que **devolver** (entregar un valor reutilizable).

## Paso 4: variables locales (predice primero)

Las variables creadas dentro de una función son **locales**: no existen afuera. **Predice** qué pasaría con la línea comentada antes de razonarlo; déjala comentada (daría un error).

```python
def cuadrado(n):
    resultado = n * n
    return resultado

print(cuadrado(5))
# print(resultado)   # predice: que pasa si quitas el # de esta linea
```

~~~text
25
~~~

<details><summary>Respuesta esperada</summary>

`cuadrado(5)` imprime `25`. Pero si quitaras el `#` de `print(resultado)`, el programa daría un `NameError`: la variable `resultado` vive **dentro** de la función `cuadrado` y no existe en el programa principal. Para sacar un valor de una función se usa `return`, no una variable local. (Por eso la dejamos comentada: un error vivo detendría el resto del notebook.)
</details>

## Tu turno (programa completo)

### Ejercicio: escribe una función de dilución

Una dilución divide la concentración entre un factor. Escribe una función `diluir(conc, factor)` que **devuelva** `conc / factor`, y llámala. Completa las partes que faltan.

```python
# TODO: quita los # y completa la definicion (el operador que falta).
# def diluir(conc, factor):
#     return conc ____ factor
# print(diluir(80.0, 4))
```

<details><summary>Respuesta esperada</summary>

~~~text
20.0
~~~

La parte que falta es el operador de división `/`: `return conc / factor`. Al llamar `diluir(80.0, 4)`, el parámetro `conc` toma `80.0` y `factor` toma `4`, así que devuelve `80.0 / 4 = 20.0`. Como cualquier función, la puedes reutilizar: `diluir(64.0, 2)` daría `32.0`.
</details>

## Resumen

- Una función se **declara** con `def nombre(parametros):` + bloque indentado, y se **llama** con `nombre(argumentos)`. Declarar no ejecuta el cuerpo; llamar sí (cuantas veces quieras).
- **Parámetro** = el nombre en la definición; **argumento** = el valor en la llamada (por posición).
- `return` devuelve un valor y termina la función; **sin** `return`, devuelve `None`. **Imprimir** no es **devolver**.
- Las variables creadas dentro de una función son **locales**: no existen afuera (para sacar un valor, `return`).
- Una función con nombre se **reutiliza** con distintos argumentos -- no repites código (`gc`, `molaridad`, `diluir`).
- **Sigue con la L16** (Módulo 2): los **archivos de texto**, donde escribirás funciones que procesan archivos.
