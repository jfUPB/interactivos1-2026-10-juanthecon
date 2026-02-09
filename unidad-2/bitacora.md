# Unidad 2

## Bitácora de proceso de aprendizaje

### Actividad 01

¿Cuáles son los estados en el programa?
¿Cuáles son los eventos en el programa?
¿Cuáles son las acciones en el programa?

### Actividad 02

```jav
Estados: estado_waitTimeout

Eventos: ENTRY, EXIT, Timeout

Acciones: encender/apagar píxel, alternar brillo, reiniciar temporizador



from microbit import *
import utime

class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration

        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
            self.start_time = utime.ticks_ms()
            self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
                self.active = False
                self.owner.post_event(self.event)

class Semaforo:
    def __init__(self,_x,_y):
        self.redX = _x
        self.redY = _y
        self.event_queue = []
        self.timers = []
        
        self.myTimer = self.createTimer("Timeout",2000)
        self.estado_actual = None
        self.transicion_a(self.estado_WaitRed)
        
    def createTimer(self,event,duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t

    def post_event(self, ev):
        self.event_queue.append(ev)

    def update(self):
        # 1. Actualizar todos los timers internos automáticamente
        for t in self.timers:
            t.update()

        # 2. Procesar la cola de eventos resultante
        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)

    def transicion_a(self, nuevo_estado):
        if self.estado_actual: self.estado_actual("EXIT")
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")

    def estado_WaitRed(self,ev):
        if ev =="ENTRY":
            display.set_pixel(self.redX,self.redY,9)
            self.myTimer.start(2000)
        if ev =="Timeout":
            display.set_pixel(self.redX,self.redY,0)
            self.transicion_a(self.estado_WaitGreen)
        if ev == "S":
            display.set_pixel(self.redX,self.redY,0)
            self.transicion_a(self.estado_WaitNightOn)
        

    def estado_WaitGreen(self,ev):
        if ev =="ENTRY":
            display.set_pixel(self.redX,self.redY+2,9)
            self.myTimer.start(1000)
        if ev =="Timeout":
            display.set_pixel(self.redX,self.redY+2,0)
            self.transicion_a(self.estado_WaitYellow)
        if ev =="A":
            self.myTimer.stop()
            display.set_pixel(self.redX,self.redY+2,0)
            self.transicion_a(self.estado_WaitYellow)

    def estado_WaitYellow(self,ev):
        if ev =="ENTRY":
            display.set_pixel(self.redX,self.redY+1,9)
            self.myTimer.start(500)
        if ev =="Timeout":
            display.set_pixel(self.redX,self.redY+1,0)
            self.transicion_a(self.estado_WaitRed)

    def estado_WaitNightOn(self,ev):
        if ev == "S":
            if ev =="ENTRY":
                display.set_pixel(self.redX,self.redY+1,9)
                self.myTimer.start(500)
        if ev =="Timeout":
            self.transicion_a(self.estado_WaitNightOff)

    def estado_WaitNightOff(self,ev):
        if ev =="B":
            if ev =="ENTRY":
                display.set_pixel(self.redX,self.redY+1,9)
                self.myTimer.start(500)
        if ev =="Timeout":
            self.transicion_a(self.estado_WaitRed)
            


semaforo1=Semaforo(0,0)

while True:
    
    if button_a.was_pressed():
        semaforo1.post_event("A")
        
    if accelerometer.was_gesture('shake'):
        semaforo1.post_event("S")
    
    semaforo1.update()
    utime.sleep_ms(20)
```

## Bitácora de aplicación 

### actividad 04

```
@startuml
[*] --> CONFIGURACION

state CONFIGURACION {
  [*] --> Esperando
  Esperando --> Esperando : Botón A ↑ tiempo (máx 25)
  Esperando --> Esperando : Botón B ↓ tiempo (mín 15)
  Esperando --> ARMADO : Shake (S)
}

state ARMADO {
  [*] --> Contando
  Contando --> Contando : Timeout cada 1s (Timer)
  Contando --> FINALIZADO : contador == 0
}

state FINALIZADO {
  [*] --> Skull
  Skull --> CONFIGURACION : Botón A (reinicia a 20)
}

@enduml
```


```jav
from microbit import *
import utime
import music

def make_fill_images(on='9', off='0'):
    imgs = []
    for n in range(26):
        rows = []
        k = 0
        for y in range(5):
            row = []
            for x in range(5):
                row.append(on if k < n else off)
                k += 1
            rows.append(''.join(row))
        imgs.append(Image(':'.join(rows)))
    return imgs

FILL = make_fill_images()

class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration
        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
                self.active = False
                self.owner.post_event(self.event)

class Task:
    def __init__(self):
        self.event_queue = []
        self.timers = []
        self.timerSeg = self.createTimer("Timeout", 1000)

        self.valor = 20   # pixeles iniciales
        self.contador = 0 # para cuenta regresiva
        self.estado_actual = None
        self.transicion_a(self.estado_config)

    def createTimer(self, event, duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t

    def post_event(self, ev):
        self.event_queue.append(ev)

    def update(self):
        for t in self.timers:
            t.update()
        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)

    def transicion_a(self, nuevo_estado):
        if self.estado_actual: self.estado_actual("EXIT")
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")


    
    def estado_config(self, ev):
        if ev == "ENTRY":
            self.valor = 20
            display.show(FILL[self.valor])
        elif ev == "A":
            if self.valor < 25:
                self.valor += 1
                display.show(FILL[self.valor])
        elif ev == "B":
            if self.valor > 15:
                self.valor -= 1
                display.show(FILL[self.valor])
        elif ev == "S":
            self.contador = self.valor
            self.transicion_a(self.estado_armado)



    
    def estado_armado(self, ev):
        if ev == "ENTRY":
            self.timerSeg.start()
        elif ev == "Timeout":
            self.contador -= 1
            display.show(FILL[self.contador])
            if self.contador > 0:
                self.timerSeg.start()
            else:
                self.transicion_a(self.estado_final)


    
    def estado_final(self, ev):
        if ev == "ENTRY":
            display.show(Image.SKULL)
            music.play(music.POWER_DOWN)
        elif ev == "A":
            self.transicion_a(self.estado_config)


task = Task()

while True:
    if button_a.was_pressed():
        task.post_event("A")
    if button_b.was_pressed():
        task.post_event("B")
    if accelerometer.was_gesture("shake"):
        task.post_event("S")

    task.update()
    utime.sleep_ms(20)
```



## Bitácora de reflexión



