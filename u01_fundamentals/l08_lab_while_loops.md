---
title: "Lab: Ciclos while"
unit: "1"
lesson: 8
type: lab
tags: [python, ciclos, while, contador, acumulador, asignacion-con-operador, terminacion]
difficulty: introductory
duration: "60 mins"
---

# Lab: Ciclos while

**Meta:** escribir ciclos `while` con los patrones contador y acumulador, usar las asignaciones con operador y reconocer cuándo un ciclo no termina. Acompaña a la nota de concepto [Ciclos while](l08_concept_while_loops.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`. Donde diga **Predice** o **Tu turno**, las líneas vienen comentadas con `#`: quita el `#` **y completa lo que falte**. Los bloques marcados **"NO lo ejecutes"** son solo para leer. Compara con la respuesta esperada de cada bloque desplegable.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u01_fundamentals/l08_lab_while_loops.ipynb)

## Preparación

Solo Python estándar: no hay que instalar nada. Corre esta celda primero.

```python
print("Listo para los ciclos while")
```

~~~text
Listo para los ciclos while
~~~

## Paso 1: patrón contador

Un contador inicializa una variable de control, la compara en la condición y la incrementa cada vuelta. Este programa imprime los primeros `n` números naturales. En Colab pedirías `n` con `input()`; aquí usamos un valor fijo:

~~~python
n = int(input("Ingrese el numero n: "))
~~~

```python
n = 5         # en Colab lo pedirias con input()
i = 1
while i <= n:
    print(i)
    i = i + 1
print("Termine!")
```

~~~text
1
2
3
4
5
Termine!
~~~

`i` empieza en `1` y, mientras `i <= 5` sea `True`, imprime `i` y le suma `1`. Cuando `i` llega a `6`, la condición es `False` y el ciclo termina, así que `Termine!` se imprime una sola vez.

## Paso 2: patrón acumulador

Un acumulador inicializa una variable **fuera** del ciclo y le va agregando un valor **dentro**. Este programa suma los primeros `n` naturales:

```python
n = 5
suma = 0
i = 1
while i <= n:
    suma = suma + i
    i = i + 1
print("La suma es:", suma)
```

~~~text
La suma es: 15
~~~

Hay dos variables que cambian: el contador `i` (controla cuándo parar) y el acumulador `suma` (guarda `0+1+2+3+4+5 = 15`). El `print` va **después** del ciclo, cuando la suma ya está completa.

## Paso 3: asignaciones con operador (predice primero)

`i += 1` es un atajo de `i = i + 1`. Funciona con `+ - * / // % **`. **Predice** el valor final de `i` antes de ejecutar (cuidado: `/` siempre da `float`). Luego quita los `#` y comprueba.

```python
# Predice el valor final, luego quita los # y comprueba.
# i = 2
# i += 3    # i = 5
# i -= 1    # i = 4
# i *= 2    # i = 8
# i /= 8    # i = 1.0   (la division / da float)
# i **= 3   # i = 1.0
# i //= 3   # i = 0.0
# i %= 2    # i = 0.0
# print(i)
```

<details><summary>Respuesta esperada</summary>

~~~text
0.0
~~~

Paso a paso desde `i = 2`: `+= 3` da `5`; `-= 1` da `4`; `*= 2` da `8`; `/= 8` da `1.0` (aquí `i` se vuelve `float` por el `/`); `**= 3` da `1.0`; `//= 3` da `0.0`; `%= 2` da `0.0`. El valor final es `0.0`.
</details>

## Paso 4: validación con un centinela (solo lectura)

Un `while` sirve para insistir hasta que un dato sea válido. La forma interactiva usa `input()`, que **no corre** en una ejecución automática, así que este bloque es **solo para leer** (en Colab sí funcionaría):

~~~python
horas = int(input("Ingrese hora [1...12]: "))
while horas < 1 or horas > 12:
    horas = int(input("Vuelva a ingresar hora [1...12]: "))
~~~

Si la primera lectura es válida (por ejemplo `5`), la condición `horas < 1 or horas > 12` es `False` y el bloque no corre ni una vez. Si es inválida (por ejemplo `15`), el ciclo vuelve a pedir el dato hasta que esté entre `1` y `12`. La variable que controla la salida (`horas`) es el **centinela**.

## Paso 5: el ciclo infinito (NO lo ejecutes)

Si el bloque no cambia la variable de la condición, el ciclo **nunca termina**. Este bloque es **solo para leer y trazar a mano** (si lo corres, se queda imprimiendo `0` para siempre y tendrías que interrumpir el kernel):

~~~python
i = 0
while i < 5:
    print(i)
~~~

`i` siempre vale `0`, así que `i < 5` es **siempre** `True`: falta la tercera pieza, cambiar el estado. El arreglo es una sola línea dentro del bloque:

~~~python
i = 0
while i < 5:
    print(i)
    i = i + 1   # ahora i avanza hacia 5 y el ciclo termina
~~~

## Tu turno (programa completo)

### Ejercicio: contar generaciones de duplicación celular

Un cultivo empieza con `10` células y se **duplica** cada generación. Escribe un `while` que cuente cuántas generaciones hacen falta para superar las `1000` células. Usa un contador `generaciones` y actualiza `celulas` con `*= 2`.

```python
celulas = 10
generaciones = 0
# TODO: quita los # y completa la condicion y el cuerpo del while.
# while ____:
#     celulas ____
#     generaciones ____
# print("Generaciones hasta pasar 1000:", generaciones)
# print("Celulas:", celulas)
```

<details><summary>Respuesta esperada</summary>

~~~text
Generaciones hasta pasar 1000: 7
Celulas: 1280
~~~

La condición es `celulas < 1000`; dentro, `celulas *= 2` (duplica) y `generaciones += 1` (cuenta). Partiendo de `10`: 20, 40, 80, 160, 320, 640, 1280. A las `7` generaciones el cultivo pasa de `1000` (llega a `1280`) y el ciclo se detiene. No sabíamos de antemano cuántas vueltas harían falta: por eso un `while` encaja mejor que copiar líneas.
</details>

## Resumen

- `while` repite su bloque **indentado** mientras la condición sea `True`; se detiene cuando es `False`.
- Todo `while` seguro tiene **tres piezas**: inicializar el estado, probar la condición y **cambiar el estado** hacia la salida.
- **Contador:** una variable de control que se inicializa y se incrementa cada vuelta. **Acumulador:** una variable que se inicializa fuera y acumula dentro.
- **Asignaciones con operador:** `i += 1` es `i = i + 1` (sirve con `+ - * / // % **`); recuerda que `/` da `float`.
- Sin cambio de estado hay **ciclo infinito**; traza a mano para evitar el error de uno-en-uno (`<` vs `<=`).
- **Sigue con la L11** (Módulo 2): el ciclo `for`, la forma natural de recorrer una colección. Antes, el **laboratorio calificado** (S9) y el **Examen Parcial 1** (S10), que incluye `while`.
