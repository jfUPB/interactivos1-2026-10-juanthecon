# Unidad 1

## Bitácora de proceso de aprendizaje

### Actividad 01

Primer texto de la actividad 1

### Actividad 03
Sacude el micro:bit. ¿Qué pasa? = cambia a verde
Presiona el botón Send Love. ¿Qué pasa? = muestra un corazon

### Actividad 04

funciona con is_pressed() por que el sistema reconoce cada vez que se presiona y no si se presiono alguna vez como lo hace was_pressed()

### Actividad 05

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



## Bitácora de aplicación 



## Bitácora de reflexión


