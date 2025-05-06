# 

# 

![][image1]\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

[**OBJETIVO**](#objetivo)	**[2](#objetivo)**

[**Alta de ubicación**](#alta-de-ubicación)	**[2](#alta-de-ubicación)**

[Migración de servicios en Raspberry Pi](#migración-de-servicios-en-raspberry-pi)	[2](#migración-de-servicios-en-raspberry-pi)

[Reservación de prueba](#reservación-de-prueba)	[4](#reservación-de-prueba)

[**Consulta y actualización de Raspberry Pi**](#consulta-y-actualización-de-raspberry-pi)	**[5](#consulta-y-actualización-de-raspberry-pi)**

[tower-rpi](#tower-rpi)	[5](#tower-rpi)

[Consulta](#consulta)	[5](#consulta)

[Actualización](#actualización)	[6](#actualización)

[ms-rpi](#ms-rpi)	[6](#ms-rpi)

[Consulta](#consulta-1)	[6](#consulta-1)

[Actualización](#actualización-1)	[6](#actualización-1)

[**Errores comunes**](#errores-comunes)	**[7](#errores-comunes)**

[Token Incorrecto](#token-incorrecto)	[7](#token-incorrecto)

[Pantalla de error](#pantalla-de-error)	[8](#pantalla-de-error)

[Lok desconectado](#lok-desconectado)	[9](#lok-desconectado)

[Microcontrolador desconectado](#microcontrolador-desconectado)	[10](#microcontrolador-desconectado)

[Reservación Fallida](#reservación-fallida)	[10](#reservación-fallida)

[API v1](#api-v1)	[10](#api-v1)

[API v2](#api-v2)	[10](#api-v2)

[Pantalla del Lok apagada](#pantalla-del-lok-apagada)	[11](#pantalla-del-lok-apagada)

[Webhook no recibido](#webhook-no-recibido)	[11](#webhook-no-recibido)

[**Procesos relacionados**](#procesos-relacionados)	**[12](#procesos-relacionados)**

[Conectarse a la RPi](#conectarse-a-la-rpi)	[12](#conectarse-a-la-rpi)

[Reiniciar la RPi](#reiniciar-la-rpi)	[13](#reiniciar-la-rpi)

[Reiniciar ms-rpi](#reiniciar-ms-rpi)	[13](#reiniciar-ms-rpi)

[Liberar memoria de la RPi](#liberar-memoria-de-la-rpi)	[13](#liberar-memoria-de-la-rpi)

[Reconectar iPad](#reconectar-ipad)	[14](#reconectar-ipad)

[Reiniciar iPad](#reiniciar-ipad)	[14](#reiniciar-ipad)

[Abrir puerta](#abrir-puerta)	[14](#abrir-puerta)

[Buscar AP en InControl](#buscar-ap-en-incontrol)	[15](#buscar-ap-en-incontrol)

[**Acciones de Moleculer**](#acciones-de-moleculer)	**[15](#acciones-de-moleculer)**

[Conectarse al cliente](#conectarse-al-cliente)	[15](#conectarse-al-cliente)

[Reenviar eventos](#reenviar-eventos)	[15](#reenviar-eventos)

[Obtener tokens de una reserva](#obtener-tokens-de-una-reserva)	[16](#obtener-tokens-de-una-reserva)

[Reenviar tokens a la RPi](#reenviar-tokens-a-la-rpi)	[16](#reenviar-tokens-a-la-rpi)

[**Consultas SQLite**](#heading=h.1664s55)	**[16](#heading=h.1664s55)**

[Conectarse a la base](#conectarse-a-la-base)	[16](#conectarse-a-la-base)

[Obtener los tokens activos](#obtener-los-tokens-activos)	[16](#obtener-los-tokens-activos)

[Revisar torres](#revisar-torres)	[16](#revisar-torres)

[Eventos sin sincronizar](#eventos-sin-sincronizar)	[17](#eventos-sin-sincronizar)

[Revisar tablas](#revisar-tablas)	[17](#revisar-tablas)

[Esquema de tabla](#esquema-de-tabla)	[17](#esquema-de-tabla)

[**Consultas MongoDB**](#consultas-mongodb)	**[17](#consultas-mongodb)**

[Buscar reservación por Shipment](#buscar-reservación-por-shipment)	[17](#buscar-reservación-por-shipment)

[Buscar reservación por Id](#buscar-reservación-por-id)	[17](#buscar-reservación-por-id)

[Buscar tokens](#buscar-tokens)	[17](#buscar-tokens)

[Buscar reservación con un token](#buscar-reservación-con-un-token)	[18](#buscar-reservación-con-un-token)

[Buscar reservación erróneas en un periodo de tiempo](#buscar-reservación-erróneas-en-un-periodo-de-tiempo)	[19](#buscar-reservación-erróneas-en-un-periodo-de-tiempo)

# Manual de soporte remoto Anexo: Procesos de soluciones

## **Versión de documento: 1.0**

Elaboró: Maurcio Esquivel Reyes.

# **OBJETIVO** {#objetivo}

Ejemplificar los pasos necesarios para resolver cada uno de los problemas que se presentan al momento de dar atención de manera remota.

# **Alta de ubicación** {#alta-de-ubicación}

##  **Migración de servicios en Raspberry Pi** {#migración-de-servicios-en-raspberry-pi}

La imagen utilizada en la RPi para el ensamble de los Lok cuenta con los servicios de *tower-rpi*,  es necesario migrar a *ms-rpi*. Estos son los pasos necesarios:

* [Ingresar a la rasp vía ssh](#conectarse-a-la-rpi)  
* (Opcional) Borrar archivos basura en ***\~(/home/pi)*** con ***rm \*.png*** y ***rm \-rf builds***  
* Confirmar que se encuentran los servicios de *tower-rpi* con el comando ***sudo pm2 list***  
* Remover repositorio de GitLab Runner   
  ***sudo rm /etc/apt/sources.list.d/runner\_gitlab-runner.list***  
* Agregar repositorio de Ansible

***sudo sh \-c 'echo "deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main" \>\> /etc/apt/sources.list'***

* Agregar llave del repositorio de Ansible

***sudo apt-key adv \--keyserver keyserver.ubuntu.com \--recv-keys 93C4A3FD7BB9C367***

* Actualizar repositorios e instalar Ansible y Redis (usar comandos separados)

***sudo apt update***  
***sudo apt install \-y ansible***  
***sudo apt install \-y redis-server***

* Revisar que la versión de Ansible sea 2.9 para arriba ***ansible \--version***  
* Cambiar a la carpeta de Documentos con el comando ***cd Documents***  
* Clonar el repositorio de Ansible Playbooks ***git clone [https://gitlab.com/clicknbox/devsecops/development/firmware/ansible-playbooks.git](https://gitlab.com/clicknbox/devsecops/development/firmware/ansible-playbooks.git)***  
  * Usuario: firmware  
  * Contraseña: eCXQVodZq-CbM\_mg1Y1k  
* Ingresar a la carpeta con ***cd ansible-playbooks***

* Crear el archivo con  ***nano setup.json*** con la siguiente información:  
  {  
  "git\_user": "firmware",

  "git\_pass": "eCXQVodZq-CbM\_mg1Y1k",  
  "base\_path": "/home/pi/Documents"

}

* Correr el playbook de configuración   
  ***ansible-pull \-d /home/pi/Documents/ansible-playbooks \-U https://firmware:eCXQVodZq-CbM\_mg1Y1k@gitlab.com/clicknbox/firmware/ansible-playbooks.git  /home/pi/Documents/ansible-playbooks/microservices/local/setup.yml \-e @setup.json***  
* Crear el archivo ***vars.json*** con la siguiente información, agregando el uuid y modelo correspondiente:  
  {

     "env": "prod",

     "mqtt\_user": "moleculer",

     "mqtt\_pass": "moleculer",

     "mqtt\_host": "mqtt-dev.clicknbox.com",

     "db": "../database/locationDB.db",

     "log\_level": "trace",

     "rpi\_serial": "/dev/ttyACM0",

     "rpi\_port": 8080,

     "api\_url": "https://robot.clicknbox.com/api/v2",

     "api\_token": "Y2xzpY2tuYm94OnNlY3JldF9jbGlja25ib3gyMDE5",

     "api\_user": "rpi-v2@clicknbox.com",

     "api\_pass": "FrgPass2019Api",

     "client\_id": "Vu7Aqy3ICtz7nHu",

     "client\_secret": "u27OLOvn0FnHUELzfFo1tMCS",

     "particle\_mail": "dev@clicknbox.com",

     "particle\_pass": "WE5kthQL4pbAjra",

     "particle\_token\_duration": 7776000,

     "uuid": "XXXX",

     "version": "release/pre-particle",

     "git\_pass": "eCXQVodZq-CbM\_mg1Y1k",

     "git\_user": "firmware",

     "base\_path": "/home/pi/Documents",

     "model": \["cnc", "p2p"\]

  }


* Correr el playbook de migración:  
  ***ansible-playbook microservices/local/migration.yml \-e @vars.json \--ask-become-pass***  
* Esperar a que termine y verificar la instalación con:  
  ***systemctl status moleculer.service***


Puede suceder que el playbook no termine de correr por una desconexión, por lo cual se deben tomar las siguientes acciones para terminar la instalación:

* Borrar la carpeta de *ms-rpi* con el comando:   
  ***sudo rm \-rf /home/pi/Documents/ms-rpi***  
* En la carpeta *ansible-playbooks* correr el playbook de instalación con el archivo [***vars.json***](#bookmark=id.3dy6vkm)  
  ***sudo ansible-playbook microservices/install.yml \-e @vars.json***  
* Esperar a que termine y verificar la instalación con:  
  ***systemctl status moleculer.service***

Al finalizar la instalación de *ms-rpi* moverse a ***cd /home/pi/Documents*** y eliminar las siguientes carpetas:  
	***sudo rm \-rf P2P/ demo-tower/ standalone-rpi/ tower-rpi/***

## **Reservación de prueba** {#reservación-de-prueba}

Ya que el Lok se encuentre instalado no importa si tiene ***tower-rpi*** o ***ms-rpi*** se puede realizar una reserva de prueba de la siguiente forma:

* Ingresar a [https://network.clicknbox.com](https://network.clicknbox.com)  
* En la pestaña de inicio buscar el Lok por su **uuid** en la barra de búsqueda.  
* Presionar sobre el botón verde y luego en **Reservación de Prueba**.  
* Elegir torre y caja para realizar la reserva.  
  * Si tiene una reservación activa no se podrá crear.  
* Utilizar los tokens con el flujo de éxito.

	También se puede revisar el funcionamiento del Lok desde la [consola de Particle](https://console.particle.io/events).

* En la página dar click en el icono **Publica un evento**.  
* Llenar **Nombre del evento** con el **uuid** del Lok a probar![][image1]   
* Publicar el evento y esperar el resultado.  
  * Si la consola muestra un evento **reservation/0079** con data **0**, funciona de manera correcta. De caso contrarío revisar los servicios respectivos.

# **Consulta y actualización de Raspberry Pi** {#consulta-y-actualización-de-raspberry-pi}

## **tower-rpi** {#tower-rpi}

#### **Consulta** {#consulta}

	Para saber si los servicios están activos se necesita usar el comando  
		***sudo pm2 list***   
	Si nos muestra algo parecido a:  
![][image2]  
	Significa que todo está corriendo de manera correcta. Si alguno de los tres servicios se reinicia de manera constante, es decir que el contador **restart** está incrementando y **uptime** no pasa de la marca de un minuto, es necesario parar el servicio con ***sudo pm2 stop n***, donde ***n*** es el **id** del servicio. Para reiniciar el servicio es con ***sudo pm2 restart n.***

	Para obtener los logs de un servicio se usa el comando ***sudo pm2 log n.***  
Es recomendable extraer los logs de la RPi a nuestra máquina de la siguiente manera:

* ***pwd*** (para obtener la ruta sobre la cual se está trabajando)  
* ***sudo pm2 logs 0 \--lines 100 \> error-app.log*** (imprimos la salida en un archivo)  
* ***exit*** (salimos de la RPi)  
* ***scp pi@10.10.10.10:/home/pi/error-app.log ./error-0001.log*** (ponemos la ruta que nos arrojó ***pwd*** como primer argumento y la ruta de nuestra máquina donde se guardará el archivo como segundo argumento)

#### **Actualización** {#actualización}

Dado que *tower-rpi* es código legado se debe migrar a *ms-rpi* con las instrucciones de [Migración de servicios en Raspberry Pi.](#migración-de-servicios-en-raspberry-pi)

##  **ms-rpi** {#ms-rpi}

#### **Consulta** {#consulta-1}

Para consultar el estado de *ms-rpi* se utiliza el comando   
***systemctl status moleculer.service***

Si la RPi se encuentra en buen estado nos debe mostrar un resultado similar a este:  
![][image3]

Si en la etiqueta de ***Active*** muestra un error es necesario [reiniciar la RPi](#reiniciar-la-rpi).

Si es necesario obtener las bitácoras se muestran en pantalla con:  
***journalctl \-u moleculer.service \-b***  
Esto nos da las entradas desde el último boot de la RPi (***\-b***)

Pero es mejor imprimir las entradas en un archivo y analizarlo desde nuestro ordenador:

*  ***pwd*** (para obtener la ruta sobre la cual se está trabajando)  
  * ***journalctl \-u moleculer.service | tail \--lines 100  \> ms-rpi.log*** (imprimos la salida en un archivo)  
* ***exit*** (salimos de la RPi)  
* ***scp pi@10.10.10.10:/hom e/pi/ms-rpi.log ./ms-rpi.log*** (ponemos la ruta que nos arrojó ***pwd*** como primer argumento y la ruta de nuestra máquina donde se guardará el archivo como segundo argumento)

#### **Actualización** {#actualización-1}

Ya que el Lok se encuentre funcionando con *ms-rpi* esta es la manera en la que podemos actualizar o regresar a una versión anterior:

* Conectarse a la RPi por ssh  
* Ingresar a la carpeta   
  ***cd Documents/ansible-playbooks***  
* Modificar o crear el archivo **vars.json** con la versión deseada:  
  {

     "version": "vX.X.X"

  }

* Correr el playbook   
  ***sudo ansible-playbook microservices/update.yml \-e @vars.json \--ask-become-pass***  
* Reiniciar la RPi  
  ***sudo reboot***  
* Conectarse y revisar el estado de los microservicios   
  ***systemctl status moleculer.service***

# **Errores comunes** {#errores-comunes}

## **Token Incorrecto** {#token-incorrecto}

Cuando es reportado que un token no está funcionando es necesario revisar que la estructura sea la siguiente:

* Debe ser una cadena de 8 caracteres hexadecimales.  
  * Contenga números.  
  * Contenga las letras A, B, C, D, E, F.

	  
	Si la cadena cumple con los requerimientos se debe solicitar el tipo de token y la reserva a la que corresponde, o buscar la reserva con el token. Posteriormente revisamos que el evento del token no se encuentre marcado aún en la reserva.  
                                                      ![][image4]  
Si ya se encuentra registrado el evento como en el ejemplo anterior es necesario [abrir la caja](#abrir-puerta).

Si el evento no ha sido registrado, hay que [conectarse a la RPi](#conectarse-a-la-rpi) y [buscar los tokens en la base](#obtener-los-tokens-activos).

Si los tokens aún se encuentran en la base es necesario [consultar los servicios](#consulta-y-actualización-de-raspberry-pi) y reiniciarlos de ser necesario.  
                            

## 

## 

## **Pantalla de error**  {#pantalla-de-error}

Cuando el reporte indica que la pantalla muestra un mensaje de error es necesario revisar su estado e intentar [reiniciar el iPad](#reiniciar-ipad). Si no es posible el paso anterior hay que revisar si se encuentra conectada al [AP](#buscar-ap-en-incontrol), si este no es el caso es necesario [conectar el iPad](#reconectar-ipad).

Si el router ya no levanto es necesario pasar el ticket a Atención en Sitio; si el iPad ya se conectó al AP pero sigue mostrando el mensaje de error hay que ingresar a InControl, buscar el AP, revisar la pestaña de **Clientes** y confirmar que la **Dirección IP** se encuentre en el segmento **192.168.1.xx**  
![][image5]  

Si el iPad no se encuentra en el segmento correcto es necesario solicitar a Vialterna que asigne una ip en ese segmento, pero si se encuentra bien entonces hay que revisar que la RPi se encuentre en la ip indicada.

* Ingresar a la RPi  
* Correr el siguiente comando

  ***ip route show***

* Si se encuentra en la ip correcta creamos un issue para dev, en caso contrario hay que cambiar el valor abriendo el archivo 

  ***sudo nano /etc/dhcpcd.conf***


* Cambiar el valor de **static ip\_address** al siguiente:***![][image6]***  
* Reiniciar la RPi.

Si se sigue igual después de estos pasos hay que pasar el ticket a Atención en Sitio. 

## **Lok desconectado** {#lok-desconectado}

Si es reportado que el Lok no tiene internet es necesario [buscar el AP](#buscar-ap-en-incontrol), revisar su estatus y si no cuenta con señal hay que reiniciar el chip en Jasper de ser posible. 

Si el internet es restaurado hay que revisar en ScaleFusion la batería del iPad. Esto se encuentra explicado en este [manual](https://docs.google.com/document/d/16K-3Gcpq-m4fFbXq_aBxyVYrYnLQUXwXMLN3fgHhfiI/edit), en la sección de **Carga de Batería**.

	Si se encuentra debajo del 100% de carga significa que fue desconectado de la luz, hay que solicitar al encargado que se vuelva a conectar o en su defecto programar una visita a locación.

## **Microcontrolador desconectado** {#microcontrolador-desconectado}

En Particle hay que verificar que el Argón y Xenones cuenten con conexión a internet, de no ser este el caso hay que reiniciar la mesh de la consola de Particle.

Después hay que revisar si el [Lok cuenta con conexión a internet](#lok-desconectado); si el iPad y RPi se encuentran conectados pero no la mesh es necesario hacer una visita a locación. 

## **Reservación Fallida** {#reservación-fallida}

Antes de seguir con alguna de estas soluciones es necesario revisar que **el total de reservas fallidas** no sea igual al **límite de reservas fallidas** en la base de datos Mongo.

#### **API v1** {#api-v1}

Si es reportado que no se ha logrado hacer una reservación en una locación es necesario revisar si la petición fue realizada en los logs, por lo cual hay que entrar al siguiente [enlace](https://us-west-2.console.aws.amazon.com/elasticbeanstalk/home?region=us-west-2#/environment/logs?applicationName=Clicknbox%20API&environmentId=e-a4y3m8hm54) y solicitar todos los logs de la siguiente manera:  
![][image7]

Ya que han sido descargados hay que descomprimir los archivos y buscar el archivo que se encuentra en la ruta **var/log/nodejs/nodejs.log**

	La forma más fácil de buscar si existe el intento de reservación es buscando el **shipment** o el **uuid** de la locación  solicitado al momento de hacer la reservación. Si existe la reservación y muestra un mensaje de **Max time limit**  es necesario revisar si el [Lok se encuentra conectado](#lok-desconectado) y si el [microcontrolador está desconectado](#microcontrolador-desconectado). Si no existe registro de la reservación es necesario pasar el issue a dev.

#### **API v2** {#api-v2}

Si la reserva fue realizada desde el API v2 es necesario obtener los logs del siguiente [enlace](https://us-west-2.console.aws.amazon.com/cloudwatch/home?region=us-west-2#logsV2:log-groups)  
![][image8]

	Se deben buscar los logs de **/ecs/prod/ms-api-gateway** para ver si la reserva fue solicitada; si fue solicitada pero falló podemos buscar la razón de la falla en los logs de **/ecs/prod/ms-reservation**. Estos logs pueden ser consultados directamente en la página y tienen herramientas para buscar los eventos por fecha o información enviada por en la solicitud.

Si la solicitud existe es necesario revisar si el [Lok se encuentra conectado](#lok-desconectado) y si el [microcontrolador está desconectado](#microcontrolador-desconectado). Si no es posible levantar el Lok es necesario realizar una visita a la locación.

## **Pantalla del Lok apagada** {#pantalla-del-lok-apagada}

Hay que revisar el estado del iPad en ScaleFusion. Si tiene conexión se deberá [reiniciar](#reiniciar-la-rpi), de caso contrario hay que hacer una visita a locación para acomodarla y que el sensor de luz funcione de manera correcta.

Si no se encuentra disponible en SF hay que revisar la [conexión del Lok.](#lok-desconectado)

## 

## 

## 

## **Webhook no recibido** {#webhook-no-recibido}

Si es reportado que no llegó una notificación es necesario buscarla en [bull](https://monitoring.clicknbox.com/admin/queues) con las credenciales:   
	Usuario: **admin**  
	Password: **IdeRacRYaRDENhEMeNItA**

      ![][image9]

Primero revisaremos que no se encuentre en la cola de **failed**, si no se encuentra ahí significa que fue enviada con éxito. Opcionalmente se puede revisar en la cola de **completed**.

Si no se encuentra en ninguna de esas colas es necesario conectarse a la RPi y buscar las reservas que no han sido sincronizadas. Si no se encuentra ahí es necesario [volver a mandar el evento](#reenviar-eventos). En otro caso se le asigna un issue a dev.

# 

# 

# 

# 

# 

# **Procesos relacionados** {#procesos-relacionados}

## **Conectarse a la RPi**  {#conectarse-a-la-rpi}

Para ingresar a la RPi con la **VPN** es necesario usar el siguiente comando:

***ssh pi@10.10.10.11***

## **Reiniciar la RPi**  {#reiniciar-la-rpi}

Una vez dentro de la RPi el comando necesario para reiniciarla es:

***sudo reboot***  
											

## **Reiniciar ms-rpi** {#reiniciar-ms-rpi}

Cuando una configuración es cambiada es posible reiniciar el microservicio sin tener que reiniciar la RPi, estos son los pasos necesarios:

* **sudo systemctl stop moleculer.service** (detenemos el servicio)  
* **ps \-aux | grep moleculer** (buscamos el proceso detenido y su pid)  
  root  731  0.1  8.0 154816 76020 ? Sl    2020  42:47 node /home/pi/Documents/ms-rpi/node\_modules/.bin/moleculer-runner \--env  
* **sudo kill \-9 pid** (terminamos el proceso)  
  sudo kill \-9 731  
* **sudo systemctl start moleculer.service** (reiniciamos el servicio)  
* 

  ## **Liberar memoria de la RPi** {#liberar-memoria-de-la-rpi}

En ocasiones los logs de los servicios llenan la memoria de la RPi y no permite que los servicios sean lanzados de nuevo. 

                                ![][image10]

	Vamos a realizar lo siguiente

* ***sudo du \-xh / | grep \-P “G\\t”*** (Mostramos las carpetas de 1G o más en ***/*** )


* ***sudo rm \-rf /root/.pm2/logs*** (Borrar carpetas si son logs, recuperarlos si es necesario)    
* Reiniciar la RPi

  ## **Reconectar iPad**  {#reconectar-ipad}

* [Buscar el AP de la locación.](#buscar-ap-en-incontrol)  
* Verificar que el iPad no se encuentre conectado en la tabla de **Clientes**.  
  * Si se encuentra conectada se puede reiniciar desde [ScaleFusion](#reiniciar-ipad)

     *** ![][image11]***

* En la pestaña **Detalles del dispositivo** dar click sobre el botón **On**, para apagarlo. Esperar a que nos marque si aún está disponible. Si se encuentra disponible volvemos a encender el router.   
                       ** ![][image12]**


  ## **Reiniciar iPad**  {#reiniciar-ipad}

Si el iPad se encuentra disponible a través de ScaleFusion es posible reiniciar el iPad, las instrucciones se encuentran con detalle en el siguiente [manual](https://docs.google.com/document/d/16K-3Gcpq-m4fFbXq_aBxyVYrYnLQUXwXMLN3fgHhfiI/edit), en la sección de **Reinicio Remoto**.

## **Abrir puerta**  {#abrir-puerta}

Ya que el Lok se encuentra instalado y cuenta con ***ms-rpi*** se puede abrir la puerta de las cajas de la siguiente manera:

* Ingresar a [https://network.clicknbox.com](https://network.clicknbox.com)  
* En la pestaña de inicio buscar el Lok por su **uuid** en la barra de búsqueda.  
* Presionar sobre el botón verde y luego en **Abrir Puerta**.  
* Elegir torre y la caja a abrir.

	Si el Lok aún tiene instalado ***tower-rpi***  es posible abrir la puerta desde el portal de [Particle](https://console.particle.io/devices): 

* Obtenemos la caja de la reserva (ej. **uuid** 006900105).  
* En Particle buscamos el Xenon/Argon por el número de la locación  (primeros cuatro números del **uuid**, ej 0069\) y el número de la torre (siguientes tres números del **uuid**, ej. 001\)  
* En la sección de **Functions** agregar el número de la caja menos uno.  
                                                 ![][image13]

  ## **Buscar AP en InControl** {#buscar-ap-en-incontrol}


* Ingresar al portal  [InControl](https://incontrol2.peplink.com/login)  
* Seleccionar el estado en el cual se encuentra la locación que se desea encontrar.  
* Seleccionar el nombre del AP de la locación.

# **Acciones de Moleculer** {#acciones-de-moleculer}

## **Conectarse al cliente** {#conectarse-al-cliente}

Es necesario instalar el cliente de moleculer en nuestro ordenador:  
	***npm i moleculer-cli notepack.io mqtt \-g***

Para conectarse es necesario contar con el archivo de configuración y que este se encuentre en la misma ruta en la que se correrá el comando:  
	***moleculer connect \--ns prod \--config ./moleculer.config.js***

## **Reenviar eventos** {#reenviar-eventos}

	Cuando se haya establecido la conexión del cliente es necesario correr el siguiente comando con la información correspondiente:  
***call notifications.scheduleEventJob '{"reservationId":"639bf5e8e32e5e001a499b2e","type":"cancelled","shipmentId":"aIzsH3PaEis8Wp4+tMRNPQ==","companyId":"600f088b1a373d090a7dce07","url":"https://www.movil.farmaciasguadalajara.com/Clicknbox/ext/v1/notify\_events"}'***

## **Obtener tokens de una reserva** {#obtener-tokens-de-una-reserva}

	Ya que el cliente tiene conexión, podemos consultar los tokens de una reserva a partir de su ***\_id*** con el comando:  
		***call codes.getReservationTokens '{"reservationId" :  "5fdb7fd6075645001933c3c9"}'***

## **Reenviar tokens a la RPi** {#reenviar-tokens-a-la-rpi}

	Después de conectarse al cliente, esta acción necesita contar con el ***nodeID***, ***boxUuid*** y los ***tokens*** de la reservación.  El comando es el siguiente:  
dcall rpi-00 locker.createReservation '{ "tokens": \[ "", "", "", "", "" \], "boxUuid": ""}'  
		***dcall rpi-0073 locker.createReservation '{ "tokens": \[ "EEC916BC", "AE47640B", "D1A6E410", "FEFAF32F", "D0746DB2" \], "boxUuid": "007300201"}'***

## **Actualizar reservas de P2P**

Es necesario obtener el id de mongo de la locacion para poder llamar el siguiente comando:  
	***call reservationHistory.syncFromNode \--locationId 6010621be984ce0014e397eb \--since "02-01-2021"***

## **Integrity Check**

Es necesario obtener el id de mongo de la locacion para poder llamar el siguiente comando:  
	***call location.integrityCheck \--locationId 5fab39ee625014061f6c3e0c***

# **Consultas SQLite**

## **Conectarse a la base**  {#conectarse-a-la-base}

Si nos encontramos conectado a la [RPi](#conectarse-a-la-rpi) en la carpeta **\~ (home)** es necesario correr el siguiente comando:  
	***sudo sqlite3 Documents/database/locationDB.db***

Para ingresar a la base desde cualquier carpeta lo podemos lograr de la siguiente manera:  
	***sudo sqlite3 \~/Documents/database/locationDB.db***

## **Obtener los tokens activos**  {#obtener-los-tokens-activos}

Ya que se tiene una conexión a la base es necesario correr la query:  
	***select \* from tokens;***

Esto nos mostrará todos los tokens activos, si es necesario obtener los de una reserva en específico se puede obtener de la siguiente manera  
	***select \* from tokens where box\_id=6;***

Donde **box\_id** es el número de la caja general del Lok en cuestión. 

## **Revisar torres**  {#revisar-torres}

Si necesitamos revisar que los Particle Id de una locación son correctos es necesario utilizar la siguiente query:  
	***select \* from towers;***  
***![][image14]***

Revisamos que la segunda columna contenga los valores apropiados.

## **Eventos sin sincronizar**  {#eventos-sin-sincronizar}

La RPi puede no haber enviado los eventos de manera oportuna por lo cual se encuentran en la tabla de ***events\_pending\_syncs*** la cuál puede ser consultada con la query:  
	***select \* from events\_pending\_syncs;***  
  ![][image15]

La única manera de poder de identificar una reserva es por el **uuid** (segunda columna) y el **tipo de evento** (tercera columna)

Si es necesario intentar enviar el evento se debe reiniciar el contador (cuarta columna) así:  
	***update events\_pending\_syncs set count\_attempts \= 0 where id=80;***

## **Revisar tablas**  {#revisar-tablas}

Si no se esta seguro de qué tablas existen es posible saberlo con el comando  
	***.tables***

## **Esquema de tabla**  {#esquema-de-tabla}

Si se necesita conocer la estructura de una tabla se tiene el comando  
	***.schema table\_name***

Cambiando **table\_name** por la tabla a consultar.

# **Consultas MongoDB** {#consultas-mongodb}

## **Buscar reservación por Shipment**  {#buscar-reservación-por-shipment}

Para buscar una reservación en mongo se puede hacer las siguientes consultas desde la colección ***reservation\_history*** de dos maneras:  
	  
***{ shipmentId: “v23234324ekt-01” }*** o ***{ shipmentId: /23234324/ }***

En la primera usamos la cadena exacta y la segunda utiliza una expresión regular.

## **Buscar reservación por Id**  {#buscar-reservación-por-id}

Para buscar una reservación en mongo se puede hacer la siguiente consulta desde la colección ***reservation\_history***:

	***{ \_id: ObjectId('5cf70f7417442761ce16091a') }***

## **Buscar tokens**  {#buscar-tokens}

Para buscar los tokens de una reservación primero debemos obtener el ***id*** de dicha reservación para poder usarla como parámetro en la colección ***codes*** de la siguiente manera:

	***{ reservation:  ObjectId('5cf70f7417442761ce16091a') }***  
                                   *** ![][image16]***

## **Buscar reservación con un token**  {#buscar-reservación-con-un-token}

Si se nos da solamente el token de una reservación podemos utilizar la siguiente consulta en la colección de ***codes***  
	  
	***{ tokens: { $in: \["7E2A9E48"\] } }***  
	  
Después obtenemos el **id** del campo **reservation** 

                     ![][image17]  
Y lo utilizamos en la colección **reservation\_history** de la siguiente manera:

	***{ \_id: ObjectId(‘5cf70f7417442761ce16091a’) }***

## **Buscar reservación erróneas en un periodo de tiempo**  {#buscar-reservación-erróneas-en-un-periodo-de-tiempo}

Esta consulta es más que nada para los reportes individuales que se le hacen a cada empresa en la colección ***reservation\_history***:

	***{***  
		***$and: \[***  
			***{***  
			***updatedAt: {***  
				***$gte: ISODate(‘2020-11-09’)***  
***}***  
***},***  
***{***  
			***updatedAt: {***  
				***$lt: ISODate(‘2020-11-16’)***  
***}***  
***},***  
***\],***  
***“locker.company”: ObjectId(‘id’),***  
***isConfirmed: false,***  
***}***  



[image1]: <./Images/Support/support0.png>

[image1]: <./Images/Support/support1.png>

[image2]: <./Images/Support/support2.png>

[image3]: <./Images/Support/support3.png>

[image4]: <./Images/Support/support4.png>

[image5]: <./Images/Support/support5.png>

[image6]: <./Images/Support/support6.png>

[image7]: <./Images/Support/support7.png>

[image8]: <./Images/Support/support8.png>

[image9]: <./Images/Support/support9.png>

[image10]: <./Images/Support/support10.png>

[image11]: <./Images/Support/support11.png>

[image12]: <./Images/Support/support12.png>

[image13]: <./Images/Support/support13.png>

[image14]: <./Images/Support/support14.png>

[image15]: <./Images/Support/support15.png>

[image16]: <./Images/Support/support16.png>

[image17]: <./Images/Support/support17.png>