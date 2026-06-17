---
title: "Lab: Lógica booleana"
unit: "1"
lesson: 6
type: lab
tags: [python, booleanos, comparaciones, and-or-not, condicional-if]
difficulty: introductory
duration: "60 mins"
---

# Lab: Lógica booleana

**Meta:** evaluar condiciones con `not`/`and`/`or` y comparaciones, y escribir tu primer condicional `if`. Acompaña a la nota de concepto [Lógica booleana](l06_concept_boolean_logic.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`. Donde diga **Completa tú** o **Tu turno**, las líneas vienen comentadas con `#`: quítalo y complétalas. Compara con la respuesta esperada de cada bloque desplegable.

## Preparación

Solo Python estándar: no hay que instalar nada. Corre esta celda primero.

```python
print("Listo para la logica booleana")
```

~~~text
Listo para la logica booleana
~~~

## Paso 1: not, and, or

Los operadores booleanos combinan o niegan condiciones. Ejecuta y compara con las tablas de verdad de la lección.

```python
print(not True)          # niega
print(True and True)     # ambos verdaderos
print(True and False)    # uno falso
print(False or True)     # al menos uno verdadero
print(False or False)    # ninguno verdadero
```

~~~text
False
True
False
True
False
~~~

Recuerda: `and` es verdadero solo si **ambos** lo son; `or`, si **al menos uno** lo es; `not` invierte.

## Paso 2: comparaciones

Cada comparación devuelve un booleano (`True` o `False`). Ejecuta las seis.

```python
print(5 == 6)
print(5 != 6)
print(5 < 6)
print(5 <= 6)
print(5 > 6)
print(5 >= 6)
```

~~~text
False
True
True
True
False
False
~~~

Cuidado: `==` **compara** (pregunta si dos valores son iguales); `=` **asigna** (guarda un valor en una variable). No son lo mismo.

## Paso 3: combinar condiciones

**Completa tú.** Antes de ejecutar, **predice** cada resultado usando las tablas de verdad. Luego quita los `#` y comprueba.

```python
# Predice primero, luego quita los # y comprueba.
# print((5 < 6) and (6 < 7))    # ambas se cumplen?
# print((5 > 6) or (3 == 3))    # al menos una?
# print(not (5 == 5))           # niega una verdad
```

<details><summary>Respuesta esperada</summary>

~~~text
True
True
False
~~~

`(5 < 6) and (6 < 7)`: las dos comparaciones son `True`, y `and` exige ambas, asi que da `True`. `(5 > 6) or (3 == 3)`: la primera es `False` pero la segunda es `True`, y `or` pide al menos una, asi que da `True`. `not (5 == 5)`: `5 == 5` es `True`, y `not` lo invierte a `False`.
</details>

## Paso 4: tu primer condicional `if`

El `if` ejecuta un bloque **indentado** solo si la condicion es verdadera. Este programa resuelve `a*x + b = 0` solo si `a` no es cero (para no dividir entre cero).

En Colab pedirias los coeficientes al usuario (NO ejecutes esto aqui; solo observalo):

~~~python
a = float(input("Ingrese a: "))
b = float(input("Ingrese b: "))
~~~

Aqui usamos valores fijos para que el notebook corra completo. Ejecuta:

```python
a = 2.0       # en Colab lo pedirias con input()
b = -6.0
if a != 0:                # evita dividir entre cero
    x = -b / a
    print("Solucion x =", x)
```

~~~text
Solucion x = 3.0
~~~

Como `a` vale `2.0` (distinto de `0`), la condicion `a != 0` es `True` y el bloque se ejecuta. Si `a` fuera `0`, el bloque se saltaria y el programa seguiria sin imprimir nada.

## Tu turno (programa completo)

### Ejercicio: clasificar por un umbral

Una concentracion de `0.8` mol/L se compara con un limite de `0.5`. Escribe un `if` que imprima `"Concentracion alta"` solo si la concentracion **supera** el limite.

```python
concentracion = 0.8       # mol/L
limite = 0.5
# TODO: quita los # y completa la condicion del if.
# if ____:
#     print("Concentracion alta")
```

<details><summary>Respuesta esperada</summary>

~~~text
Concentracion alta
~~~

La condicion es `concentracion > limite`. Como `0.8 > 0.5` es `True`, el bloque se ejecuta y se imprime el aviso. Si la concentracion fuera, por ejemplo, `0.3`, la condicion seria `False` y no se imprimiria nada.
</details>

## Resumen

- Las **comparaciones** (`== != < <= > >=`) y los **operadores booleanos** (`not`, `and`, `or`) producen `True` o `False`.
- `=` asigna; `==` compara: no son lo mismo.
- El **flujo** es secuencial hasta que una estructura de control lo cambia.
- El `if` ejecuta su bloque **indentado** solo si la condicion es `True`.
- **Sigue con la L07:** aprenderas `else`, `elif` y condiciones anidadas para cubrir todos los casos.
