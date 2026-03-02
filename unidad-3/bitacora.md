# Unidad 3

## Bitácora de proceso de aprendizaje

comandos de terminal
pwd (path working directory)
ls -al (ver el contenido de pwd)
clear (borrar la terminal)
cd (cambiar ruta de direccion)


## Bitácora de aplicación 


p5 

``` jav
let port;
let connectBtn;
let writer;

const TIMER_LIMITS = {
  min: 15,
  max: 25,
  defaultValue: 20,
};

const EVENTS = {
  DEC: "A",
  INC: "B",
  START: "S",
  TICK: "Timeout",
};

const UI = {
  dialSize: 250,
  ringWeight: 20,
  bigText: 100,
  configText: 120,
  helpText: 18,
};


class Temporizador extends FSMTask {
  constructor(minValue, maxValue, defaultValue) {
    super();

    this.minValue = minValue;
    this.maxValue = maxValue;
    this.defaultValue = defaultValue;
    this.configValue = defaultValue;
    this.totalSeconds = defaultValue;
    this.remainingSeconds = defaultValue;

    this.myTimer = this.addTimer(EVENTS.TICK, 1000);
    this.transitionTo(this.estado_config);

    //this.keyBuffer = [];
  }


handleKeySequence(key) {
    if (!this.keySequence) this.keySequence = [];
    this.keySequence.push(key);
    if (this.keySequence.length > 3) this.keySequence.shift();

    if (
      this.keySequence.length === 3 &&
      this.keySequence[0] === 'S' &&
      this.keySequence[1] === 'B' &&
      this.keySequence[2] === 'S'
    ) {
      this.postEvent(EVENTS.START); 
      this.keySequence = [];
      return;
    }

    if (key === 'A') this.postEvent(EVENTS.DEC);
    if (key === 'B') this.postEvent(EVENTS.INC);
    if (key === 'S') this.postEvent(EVENTS.START);
  }

  get currentState() {
    return this.state;
  }



  estado_config = (ev) => {
    if (ev === ENTRY) {
      this.configValue = this.defaultValue;
      this.keyBuffer = [];
    }
    else if (ev === EVENTS.DEC) {
      if (this.configValue > this.minValue) this.configValue--;
    } else if (ev === EVENTS.INC) {
      if (this.configValue < this.maxValue) this.configValue++;
    } else if (ev === EVENTS.START) {
      this.totalSeconds = this.configValue;
      this.remainingSeconds = this.totalSeconds;
      this.transitionTo(this.estado_armed);
    }
  };



  estado_armed = (ev) => {
    if (ev === ENTRY) {
      this.myTimer.start();
      this.keyBuffer = [];
    } else if (ev === EVENTS.TICK) {
      if (this.remainingSeconds > 0) {
        this.remainingSeconds--;
        if (this.remainingSeconds === 0) {
          this.transitionTo(this.estado_timeout);
        } else {
          this.myTimer.start();
        }
      }
    } else if (ev === EVENTS.DEC) {

      this.transitionTo(this.estado_paused);
    } else if (ev === EXIT) {
      this.myTimer.stop();
    }
  };


  estado_paused = (ev) => {
    if (ev === ENTRY) {
      this.myTimer.stop();
    } else if (ev === EVENTS.DEC) {

      this.transitionTo(this.estado_armed);
    } else if (ev === EVENTS.INC) {
      this.keyBuffer.push("B");
    }
  };

  estado_timeout = (ev) => {
    if (ev === ENTRY) {
      console.log("¡TIEMPO!");
      this.keyBuffer = [];
    } else if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_config);
    }
  };

  handleKeySequence(key) {
    this.keyBuffer.push(key);
    if (this.keyBuffer.length > 3) this.keyBuffer.shift();

    if (this.keyBuffer.join("") === "SBS" && this.currentState === this.estado_armed) {
      this.transitionTo(this.estado_config);
    }
  }
}


let temporizador;
const renderer = new Map();

function setup() {
  createCanvas(windowWidth, windowHeight);
  console.log("Setup started");
  temporizador = new Temporizador(
    TIMER_LIMITS.min,
    TIMER_LIMITS.max,
    TIMER_LIMITS.defaultValue
  );
  textAlign(CENTER, CENTER);

  renderer.set(temporizador.estado_config, () => drawConfig(temporizador.configValue));
  renderer.set(temporizador.estado_armed, () => drawArmed(temporizador.remainingSeconds, temporizador.totalSeconds));
  renderer.set(temporizador.estado_timeout, () => drawTimeout());


    port = createSerial();
    connectBtn = createButton('Connect to micro:bit');
    connectBtn.position(80, 300);
    connectBtn.mousePressed(connectBtnClick);

}


async function connectBtnClick() {
    try {
        if (typeof port.requestPort === 'function') {
            await port.requestPort();
        }
        await port.open(115200);
        connectBtn.html('Disconnect');
        console.log('Serial opened');
        if (typeof port.on === 'function') {
            port.on('data', serialEvent);
            port.on('error', e => console.error('serial error', e));
        }
    } catch (err) {
        console.error('Could not open serial port:', err);
        alert('Serial error: ' + (err.message || err));
    }
}



/*
function connectBtnClick() {
  if (!port.opened()) {
    port.open(115200);
    connectBtn.html('Disconnect');
  } else {
    port.close();
    connectBtn.html('Connect to micro:bit');
  }
}
*/



function draw() {
  temporizador.update();
  renderer.get(temporizador.currentState)?.();
}

function drawConfig(val) {
  background(20, 40, 80);
  fill(255);
  textSize(120);
  text(val, width / 2, height / 2);
  textSize(18);
  fill(200);
  text("A(-) B(+) S(start)", width / 2, height / 2 + 100);
}

function drawArmed(val, total) {
  background(20, 20, 20);
  let pulse = sin(frameCount * 0.1) * 10;

  noFill();
  strokeWeight(20);
  stroke(255, 100, 0, 50);
  ellipse(width / 2, height / 2, 250);

  stroke(255, 150, 0);
  let angle = map(val, 0, total, 0, TWO_PI);
  arc(width / 2, height / 2, 250, 250, -HALF_PI, angle - HALF_PI);

  fill(255);
  noStroke();
  textSize(100 + pulse);
  text(val, width / 2, height / 2);
}

function drawTimeout() {
  let bg = frameCount % 20 < 10 ? color(150, 0, 0) : color(255, 0, 0);
  background(bg);
  fill(255);
  textSize(100);
  text("¡TIEMPO!", width / 2, height / 2);
}

function keyPressed() {
  if (key === "a" || key === "A") {
    temporizador.postEvent("A");
  }
  if (key === "b" || key === "B") {
    temporizador.postEvent("B");
    temporizador.handleKeySequence("B");
  }
  if (key === "s" || key === "S") {
    temporizador.postEvent("S");
    temporizador.handleKeySequence("S");
  }
}



function serialEvent() {
  let data = port.readUntil("\n");
  if (data) {
    data = data.trim();
    if (data === "A") temporizador.postEvent("A");
    if (data === "B") temporizador.postEvent("B");
    if (data === "S") temporizador.postEvent("S");
  }
}


function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}

```


micro:bit

``` jav

from microbit import *

uart.init(baudrate=115200)

while True:
    if button_a.was_pressed():
        uart.write('A')
        display.show('A')
        sleep(200)
    elif button_b.was_pressed():
        uart.write('B')
        display.show('B')
        sleep(200)

    if accelerometer.was_gesture('shake'):
        uart.write('S')
        display.show('S')
        sleep(300)
    sleep(50)

```
## Bitácora de reflexión



