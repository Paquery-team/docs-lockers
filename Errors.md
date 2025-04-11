# Chazki México 2024

## Plataformas utilizadas

| # | Plataforma     | Uso         |
|---|----------------|-------------|
| 1 | VPN            | Iniciar sesión en raspberry asociada al locker <br> Utilizar comando journatl <br> Buscar línea por línea -new message    |
| 2 | Compass Mongo        | Conectarse a base de datos<br>Abrir coleciones <br> Crear Query <br> Estados de reservas <br> Obtener UUID asociado <br> Fecha y Hora del evento    |
| 3 | Admin IOT          | Verificar estatus de locker (Offline - Online) <br> Abrir puertas <br> Abrir torres   |
| 4 | Scalefusion    | Verificar estado del ipad <br> Comando de actualizacion <br> Comando de reinicio   |
| 5 | Incontrol      | Seleccionar dispositivo asociado al locker <br> Verificar estatus Offline - Online <br> Validar clientes conectados al locker <br> Verificar estatus respberry principal <br> Reiniciar dispositivo <br> Reconectar router  |
| 6 | Excel Maaster Ubicaciones          | Buscar UUID de locker<br>Estado al que pertenece el locker      |

## Pantalla congelada (Tiempo solución 15 min a 1 hr)

Companies es la plataforma que utiliza el clientePantalla congelada (Tiempo solución 15 min a 1 hr)

- Abrir excel de master ubicaciones
- Buscar el número de sucursal reportado para saber cual es el UUID del locker
- Abrir Scalefusión y loguearse (dev@clicknbox.com)
- Buscar ipad asociado al locker
- Verificar el estado del ipad (Inactivo, activo, bloqueado y/o desbloqueado)
- Enviar comando de actualización de dispositivo
- Enviar comando de reinicio de dispositivo
- Esperar un mínimo de 3 minutos para solicitar revisión

## Solicitan apertura remota (Tiempo solución 15 min a 3:30 hr)

Si notifican que el box no abrió, se realiza apertura de manera remota siempre validando que el retiro se haya intentado en el momento.

1. Abrir Admin IOT y loguearse  
2. Buscar locker por UUID  
3. Verificar su estatus (Offline - Online)  
   - **Online:**
     1. Seleccionar Administrar  
     2. Seleccionar abrir puerta  
     3. Seleccionar torre y puerta (Con el UUID del box asociado - Construcción - 011400208)  
     4. Se informa que se realizó apertura  
   - **Offline:**
     1. Abrir Excel de master ubicaciones  
     2. Buscar por número de sucursal el estado al que pertenece  
     3. Abrir Incontrol y loguearse  
     4. Seleccionar el estado en el dashboard principal  
     5. Seleccionar el dispositivo asociado al locker por UUID  
     6. Verificar estatus (Offline - Online)  
        - **Offline:**
          - Se solicita desconexión por 3 hrs.  
        - **Online:**
          1. Validar clientes conectados  
          2. Verificar Raspberry principal (conectada o desconectada)  
             - **Conectada:**
               1. Seleccionar en settings "Device Tools"  
               2. Seleccionar "Reiniciar dispositivo"  
               3. Esperar reconexión de router (aprox. 5 minutos)  
               4. Validar clientes conectados  
               5. Verificar Raspberry principal (conectada o desconectada)  
                  - **Conectada:** Regresar a Admin IOT y realizar apertura de box  
                  - **Desconectada:** Se manda ticket a mantenimiento  
             - **Desconectada:** 
               1. Se manda ticket a mantenimiento  


## Error 404 - Reservación no encontrada (Tiempo solución 15 min a 3:30 hrs)

1. Abrir compass (mongo)
2. Conectarse a la base de datos de clicknbox
3. Abrir la colección de reservation_history
4. Realizar query en compass del shipment reportado - (1 query utilizada)
5. Verificar el estado de la reserva en el event history
6. Identificar el UUID del box asociado
7. Se revisa la fecha del evento y se debe convertir la hora de UTC a hora de México.

## Error 404 - Reservación no encontrada (Tiempo solución 15 min a 3:30 hrs)

### Escenario 1 - Picked up (3 plataformas)

Se notifica que el item ya fue recolectado y se proporciona la fecha.

  - Si notifican que el box no abrió, se realiza apertura de manera remota. (Se realizan todos los pasos de 2do caso )

### Escenario 2- Reserved (1 plataforma)

Si el estatus es reserved, se notifica que aún no se ha depositado el pedido en el box.

### Escenario 3 - Delivery (2 plataformas)

Si el estatus es delivery:

1. Abrir la colección en Compass de `codes`  
2. Realizar query del token reportado en Compass  
   - **El token proporcionado es incorrecto:**  
     - Se notifica  
   - **El token proporcionado es correcto:**  
     1. Conectar equipo a la VPN  
     2. Mediante consola, iniciar sesión SSH en la Raspberry asociada al locker  
     3. Mediante el comando `journalctl`, desplegar los logs del sistema  
     4. Buscar línea por línea en el log el mensaje `"New message"` que contenga el UUID del box asociado  
     5. Revisar el token que se está ingresando  
        - **El token ingresado es incorrecto:**  
          - Se notifica  

