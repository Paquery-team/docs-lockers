# 

# 

![][image1]\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

# **Especificaciones técnicas smart locker V1.6**

# 

# Índice {#índice}

[Índice	2](#índice)

[HISTORIAL DE CAMBIOS	3](#historial-de-cambios)

[Glosario	4](#glosario)

[Diagrama bloques	7](#diagrama-bloques)

[Conexiones	8](#conexiones)

[Componentes	11](#componentes)

[Raspberry Pi Central	11](#raspberry-pi-central)

[Montaje físico en torre	12](#montaje-físico-en-torre)

[PCB de control	12](#pcb-de-control)

[Esquemático	12](#esquemático)

[Diseño de PCB	12](#diseño-de-pcb)

[Descripción PCB	14](#descripción-pcb)

[Armado físico PCB	15](#armado-físico-pcb)

[Montaje físico en torre	16](#montaje-físico-en-torre-1)

[Módulo de relevadores	17](#módulo-de-relevadores)

[Montaje físico en torre	17](#montaje-físico-en-torre-2)

[Chapa electrónica	18](#chapa-electrónica)

[Montaje físico en torre	19](#montaje-físico-en-torre-3)

[iPad	20](#ipad)

[Montaje físico en torre	20](#montaje-físico-en-torre-4)

[Arnés de conexiones	22](#arnés-de-conexiones)

[Apertura de emergencia	23](#apertura-de-emergencia)

[Montaje físico en torre	24](#montaje-físico-en-torre-5)

[No break	25](#no-break)

[Montaje físico en torre	25](#montaje-físico-en-torre-6)

[Switch de puertos ethernet	26](#switch-de-puertos-ethernet)

[Montaje físico en torre	26](#montaje-físico-en-torre-7)

[Fuente ATX	26](#fuente-atx)

[Montaje físico en torre	26](#montaje-físico-en-torre-8)

## Historial de cambios {#historial-de-cambios}

| Version | Autor | Fecha | Comentarios |
| :---: | ----- | ----- | ----- |
| 1.0 | [Erik Domínguez Vargas](mailto:erik.dominguez@chazki.com) | 15 de mayo de 2023 | Primera definición de especificaciones |
| 1.1 | [Erik Domínguez Vargas](mailto:erik.dominguez@chazki.com) | 22 de junio de 2023 | Se agregan diagramas de bloques y conexión |
|  |  |  |  |

# Glosario {#glosario}

Locker. Se refiere al hardware TOTAL instalado en una sucursal por Chazki, puede estar integrado por múltiples torres.

![][image2]

Torre. Se refiere al número de separaciones verticales que se pueden hacer de un locker completo. Está integrado por múltiples boxes. 

![][image3]

Box. Se refiere a la unidad mínima en la que se pueden dividir las torres de cada locker. Se abre una por cada operación que realice un cliente del locker

![][image4]

Torre central. Se refiere a la torre donde se encuentra instalada la pantalla del locker así como parte de los componentes que permiten el funcionamiento del locker.  
![][image5]

Canaleta. Se refiere al espacio físico que resguarda los componentes electrónicos específicos para la apertura de cada puerta. Cada torre tiene construida una canaleta.

# Diagrama bloques {#diagrama-bloques}

El objetivo final de la electrónica montada en el locker siempre será la correcta apertura de las chapas, para ello, se interconectan una serie de componentes como se muestra a continuación:

La electrónica desarrollada en el locker se puede considerar modular y los módulos que se pueden diferenciar son los siguientes:

- Raspberry Pi Central (3B). Usada para el montaje del software en local del sistema de lockers, se comunica con el iPad de manera inalámbrica y con cada Raspberry Zero por medio de cableado ethernet, todos teniendo un punto de acceso en común que es el modem/router del locker. El sistema operativo se graba en una memoria Micro SD de 32 Gb y se monta en la Raspberry, se emplea una Raspberry 3B por locker y se monta en el box central. Se energiza con 5V que se toman del PCB de control. El sistema operativo se encuentra configurado de manera genérica. Se requiere una configuración externa por parte de Chazki y se lleva a cabo una vez que se ha conectado y energizado todo en el locker. 

 

- PCB de control. Desarrollado por Chazki, tiene como finalidad comunicarse con la Raspberry Central y enviar la señal de activación a cada  uno de los módulos de relé. Su salida de 5V alimenta la Raspberry Central del locker. En este PCB se monta una raspberry Zero mediante los headers hembra. El sistema operativo se graba en una memoria Micro SD de 32 Gb y se monta en la Raspberry, se emplea una Raspberry por torre y se monta en la canaleta. La Raspberry Zero se conecta al router mediante un adaptador y un cable ethernet. Este sistema operativo se encuentra configurado de manera genérica. Se requiere una configuración externa por parte de Chazki y se lleva a cabo una vez que se ha conectado y energizado todo en el locker.

- Módulo de relevadores. Esta placa tiene como objetivo único recibir las señales de salida de la Raspberry de cada torre, cambiar el estado de una bobina y con ello energizar el solenoide de la chapa que tenga conectada. 

- Chapa electronica. Es el actuador final del locker, se energiza para poder liberar un solenoide que empuja el pestillo de cada puerta y así se abre la puerta.

- iPad. El cargador del iPad se encuentra permanentemente conectado al no break, así el iPad no se apaga, por otro lado, para poder comunicarse con la raspberry central se conecta de manera inalámbrica al router/modem. 

- No break. El no break utilizado se emplea para generar la conexión eléctrica para todos los componentes usados en el locker y a su vez, permite que el locker siga encendido por un periodo después de perder la energía proporcionada por el enchufe asignado por el cliente. 

## Conexiones {#conexiones}

Del mismo diagrama a bloques se pueden describir las siguientes conexiones: 

1. Se refiere a la conexión eléctrica que se requiere para energizar todos los componentes del locker, se coloca en la parte trasera de la torre maestra del locker, solo se ocupa una por locker. 

 Los componentes ocupados para generar esta conexión son:

| Componente | Cantidad | Marca | Modelo | Link |
| ----- | ----- | ----- | ----- | ----- |
| Conector de tres hilos, Hoja recta | 1 | Leviton | **NEMA:** 5-15 R | https://www.leviton.com/es/products/515cv |
| Clavija reforzada de hoja recta | 1 | Leviton | **NEMA:** 5-15 R | https://www.leviton.com/es/products/515pv |
| Cable uso rudo 3 líneas | N/A | N/A | **N/A** | N/A |

El objetivo será construir un cable que dentro del box central del locker tenga un conector y al exterior quede una clavija. La distancia del cable no está determinada, ya que se debe personalizar al cliente y el scouting que se haya realizado. 

2. Se refiere a la conexión electrónica que existe entre la salida de \+5VDC del PCB de control y la raspberry central de cada locker. Se debe cablear con el mismo calibre del arnés de conexión del locker (AWM 1560). Se usan la bornera de salida de \+5VDC y la de GND.  
     
3. Se refiere a la conexión electrónica que existe entre la salida de \+5VDC y \+12VDC de la fuente ATX y el PCB de control. Para realizar esta conexión se debe colocar un conector molex hembra en el interior del box central e integrarlo a todo el arnés del locker.  
     
4. Se refiere a la conexión eléctrica que existe entre una de las salidas “soportadas” del no break y la fuente ATX, se conecta mediante el cable incluido siempre en la fuente.  
     
5.  Se refiere a la conexión eléctrica que existe entre una de las salidas “aterrizadas” del no break y el cargador del iPad mediante un cable lightning. Es importante que la conexión no se realice en uno de los contactos soportados del no break, ya que el iPad cuenta con su propia batería y se mantendrá encendida un tiempo aún después de cortar la energía eléctrica del contacto asignado por el cliente.

6. Se refiere a la conexión de datos que se requiere entre la raspberry y el router, se requiere cablear con un cable ethernet cat5 o superior. Estos componentes se cablean fácilmente dentro del box central por medio de un switch de entre 5 y 8 puertos.

7. Se refiere a la conexión de datos que se requiere entre la raspberry y el router, se requiere cablear con un cable ethernet CAT5 o superior. Estos componentes se cablean fácilmente dentro del box central por medio de un switch de entre 5 y 8 puertos.  
     
8. Se refiere a las señales de salida que activarán cada uno de los módulos de relé, este cableado deberá estar incluido en el arnés de conexiones.

9. Se refiere a la conexión necesaria entre la fuente de alimentación y el módulo de relé, se debe alimentar con \+12VDC y en caso de ser la torre central se debe tomar \+5VDC para energizar desde el PCB de control la Raspberry Central, como más adelante se detalla.

10. Se refiere a la señal de alimentación de las chapas, cuando la bobina del módulo de relé cambia de estado, la chapa deberá ser alimentada con \+12VDC lo que provocará la activación del solenoide, por tanto se abrirá la puerta.  
      
11. Se refiere a la conexión entre el no break y el router, se debe energizar siempre mediante la fuente asignada por el fabricante, siempre se debe recibir el router junto con la fuente de éste router. 

12. Se refiere a la conexión de datos existente entre el iPad y el router, ésta se realiza de manera inalámbrica y solo se configura para las pruebas finales del locker justo antes de la instalación en sitio.

13. Se refiere al cable de interconexión de energía entre torres, este debe llevar los voltajes \+12VDC y \+5VDC desde la fuente de alimentación en la torre maestra hasta la esclava, esta conexión se logra mediante el arnés de conexiones.

14. Se refiere a la conexión que debe haber entre el PCB de control de la(s) torre(s) esclava(s) y el router montado en la torre maestra. se realiza mediante un par de cables ethernet. El primer cable se coloca desde la conexión con el adaptador ethernet de la Raspberry Zero montado en el PCB de control, hasta pasar por debajo del locker, y salir por el barreno para pasar los cables en la parte trasera, ahí se conecta a un cople y después se conecta el segundo cable ethernet que se ingresa por el barreno para pasar cables de la torre maestra, sube hasta el box central y se conecta al switch localizado en el box central. 

# Componentes {#componentes}

## Raspberry Pi Central {#raspberry-pi-central}

Se emplea una [Raspberry Pi 3B](https://www.raspberrypi.com/products/raspberry-pi-3-model-b/) en ella se carga un sistema operativo que además contiene el software necesario para controlar el locker, se deben configurar la BD y archivos de configuración, sin embargo estas configuraciones se pueden ver en el documento “Preparación de componentes”.

Se energiza con un par de cables integrados al arnés, estos salen de la bornera \+5VDC y GND del PCB de control y se suelda a los pines 2 y 6 como se muestra en la siguiente imagen:

![][image6]

Se conecta al router por medio de un cable ethernet Cat5 o superior.

## Montaje físico en torre {#montaje-físico-en-torre}

La Raspberry Pi Central solo se acomoda con su carcasa en el box central, ahí mismo se realiza la conexión al router mediante el cable ethernet. 

## PCB de control {#pcb-de-control}

### Esquemático {#esquemático}

El esquemático se debe analizar en el documento aparte [adjunto](https://drive.google.com/file/d/1q9q7bDgcV_Ge9U0btLgP8y4yJ90zJMVM/view?usp=share_link)

### Diseño de PCB {#diseño-de-pcb}

Tomando en cuenta el diagrama a bloques mencionado anteriormente junto con el esquemático, el Diseño de PCB sería el siguiente:   
Descripción  general del diseño: 

![][image7]![][image8]

Base de circuito impreso en cobre para control de tarjeta raspberry zero con expansión de puertos y base para alimentación directa de 12 volts.  
**Dimensiones**(máximas): 

- 9.5cm(alto) x 8.7cm(ancho)

**Material**: FR-4(kingboard)  
**Componentes**: pasivos, RoHS compliance

BOM:

![][image9]

Todos los componentes pueden ser sustituidos por sus partes equivalentes de otros fabricantes

### Descripción PCB {#descripción-pcb}

1. Sección de conversión de señales de salida. Se utiliza esta etapa debido a que los relevadores usan   **como señal de activación un pulso bajo** y con esta etapa se evita que mientras la Raspberry Zero no esté encendida o inicializada se abran las puertas, dada la configuración de inicialización de pines según el SO instalado. Básicamente los pines de la Raspberry se inicializan como bajos y al activarse, envían pulsos altos, por lo que solamente cuando se activa el pulso en el GPIO de la raspberry, se “cortocircuita” el pin de entrada del módulo de relé con GND.  
2. Está sección se usa en caso de que en las torres se desee emplear una Raspberry 3B como dispositivo esclavo, esta implementación en teoría funciona, sin embargo nunca se ha llegado a probar y no se sabe si los pines de las diferentes versiones de Raspberries son compatibles. Cuando se desea colocar una Raspberry Zero, solamente se debe “romper” la sección sobrante.  
3. Son headers hembra de paso común (2.54 mm) para colocar encima la Raspberry Zero y así conectar con las demás etapas del PCB.  
4. Aquí se debe montar un Step Down, para el diseño de este PCB se debe implementar específicamente Step Down del tipo LM2596 con su PCB ya armado que proporciona hasta 3A de salida. Siempre se debe regular la salida del Step Down a 5V, con una entrada de 12V, antes o ya montado en el PCB, pero **siempre antes de que se monte la Raspberry Zero** o ésta se quemará. El diseño a emplear de Step Down es [este](https://uelectronics.com/producto/modulo-regulador-ajustable-lm2596-dc-dc-step-down-3a-1-25-30v/), no se puede cambiar a menos que las medidas correspondan de manera exacta, o se haga la adaptación para montarlo, sin embargo no es aconsejable cambiarlo, sería mejor llevar a cabo otro diseño.  
5. LEDs indicadores, se implementaron para que el proceso de energizar y el procesamiento de la Raspberry sea fácil de reconocer de manera empírica.  
6. Dip Switch de 4 posiciones, dependiendo de la configuración que tenga de encendido/apagado en cada canal, la Raspberry reconocerá la combinación y asignará una IP diferente a la torre, lo que permitirá diferenciar entre las diferentes torres.  
7. Bornera de alimentación, por aquí se deben ingresar los \+12 VDC provenientes de la fuente ATX en caso de ser torre principal, y por otro lado, se debe usar la salida de \+5VDC en las torres maestras para alimentar la Raspberry 3B, en las demás torres no se ocupa esta salida.   
8. Borneras de salida, aquí se conectan las entradas de cada uno de los relés que a su vez abrirán las chapas. Es importante tener correctamente identificadas las entradas de los módulos de relé, ya que el PCB tiene numerado del 1 al 10 las salidas y se deben asignar a los módulos de relé de tal forma que las puertas de la torre se abran como la 1 la más alta (lejana al suelo) y la 10 la más baja (cercana al suelo). Este PCB no tiene la posibilidad de aumentar su número de salidas, para ello se debe hacer un diseño nuevo o agregar otro diseño conectado a este mismo PCB.

### Armado físico PCB {#armado-físico-pcb}

Como en la imagen anterior, se puede ver, existen varios grandes módulos que se describen a continuación:

1. Header hembra doble para montaje de Raspberry Pi Zero.   
2. LEDs indicadores de estado  
3. Dip switch de 4 botones para direccionamiento y asignación de IP para cada torre.  
4. Bornera de alimentación, \+12 VDC, \+5 VDC y GND  
5. Step Down de alimentación, configurado a convertir \+12VDC en \+5VDC  
6. Borneras de señales de salidas para conectar los relés desde la raspberry Zero \+3.3VDC. Las salidas están limitadas a 10\. (Máximo 10 boxes por torre)

### Montaje físico en torre {#montaje-físico-en-torre-1}

El PCB está diseñado para caber en la canaleta de los lockers versión 1.6, esto incluyendo el montaje de la raspberry así como el conector que usa para convertir micro USB en ethernet. 

Normalmente el PCB se monta junto con el arnés, por lo que el arnés completo puede probarse sin la necesidad de estar en la torre, solamente se requiere que se energice para realizar pruebas necesarias. 

Siempre será importante que el PCB este montado en un material no conductor 

![][image10]

## Módulo de relevadores {#módulo-de-relevadores}

Los detalles del módulo de relevadores pueden verse [aqui](http://www.handsontec.com/dataspecs/2Ch-relay.pdf). El módulo se conecta mediante cables ya colocados en el arnés. Las bobinas se energizan mediante \+5VDC de la fuente ATX, así mismo la potencia se toma de \+12VDC de la fuente ATX. Es importante dejar el header que trae colocado por default (entre JDVCC y VCC el módulo de relé para que funcione de manera correcta. 

### Montaje físico en torre {#montaje-físico-en-torre-2}

Para montar el relevador, se emplea la superficie de las chapas, sin embargo al ser metálicas, se debe poner detrás de cada módulo una placa de madera o de otro material aislante. Para sujetar el módulo de la chapa se emplea silicón caliente, es importante que se encuentren lo mejor montados posible ya que si se llegan a despegar, pudieran hacer contacto con la lámina del locker y hacer un corto. 

![][image11]

# Chapa electrónica {#chapa-electrónica}

Se emplea un modelo genérico con el siguiente diseño:

![][image12] 

Se energiza con \+12VDC y tiene un par de cables que funcionan como sensor de sí la chapa está cerrada o abierta, para el locker versión 1.6 estos cables no tienen ninguna funcionalidad, por lo tanto se llegan incluso a cortar los cables. 

Se debe respetar el modelo pidiendo a los fabricantes las especificaciones y comparándolas con el modelo actual, ya que mecánicamente el locker ya se encuentra hecho para este tipo de chapas, por lo que al intentar cambiarlo, se debe cambiar también el diseño.

### Montaje físico en torre {#montaje-físico-en-torre-3}

En la siguiente imagen puede verse cómo se debe montar la chapa en cada torre del locker. Se puede apreciar que algunas se montan con un módulo de relé “encima” y otras no, esto debido a que cada módulo controla dos chapas.

![][image13]

# iPad {#ipad}

Se emplean iPad desde el modelo 6 hasta el 9, sin embargo la 6 es de un cierto tamaño (9.7’’) y los iPads restantes 7,8 y 9 son de un tamaño diferente (10.2’’) por lo que debe considerarse de qué tamaño se colocará el iPad.

Se conecta a la energía eléctrica por medio del cable Lightning propio del iPad y su cargador, para la instalación y configuración de la app Lok Hub, solamente se emplea conexión Wifi que se configura al tener el router ya asignado al locker. 

### Montaje físico en torre {#montaje-físico-en-torre-4}

En la siguiente imagen puede verse cómo se debe montar el iPad en el box central.

![][image14]

# Arnés de conexiones {#arnés-de-conexiones}

En el siguiente diagrama se puede observar un diagrama del arnés y las conexiones que efectúa.

Como se puede observar, las líneas son una representación de la conexión necesaria para hacer funcionar cada una de las torres, ya que por ejemplo la línea de señales, está representada con un solo trazo, sin embargo ahí se conectan las 10 señales de activación de las chapas que salen desde el PCB de control. 

### Apertura de emergencia {#apertura-de-emergencia}

Se trata de una conexión que se hace hacia todas las chapas en caso de perder conexión al locker (por falta de energía o internet). Es una inyección de voltaje de 12V desde un conector “oculto” en la parte baja de la canaleta de la torre en cuestión.  

Se puede sustituir por un diseño de apertura mecánica. Sin embargo hasta ahora no se tiene ninguna propuesta. Se puede además mejorar la implementación actual, ya que solo se necesitan abrir la puerta superior y la inferior para poder así retirar la canaleta y posteriormente en caso de ser necesario abrir de manera manual las chapas que sea necesario.

Aqui el diagrama de conexión para generar la apertura de emergencia: 

Como se puede observar, se emplea un diodo ya que entre el cátodo del diodo y la chapa se tiene la conexión de la apertura y para que no se genere una conexión común entre todas las chapas, se pone el diodo. 

El diodo se monta junto con el cable al arnés correspondiente y se emple a el [1N4001](https://pdf1.alldatasheet.com/datasheet-pdf/view/14618/PANJIT/1N4001.html) que es un diodo de uso general 

### Montaje físico en torre {#montaje-físico-en-torre-5}

Como ya se mencionó, la apertura de emergencia forma parte del arnés de conexiones y ambos se montan a las torres en la canaleta de cada torre. Junto a las chapas. Se usan sujeta cinchos y cinchos para mantener el cableado en su lugar. El conector de la apertura de emergencia se coloca en la parte baja de la canaleta para poder sacarlo por la ranura existente con el conector de GND y \+12VDC listo y conectado como se mencionó anteriormente.

![][image15]

# No break {#no-break}

Como se mencionó anteriormente, se usa para generar la conexión eléctrica necesaria para el locker y además garantizar que el locker permanezca encendido por un cierto tiempo después de faltar la energía eléctrica en el enchufe proporcionado por el cliente. 

Se acomoda en el box central del locker y se energiza mediante la conexión descrita en el número 1 de la sección “Conexiones”.

Los componentes del locker se deben de conectar de la siguiente manera: 

**Contactos respaldados:** 

- Fuente ATX  
- Router  
- Switch de puertos ethernet

**Contactos aterrizados:** 

- Cargador iPad

Como se puede observar, para que el locker siga funcionando, después de una conexión eléctrica, se requiere que todos los dispositivos estén conectados a la sección de respaldados, ya que el iPad cuenta con su propia batería y continúa encendida por un tiempo después de desconectarlo de la energía. 

El no break usado hasta ahora y sus características, pueden verse en el siguiente link:

[https://koblenz.com.mx/productos/energia/no-break-nivel-de-respaldo-gold-5216-r/](https://koblenz.com.mx/productos/energia/no-break-nivel-de-respaldo-gold-5216-r/)

Hasta ahora no se tienen graves problemas con este modelo, por lo que ningún cambio ha sido propuesto.

### Montaje físico en torre {#montaje-físico-en-torre-6}

Como también ya se mencionó anteriormente, el no break se coloca dentro del box central y ahí se conectan todos los componentes. por lo que no debe haber mayor complicación a la hora de instalar este componente. 

# Switch de puertos ethernet {#switch-de-puertos-ethernet}

Como se puede ver en el diagrama de bloques al principio de este documento, al router se conectan por medio de cable ethernet por lo menos 3 dispositivos: dos de las torres del locker y la raspberry central. Sin embargo el router empleado solamente tiene un puerto ethernet, por lo que se debe emplear un switch de puertos para ampliar las conexiones disponibles. Actualmente se emplea el siguiente switch: 

[https://www.tp-link.com/es/home-networking/soho-switch/tl-sf1005d/](https://www.tp-link.com/es/home-networking/soho-switch/tl-sf1005d/)

Solo se debe considerar que el número de puertos disponibles en el switch sea suficiente para el locker, por ello si se emplean más de 3 torres, se debe considerar un switch con más puertos. 

### Montaje físico en torre {#montaje-físico-en-torre-7}

De igual manera como se hace con el no break, simplemente se ingresa en el box central y ahí es que se conectan todos los dispositivos por puerto ethernet. 

# Fuente ATX {#fuente-atx}

Como fuente de todo el sistema hasta ahora se ha empleado una fuente ATX de 600 W, como se sabe al estar pensada para una computadora, tiene una variedad de voltajes que no se necesitan en el locker. Por lo que una mejora sería utilizar una fuente solo con los voltajes necesarios para el locker. Sin embargo esto nunca se ha probado y no se ha medido el consumo de corriente en *standy by* ni en la activación de chapas. Se debe tener en cuenta además que para que una fuente ATX encienda sin estar conectada a una motherboard, se debe puentear el cable verde del arnes principal de cables a un cable negro o de GND. 

El modelo utilizado como fuente de locker es el siguiente: 

https://www.gamefactor.mx/detalle/29/psg650

### Montaje físico en torre {#montaje-físico-en-torre-8}

De igual manera como se hace con el no break, simplemente se ingresa en el box central y se conecta con el arnés por medio de una conexión principal. 

![][image16]

[image1]: <./Images/Specs/specs1.png>

[image2]: <./Images/Specs/specs2.png>

[image3]: <./Images/Specs/specs3.png>

[image4]: <./Images/Specs/specs4.png>

[image5]: <./Images/Specs/specs5.png>

[image6]: <./Images/Specs/specs6.png>

[image7]: <./Images/Specs/specs7.png>

[image8]: <./Images/Specs/specs8.png>

[image9]: <./Images/Specs/specs9.png>

[image10]: <./Images/Specs/specs10.png>

[image11]: <./Images/Specs/specs11.png>

[image12]: <./Images/Specs/specs12.png>

[image13]: <./Images/Specs/specs13.png>

[image14]: <./Images/Specs/specs14.png>

[image15]: <./Images/Specs/specs15.png>

[image16]: <./Images/Specs/specs16.png>
