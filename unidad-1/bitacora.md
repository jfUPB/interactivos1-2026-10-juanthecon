# Unidad 1

## Bitácora de proceso de aprendizaje

### Actividad 01

Primer texto de la actividad 1

### Actividad 03
Sacude el micro:bit. ¿Qué pasa? = 
Presiona el botón Send Love. ¿Qué pasa?

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
  ellipse(300, 100, 100, 100);
}

function draw() {
  if (port.availableBytes() > 0) {
    let dataRx = port.read(1);
    if (dataRx == "A") {
      posX = posX-20;
      
      
    } else if (dataRx == "B") {
      posX = posX+20;
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

