# Unidad 6

## Bitácora de proceso de aprendizaje

¿Cuál es la diferencia entre recibir un mensaje y ejecutarlo?
recibir un mensaje es solo que el programa registra que ocurrio un evento (solo la llegada de información)
ejecutar un mensaje es que aparte de registrar lo ocurrido el programa ejecuta una accion correspondiente a ese mensaje (responder con una accion)

¿Por qué un sistema audiovisual puede necesitar timestamp además de los datos del evento?
para que se pueda sincronizar las salidas de audio con las visuales

¿Qué aspectos de la arquitectura de las unidades 4 y 5 permanecen intactos aunque ahora la fuente de datos ya no sea hardware?
maquina de estados, temporizador, logica de los eventos, separacion de entrada, procesamiento y salida

### Paso 1

Si Strudel fuera “el dispositivo” de esta unidad, ¿Cuál sería su protocolo?
cada mensaje incluye que evento ocurrió, se acompaña de un timestamp, se interpretan los mensajes como comandos para activar los sonidos

¿Qué variables mínimas necesitarías extraer para poder construir una visualización útil?
tipo de evento( cual nota fue tocada), timestamp(en que momento ocurrió), identificador de fuente(que nota fue tocada)

### Paso 2

¿Qué problema resuelve la cola de eventos?
evita que haya latencia 

¿Por qué esta capa no pertenece al bridge sino al lado que interpreta el evento?
el bridge solo deberia de encargarse de mover los mensajes de una aplicacion a otra, decidir cuando se ejecuta es responsabilidad del frontend

### Paso 3

¿Qué papel cumple el Adapter en U4 y U5?
traduce las señales del hardware a mensajes para la maquina de estados

¿Qué Adapter necesitas ahora para que los eventos de Strudel no entren “crudos” al sistema visual?
un adapter que convierta las notas y instrucciones en mensajes con tipo de evento, timestamp, intensiad/valor, fuante/canal

## Bitácora de aplicación 


## Bitácora de reflexión
