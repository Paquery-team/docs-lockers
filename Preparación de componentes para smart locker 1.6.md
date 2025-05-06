# 

# 

![][image1]\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

# **Preparación de componentes para smart locker 1.6**

# 

# Índice {#índice}

[Índice	2](#índice)

[Historial de cambios	3](#historial-de-cambios)

[Objetivo	4](#objetivo)

[Alcance	4](#alcance)

[Lista de componentes y condiciones previas	4](#lista-de-componentes-y-condiciones-previas)

[Condiciones previas para la preparación	4](#condiciones-previas-para-la-preparación)

[Lista de componentes	5](#lista-de-componentes)

[Componentes electrónicos	5](#componentes-electrónicos)

[Preparación de componentes	6](#preparación-de-componentes)

[Raspberry Pi 3B / Memoria micro SD	6](#raspberry-pi-3b-/-memoria-micro-sd)

[Parte 1\. Condiciones previas para la preparación	6](#parte-1.-condiciones-previas-para-la-preparación)

[Parte 2\. Condiciones previas para la preparación	10](#parte-2.-condiciones-previas-para-la-preparación)

[Configuración de las variables de entorno	11](#configuración-de-las-variables-de-entorno)

[Configuración de la BD local	14](#configuración-de-la-bd-local)

[Raspberry Pi Zero / Memoria micro SD	17](#raspberry-pi-zero-/-memoria-micro-sd)

[Parte 1\. Condiciones previas para la preparación	17](#parte-1.-condiciones-previas-para-la-preparación-1)

[Parte 2\. Condiciones previas para la preparación	22](#parte-2.-condiciones-previas-para-la-preparación-1)

[Mobile router Peplink MAX BR1 Mini LTE	24](#mobile-router-peplink-max-br1-mini-lte)

[Condiciones previas para la preparación	24](#condiciones-previas-para-la-preparación-1)

[Fuente ATX de 600 W	25](#fuente-atx-de-600-w)

[Condiciones previas para la preparación	25](#condiciones-previas-para-la-preparación-2)

# Historial de cambios {#historial-de-cambios}

| Versión | Autor | Fecha | Comentarios |
| :---: | ----- | ----- | ----- |
| 1.0 | [Erik Domínguez Vargas](mailto:erik.dominguez@chazki.com) | 7 de agosto de 2023 | Primera definición de procedimientos |
|  |  |  |  |
|  |  |  |  |

# Objetivo {#objetivo}

Establecer los lineamientos y procedimientos necesarios para la correcta preparación y verificación de los componentes utilizados en el ensamblaje del Smart Locker versión 1.6, con el fin de asegurar la calidad, funcionalidad y trazabilidad del producto antes de su integración final.

# Alcance {#alcance}

Este procedimiento aplica a todo el personal técnico y operativo responsable de la preparación de componentes previos al ensamblaje del smart locker 1.6, desde la recepción de materiales hasta su entrega al área de integración.

# Lista de componentes y condiciones previas {#lista-de-componentes-y-condiciones-previas}

## Condiciones previas para la preparación {#condiciones-previas-para-la-preparación}

Antes de iniciar la preparación de los componentes, deben cumplirse las siguientes condiciones:

1. Se debe tener una verificación de orden de trabajo activa:

- Confirmar que existe una orden de producción autorizada para el proyecto.

2. Recepción física y validación de materiales:

- Todos los componentes deben estar físicamente presentes y haber sido verificados por el área de almacén.

- No debe iniciarse ningún proceso con partes incompletas o sin revisión de calidad inicial.

3. Espacio de trabajo limpio y habilitado:

- El área debe estar despejada, con los kits de herramientas disponibles y en condiciones óptimas.

4. Checklist de componentes impresa y firmada:

- Debe estar disponible la hoja de verificación para registro manual durante el proceso de preparación.

## Lista de componentes {#lista-de-componentes}

A continuación se enlistan los componentes que deben prepararse para el ensamblaje del smart locker versión 1.6.

## Componentes electrónicos {#componentes-electrónicos}

| Componente | Unidad | Cantidad necesaria | ¿Requiere preparación previa? |
| ----- | ----- | ----- | ----- |
| Chapa electronica | Pieza | 1 Por cada puerta de la configuración de espacios del locker. | No |
| Modulo de relevadores dobles | Pieza | 1 Por cada dos puertas de la configuración de espacios del locker. | No |
| PCB Armado de torre | Pieza | 1 Por cada torre | No |
| Raspberry Pi 3B | Pieza | 1 Por cada locker | Sí |
| Raspberry Zero | Pieza | 1 Por cada torre del locker | Sí |
| Memoria micro SD | Pieza | 1 Por cada Raspberry zero y 1 Para Raspberry 3B | Sí |
| Mobile router Peplink MAX BR1 Mini LTE | Pieza | 1 Por cada locker | Sí |
| No break o multicontacto eléctrico | Pieza | 1 Por cada locker | No |
| Conjunto iPad, cable lightning y cargador 20 W  | Juego | 1 Por cada locker | Sí |
| Switch de red | Pieza | 1 Por cada locker | No |
| Conjunto de cables ethernet | Juego | 1 Por cada locker | No |
| Fuente ATX de 600 W | Juego | 1 Por cada locker | Sí |

# Preparación de componentes {#preparación-de-componentes}

## Raspberry Pi 3B / Memoria micro SD {#raspberry-pi-3b-/-memoria-micro-sd}

### Parte 1\. Condiciones previas para la preparación  {#parte-1.-condiciones-previas-para-la-preparación}

1. Computadora disponible con archivo de imagen “image-master-resize-rc-particle-removal.img”  y software [Raspberry Pi Imager](https://www.raspberrypi.com/software/) instalado.  
2. Recepción física de Raspberry Pi 3B y memoria micro SD de por lo menos 32 Gb de capacidad.

Parte 1: Para la parte 1 se debe grabar la imagen “limpia” con la imagen del Sistema Operativo proporcionado por el área de desarrollo, por lo que se vuelve imperativo tener el archivo de imagen en la computadora donde se llevará a cabo el procedimiento, los pasos a seguir son los siguientes: 

Abrir el software Raspberry Pi Imager:  
 ![][image2]

Seleccionamos la opción CHOOSE OS, y en el listado, buscamos la opción Use custom:

![][image3]

Insertar la memoria microSD en la computadora

Aquí es donde tendremos que buscar el archivo “image-master-resize-rc-particle-removal.img” en nuestra computadora y deberemos seleccionarlo: 

![][image4]

Ahora, insertamos la memoria MicroSD en la computadora y dejamos que Windows la reconozca, una vez reconocida por windows, seleccionamos CHOOSE STORAGE: 

![][image5]  
Una vez seleccionada la tarjeta que hemos insertado, seleccionamos la opción WRITE, lo que comenzará el proceso de grabación de imagen y también se realizará la comprobación del Sistema Operativo grabado, el mismo programa nos indicará cuando debemos retirar la memoria microSD y con ello concluirá la primera parte. 

![][image6]  
![][image7]

### Parte 2\. Condiciones previas para la preparación {#parte-2.-condiciones-previas-para-la-preparación}

	

1. Recepción física de Raspberry Pi 3B y memoria micro SD de por lo menos 32 Gb de capacidad con imagen “limpia” grabada.  
2. Asignación de UUID de locker por parte del área de desarrollo (alta de locker en Admin IoT)   
3. Fuente de alimentación para Raspberry Pi 3B en locker o por medio de USB.  
4. Recepción de cables ethernet para conectar el router con la Raspberry, puede ser directamente o a través de un switch  
5. Router con VPN preparada y computadora configurada para conectar a VPN.

Parte 2: Para esta parte ya tendremos una imagen “limpia” del Sistema Operativo del locker, por lo que ahora debemos conectarnos a la Raspberry por medio de una sesión SSH y usando la VPN del router y así podremos modificar los archivos de configuración del locker y su base de datos local, lo que lo convertirá en un locker único dentro de la red productiva de lockers. Los pasos a seguir son los siguientes: 

Debemos conectarnos a la VPN desde la computadora con la cual vamos a realizar el procedimiento de configuración. Una vez conectados, con el cliente SSH que usemos habitualmente, debemos iniciar sesión, con la siguiente estructura: 

| ssh pi@10.10.10.xx |
| :---- |

Donde las xx deben ser sustituidas por lo que nos indique el proveedor de routers cuando haya configurado la VPN, el proveedor asigna normalmente una IP que coincide con el UUID del locker, así por ejemplo para conectarse a el locker con UUID 0080, el proveedor asigno la IP 10.10.10.80, por lo que el comando quedaría:

| ssh pi@10.10.10.80 |
| :---- |

Una vez que hayamos ingresado el comando, nos solicitará la contraseña, la cual para todas las imágenes limpias es: 

Ingresada la contraseña, veremos que hemos iniciado sesión en la raspberry que se asignará al locker, con el usuario pi, por lo que deberíamos ver algo así:

![][image8]  
	

### Configuración de las variables de entorno {#configuración-de-las-variables-de-entorno}

Una vez conectado a la Raspberry, debemos asignar las variables de entorno que identificarán al locker, para ello, ingresamos el siguiente comando:

| cd Documents/ms-rpi/ |
| :---- |

Lo que nos llevará a la carpeta donde haremos la modificación: 

![][image9]

Una vez ahí usaremos nuestro editor de texto preferido para hacer los cambios, que deberán ser en el documento. env, en este caso usaremos nano:

| sudo nano .env |
| :---- |

Lo que nos mostrará el contenido de nuestro archivo: 

![][image10]

De este archivo debemos modificar, las variables: 

| UUID=XXXX |
| :---- |

Donde XXXX, será el UUID que nos proporcione el área de desarrollo al dar de alta el locker. Por ejemplo, se muestra la configuración para el locker con UUID 0080:

![][image11]

Guardamos los cambios, en el caso de nano, se guardan con 

| ctrl \+ x |
| :---- |

Se confirma que se guardan los cambios en el mismo archivo con “y” y esto nos regresa a la sesión principal.

### Configuración de la BD local {#configuración-de-la-bd-local}

Estando en la misma carpeta “ms-rpi”, debemos ingresar a la base de datos, se hace con el siguiente comando: 

| sudo sqlite3 prod.sqlite  |
| :---- |

Con ello, entraremos a la base de datos y deberemos configurar con los siguientes comandos: 

| UPDATE towers SET particle\_id \= '192.168.1.100', uuid= 'XXXX001' WHERE id \= 1; UPDATE towers SET particle\_id \= '192.168.1.101', uuid= 'XXXX002' WHERE id \= 2;  |
| :---- |

Donde cambiaremos las XXXX por el UUID del locker, por ejemplo para configurar el locker con UUID 0080, quedaría así: 

| UPDATE towers SET particle\_id \= '192.168.1.100', uuid= '0080001' WHERE id \= 1; UPDATE towers SET particle\_id \= '192.168.1.101', uuid= '0080002' WHERE id \= 2;  |
| :---- |

Y para corroborar nuestra configuración, podemos usar el comando: 

| SELECT \* FROM towers;  |
| :---- |

Y como podemos ver, se muestran correctamente los UUID de las torres, 0080001 y 0080002\.

![][image12]

Ahora debemos configurar los boxes con los siguientes comandos: 

| UPDATE boxes SET uuid \= 'XXXX00101' WHERE id \= 1; UPDATE boxes SET uuid \= 'XXXX00102' WHERE id \= 2; UPDATE boxes SET uuid \= 'XXXX00103' WHERE id \= 3; UPDATE boxes SET uuid \= 'XXXX00104' WHERE id \= 4; UPDATE boxes SET uuid \= 'XXXX00105' WHERE id \= 5; UPDATE boxes SET uuid \= 'XXXX00106' WHERE id \= 6; UPDATE boxes SET uuid \= 'XXXX00107' WHERE id \= 7; UPDATE boxes SET uuid \= 'XXXX00108' WHERE id \= 8; UPDATE boxes SET uuid \= 'XXXX00109' WHERE id \= 9; UPDATE boxes SET uuid \= 'XXXX00110' WHERE id \= 10; UPDATE boxes SET uuid \= 'XXXX00201' WHERE id \= 11; UPDATE boxes SET uuid \= 'XXXX00202' WHERE id \= 12; UPDATE boxes SET uuid \= 'XXXX00203' WHERE id \= 13; UPDATE boxes SET uuid \= 'XXXX00204' WHERE id \= 14; UPDATE boxes SET uuid \= 'XXXX00205' WHERE id \= 15; UPDATE boxes SET uuid \= 'XXXX00206' WHERE id \= 16; UPDATE boxes SET uuid \= 'XXXX00207' WHERE id \= 17; |
| :---- |

Donde de igual manera, se deben sustituir todos los XXXX, por el UUID del locker. Finalmente para corroborar nuestra configuración, podemos ingresar el siguiente comando: 

| SELECT \* FROM towers;  |
| :---- |

Y con ello se nos mostrará la configuración que acabamos de realizar: 

![][image13]  
Para salir de la edición de la base de datos, usamos el comando:

| .quit  |
| :---- |

Y con ello, habremos terminado la configuración de la Raspberry central, por lo que debemos de únicamente reiniciar con el siguiente comando: 

| sudo reboot  |
| :---- |

Esto además interrumpirá nuestra conexión con la raspberry, pero al reiniciar, la Raspberry central estará lista para colocarse en el locker. 

## Raspberry Pi Zero / Memoria micro SD {#raspberry-pi-zero-/-memoria-micro-sd}

### Parte 1\. Condiciones previas para la preparación  {#parte-1.-condiciones-previas-para-la-preparación-1}

3. Computadora disponible con archivo de imagen “image-slave-resize-V1-4-rev3-008”  y software [Raspberry Pi Imager](https://www.raspberrypi.com/software/) instalado.  
4. Recepción física de Raspberry Pi Zero y memoria micro SD de por lo menos 32 Gb de capacidad.

Parte 1: Para la parte 1 se debe grabar la imagen “limpia” con la imagen del Sistema Operativo proporcionado por el área de desarrollo, por lo que se vuelve imperativo tener el archivo de imagen en la computadora donde se llevará a cabo el procedimiento, los pasos a seguir son los siguientes: 

Abrir el software Raspberry Pi Imager:  
 ![][image2]

Seleccionamos la opción CHOOSE OS, y en el listado, buscamos la opción Use custom:

![][image3]

Insertar la memoria microSD en la computadora

Aquí es donde tendremos que buscar el archivo “image-slave-resize-V1-4-rev3-008” en nuestra computadora y deberemos seleccionarlo: 

![][image14]

Ahora, insertamos la memoria MicroSD en la computadora y dejamos que Windows la reconozca, una vez reconocida por windows, seleccionamos CHOOSE STORAGE: 

![][image5]  
Una vez seleccionada la tarjeta que hemos insertado, seleccionamos la opción WRITE, lo que comenzará el proceso de grabación de imagen y también se realizará la comprobación del Sistema Operativo grabado, el mismo programa nos indicará cuando debemos retirar la memoria microSD y con ello concluirá la primera parte. 

![][image6]

![][image15]

### Parte 2\. Condiciones previas para la preparación {#parte-2.-condiciones-previas-para-la-preparación-1}

	

1. Recepción física de Raspberry Pi Zero y memoria micro SD de por lo menos 32 Gb de capacidad con imagen “limpia” grabada.  
2. Asignación de UUID de locker por parte del área de desarrollo (alta de locker en Admin IoT)   
3. Fuente de alimentación para Raspberry Pi Zero en locker o por medio de USB.  
4. Recepción de cables ethernet para conectar el router con la Raspberry, puede ser directamente o a través de un switch  
5. Router con VPN preparada y computadora configurada para conectar a VPN.  
6. Raspberry 3B configurada y conectada correctamente a la VPN

Debemos conectarnos a la VPN desde la computadora con la cual vamos a realizar el procedimiento de configuración, esta conexión con la VPN será hacia la Raspberry 3B, por lo que de ahí, deberemos conectarnos hacia la Raspberry Zero Una vez, primero conectamos nuestra computadora a la VPN conectados, con el cliente SSH que usemos habitualmente, nos conectamos a la Raspberry 3B en la cual deseamos configurar una Raspberry Zero. 

| ssh pi@192.168.1.100 |
| :---- |

Por lo que conectando desde la Raspberry 3B, debe verse algo así:

![][image16]

Y pedirá la contraseña, que será para todas las imagenes limpias: 

Una vez ingresando, debemos ingresar el siguiente comando: 

| cd Documents/slave-rpi/ |
| :---- |

Lo que nos abrirá la carpeta donde se encuentra el documento con las variables de entorno que debemos configurar: 

![][image17]

Aquí usaremos el editor de texto que mejor nos acomode, en este caso usaré nano, por lo que se debe ingresar el comando: 

| sudo nano .env |
| :---- |

Lo que abrirá el archivo y debe verse de la siguiente manera: 

![][image18]

Aquí la variable que debemos modificar es UUID, que por defaul, se encuentra como 8889, aquí deberemos colocar el UUID del locker, por ejemplo el UUID 0080\.   
Por otro lado, se debe colocar una comilla simple al final de la variable MQTT\_BROKER, para corregir un error de edición, por lo que quedaría de la siguiente manera:

![][image19]

Una vez realizado el cambio, guardamos los cambios realizados, en el caso de nano, con:

| ctrl \+ x |
| :---- |

Se confirma que se guardan los cambios en el mismo archivo con “y” y esto nos regresa a la sesión principal.

Para salir de la sesión principal, simplemente usamos el comando 

| exit |
| :---- |

Una vez que hayamos realizado esto, la configuración de la Raspberry Zero y su tarjeta microSD estará terminada, debemos repetir este proceso para el total de torres que tenga el locker, normalmente dos veces. 

## Mobile router Peplink MAX BR1 Mini LTE {#mobile-router-peplink-max-br1-mini-lte}

### Condiciones previas para la preparación {#condiciones-previas-para-la-preparación-1}

- Haber realizado la compra del router y haber pedido la configuración habitual de los routers proporcionados para el locker.   
- Tener la imagen del SO “limpio” grabado en la Raspberry 3B para solicitar la configuración de la VPN.  
- Haber adquirido la SIM de datos celulares que se asignará al locker. 

Comenzaremos por ingresar la SIM de datos en la ranura principal del router, colocaremos sus antenas.

![][image20]

Conectamos la Raspberry principal, con el sistema “limpio” por medio de un switch o directamente cableada al router. 

Solicitamos al proveedor que realice la configuración de la VPN del router y pedimos la IP resultante de la configuración.

Posteriormente, verificamos en InControl que la Raspberry central se encuentre conectada entre los clientes del dispositivo, lo que ahora nos permitirá terminar la configuración de la memoria microSD de la raspberry.

Conjunto iPad, cable lightning y cargador 20 W 

Para este paso, se debe seguir el manual de enrolamiento de iPads a Scalefusion MDM.

## Fuente ATX de 600 W {#fuente-atx-de-600-w}

### Condiciones previas para la preparación {#condiciones-previas-para-la-preparación-2}

1. Fuente ATX comprada y recibida.  
2. Contar con un enchufe eléctrico disponible para poder realizar las pruebas. 

Para poder dejar lista la Fuente ATX, solamente se debe generar una conexión entre los cables verde y negro del conector de 20 pines, tal como se muestra en la imagen: 

![][image21]

Después de realizar esta conexión, la fuente de alimentación se encenderá al conectarse, o al activar el switch integrado en la fuente.



[image1]: <./Images/Component/component1.png>

[image2]: <./Images/Component/component2.png>

[image3]: <./Images/Component/component3.png>

[image4]: <./Images/Component/component4.png>

[image5]: <./Images/Component/component5.png>

[image6]: <./Images/Component/component6.png>

[image7]: <./Images/Component/component7.png>

[image8]: <./Images/Component/component8.png>

[image9]: <./Images/Component/component9.png>

[image10]: <./Images/Component/component10.png>

[image11]: <./Images/Component/component11.png>

[image12]: <./Images/Component/component12.png>

[image13]: <./Images/Component/component13.png>

[image14]: <./Images/Component/component14.png>

[image15]: <./Images/Component/component15.png>

[image16]: <./Images/Component/component16.png>

[image17]: <./Images/Component/component17.png>

[image18]: <./Images/Component/component18.png>

[image19]: <./Images/Component/component19.png>

[image20]: <./Images/Component/component20.png>

[image21]: <./Images/Component/component21.png>

