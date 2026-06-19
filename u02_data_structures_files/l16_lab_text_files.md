---
title: "Lab: Archivos de texto"
unit: "2"
lesson: 16
type: lab
tags: [python, archivos, open, with, lectura, escritura, lineas, fasta, conteo]
difficulty: introductory
duration: "60 mins"
---

# Lab: Archivos de texto

**Meta:** escribir un archivo de texto con `with open(..., 'w')`; leerlo iterando línea por línea y limpiando el `\n` con `.strip()`; procesarlo (contar líneas, contar secuencias, buscar); y escribir una **función que procesa un archivo**. Cierre del Módulo 2: junta `for`, cadenas, listas y `def` sobre un archivo FASTA real. Acompaña a la nota de concepto [Archivos de texto](l16_concept_text_files.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`. Donde diga **Predice**, hay una línea comentada con `#`: piensa primero qué pasaría y, si no daría error, quita el `#` para comprobarlo (la del Paso 4 daría un error a propósito -- déjala comentada). Donde diga **Tu turno**, hay un bloque comentado con una parte marcada con `____` que debes **completar** al quitar los `#`. Compara con la respuesta esperada de cada bloque desplegable.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u02_data_structures_files/l16_lab_text_files.ipynb)

## Preparación

Solo Python estándar: no hay que instalar nada. El archivo que vas a leer lo crea este mismo lab al escribirlo. Corre esta celda primero.

```python
print("Listo para trabajar con archivos")
```

~~~text
Listo para trabajar con archivos
~~~

## Paso 1: escribir un archivo (modo `'w'`)

Para crear un archivo se abre en modo `'w'` (escribir) y se usa `f.write(texto)`. A diferencia de `print`, `write` **no** agrega el salto de línea: lo incluyes tú con `\n`. Una cadena de triple comilla ya trae los saltos. Aquí escribimos un archivo FASTA con dos secuencias de ADN:

```python
fasta = """>seq1 muestra_A
ATGCATGCAT
>seq2 muestra_B
GCGCGCGCGC
"""

with open('secuencias.fasta', 'w') as f:
    f.write(fasta)

print('Archivo secuencias.fasta creado')
```

~~~text
Archivo secuencias.fasta creado
~~~

El bloque `with` abrió el archivo, escribió el texto y lo **cerró solo** al terminar. Como usamos `'w'`, si `secuencias.fasta` ya existía, su contenido se habría sobreescrito. (Con `'a'` en vez de `'w'`, el texto se añadiría al final.)

## Paso 2: leer un archivo

La forma natural de leer para procesar es iterar con un `for`: cada `linea` es una línea del archivo. Ojo: cada línea conserva su salto de línea final, así que la limpiamos con `.strip()`:

```python
with open('secuencias.fasta', 'r') as f:
    for linea in f:
        print(linea.strip())
```

~~~text
>seq1 muestra_A
ATGCATGCAT
>seq2 muestra_B
GCGCGCGCGC
~~~

Para **ver** ese `\n` que viene en cada línea, lee el archivo completo como una lista con `.readlines()`:

```python
with open('secuencias.fasta', 'r') as f:
    lineas = f.readlines()

print(lineas)
```

~~~text
['>seq1 muestra_A\n', 'ATGCATGCAT\n', '>seq2 muestra_B\n', 'GCGCGCGCGC\n']
~~~

Cada elemento de la lista termina en `\n`: por eso usamos `.strip()` al imprimir. (También existe `f.read()`, que devuelve todo el archivo en una sola cadena.)

## Paso 3: procesar por líneas

Con el `for`, un contador y un `if` puedes analizar el archivo. Contemos cuántas líneas tiene en total y cuántas son **encabezados** (las que empiezan con `>`). Para chequear el inicio de una línea se usa `.startswith(prefijo)`, que devuelve `True` o `False`:

```python
with open('secuencias.fasta', 'r') as f:
    total = 0
    secuencias = 0
    for linea in f:
        total += 1
        if linea.startswith('>'):
            secuencias += 1

print('Lineas:', total)
print('Secuencias:', secuencias)
```

~~~text
Lineas: 4
Secuencias: 2
~~~

También puedes **buscar** las líneas que contienen una cadena con el operador `in` e imprimir solo esas:

```python
with open('secuencias.fasta', 'r') as f:
    for linea in f:
        if 'GC' in linea:
            print(linea.strip())
```

~~~text
ATGCATGCAT
GCGCGCGCGC
~~~

Las dos líneas de secuencia contienen `'GC'`; los encabezados, no.

## Paso 4: una función que procesa un archivo (predice primero)

Empaquetemos el conteo de secuencias en una **función** para llamarla con cualquier archivo. La última línea está comentada: **predice** qué pasaría al quitarle el `#` (daría un error a propósito; déjala comentada).

```python
def contar_secuencias(nombre):
    conteo = 0
    with open(nombre, 'r') as f:
        for linea in f:
            if linea.startswith('>'):
                conteo += 1
    return conteo

print(contar_secuencias('secuencias.fasta'))
# print(contar_secuencias('noexiste.txt'))   # predice: que pasa si quitas el # de esta linea
```

~~~text
2
~~~

<details><summary>Respuesta esperada</summary>

`contar_secuencias('secuencias.fasta')` imprime `2`. Pero si quitaras el `#` de la última línea, el programa daría un `FileNotFoundError`: `open(..., 'r')` no puede leer un archivo que no existe (`noexiste.txt`). Para **crear** un archivo que falta se usa el modo `'w'` o `'a'`, no `'r'`. (Por eso la dejamos comentada: un error vivo detendría el resto del notebook.)
</details>

Esa función junta casi todo el módulo: un `def` con un parámetro, un `with`, un `for`, un `if` con `.startswith` y un acumulador con `return`. Ahora cerremos el arco del Módulo 2 reutilizando la función `gc` de la L15, pero leyendo la secuencia **desde el archivo**:

```python
def gc(adn):
    return (adn.count('G') + adn.count('C')) / len(adn) * 100

def gc_de_archivo(nombre):
    with open(nombre, 'r') as f:
        for linea in f:
            if not linea.startswith('>'):
                return gc(linea.strip())

print(gc_de_archivo('secuencias.fasta'))
```

~~~text
40.0
~~~

`gc_de_archivo` se salta los encabezados (`not linea.startswith('>')`) y, en la primera línea de secuencia, devuelve su contenido GC. Para `'ATGCATGCAT'` da `40.0`: las 2 `G` y 2 `C` de 10 bases que contaste a mano en la L14, encapsuladas en `gc(...)` en la L15, ahora **leídas desde un archivo**. Ese es el arco completo: una cadena (L12) -> una función (L15) -> un archivo (L16).

## Tu turno (programa completo)

### Ejercicio: promedio de un registro de laboratorio

Un instrumento guardó cinco lecturas de pH, una por línea, en un archivo de texto. Escribe una función `promedio_archivo(nombre)` que lea esas líneas, las convierta a `float` y devuelva el promedio. Completa la parte que falta.

```python
# TODO: quita los # y completa la parte que falta (el operador ____).
# mediciones = """7.1
# 6.8
# 7.4
# 7.0
# 6.9
# """
# with open('mediciones.txt', 'w') as f:
#     f.write(mediciones)
#
# def promedio_archivo(nombre):
#     suma = 0
#     conteo = 0
#     with open(nombre, 'r') as f:
#         for linea in f:
#             suma += float(linea.strip())
#             conteo ____ 1
#     return suma / conteo
#
# print('Promedio de pH: {:.2f}'.format(promedio_archivo('mediciones.txt')))
```

<details><summary>Respuesta esperada</summary>

~~~text
Promedio de pH: 7.04
~~~

La parte que falta es `+=` (el acumulador): `conteo += 1`, que cuenta cuántas líneas hay. La función escribe las cinco mediciones (modo `'w'`), las lee convirtiendo cada línea a `float` con `float(linea.strip())`, acumula la `suma` y la divide entre `conteo`. El `{:.2f}` de la L12 muestra el promedio con dos decimales: `(7.1 + 6.8 + 7.4 + 7.0 + 6.9) / 5 = 7.04`. Como cualquier función de archivo, la reutilizas con otro registro: `promedio_archivo('otras_mediciones.txt')`.
</details>

## Resumen

- Un archivo se abre con `with open(nombre, modo) as f:`, que lo **cierra solo** al terminar el bloque. Modos: `'r'` leer, `'w'` crear/sobreescribir, `'a'` agregar.
- Para **escribir** se usa `f.write(texto)`, incluyendo tú el `\n`; `'w'` borra lo que hubiera.
- Para **leer** y procesar, lo natural es iterar `for linea in f:`; cada línea trae su `\n`, que se quita con `.strip()`. (También están `f.read()` y `f.readlines()`.)
- Para **procesar** por líneas combinas `for` + cadenas + condicionales: contar (acumulador), buscar (`in`, `.startswith`), contar ocurrencias (`.count`).
- Una **función que procesa un archivo** empaqueta abrir + recorrer + procesar en un `def ... return` reutilizable -- el cierre integrador del Módulo 2 (`contar_secuencias`, `gc_de_archivo`).
- **Sigue el Módulo 2** con el laboratorio calificado y el Parcial 2; después, en el **Módulo 3**, leerás datos en tablas con archivos **CSV** -- primero a mano, como hoy, y luego con Pandas.
