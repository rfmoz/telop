telop
=======

Telop (TELégrafoÓPtico) - Utilidad para codificar y descodificar mensajes de texto empleando una interpretación del código telegráfico desarrollado por José María Mathé. Permite recrear el sistema utilizado en la red de telegrafía óptica de España a mediados del s.XIX.


### Uso Básico

**Codificar mensaje:**
```
$ telop 'Telegrama de prueba'
--------------------------------------------------------------------------------
Tipo:		 0 - General
T. Origen:	 001
T. Destino:	 052
Hora y Día:	 23:10 08
Registro:	 00
Novenales:	 04.2
--------------------------------------------------------------------------------

Mensaje:	 0/0x1052/2310x80x/042/5x1421x41/627102x10/9x13149x2/52730141x/10/0

--------------------------------------------------------------------------------
```

**Descodificar mensaje:**
```
$ telop '0/0x1052/2310x80x/042/5x1421x41/627102x10/9x13149x2/52730141x/10/0'
--------------------------------------------------------------------------------
Tipo:		 0 - General
T. Origen:	 001
T. Destino:	 052
Hora y Día:	 23:10 08
Registro:	 00
Novenales:	 04.2
--------------------------------------------------------------------------------

Mensaje:	 Telegrama de prueba

--------------------------------------------------------------------------------
```

### Ejemplos de cada tipo de mensaje

**0 - General**
**4 - General urgente**
**8 - General urgentísimo**

Mensaje habitual para enviar el texto consignado.

- Codificar texto de la manera más sencilla de la torre '001' (por defecto) a la '041':
    > telop -d 41 'Texto ejemplo' 
- Mensaje urgentísimo '8' con registro '12', origen '010' y destino '050' :
    > telop -t 8 -r 12 -o 10 -d 50 'Texto'


**2 - Servicio interno**

Utilizado sólo para dar indicaciones de servicio.

- Mensaje interno de la torre '001' (por defecto) a la '045' con código 10:
    > telop -t 2 -d 45 -s 10 
- Mensaje interno de la torre '045'a la '001' con código 11 y formato de fecha breve:
    > telop -t 2 -o 45 -d 1 -s 11 -b 


**3 - Vigilancia**

Para controlar y mantener la atención sobre la línea. Se envían cada media hora, desde la cabecera al final de la línea y los ramales.

Su recepción se confirma mediante la devolución de otro mensaje de vigilancia, indicando las torres oportunas.

- Mensaje inicial, por ejemplo, con valor '99' a modo de comodín a todos los extremos de línea y ramales, origen ímplicito (sin indicar con '0'), formato fecha breve:
    > telop -t 3 -o 0 -d 99 -c -b
- Mensaje de respuesta con recepción correcta '0'. Comandancia de origen '07' y destino '01':
    > telop -t 3 -o 7 -d 1 -c -s 0


**5 - Continuación**

Aviso para indicar la reanudación de un mensaje general detenido en cualquier torre, habitualmente por causas meteorológicas o cruce con otra comunicación de mayor prioridad.

- Retomar la transmisión del mensaje con torre de origen '009' y refrencia '43':
    > telop -t 5 -o 9 -r 43


**6 - Acuse de recibo**

Confirmar la recepción de un mensaje general junto con el motivo que lo provoca.

- Recepción correcta de mensaje con registro '12' desde la torre '040' a la torre '001':
    > telop -t 6 -o 40 -d 1 -r 12
- Recepción por niebla '1' de mensaje con registro '17' desde la torre '030' a la torre '001':
    > telop -t 6 -o 30 -d 1 -r 17 -s 1


**1 - Rectificación**

Solicitar la anulación o retransmisión de un mensaje general, según su número de registro.

- Repetir '6' mensaje con registro '23' desde la torre '021' a la '001':
    > telop -t 1 -o 21 -d 1 -s 6 -r 23
- Anular '9' mensaje con registro '12' desde la torre '021' a la '001':
    > telop -t 1 -o 21 -d 1 -s 9 -r 12 


### Modificaciones del formato en cabecera

**Fecha con formato corto**

En cualquier mensaje con fecha se puede pasar el argumento '-b' para utilizar el formato reducido:

- Mensaje con origen '010', destino '021' y formato de fecha breve:
    > telop -o 10 -d 21 -b 'Texto'


**Sustituir indicación de torre por comandancia**

Empleando el argumento '-c', en cualquier mensaje se puede sustituir el formato de torre, de tres cifras, por el de comandancia, de dos cifras.

- Mensaje con origen '01', destino '07' y opción de comandancia:
    > telop -o 1 -d 7 -c 'Texto'


**Indicar sólo una torre o comandancia**

Es posible generar un mensaje con sólo un número de torre en vez del formato habitual que lleva dos, origen y destino. Con sólo una torre, se deduce el origen o destino según el sentido del mensaje y la posición que ocupa la torre en la línea. Para ello, se pasa el valor '0' a la opción de origen `telop -o 0` o destino `telop -d 0`.


### Opciones del programa:
```
usage: telop [-h] [-t {0,1,2,3,4,5,6,8}] [-o [nº]] [-d [nº]] [-b] [-c] [--diccionario] [--pwd PWD] [-r [nº]] [-s SUFIJO] [--solo] [-v] [--version] [-z {0,1}] [mensaje]

positional arguments:
  mensaje               texto del mensaje entre ' '

options:
  -h, --help            show this help message and exit
  -t {0,1,2,3,4,5,6,8}, --tipo {0,1,2,3,4,5,6,8}
                        tipo de servicio -> 0-general | 4-gral. urgente | 8-gral. urgentísimo | 1-rectificación | 2-interno | 3-vigilancia | 5-continuación | 6-acuse recibo
  -o [nº], --origen [nº]
                        torre de origen
  -d [nº], --destino [nº]
                        torre de destino
  -b, --breve           formato fecha y hora reducido
  -c, --comandancia     emplear n. de comandancia en origen / destino
  --diccionario         mostrar diccionario codificación
  --pwd PWD             codificar mensaje con contraseña
  -r [nº], --registro [nº]
                        nº registro despacho
  -s SUFIJO, --sufijo SUFIJO
                        sufijo aplicable a los mensajes tipo 1, 2, 3 y 6
  --solo                sólo imprime mensaje resultante
  -v, --verbose         debug
  --version             show program's version number and exit
  -z {0,1}              proceso a ejecutar -> (auto) | 0-codificar | 1-descodificar
```

### Diccionario codificación:

```
$ telop --diccionario
--------
Nº - Valor
--------
00 - 0       20 - k       40 - E       60 - Y       80 - =
01 - 1       21 - l       41 - F       61 - Z       81 - >
02 - 2       22 - m       42 - G       62 - !       82 - ?
03 - 3       23 - n       43 - H       63 - "       83 - @
04 - 4       24 - o       44 - I       64 - #       84 - [
05 - 5       25 - p       45 - J       65 - $       85 - \
06 - 6       26 - q       46 - K       66 - %       86 - ]
07 - 7       27 - r       47 - L       67 - &       87 - ^
08 - 8       28 - s       48 - M       68 - '       88 - _
09 - 9       29 - t       49 - N       69 - (       89 - `
10 - a       30 - u       50 - O       70 - )       90 - {
11 - b       31 - v       51 - P       71 - *       91 - |
12 - c       32 - w       52 - Q       72 - +       92 - }
13 - d       33 - x       53 - R       73 - ,       93 - ~
14 - e       34 - y       54 - S       74 - -       94 - Ñ
15 - f       35 - z       55 - T       75 - .       95 - ñ
16 - g       36 - A       56 - U       76 - /       96 - ¿
17 - h       37 - B       57 - V       77 - :       97 - €
18 - i       38 - C       58 - W       78 - ;       98 - n/a
19 - j       39 - D       59 - X       79 - <       99 -  
```

### Instalación

Requiere Python 3. Descargar y ejecutar el archivo "telop"



### Notas

- Cada dígito del mensaje de texto se codifica según la posición que ocupa en el diccionario definido internamente en el programa (telop --diccionario). Se sustituye así el diccionario frasológico del sistema original. Resulta un telegrama de mayor extensión, pero más versátil y fácil de implementar.

- En el siguiente esquema se establece el orden de los valores en la cabecera:

```
A/___B__/___C____/D
|    |      |     |
|    |      |     --- D sufijo particular a cada tipo de mensaje([0-3])
|    |      --------- C hora(2) + minutos(2) + dia(2) + registro(2)
|    ---------------- B torre de origen(3) + torre de destino(3)
--------------------- A tipo de servicio y prioridad(1)


0/0x10x5/2341040x/013/252730141/1x0/0 -> Mensaje general
|    |       |    |   \          /  |
|    |       |    |    \        /   -- A tipo de servicio y prioridad(1)
|    |       |    |     -------------- - novenales de mensaje
|    |       |    -------------------- D sufijo nº de novenales completos(2) y nº digitos resto(1)
|    |       ------------------------- C hora(2) + minutos(2) + dia(2) + registro(2)
|    --------------------------------- B torre de origen(3) + torre de destino(3)
-------------------------------------- A tipo de servicio y prioridad(1)

2/0x10x5/ 234104 /0x1/2 -> Comunicación interna
|    |       |     |  |
|    |       |     |  -- A tipo de servicio y prioridad(1)
|    |       |     ----- D sufijo código interno
|    |       ----------- C hora(2) + minutos(2) + dia(2)
|    ------------------- B torre de origen(3) + torre de destino(3)
------------------------ A tipo de servicio y prioridad(1)

3/0x10x5/ 234104 /0x -> Vigilancia
|    |       |    |
|    |       |    --- D sufijo estado opcional, sólo en recepción([0-2])
|    |       -------- C hora(2) + minutos(2) + dia(2)
|    ---------------- B torre de origen(3) + torre de destino(3)
--------------------- A tipo de servicio y prioridad(1)

6/0x10x5/2341040x/0x -> Acuse de recibo
|    |       |    |
|    |       |    --- D sufijo estado acuse de recibo([1-2])
|    |       -------- C hora(2) + minutos(2) + dia(2) + registro mensaje recibido(2)
|    ---------------- B torre de origen(3) + torre de destino(3)
--------------------- A tipo de servicio y prioridad(1)

5/  0x1 /   03  -> Continuación
|    |       |
|    |       |   D sufijo(0)
|    |       --- C registro mensaje a continuar(2)
|    ----------- B torre de origen(3)
---------------- A tipo de servicio y prioridad(1)

1/0x10x5/   04   /6/2/3/4 -> Rectificación
|    |       |    | \   /
|    |       |    |  ----- - novenales a repetir(opcional)
|    |       |    -------- D sufijo tipo de petición(1)
|    |       ------------- C registro mensaje rectificado(2)
|    --------------------- B torre de origen(3) + torre de destino(3)
-------------------------- A tipo de servicio y prioridad(1)
```

- Cada mensaje puede llevar un sufijo indicando las interrupciones sufridas durante la transmisión, si así corresponde. Se puede repetir el número de veces necesario.
  El grupo que comprende la hora, los minutos y día es opcional y puede estar compuesto por todos esos valores o sólo de la hora y los minutos. La causa corresponde a la misma numeración que se utiliza en el acuse de recibo -> 1-niebla | 2-ausencia | 3-ocupada | 4-avería. El formato es el siguiente:

```
/_X_/__Y__/Z -> Sufijo interrupción
  |    |   |
  |    |   -- Z causa(1)
  |    ------ Y hora(2) + minutos(2) + dia(2)
  ----------- X torre(3)

/011/183012/2 -> Sufijo interrupción
  |    |    |
  |    |    -- Z causa(1)
  |    ------- Y hora(2) + minutos(2) + dia(2)
  ------------ X torre(3)

/011/1830  /2 -> Sufijo interrupción
  |    |    |
  |    |    -- Z causa(1)
  |    ------- Y hora(2) + minutos(2)
  ------------ X torre(3)

/011       /2 -> Sufijo interrupción
  |         |
  |         -- Z causa(1)
  ------------ X torre(3)
```

- En la cabecera se puede emplear un formato de fecha y hora reducido con la opción `--breve`, a costa de obtener una precisión de 15 minutos.
  Son dos dígitos los que representan la hora y los minutos, el resultado se obtiene teniendo en cuenta el cuarto de hora en que se encuentran los minutos. Se suma 0, 25, 50 o 75 a la hora (00 a 24) según si es el primer, segundo, tercer o último cuarto de hora. Como ejemplo las 12:05 sería un 12, las 12:20 sería 12+25 = 37, las 12:40 sería 12+50 = 62 y las 12:55 12+75 = 87.
  El día sólo mantiene el último dígito, es decir, se representa igual el día 1 que el 11 que el 21.
  Ésta fue la modificación más curiosa de las empleadas y conocidas, por eso su codificación, el resto básicamente conseguían reducir tamaño a base de omitir información fácilmente interpretable por la situación del emisor y receptor.
  El grupo de cabecera podría tener cualquiera de los siguientes formatos (aunque no todos se llegan a utilizar):
```
/12150199/ -> hora(2) + minutos(2) + dia(2) + registro(2)
/121501/   -> hora(2) + minutos(2) + dia(2)
/37199/    -> hora_breve(2) + dia_breve(1) + registro(2)
/1215/     -> hora(2) + minutos(2)
/371/      -> hora_breve(2) + dia_breve(1)
/99/       -> registro(2)
```
- En la cabecera se puede indicar un número de comandancia en sustitución de la torre con la opción `--comandancia`. Pasando de emplear tres dígitos por torre a dos.
  El grupo de cabecera pasaría del formato `torre de origen(3) + torre de destino(3)` a `comandancia de origen(2) + comandancia de destino(2)`.

- Si se indica alguna torre con valor '0', se suprime la representación de la misma. El mensaje generado sólo lleva registro de una única torre o comandancia.
  El grupo de torre/comandancia podría tener cualquiera de los siguientes formatos:
```
/001051/ -> torre de origen(3) + torre de destino(3)
/0109/   -> comandancia de origen(2) + comandancia de destino(2)
/051/    -> torre(3)
/09/     -> comandancia(2)
```
- Opcionalmente, mediante el uso de una contraseña `telop --pwd '123'`, se permite encriptar/desencriptar el contenido del mensaje, manteniendo libre la cabecera. El método emplea Format-preserving, Feistel-based encryption (FFX), generando una cadena de números de apariencia aleatoria para quien intente descodificar el mensaje sin emplear la contraseña de encriptación.


### Más información

Revisión Código Telégrafo Óptico Mathé - [Academia.edu](https://www.academia.edu/109790572/Revisi%C3%B3n_C%C3%B3digo_Tel%C3%A9grafo_%C3%93ptico_Math%C3%A9)

Análisis sobre la presencia de niebla en las líneas civiles de telegrafía óptica - [Academia.edu](https://www.academia.edu/110399137/An%C3%A1lisis_sobre_la_presencia_de_niebla_en_las_l%C3%ADneas_civiles_de_telegraf%C3%ADa_%C3%B3ptica)

Instrucción General para el Servicio de Transmisión - [Academia.edu](https://www.academia.edu/122482646/Instrucci%C3%B3n_General_para_el_Servicio_de_Transmisi%C3%B3n)


```   
Título:		Historia de la telegrafía óptica en España
Autor:		Olivé Roig, Sebastián
Fecha de pub.:	1990
Páginas: 	101
Fuente:		Foro Histórico de las Telecomunicaciones

Título:		Instrucción general para el servicio de transmisión 
Autor:		José Maria Mathé
Fecha de pub.:	1850
Páginas:	24
Fuente:		Biblioteca Museo Postal y Telegráfico

Título:		Instrucción general para los torreros en el servicio telegráfico
Autor:		Manuel Varela y Limia
Fecha de pub.:	1846
Páginas:	22(incompleto)
Fuente:		Biblioteca Museo Postal y Telegráfico

Título:		Telégrafos militares : instrucción para los torreros y cartilla de servicio interior y señales particulares
Autor:		José Maria Mathé
Fecha de pub.:	1849
Páginas:	25
Fuente:		Biblioteca Virtual de Defensa

Wikipedia:	https://es.wikipedia.org/wiki/Tel%C3%A9grafo_%C3%B3ptico

Título:		Historia de la telegrafía
Fecha de pub.:	2012
Autor:		Fernando Fernández de Villegas / Amateur radio club Orense
Url:		http://www.ea1uro.com/eb3emd/Telegrafia_hist/Telegrafia_hist.htm

Título:		Estudio de la red de telegrafía óptica en España
Autor:		Capdevila Montes, Enrique. Slepoy Benites, Paula
Fecha de pub.:	2012
Páginas: 	456
Fuente:		Internet

Título:		Tratado de telegrafía
Autor:		Suárez Saavedra, Antonino  
Fecha de pub.:	1880-1882
Páginas:	665 tomo 1 (interesantes 148-153)
Fuente:		Biblioteca Digital Hispánica

Título:		Tratado de telegrafía y nociones suficientes de la posta 
Autor:		Suárez Saavedra, Antonino  
Fecha de pub.:	1870
Páginas:	605 (interesantes 51-55)
Fuente:		Biblioteca Digital Hispánica

Título:		Diccionario y tablas de transmisión para el telégrafo militar de noche y día
Autor:		José Maria Mathé
Fecha de pub.:	1849
Páginas:	47
Fuente:		Biblioteca Nacional

Título:		Diccionario y tablas de transmisión para el telégrafo militar de noche y día compuesto de órden del Exmo. señor Marqués del Duero
Autor:		José Maria Mathé
Fecha de pub.:	1849
Páginas:	310
Fuente:		Biblioteca Virtual de Defensa
Url:		http://bibliotecavirtualdefensa.es/BVMDefensa/es/consulta/registro.do?id=42216

Título:		Diccionario de Telégrafos (diccionario frasológico)
Autor:		Dirección General de Telégrafos
Fecha de pub.:	1858
Páginas:	415
Fuente:		Universidad Complutense / Google Books
Url:		https://books.google.es/books?id=mMIPmN8YXc8C

Título:		De torre en torre: Mensajes codificados en los cielos de la meseta
Autor:		Pasquale de Dato / Yolanda Hernández Navarro
Fecha de pub.:	2015
Páginas:	18
Fuente:		Revista Oleana Nº 30 - Ayuntamiento de Requena

Título:		The early history of data networks
Autor:		Gerard J. Holzmann / Björn Pehrson
Fecha de pub.:	1994
Páginas:	304
Fuente:		Internet Archive Open Library / https://archive.org/details/earlyhistoryofda0000holz

Título:		El progreso con retraso: La telegrafía óptica en la provincia de Cuenca
Autor:		Jesús López Requena
Fecha de pub.:	2012
Páginas:	354

Título:		Distintas etapas de la telegrafía óptica en España
Autor:		Olivé Roig, Sebastián
Fecha de pub.:	2007
Páginas: 	16
Fuente:		Cuadernos de Historia Contemporánea Nº 29
Url:		https://revistas.ucm.es/index.php/CHCO

Título:		Semanario Pintoresco Español. Segunda Serie - Tomo III
Artículo:	Telégrafos Españoles (p.155) y Rectificación (p.168)
Autor:		F. Navarro Villoslada
Fecha de pub.:	1841
Url:		Google Books - https://books.google.es/books?id=pVJfAAAAcAAJ

VV/AA:		https://forohistorico.coit.es/index.php/wiki-telegrafia-optica/category/bibliografia-y-referencias
```   

### Versión web

https://rfmoz.github.io/telop/

