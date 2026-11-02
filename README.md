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
Servicio:	 Despacho
Prioridad:	 0 - General
T. Origen:	 001
T. Destino:	 052
Hora y Día:	 23:10 08
Registro:	 00
Novenales:	 04.2
Mensaje txt:	 'Telegrama de prueba'
--------------------------------------------------------------------------------

Mensaje:	 0/0x1052/2310x80x042/5x1421x41/627102x10/9x13149x2/52730141x/10/0

--------------------------------------------------------------------------------
```

**Descodificar mensaje:**
```
$ telop '0/0x1052/2310x80x042/5x1421x41/627102x10/9x13149x2/52730141x/10/0'
--------------------------------------------------------------------------------
Servicio:	 Despacho
Prioridad:	 0 - General
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
usage: telop [-h] [-s {0,1,2,3,5,6}] [-p {0,4,8}] [-t ORIGEN:DESTINO] [-b] [--diccionario] [--pwd PWD] [-r [nº]] [-i [nº]] [--raw] [-v] [--version] [-z {0,1}] [mensaje]

positional arguments:
  mensaje               texto del mensaje entre ' '

options:
  -h, --help            show this help message and exit
  -s {0,1,2,3,5,6}, --servicio {0,1,2,3,5,6}
                        servicio/tipo de mensaje -> 0-despacho | 1-rectificación | 2-interno | 3-vigilancia | 5-continuación | 6-acuse recibo
  -p {0,4,8}, --prioridad {0,4,8}
                        prioridad -> 0-general | 4-urgente | 8-urgentísimo
  -t ORIGEN:DESTINO, --torres ORIGEN:DESTINO
                        torres origen:destino (001:051) o torre única (051) o comandancias (01:05) o comandancia única (05)
  -b, --breve           formato fecha y hora reducido
  --diccionario                muestra el diccionario de codificación
  --pwd PWD             cifra el mensaje con la contraseña indicada
  -r [nº], --registro [nº]
                        nº registro despacho
  -i [nº], --indicación [nº]
                        indicación particular del mensaje (solo en servicios 1, 2, 3 y 6)
  --raw                 salida sin formato
  -v, --verbose         debug
  --version             show program's version number and exit
  -z {0,1}              proceso a ejecutar -> (auto) | 0-codificar | 1-descodificar
```


### Ejemplos de cada tipo de mensaje

**Tipo 0 — Despacho**

Mensaje para transmitir texto de uso general con diferentes prioridades.

- Codificar texto de forma básica de la torre '001' (por defecto) a la '041':
    > telop -t 001:041 'Texto ejemplo'
- Mensaje urgentísimo con registro '12', origen '010' y destino '050':
    > telop -p 8 -r 12 -t 010:050 'Texto'


**Tipo 2 — Servicio interno**

Para indicaciones internas de servicio.

- Mensaje interno de la torre '001' (por defecto) a la '045' con código 10:
    > telop -s 2 -t 001:045 -i 10
- Mensaje interno de la torre '045' a la '001' con código 11 y formato de fecha breve:
    > telop -s 2 -t 045:001 -i 11 -b


**Tipo 3 — Vigilancia**

Para verificar el estado de la línea. Se envían cada media hora, desde la cabecera al final de la línea y los ramales.

Su recepción se confirma mediante la devolución de otro mensaje de vigilancia, indicando las torres oportunas.

- Mensaje inicial con valor '99' como comodín para todos los extremos de línea y ramales. Origen implícito ('0'), formato fecha breve:
    > telop -s 3 -t 00:99 -b
- Mensaje de respuesta con recepción correcta '0'. Comandancia de origen '07' y destino '01':
    > telop -s 3 -t 07:01 -i 0


**Tipo 5 — Continuación**

Indica la reanudación de un despacho detenido en cualquier torre, habitualmente por causas meteorológicas o cruce con otra comunicación de mayor prioridad.

- Retomar la transmisión del mensaje con torre de origen '009' y referencia '43':
    > telop -s 5 -t 009 -r 43


**Tipo 6 — Acuse de recibo**

Confirmación de recepción de un despacho, indicando el estado.

- Recepción correcta de despacho con registro '12' desde la torre '040' a la torre '001':
    > telop -s 6 -t 040:001 -r 12
- Recepción por niebla '1' de mensaje con registro '17' desde la torre '030' a la torre '001':
    > telop -s 6 -t 030:001 -r 17 -i 1


**Tipo 1 — Rectificación**

Solicitud de anulación o retransmisión de un despacho, según su número de registro.

- Repetir '6' mensaje con registro '23' desde la torre '021' a la '001':
    > telop -s 1 -t 021:001 -i 6 -r 23
- Anular '9' mensaje con registro '12' desde la torre '021' a la '001':
    > telop -s 1 -t 021:001 -i 9 -r 12


### Modificaciones del formato en cabecera

**Fecha con formato corto**

En cualquier mensaje con fecha se puede pasar el argumento '-b' para utilizar el formato reducido:

- Mensaje con origen '010', destino '021' y formato de fecha breve:
    > telop -t 010:021 -b 'Texto'


**Sustituir indicación de torre por comandancia**

Con `-t` se indica el par origen:destino. Ambos números deben tener 3 dígitos (torre) o ambos 2 (comandancia).
El grupo de cabecera puede tener el formato `torre de origen(3) + torre de destino(3)` o `comandancia de origen(2) + comandancia de destino(2)`.


- Mensaje con comandancia de origen '01' y destino '07':
    > telop -t 01:07 'Texto'
- Mensaje con torre origen '001' y destino '012':
    > telop -t 001:012 'Texto'
- Mensaje con una sola torre '052':
    > telop -t 052 'Texto'
- Mensaje con una sola comandancia '02':
    > telop -t 02 'Texto'




Nota: Telop permite cifrar el contenido del mensaje con una contraseña (`telop --pwd '123'`), preservando la cabecera en claro. Se utiliza cifrado FFX (Format-preserving, Feistel-based encryption), que produce una cadena numérica de apariencia aleatoria para quien intente descodificar el mensaje sin la contraseña.


### Estructura del mensaje

En el siguiente esquema se establece el orden de los valores en el mensaje:

```
 A/B/   C  /     D     / E / - /
 | |    |        |       |   |
 | |    |        |       |   +-- - texto(*)
 | |    |        |       +------ E indicación particular a cada tipo de mensaje([0-3])
 | |    |        +-------------- D hora(2) + minutos(2) + día(2) + registro(2) + extensión(3)
 | |    +----------------------- C torre de origen(3) + torre de destino(3)
 | +---------------------------- B prioridad(1)
 +------------------------------ A servicio(1)

   0/0x10x5/2341040x013/252730141/1x0/0 -> Mensaje general
   |    |        |       \         /  |
   |    |        |        \       /   +-- B prioridad(1)
   |    |        |         +-----+------- - novenales de mensaje(*)
   |    |        +----------------------- D hora(2) + minutos(2) + día(2) + registro(2) + ext.(3)
   |    +-------------------------------- C torre de origen(3) + torre de destino(3)
   +------------------------------------- B prioridad(1)

 2  /0x10x5/234104     /0x1    /2 -> Comunicación interna
 |      |        |       |      |
 |      |        |       |      +-- A servicio(1)
 |      |        |       +--------- E indicación código interno([2-3])
 |      |        +----------------- D hora(2) + minutos(2) + día(2)
 |      +-------------------------- C torre de origen(3) + torre de destino(3)
 +--------------------------------- A servicio(1)

 3  /0x10x5/234104     /0x -> Vigilancia
 |      |        |       |
 |      |        |       +-- E indicación estado, solo en recepción (opcional)([0-2])
 |      |        +---------- D hora(2) + minutos(2) + día(2)
 |      +------------------- C torre de origen(3) + torre de destino(3)
 +-------------------------- A servicio(1)

 6/0/0x10x5/2341040x   /0x -> Acuse de recibo
 | |    |        |       |
 | |    |        |       +-- E indicación estado acuse de recibo([1-2])
 | |    |        +---------- D hora(2) + minutos(2) + día(2) + registro mensaje recibido(2)
 | |    +------------------- C torre de origen(3) + torre de destino(3)
 | +------------------------ B prioridad(1)
 +-------------------------- A servicio(1)

 5/0/  0x1 /    03 -> Continuación
 | |    |        |
 | |    |        +-- D registro mensaje a continuar(2)
 | |    +----------- C torre de origen(3)
 | +---------------- B prioridad(1)
 +------------------ A servicio(1)

 1/0/0x10x5/    04     / 6/2/3/4 -> Rectificación
 | |    |        |       | \   /
 | |    |        |       |  +-+-- - novenales a repetir(opcional)(*)
 | |    |        |       +------- E indicación tipo de petición(1)
 | |    |        +--------------- D registro mensaje rectificado(2)
 | |    +------------------------ C torre de origen(3) + torre de destino(3)
 | +----------------------------- B prioridad(1)
 +------------------------------- A servicio(1)
```

Cada mensaje puede llevar un suplemento final indicando las interrupciones sufridas durante la transmisión. Se puede repetir el número de veces necesario.
El grupo que comprende la hora, los minutos y día es opcional y puede estar compuesto por todos esos valores o solo por la hora y los minutos. La causa corresponde a la misma numeración del acuse de recibo: 1-niebla | 2-ausencia | 3-ocupada | 4-avería. El formato es el siguiente:

```
/ X /  Y  / Z -> Suplemento de interrupción
  |    |    |
  |    |    +-- Z causa(1)
  |    +------- Y hora(2) + minutos(2) + día(2)
  +------------ X torre(3)

/011/183012/2 -> Suplemento de interrupción
  |    |    |
  |    |    +-- Z causa(1)
  |    +------- Y hora(2) + minutos(2) + día(2)
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
/12150199/ -> hora(2) + minutos(2) + día(2) + registro(2)
/121501/   -> hora(2) + minutos(2) + día(2)
/37199/    -> hora_breve(2) + día_breve(1) + registro(2)
/1215/     -> hora(2) + minutos(2)
/371/      -> hora_breve(2) + día_breve(1)
/99/       -> registro(2)
```

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
