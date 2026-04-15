# unidad 4


```py
from microbit import *
import utime

# Configurar UART a 115200 baud
uart.init(baudrate=115200)

# Función para calcular checksum
def calculate_checksum(x, y, a, b):
    return abs(x) + abs(y) + a + b

# Timestamp inicial
start_time = utime.ticks_ms()

while True:
    # Leer acelerómetro (valores en mili-g, rango típico -2048 a 2047)
    accel_x = accelerometer.get_x()
    accel_y = accelerometer.get_y()
    
    # Leer botones (1 si presionado, 0 si no)
    btn_a = 1 if button_a.is_pressed() else 0
    btn_b = 1 if button_b.is_pressed() else 0
    
    # Calcular timestamp desde arranque
    timestamp = utime.ticks_ms() - start_time
    
    # Calcular checksum
    checksum = calculate_checksum(accel_x, accel_y, btn_a, btn_b)
    
    # Construir trama ASCII
    frame = f"$T:{timestamp}|X:{accel_x}|Y:{accel_y}|A:{btn_a}|B:{btn_b}|CHK:{checksum}\n"
    
    # Enviar por UART
    uart.write(frame)
    
    # Esperar para mantener 10 Hz (100ms)
    sleep(100)
```
## 1. Componentes del sistema

| Componente | Tipo | Función principal | Entrada | Salida | Interacción |
|---|---|---|---|---|---|
| Hardware sensor | Dispositivo físico | Envía tramas ASCII con datos de acelerómetro y botones | - | `$T:...|X:...|Y:...|A:...|B:...|CHK:...\n` | Puerto serial 115200 |
| `MicrobitV2Adapter.js` | Adapter / Capa de interpretación | Parseo del protocolo, validación checksum, estandarización de datos | Tramas seriales ASCII | `{x, y, btnA, btnB}` | Emite `onData` al servidor |
| `bridgeServer.js` | Servidor Node.js / Capa de transporte | Conecta adaptador, abre WebSocket, retransmite datos al navegador | Datos estandarizados del adaptador | Mensajes WebSocket JSON | Broadcast a clientes WebSocket |
| `bridgeClient.js` | Cliente WebSocket | Recibe datos desde servidor, expone eventos de aplicación | Mensajes WebSocket JSON | Eventos `EVENTS.DATA`, `EVENTS.CONNECT`, etc. | Llama callbacks en `sketch.js` |
| `sketch.js` | Frontend p5.js / FSM | Administra estados, procesa datos de hardware, renderiza arte | Eventos `EVENTS.DATA` con payload | Dibujo en canvas | Usa `PainterTask` y `drawRunning()` |
| `PainterTask` | FSM / Lógica de aplicación | Separa ingesta de datos y renderizado | Eventos FSM (`CONNECT`, `DATA`, `DISCONNECT`) | Estado interno para dibujo | `updateLogic()` y `drawRunning()` |
| `drawRunning()` | Render | Dibujo generativo tonto basado en estado | Estado `generativeState` | Polígonos en canvas | Llamado desde `draw()` cada frame |
| `updateLogic()` | Lógica | Transforma datos crudos en parámetros gráficos | `{x,y,btnA,btnB}` | `rxData` y `generativeState` | Ejecutado en evento `DATA` |

## 2. Flujo de información detallado

| Paso | De | A | Qué se transfiere | Qué ocurre |
|---|---|---|---|---|
| 1 | Hardware sensor | Puerto serial del PC | Trama ASCII con campos `T,X,Y,A,B,CHK` | Sensor envía 10 Hz |
| 2 | Puerto serial | `MicrobitV2Adapter` | Bytes seriales | `_onChunk()` acumula hasta `\n` |
| 3 | `MicrobitV2Adapter` | Interno de adapter | Línea completa ASCII | `parseAsciiFrame()` parsea y valida |
| 4 | `MicrobitV2Adapter` | `bridgeServer` | `{x, y, btnA, btnB}` | Si checksum inválido, descarta y loguea warning |
| 5 | `bridgeServer` | WebSocket clients | `{type:"microbit", x, y, btnA, btnB, t}` | Broadcast a todos los clientes conectados |
| 6 | `bridgeClient` | `sketch.js` | Evento `EVENTS.DATA` | Despacha payload en la FSM |
| 7 | `PainterTask` | Estado interno | `data` de hardware | `updateLogic()` mapea valores |
| 8 | `PainterTask` | Render | Estado `generativeState` | `drawRunning()` dibuja polígono |
| 9 | `p5.js` | Pantalla | Canvas gráfico | Usuario ve arte generativo |

## 3. Interacciones entre componentes

- **Hardware sensor ↔ Adaptador**
  - El sensor solo envía datos.
  - El adaptador solo lee y valida; no cambia firmware.
- **Adaptador ↔ Servidor**
  - `MicrobitV2Adapter` emite datos limpios mediante `this.onData?.(...)`.
  - `bridgeServer` no conoce el protocolo del sensor, solo el objeto estandarizado.
- **Servidor ↔ Cliente**
  - WebSocket actúa como transporte transparente.
  - El servidor retransmite JSON a todos los clientes sin modificar la semántica del objeto.
- **Cliente ↔ FSM**
  - `bridgeClient` lanza eventos `EVENTS.DATA`.
  - `PainterTask` recibe esos eventos y actualiza estado.
- **FSM ↔ Render**
  - `updateLogic()` se encarga de la matemática y del mapeo.
  - `drawRunning()` solo lee estado y dibuja, sin lógica de datos.
- **Botones físicos**
  - `btnA` activa el dibujo del polígono.
  - `btnB` activa el relleno.
- **Acelerómetro**
  - Eje `X` controla el radio del polígono.
  - Eje `Y` controla la resolución (cantidad de vértices).

## 4. Notas de diseño importantes

- El servidor y la capa de transporte quedaron **sin cambios de arquitectura**.
- El nuevo adaptador actúa como una **puerta de traducción limpia**.
- La lógica del arte generativo se implementó respetando la **separación entre ingestión y renderizado**.
- El sistema queda listo para que otro hardware distinto pueda reemplazar el adaptador sin tocar el frontend ni el servidor.
