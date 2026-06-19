---
title: "Lab: Biopython -- secuencias de ADN"
unit: "3"
lesson: 25
type: lab
tags: [python, biopython, bioinformatica, adn, secuencias, fasta, gc, transcripcion, traduccion]
difficulty: introductory
duration: "75 mins"
---

# Lab: Biopython -- secuencias de ADN

**Meta:** trabajar con un tercer tipo de dato científico, la **secuencia de ADN**, usando **Biopython** (la caja de herramientas de la bioinformática). Crearás una secuencia (`Seq`) y verás que es **texto con superpoderes**: obtendrás su hebra complementaria, calcularás su **contenido GC**, la **transcribirás** a ARN y la **traducirás** a proteína, y leerás un archivo **FASTA** con `SeqIO.parse` (lo que en la L16 hiciste a mano). Acompaña a la nota de concepto [Biopython](l25_concept_biopython.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`, empezando por las dos celdas de preparación. Donde diga **Tu turno**, hay un bloque comentado con una parte marcada con `____` que debes **completar** al quitar los `#`. Compara con la respuesta esperada del bloque desplegable.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u03_scientific_computing/l25_lab_biopython.ipynb)

## Preparación

Necesitamos **Biopython**, la librería de bioinformática de Python (se importa como `Bio`). Corre estas **dos** celdas primero. La primera la instala (en Colab; si ya está, no hace nada). La segunda importa lo que usaremos: `Seq` para crear secuencias, `SeqIO` para leer FASTA y `gc_fraction` para el contenido GC.

```python
%pip install -q biopython
```

```python
from Bio.Seq import Seq
from Bio import SeqIO
from Bio.SeqUtils import gc_fraction

print("Biopython listo para trabajar")
```

~~~text
Biopython listo para trabajar
~~~

## Paso 1: crear una secuencia

Una secuencia de ADN es **texto**: una cadena de las letras `A`, `C`, `G`, `T`. Con `Seq(...)` la creamos como un objeto de Biopython. Es **como una cadena** (lo de la L12): tiene `len`, se indexa, se rebana y se cuenta con `.count`.

```python
adn = Seq("ATGGCCATTGTAATGGGCCGCTGAAAGGGTGCCCGATAG")
print("secuencia:", adn)
print("longitud:", len(adn))
print("primeras tres bases:", adn[0:3])
print("conteo de G:", adn.count("G"))
```

~~~text
secuencia: ATGGCCATTGTAATGGGCCGCTGAAAGGGTGCCCGATAG
longitud: 39
primeras tres bases: ATG
conteo de G: 14
~~~

La secuencia mide 39 bases, empieza por `ATG` (lo que en biología es el "arranque" de un gen) y tiene 14 `G`. Hasta aquí, todo es lo que ya sabes hacer con una cadena.

## Paso 2: las dos hebras

El ADN es de **doble hebra**: frente a cada base hay su complementaria (`A` con `T`, `C` con `G`). `complement()` da esa hebra base por base; `reverse_complement()` además la lee al revés (que es como corre la hebra opuesta, antiparalela).

```python
print("complemento:", adn.complement())
print("reverso-complemento:", adn.reverse_complement())
```

~~~text
complemento: TACCGGTAACATTACCCGGCGACTTTCCCACGGGCTATC
reverso-complemento: CTATCGGGCACCCTTTCAGCGGCCCATTACAATGGCCAT
~~~

Fíjate en el apareamiento: donde el original tiene `A`, el complemento tiene `T`, y donde tiene `G`, tiene `C`. Esto es algo que una cadena de texto cualquiera no sabe hacer; `Seq` sí, porque conoce de biología.

## Paso 3: contenido GC

El **contenido GC** es el porcentaje de bases que son `G` o `C`. Importa en biología: un ADN con más GC es más estable y se comporta distinto en una PCR. `gc_fraction` da la fracción (de 0 a 1); por 100 es el porcentaje.

```python
print("porcentaje GC:", round(gc_fraction(adn) * 100, 1))
```

~~~text
porcentaje GC: 56.4
~~~

El 56.4 % de las bases de esta secuencia son `G` o `C`. (Usamos `round` porque la fracción cruda tiene muchos decimales.)

## Paso 4: transcribir y traducir (el dogma central)

El **dogma central** de la biología molecular es `ADN -> ARN -> proteína`. Biopython lo hace en dos métodos: `transcribe()` pasa de ADN a ARN (cada `T` se vuelve `U`), y `translate()` lee el ARN en **codones** (grupos de 3 bases) y los traduce a **aminoácidos** según el código genético. El `*` marca un codón de **stop**; con `translate(to_stop=True)` la proteína se corta en el primer stop.

```python
arn = adn.transcribe()
print("ARN:", arn)
proteina = adn.translate()
print("proteina:", proteina)
print("proteina hasta el primer stop:", adn.translate(to_stop=True))
```

~~~text
ARN: AUGGCCAUUGUAAUGGGCCGCUGAAAGGGUGCCCGAUAG
proteina: MAIVMGR*KGAR*
proteina hasta el primer stop: MAIVMGR
~~~

En el ARN cada `T` pasó a `U`. La proteína completa es `MAIVMGR*KGAR*`: cada letra es un aminoácido (`M` es la metionina del arranque `ATG`) y los `*` son codones de stop. Con `to_stop=True` nos quedamos con `MAIVMGR`, la proteína hasta el primer stop.

## Paso 5: leer un FASTA con Biopython

En la **L16** parseaste un archivo **FASTA** a mano: distinguías las líneas de encabezado (las que empiezan con `>`) de las líneas de secuencia. Biopython lo hace en **una llamada**: `SeqIO.parse` lee el archivo y entrega un **registro** por secuencia, con su `id` y su `seq`. Es el mismo salto de "a mano -> herramienta" que viste con CSV (L19) -> `read_csv` (L21). Escribimos un FASTA y lo leemos:

```python
fasta = """>gen_alfa organismo ejemplo
ATGGCCATTGTAATGGGCCGCTGAAAGGGTGCCCGATAG
>gen_beta organismo ejemplo
ATGCGTACGTTAGCCGGGCTAAGGCACTGAC
"""

with open('genes.fasta', 'w') as f:
    f.write(fasta)

for registro in SeqIO.parse('genes.fasta', 'fasta'):
    print(registro.id, "| longitud:", len(registro.seq),
          "| GC%:", round(gc_fraction(registro.seq) * 100, 1))
```

~~~text
gen_alfa | longitud: 39 | GC%: 56.4
gen_beta | longitud: 31 | GC%: 58.1
~~~

`SeqIO.parse` recorrió el archivo, separó los dos registros y nos dio su `id` y su secuencia (`registro.seq`), lista para medir. Lo que en la L16 eran varias líneas de código, aquí es un `for` sobre `SeqIO.parse`.

## Tu turno (programa completo)

### Ejercicio: un gen nuevo

Tienes una secuencia de gen nueva. Calcula su **contenido GC** y **tradúcela** a proteína, cortando en el primer codón de stop. Completa la parte que falta.

```python
# TODO: quita los # y completa el ____ para cortar la proteina en el primer stop.
# gen = Seq("ATGTTTGGCCGCGCATAA")
# print("GC%:", round(gc_fraction(gen) * 100, 1))
# print("proteina:", gen.translate(____))
```

<details><summary>Respuesta esperada</summary>

La parte que falta es el argumento `to_stop=True`, que corta la proteína en el primer stop:

~~~python
gen = Seq("ATGTTTGGCCGCGCATAA")
print("GC%:", round(gc_fraction(gen) * 100, 1))
print("proteina:", gen.translate(to_stop=True))
~~~

Al ejecutarlo imprime `GC%: 50.0` y `proteina: MFGRA`. La secuencia mide 18 bases (6 codones): `ATG TTT GGC CGC GCA TAA` se traduce a `M F G R A` y para en el stop `TAA`. Sin `to_stop=True`, la proteína saldría como `MFGRA*` (con el stop incluido).
</details>

## Resumen

- Una **secuencia de ADN es texto**; con `Seq(...)` es una cadena (`len`, indexar, `.count`) **con métodos de biología**.
- **Dos hebras:** `complement()` / `reverse_complement()` dan la hebra complementaria (apareamiento `A-T`, `C-G`).
- **Contenido GC:** `gc_fraction(seq)` da la fracción de `G`/`C`; por 100 es el porcentaje.
- **Dogma central:** `transcribe()` pasa ADN a ARN (`T`->`U`); `translate()` traduce codones a proteína (`*` = stop; `to_stop=True` corta en el primer stop).
- **Leer FASTA:** `SeqIO.parse('archivo.fasta', 'fasta')` da un registro por secuencia (`registro.id`, `registro.seq`) en una llamada -- el espiral de la L16.
- **Sigue en la L26:** la librería de dominio de **química** (mendeleev y RDKit): la tabla periódica como datos.
