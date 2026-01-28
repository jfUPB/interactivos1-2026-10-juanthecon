# Unidad 1

## Bitácora de proceso de aprendizaje

### Actividad 01

Primer texto de la actividad 1

### Actividad 03
Sacude el micro:bit. ¿Qué pasa? = el circulo cambia a verde
Presiona el botón Send Love. ¿Qué pasa? = el microbit muestra un corazon

### Actividad 04

funciona con is_pressed() por que el sistema reconoce cada vez que se presiona y no si se presiono alguna vez como lo hace was_pressed()


## Bitácora de aplicación 


### Actividad 05


```js
let port;
let connectBtn;
let posX=0;

function setup() {
  createCanvas(400, 400);
  background(220);
  port = createSerial();
  connectBtn = createButton("Connect to micro:bit");
  connectBtn.position(130, 300);
  connectBtn.mousePressed(connectBtnClick);
  fill("white");
  ellipse(50, 200, 100, 100);
}

function draw() {
  if (port.availableBytes() > 0) {
    let dataRx = port.read(1);
    if (dataRx == "A") {
      posX = posX-30;
      
      
    } else if (dataRx == "B") {
      posX = posX+30;
    }
    

    background(220);
    ellipse(posX, 200, 100, 100);

  }

  if (!port.opened()) {
    connectBtn.html("Connect to micro:bit");
  } else {
    connectBtn.html("Disconnect");
  }
}

function connectBtnClick() {
  if (!port.opened()) {
    port.open("MicroPython", 115200);
  } else {
    port.close();
  }
}
```
Se crea un circulo de color blanco y un boton para conectar y desconectar el microbit, luego el programa espera recibir la letra A para mover ese circulo 30 pixeles a la izquierda o espera recibir la letra B para mover 30 pixeles hacia la derecha

```py
from microbit import *

uart.init(baudrate=115200)

while True:

    if button_a.is_pressed():
        uart.write('A')
    elif button_b.is_pressed():
        uart.write('B')

    sleep(100)
```
El programa espera a que se presione el boton a para enviar la letra A o espera que se presione el boton b para enviar la letra B


## Bitácora de reflexión


### Actividad 06
El programa de p5 crea un cuadrado blanco y un boton para conectar y desconectar el microbit y espera recibir la letra A para que el cuabrado cambie a color rojo y tambien espera recibir la letra B para que cambie a verde


