---
title: "Lab: Ciclos for"
unit: "2"
lesson: 11
type: lab
tags: [python, ciclos, for, range, ciclos-anidados, break, continue]
difficulty: introductory
duration: "60 mins"
---

# Lab: Ciclos for

**Meta:** escribir ciclos `for` con `range`, recorrer ciclos anidados, alterar el flujo con `break` y `continue`, y aplicar un `for` a una dilución en serie. Acompaña a la nota de concepto [Ciclos for](l11_concept_for_loops.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`. Donde diga **Predice**, las líneas vienen comentadas con `#` pero están completas: quita el `#` y ejecútalas para comprobar tu predicción. Donde diga **Tu turno**, además hay una parte marcada con `____` que debes **completar** antes de ejecutar. Compara con la respuesta esperada de cada bloque desplegable.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u02_data_structures_files/l11_lab_for_loops.ipynb)

## Preparación

Solo Python estándar: no hay que instalar nada. Corre esta celda primero.

```python
print("Listo para los ciclos for")
```

~~~text
Listo para los ciclos for
~~~

## Paso 1: el `for` y `range`

Un `for` ejecuta su bloque una vez por cada valor que le entrega un iterable. Con `range(1, n + 1)` recorres los primeros `n` números naturales: no hay que inicializar ni incrementar un contador a mano. En Colab pedirías `n` con `input()`; aquí usamos un valor fijo:

~~~python
n = int(input("Ingrese el numero n: "))
~~~

```python
n = 5         # en Colab lo pedirias con input()
for i in range(1, n + 1):
    print(i)
```

~~~text
1
2
3
4
5
~~~

`range(1, n + 1)` (es decir, `range(1, 6)`) entrega `1, 2, 3, 4, 5`, e `i` toma cada uno por turno. Recuerda que `stop` no se incluye: por eso se escribe `n + 1` para llegar hasta `n`.

## Paso 2: las tres formas de `range`

`range` produce los valores según cuántos argumentos le des. Corre cada celda y observa la diferencia:

```python
for i in range(5):
    print(i)
```

~~~text
0
1
2
3
4
~~~

```python
for i in range(5, 10):
    print(i)
```

~~~text
5
6
7
8
9
~~~

```python
for i in range(0, 10, 2):
    print(i)
```

~~~text
0
2
4
6
8
~~~

`range(5)` empieza en `0` y llega hasta `4`; `range(5, 10)` va de `5` a `9`; `range(0, 10, 2)` salta de `2` en `2`. En las tres, `stop` queda fuera.

## Paso 3: ciclos anidados (predice primero)

En un `for` anidado, el bloque interior corre **completo** por cada vuelta del exterior. **Predice** cuántas líneas imprime esta tabla de multiplicar antes de ejecutar. Luego quita los `#` y comprueba.

```python
# Predice cuantas lineas imprime, luego quita los # y comprueba.
# for i in range(1, 4):
#     for j in range(1, 4):
#         print(i, "x", j, "=", i * j)
```

<details><summary>Respuesta esperada</summary>

~~~text
1 x 1 = 1
1 x 2 = 2
1 x 3 = 3
2 x 1 = 2
2 x 2 = 4
2 x 3 = 6
3 x 1 = 3
3 x 2 = 6
3 x 3 = 9
~~~

El exterior da `3` vueltas (`i` = 1, 2, 3) y por cada una el interior da otras `3` (`j` = 1, 2, 3), así que el `print` corre `3 x 3 = 9` veces. Para cada `i`, el `for` interior recorre todos los valores de `j` antes de que `i` avance.
</details>

## Paso 4: `break` y `continue`

`continue` salta al siguiente valor (omite el resto del bloque en esa vuelta); `break` termina el ciclo de inmediato. Primero, saltar los impares con `continue`:

```python
for i in range(1, 11):
    if i % 2 == 1:
        continue
    print(i)
```

~~~text
2
4
6
8
10
~~~

Cuando `i` es impar, `continue` salta el `print` y pasa al siguiente valor, así que solo se imprimen los pares. Ahora, detenerse temprano con `break`: sumar `1, 2, 3, ...` hasta que el total pase de `20`.

```python
suma = 0
for i in range(1, 100):
    suma += i
    if suma > 20:
        print("Se detuvo en i =", i, "con suma =", suma)
        break
```

~~~text
Se detuvo en i = 6 con suma = 21
~~~

Aunque `range(1, 100)` ofrecía valores hasta `99`, el `break` corta el ciclo en `i = 6`, cuando la suma llega a `21`.

## Tu turno (programa completo)

### Ejercicio: una dilución en serie 1:2

Una muestra empieza en `64.0 mg/mL` y se diluye **a la mitad** en cada uno de `5` pasos. Como el número de pasos se conoce de antemano, un `for` es la elección natural. Completa el ciclo: en cada paso, divide `concentracion` entre `2` (usa `/=`) e imprime el paso.

```python
concentracion = 64.0
pasos = 5
# TODO: quita los # y completa el for de la dilucion 1:2.
# for paso in range(1, pasos + 1):
#     concentracion ____            # diluir a la mitad
#     print("Paso", paso, "->", concentracion, "mg/mL")
# print("Concentracion final:", concentracion, "mg/mL")
```

<details><summary>Respuesta esperada</summary>

~~~text
Paso 1 -> 32.0 mg/mL
Paso 2 -> 16.0 mg/mL
Paso 3 -> 8.0 mg/mL
Paso 4 -> 4.0 mg/mL
Paso 5 -> 2.0 mg/mL
Concentracion final: 2.0 mg/mL
~~~

La línea que falta es `concentracion /= 2` (dividir a la mitad). Con `range(1, pasos + 1)` el ciclo da exactamente `5` vueltas: 64 -> 32 -> 16 -> 8 -> 4 -> 2. A diferencia del cultivo celular de la L08 (donde no sabías cuántas generaciones harían falta y usabas `while`), aquí el número de pasos es fijo y conocido, así que `for` encaja mejor.
</details>

## Resumen

- `for <variable> in <iterable>:` ejecuta el bloque una vez por cada valor que el iterable entrega; `range` es la fuente de valores más común.
- `range(n)` da `0..n-1`; `range(start, stop)` da `start..stop-1`; `range(start, stop, step)` salta de `step` en `step`. **`stop` no se incluye.**
- Elige `for` cuando el número de vueltas se conoce de antemano; `while` cuando depende de una condición.
- En un ciclo **anidado**, el bloque interior corre completo por cada vuelta del exterior.
- `break` termina el ciclo de inmediato; `continue` salta al siguiente valor. Actúan sobre el ciclo más interno.
- **Sigue con la L12** (Módulo 2): las **cadenas** de caracteres, que recorrerás con `for` carácter por carácter.
