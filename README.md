telop
=======

Telop (TELégrafo ÓPtico) - Utilidad para codificar y descodificar mensajes de texto mediante una interpretación del código telegráfico de José María Mathé. Permite recrear el sistema utilizado en la red de telegrafía óptica de España a mediados del s. XIX.

También disponible como [versión web](https://rfmoz.github.io/telop/).


### Instalación

Requiere Python 3. Descargar el archivo [telop](telop) y ejecutar:

```
$ python3 telop
```


### Uso básico

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


### Opciones del programa
```
usage: telop [-h] [-t {0,1,2,3,4,5,6,8}] [-o [nº]] [-d [nº]] [-b] [-c] [--diccionario] [--pwd PWD] [-r [nº]] [-s SUFIJO] [--raw] [-v] [--version] [-z {0,1}] [mensaje]

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
  -c, --comandancia     comandancia como origen / destino
  --diccionario         muestra el diccionario de codificación
  --pwd PWD             cifra el mensaje con la contraseña indicada
  -r [nº], --registro [nº]
                        nº registro despacho
  -s SUFIJO, --sufijo SUFIJO
                        sufijo del mensaje (solo tipos 1, 2, 3 y 6)
  --raw                 salida sin formato
  -v, --verbose         debug
  --version             show program's version number and exit
  -z {0,1}              proceso a ejecutar -> (auto) | 0-codificar | 1-descodificar
```


### Ejemplos de cada tipo de mensaje

**0 - General**
**4 - General urgente**
**8 - General urgentísimo**

Mensaje para transmitir texto de uso general con diferentes prioridades.

- Codificar texto de forma básica de la torre '001' (por defecto) a la '041':
    > telop -d 41 'Texto ejemplo'
- Mensaje urgentísimo '8' con registro '12', origen '010' y destino '050':
    > telop -t 8 -r 12 -o 10 -d 50 'Texto'


**2 - Servicio interno**

Para indicaciones internas de servicio.

- Mensaje interno de la torre '001' (por defecto) a la '045' con código 10:
    > telop -t 2 -d 45 -s 10
- Mensaje interno de la torre '045' a la '001' con código 11 y formato de fecha breve:
    > telop -t 2 -o 45 -d 1 -s 11 -b


**3 - Vigilancia**

Para verificar el estado de la línea. Se envían cada media hora, desde la cabecera al final de la línea y los ramales.

Su recepción se confirma mediante la devolución de otro mensaje de vigilancia, indicando las torres oportunas.

- Mensaje inicial con valor '99' como comodín para todos los extremos de línea y ramales. Origen implícito ('0'), formato fecha breve:
    > telop -t 3 -o 0 -d 99 -c -b
- Mensaje de respuesta con recepción correcta '0'. Comandancia de origen '07' y destino '01':
    > telop -t 3 -o 7 -d 1 -c -s 0


**5 - Continuación**

Indica la reanudación de un mensaje general detenido en cualquier torre, habitualmente por causas meteorológicas o cruce con otra comunicación de mayor prioridad.

- Retomar la transmisión del mensaje con torre de origen '009' y referencia '43':
    > telop -t 5 -o 9 -r 43


**6 - Acuse de recibo**

Confirmación de recepción de un mensaje general, indicando el estado.

- Recepción correcta de mensaje con registro '12' desde la torre '040' a la torre '001':
    > telop -t 6 -o 40 -d 1 -r 12
- Recepción por niebla '1' de mensaje con registro '17' desde la torre '030' a la torre '001':
    > telop -t 6 -o 30 -d 1 -r 17 -s 1


**1 - Rectificación**

Solicitud de anulación o retransmisión de un mensaje general, según su número de registro.

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

Con el argumento '-c', en cualquier mensaje se puede sustituir el formato de torre, de tres cifras, por el de comandancia, de dos cifras.

- Mensaje con origen '01', destino '07' y opción de comandancia:
    > telop -o 1 -d 7 -c 'Texto'


**Indicar solo una torre o comandancia**

Es posible generar un mensaje con solo un número de torre en vez del formato habitual que lleva dos, origen y destino. Con solo una torre, se deduce el origen o destino según el sentido del mensaje y la posición que ocupa la torre en la línea. Para ello, se pasa el valor '0' a la opción de origen `telop -o 0` o destino `telop -d 0`.


Nota: Telop permite cifrar el contenido del mensaje con una contraseña (`telop --pwd '123'`), preservando la cabecera en claro. Se utiliza cifrado FFX (Format-preserving, Feistel-based encryption), que produce una cadena numérica de apariencia aleatoria para quien intente descodificar el mensaje sin la contraseña.


### Estructura del mensaje

En el siguiente esquema se establece el orden de los valores en el mensaje:

```
A /  B  /   C   / D / - / A
|    |      |     |   |   |
|    |      |     |   |   +-- A tipo de servicio y prioridad (opcional)(1)
|    |      |     |   +------ - texto (opcional)(*)
|    |      |     +---------- D sufijo particular a cada tipo de mensaje([0-3])
|    |      +---------------- C hora(2) + minutos(2) + dia(2) + registro(2)
|    +----------------------- B torre de origen(3) + torre de destino(3)
+---------------------------- A tipo de servicio y prioridad(1)


0/0x10x5/2341040x/013/252730141/1x0/0 -> Mensaje general
|    |      |     |   \          /  |
|    |      |     |    \        /   +-- A tipo de servicio y prioridad(1)
|    |      |     |     +------+------- - novenales de mensaje(*)
|    |      |     +-------------------- D sufijo nº de novenales completos(2) y nº digitos resto(1)
|    |      +-------------------------- C hora(2) + minutos(2) + dia(2) + registro(2)
|    +--------------------------------- B torre de origen(3) + torre de destino(3)
+-------------------------------------- A tipo de servicio y prioridad(1)

2/0x10x5/ 234104 /0x1/2 -> Comunicación interna
|    |      |     |   |
|    |      |     |   +-- A tipo de servicio y prioridad(1)
|    |      |     +------ D sufijo código interno([2-3])
|    |      +------------ C hora(2) + minutos(2) + dia(2)
|    +------------------- B torre de origen(3) + torre de destino(3)
+------------------------ A tipo de servicio y prioridad(1)

3/0x10x5/ 234104 /0x -> Vigilancia
|    |      |     |
|    |      |     +-- D sufijo estado, solo en recepción (opcional)([0-2])
|    |      +-------- C hora(2) + minutos(2) + dia(2)
|    +--------------- B torre de origen(3) + torre de destino(3)
+-------------------- A tipo de servicio y prioridad(1)

6/0x10x5/2341040x/0x -> Acuse de recibo
|    |      |     |
|    |      |     +-- D sufijo estado acuse de recibo([1-2])
|    |      +-------- C hora(2) + minutos(2) + dia(2) + registro mensaje recibido(2)
|    +--------------- B torre de origen(3) + torre de destino(3)
+-------------------- A tipo de servicio y prioridad(1)

5/  0x1 /   03  -> Continuación
|    |      |
|    |      |
|    |      +-- C registro mensaje a continuar(2)
|    +--------- B torre de origen(3)
+-------------- A tipo de servicio y prioridad(1)

1/0x10x5/   04   /6/2/3/4 -> Rectificación
|    |      |     | \   /
|    |      |     |  +-+-- - novenales a repetir(opcional)(*)
|    |      |     +------- D sufijo tipo de petición(1)
|    |      +------------- C registro mensaje rectificado(2)
|    +-------------------- B torre de origen(3) + torre de destino(3)
+------------------------- A tipo de servicio y prioridad(1)
```

Cada mensaje puede llevar un suplemento final indicando las interrupciones sufridas durante la transmisión. Se puede repetir el número de veces necesario.
El grupo que comprende la hora, los minutos y día es opcional y puede estar compuesto por todos esos valores o solo por la hora y los minutos. La causa corresponde a la misma numeración del acuse de recibo: 1-niebla | 2-ausencia | 3-ocupada | 4-avería. El formato es el siguiente:

```
/ X /  Y  / Z -> Suplemento de interrupción
  |    |    |
  |    |    +-- Z causa(1)
  |    +------- Y hora(2) + minutos(2) + dia(2)
  +------------ X torre(3)

/011/183012/2 -> Suplemento de interrupción
  |    |    |
  |    |    +-- Z causa(1)
  |    +------- Y hora(2) + minutos(2) + dia(2)
  +------------ X torre(3)

/011/1830  /2 -> Suplemento de interrupción
  |    |    |
  |    |    +-- Z causa(1)
  |    +------- Y hora(2) + minutos(2)
  +------------ X torre(3)

/011       /2 -> Suplemento de interrupción
  |         |
  |         +-- Z causa(1)
  +------------ X torre(3)
```


### Formato de cabecera

Se puede emplear un formato de fecha y hora reducido con la opción `--breve`, a costa de una precisión de 15 minutos.
Dos dígitos representan la hora y los minutos, teniendo en cuenta el cuarto de hora en que se encuentran los minutos. Se suma 0, 25, 50 o 75 a la hora (00 a 24) según el primer, segundo, tercer o último cuarto de hora. Por ejemplo: 12:05 → 12, 12:20 → 37, 12:40 → 62, 12:55 → 87.
El día solo mantiene el último dígito, es decir, se representa igual el día 1 que el 11 o el 21.
Esta fue la modificación más curiosa de las empleadas y conocidas; el resto básicamente reducían tamaño omitiendo información fácilmente interpretable por la situación del emisor y receptor.
El grupo de cabecera podría tener cualquiera de los siguientes formatos (aunque no todos se llegan a utilizar):
```
/12150199/ -> hora(2) + minutos(2) + dia(2) + registro(2)
/121501/   -> hora(2) + minutos(2) + dia(2)
/37199/    -> hora_breve(2) + dia_breve(1) + registro(2)
/1215/     -> hora(2) + minutos(2)
/371/      -> hora_breve(2) + dia_breve(1)
/99/       -> registro(2)
```

Se puede indicar un número de comandancia en sustitución de la torre con la opción `--comandancia`, pasando de tres dígitos por torre a dos.
El grupo de cabecera pasaría del formato `torre de origen(3) + torre de destino(3)` a `comandancia de origen(2) + comandancia de destino(2)`.

Si se indica alguna torre con valor '0', se suprime su representación. El mensaje generado solo lleva registro de una única torre o comandancia.
El grupo de torre/comandancia podría tener cualquiera de los siguientes formatos:
```
/001051/ -> torre de origen(3) + torre de destino(3)
/0109/   -> comandancia de origen(2) + comandancia de destino(2)
/051/    -> torre(3)
/09/     -> comandancia(2)
```


### Diccionario de codificación

Cada carácter del mensaje se codifica según la posición que ocupa en el siguiente diccionario. En el sistema original se empleaba un diccionario frasológico que asignaba un código numérico a frases completas, reduciendo la longitud del mensaje. Aquí se sustituye por una tabla de caracteres individuales, lo que resulta en un telegrama más extenso pero más versátil.

```
$ telop --diccionario
----------
Nº - Valor
----------
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


### Más información

Revisión Código Telégrafo Óptico Mathé - [Academia.edu](https://www.academia.edu/109790572/Revisi%C3%B3n_C%C3%B3digo_Tel%C3%A9grafo_%C3%93ptico_Math%C3%A9)
