# Hardware

| Componente | Modelo / Especificación |
| ----- | ----- |
| Unidad controladora central | Raspberry Pi 3 Modelo B |
| Unidad controladora de torre | Raspberry Zero W V1.1 |
| Almacenamiento para SO Raspberries | MicroSD 32 GB clase 10 en adelante |
| Fuente de alimentación para electrónica | Fuente ATX 600W en adelante. (Salidas 12V @ 15A & 5V @ 24A) |
| UPS del sistema  | No Break 240W, 520VA, Entrada 90V \- 145V, Salida 108V \- 132V, 6 Salidas al menos |
| PCB Interfaz Raspberry-Relé | Desarrollo propio del PCB |
| Módulo de relevadores | Modulo de relevadores dobleVoltaje entrada: 5V Voltaje de control: 0 v/Low trigger Voltaje de salida: 250 VCA o 30 VDC Corriente a la salida: 10 A |
| Cerraduras | Cerradura electrónica DC 12V 2A |
| Pantalla locker | Para locker 1.1 iPad 6 de 9.7 pulgadas (diagonal) con 32 GB de memoria en adelante Para locker 1.6 iPad 7,8 ó 9 de 10.2 pulgadas (diagonal) con 32 GB de memoria en adelante |
| Router  | Pepwave MAX BR1 Mini router con modulación 802.11 b/g/n, CDMA, HSPA+, LTE  El router debe tener además configurada la VPN por el proveedor para poder conectarse a la Unidad controladora central (Raspberry) Antena SMA externa  |
| Sim de datos  | SIM de datos móviles Mini 2FF: 25mm x 15mm x 0.76mm con capacidad de por lo menos 3 GB al mes Conectividad LTE o 3G mínimo |
| Cable de conexión a la energía eléctrica | Cable uso rudo 3x12Conector Leviton NEMA: 515CV Clavija Leviton NEMA: 515PV Voltaje: 125V Amperaje: 15A |
| Estructura de locker | Lámina de acero ASTM A36 calibre 14 Acabado: pintura electrostática |
| Cable ethernet | Cat6 o superior con longitud requerida para conectar componentes al router. |
| Arnés de electrónica | Cables de calibre 20AWG para fabricación de arnés, con longitudes dependiendo del número de boxes en el locker. Cable Negro para GNDCable Rojo para \+5V Cable Amarillo para \+12V Cable anaranjado o variado para señales de activación de los relés del locker. |

 

Software 

| Componente | Modelo / Especificación |
| ----- | ----- |
| Sistema operativo principal | Raspbian para Raspberry 3BPRETTY\_NAME="Raspbian GNU/Linux 10 (buster)" NAME="Raspbian GNU/Linux" VERSION\_ID="10" VERSION="10 (buster)" VERSION\_CODENAME=buster ID=raspbian ID\_LIKE=debian HOME\_URL="http://www.raspbian.org/" SUPPORT\_URL="http://www.raspbian.org/RaspbianForums" BUG\_REPORT\_URL="http://www.raspbian.org/RaspbianBugs" |
| Sistema operativo secundario | Raspbian para Raspberry Zero W PRETTY\_NAME="Raspbian GNU/Linux 10 (buster)" NAME="Raspbian GNU/Linux" VERSION\_ID="10" VERSION="10 (buster)" VERSION\_CODENAME=buster ID=raspbian ID\_LIKE=debian HOME\_URL="http://www.raspbian.org/" SUPPORT\_URL="http://www.raspbian.org/RaspbianForums" BUG\_REPORT\_URL="http://www.raspbian.org/RaspbianBugs"  |
| Librerías | ?? |
| Módulos | ?? |
| Scalefusion | Solución de gestión de dispositivos móviles (MDM) y administración de terminales (quioscos) instalado en cada iPad del locker y con licencia activa (1 licencia para cada locker) |
| Admin IoT | Software para la gestión de lockers con funciones activas de:  Reserva en locker Apertura de puerta Apertura de locker Alta y edición de información de locker |

Sistema Operativo: Raspberry Pi OS Lite (basado en Debian)  
Lenguajes soportados: Python 3, Bash  
Librerías: RPi.GPIO, Flask (para API), sqlite3 o integración con base de datos externa  
Módulos de seguridad: SSH, firewall (ufw), actualizaciones automáticas  
