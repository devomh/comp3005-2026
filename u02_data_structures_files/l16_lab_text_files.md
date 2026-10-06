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

**Meta:** escribir un archivo de texto con `with open(..., 'w')`; leerlo iterando línea por línea y limpiando el `\n` con `.strip()`; procesarlo (contar líneas, contar secuencias, buscar); y escribir una **función que procesa un archivo**. Cierre del Módulo 2: junta `for`, cadenas, listas y `def` sobre un archivo FASTA real y, en el último ejercicio, sobre un archivo de notas con valores separados por comas. Acompaña a la nota de concepto [Archivos de texto](l16_concept_text_files.qmd).

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

### Ejercicio 1: promedio de un registro de laboratorio

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

### Ejercicio 2: las notas de un curso (antesala del CSV)

En el Ejercicio 1 cada línea traía **un** número. Ahora cada línea trae **varios** valores separados por comas: el nombre de un estudiante y sus cuatro notas.

~~~text
Ana,75,90,45,100
Bruno,58,80,93,94
...
~~~

Vas a calcular el promedio, la nota máxima y la mínima de cada estudiante, y a contar cuántos tienen promedio de 70 o más. Es la idea de la L13 (`.split(',')` parte una cadena en una lista) aplicada a cada línea de un archivo. Un archivo así, con valores separados por comas, es casi un **CSV**: en la L19 le añadirás un encabezado y nombres de columna. Hazlo en cinco pasos; cada celda tiene su respuesta esperada.

**Paso A: escribe el archivo.** Esta celda no tiene nada que completar: quita los `#` y córrela.

```python
# TODO: quita los # y corre la celda.
# registro = """Ana,75,90,45,100
# Bruno,58,80,93,94
# Carla,62,70,55,68
# Diego,90,85,88,95
# Elena,40,65,72,58
# Felipe,70,70,75,65
# Gabriela,88,92,79,85
# Hugo,55,60,48,70
# Irene,95,100,98,92
# Jorge,68,74,66,71
# """
# with open('notas.txt', 'w') as f:
#     f.write(registro)
#
# print('Archivo notas.txt creado')
```

<details><summary>Respuesta esperada (Paso A)</summary>

~~~text
Archivo notas.txt creado
~~~

Diez líneas, una por estudiante: el nombre y cuatro notas, todo separado por comas.
</details>

**Paso B: tus funciones de listas.** En clase escribiste funciones que calculan el promedio, el máximo y el mínimo de una lista de números con un `for`. Si las tienes, pega las tuyas aquí y borra las de abajo: deben llamarse `promedio`, `maximo` y `minimo`, recibir una lista y **devolver** el resultado con `return` (no imprimirlo), porque los Pasos D y E las llaman con esos nombres. Renómbralas si hace falta. Si no las tienes, usa estas: `promedio` es la de la L15; en `maximo` y `minimo` completa la comparación que falta (`____`).

```python
# TODO: quita los # y completa las dos comparaciones (____), o pega tus funciones.
# def promedio(lista):
#     suma = 0
#     for x in lista:
#         suma += x
#     return suma / len(lista)
#
# def maximo(lista):
#     mayor = lista[0]
#     for x in lista:
#         if x ____ mayor:
#             mayor = x
#     return mayor
#
# def minimo(lista):
#     menor = lista[0]
#     for x in lista:
#         if x ____ menor:
#             menor = x
#     return menor
#
# print(promedio([75, 90, 45, 100]), maximo([75, 90, 45, 100]), minimo([75, 90, 45, 100]))
```

<details><summary>Respuesta esperada (Paso B)</summary>

~~~text
77.5 100 45
~~~

Las comparaciones son `if x > mayor:` en `maximo` y `if x < menor:` en `minimo` (con `>=` y `<=` también funcionan). Las dos empiezan con el primer elemento (`lista[0]`) como el mejor visto hasta ahora y lo reemplazan cada vez que aparece uno mayor (o menor): el patrón del acumulador, guardando un valor en vez de una suma.
</details>

**Paso C: de una línea a una lista de notas.** Escribe `notas_de(linea)`, que recibe una línea del archivo y devuelve sus notas como una lista de enteros. `.strip()` quita el `\n`, `.split(',')` parte la línea en campos y la comprensión de la L13 convierte cada campo a `int`. El nombre está en la posición `0`; las notas son **todo lo demás**. Una lista se corta igual que una cadena: la **porción** (*slice*) `[a:]` de la L12 va de la posición `a` hasta el final. Completa la porción (`____`) que deja fuera el nombre.

```python
# TODO: quita los # y completa la porcion de la lista (____).
# def notas_de(linea):
#     partes = linea.strip().split(',')
#     return [int(x) for x in partes[____]]
#
# print('Ana,75,90,45,100\n'.strip().split(','))
# print(notas_de('Ana,75,90,45,100\n'))
```

<details><summary>Respuesta esperada (Paso C)</summary>

~~~text
['Ana', '75', '90', '45', '100']
[75, 90, 45, 100]
~~~

La porción es `partes[1:]`: desde la posición 1 hasta el final, igual que en una cadena. (`partes[1:5]` también da las cuatro notas, pero `[1:]` sirve para cualquier cantidad de notas.) La primera línea impresa muestra que `.split(',')` devuelve **cadenas** (`'75'`, no `75`); por eso hay que convertir cada una con `int()` antes de calcular. Probamos con una línea escrita a mano, con su `\n` al final como las que vienen del archivo.
</details>

**Paso D: el reporte del curso.** Recorre el archivo y, para cada estudiante, imprime su promedio, su máximo y su mínimo. El nombre es el primer campo de la línea. Completa la llamada que convierte la línea en la lista de notas (`____`).

```python
# TODO: quita los # y completa la funcion que falta (____).
# with open('notas.txt', 'r') as f:
#     for linea in f:
#         estudiante = linea.split(',')[0]
#         notas = ____(linea)
#         p = promedio(notas)
#         alto = maximo(notas)
#         bajo = minimo(notas)
#         print('{}: promedio {:.2f}, max {}, min {}'.format(estudiante, p, alto, bajo))
```

<details><summary>Respuesta esperada (Paso D)</summary>

~~~text
Ana: promedio 77.50, max 100, min 45
Bruno: promedio 81.25, max 94, min 58
Carla: promedio 63.75, max 70, min 55
Diego: promedio 89.50, max 95, min 85
Elena: promedio 58.75, max 72, min 40
Felipe: promedio 70.00, max 75, min 65
Gabriela: promedio 86.00, max 92, min 79
Hugo: promedio 58.25, max 70, min 48
Irene: promedio 96.25, max 100, min 92
Jorge: promedio 69.75, max 74, min 66
~~~

La llamada es `notas = notas_de(linea)`. Cada vuelta del `for` toma una línea, la convierte en una lista de cuatro números y le aplica las tres funciones del Paso B. Una función por tarea: `notas_de` separa, `promedio`/`maximo`/`minimo` calculan. Con cuatro notas enteras, cada promedio termina en `.00`, `.25`, `.50` o `.75`, así que `{:.2f}` lo muestra exacto.
</details>

**Paso E: ¿cuántos tienen promedio de 70 o más?** Empaqueta el conteo en una función que procesa un archivo, como `contar_secuencias` en el Paso 4. Completa la comparación (`____`).

```python
# TODO: quita los # y completa la comparacion (____).
# def contar_aprobados(nombre):
#     conteo = 0
#     with open(nombre, 'r') as f:
#         for linea in f:
#             if promedio(notas_de(linea)) ____ 70:
#                 conteo += 1
#     return conteo
#
# print('Estudiantes con promedio de 70 o mas:', contar_aprobados('notas.txt'))
```

<details><summary>Respuesta esperada (Paso E)</summary>

~~~text
Estudiantes con promedio de 70 o mas: 6
~~~

La comparación es `>=`: "70 o más" incluye el 70. Los seis son Ana, Bruno, Diego, Felipe, Gabriela e Irene. Fíjate en Felipe: su promedio es exactamente `70.00`, así que con `>` el conteo bajaría a 5. Jorge, con `69.75`, queda fuera con cualquiera de las dos. La función reúne todo el ejercicio: abre el archivo, separa cada línea con `notas_de` y compara su `promedio`.
</details>

## Resumen

- Un archivo se abre con `with open(nombre, modo) as f:`, que lo **cierra solo** al terminar el bloque. Modos: `'r'` leer, `'w'` crear/sobreescribir, `'a'` agregar.
- Para **escribir** se usa `f.write(texto)`, incluyendo tú el `\n`; `'w'` borra lo que hubiera.
- Para **leer** y procesar, lo natural es iterar `for linea in f:`; cada línea trae su `\n`, que se quita con `.strip()`. (También están `f.read()` y `f.readlines()`.)
- Para **procesar** por líneas combinas `for` + cadenas + condicionales: contar (acumulador), buscar (`in`, `.startswith`), contar ocurrencias (`.count`).
- Una **función que procesa un archivo** empaqueta abrir + recorrer + procesar en un `def ... return` reutilizable -- el cierre integrador del Módulo 2 (`contar_secuencias`, `gc_de_archivo`).
- Una línea con **varios valores separados por comas** se procesa con `linea.strip().split(',')`: el nombre queda en `partes[0]` y los números se convierten con `[int(x) for x in partes[1:]]`. Así cada línea se vuelve una lista a la que aplicas tus funciones de listas (`promedio`, `maximo`, `minimo`); `notas_de` hace la conversión y `contar_aprobados` empaqueta todo en una función de archivo.
- **Sigue el Módulo 2** con el laboratorio calificado y el Parcial 2; después, en el **Módulo 3**, leerás datos en tablas con archivos **CSV** -- primero a mano, como hoy, y luego con Pandas.
