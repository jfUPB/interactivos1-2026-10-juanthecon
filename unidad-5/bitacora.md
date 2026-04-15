# unidad 5

---

## 1. Tabla Comparativa

| Aspecto | ASCII | Binario |
|---------|-------|---------|
| **Tamaño de paquete** | Variable (~25-30 bytes) | Fijo (8 bytes) |
| **Ejemplo de datos** | `"500,524,true,false\n"` | `AA 01 F4 02 0C 01 00 FE` |
| **Bytes reales (ejemplo)** | 19 caracteres UTF-8 + \n = **20 bytes** | **8 bytes** |
| **Delimitador/Framing** | Carácter `\n` (0x0A) al final | Byte de sincronización 0xAA + longitud fija (7 más) |
| **Verificación de integridad** | Validación de rango y tipo | Checksum: suma(bytes 1-6) % 256 |
| **Complejidad parser** | ```split(",")``` + conversión string → número | Lectura binaria directa: `readInt16BE()` + verificación |
| **Complejidad depuración** | **MÁS FÁCIL**: Legible en terminal `cat /dev/ttyACM0` | **DIFÍCIL**: Bytes no imprimibles requieren `hexdump` |
| **Robustez frente a corrupción** | Fallo completo de la línea si hay error | Resincronización automática (busca siguiente header 0xAA) |
| **Ancho de banda @ 115200 baud** | ~5.75 paquetes/s (considerando overhead) | ~14.4 paquetes/s (2.8x más eficiente) |
| **Tolerancia a ruido** | Baja: caracteres ASCII corruptos rompen parseo | Media: resincroniza buscando header válido |

### Resumen Numérico
- Cada paquete binario es **62.5% más pequeño** (8 vs 20 bytes)
- A 10 Hz, el protocolo binario usa **0.8 kbps** vs **2 kbps** ASCII (2.5x menos ancho de banda)

---

## 2. Arquitectura Desacoplada: Por qué Permite Cambios sin Afectar el Frontend

### Patrón Arquitectónico: Adapter Pattern + Bridge + FSM

```
[sketch.js] (lógica de negocio)
     ↓
[bridgeClient.js] (contrato: {type, x, y, btnA, btnB, t})
     ↓
[bridgeServer.js] (abstracción de hardware)
     ↓
[Adapter concreto] ← AQUÍ CAMBIA (ASCII vs Binary)
     ↓
[Puerto Serial]
```

### Mecanismos de Desacoplamiento

#### 1. Contrato de Datos Invariable
Todos los adaptadores emiten:
```javascript
{ x, y, btnA, btnB }  // Idéntico formato
```
- `sketch.js` no conoce si vienen de ASCII, binario, o simulador
- Solo consume los datos en un único formato

#### 2. Inyección de Dependencias
`bridgeServer.js` elige qué adaptador usar:
```javascript
const adapter = DEVICE === "microbit" 
  ? new MicrobitAsciiAdapter(...) 
  : new MicrobitBinaryAdapter(...)
// Pero ambos cumplen: adapter.onData?.({x, y, btnA, btnB})
```

#### 3. Interfaz Común: BaseAdapter
```javascript
class BaseAdapter {
  async connect()     // Contrato: todos lo implementan igual
  async disconnect()
  onData              // Callback: siempre emite {x, y, btnA, btnB}
  handleCommand(cmd)  // Manejo de comandos
}
```

#### 4. Transparencia del Transporte
`bridgeClient.js` solo recibe WebSocket:
```javascript
ws.onmessage = (msg) => {
  const data = JSON.parse(msg.data);
  // No importa si data.x vino de serial ASCII o binario
  updateSketch(data.x, data.y);
}
```

### Conclusión del Desacoplamiento
La arquitectura **descentraliza el conocimiento del protocolo** en el adaptador específico. El frontend (`sketch.js`) es **agnóstico al protocolo subyacente**.

---

## 3. Protocolo Binario vs ASCII: Casos de Uso del Mundo Real

### Preferir BINARIO cuando:

| Situación | Razón | Ejemplo |
|-----------|-------|---------|
| **Ancho de banda limitado** | 2.8x más eficiente | IoT con conexión 2G/LoRaWAN |
| **Latencia crítica** | Menos bytes = procesamiento más veloz | Sistemas de control de robots en tiempo real |
| **Múltiples sensores de alta frecuencia** | Agregación eficiente | Captura de vídeo a 60 FPS + acelerómetro paralelo |
| **Datos científicos de precisión** | Mejor representación numérica | Sensores que requieren 4+ bytes por valor |
| **Energía limitada (baterías)** | Tx/Rx menos energético | Wearables, tags RFID |
| **Almacenamiento limitado** | Tamaño más pequeño | Microcontroladores con < 256 KB RAM |

**Ejemplo concreto:**
- Micro:bit enviando captura acelerómetro (100 Hz): **ASCII = 20 kbps** | **Binario = 6.4 kbps** (3x ahorro)

---

### Preferir ASCII cuando:

| Situación | Razón | Ejemplo |
|-----------|-------|---------|
| **Depuración frecuente** | Legible en `cat /dev/ttyACM0` | Desarrollo iterativo |
| **Múltiples plataformas heterogéneas** | Portable sin compilación | Server en Python, cliente en Java, firmware en C |
| **Formato extensible dinámicamente** | Fácil agregar campos | `x,y,btnA,btnB,temp,pressure\n` |
| **Requisitos de auditoría** | Logs en texto plano | Sistemas críticos (medicina, finanzas) |
| **Ancho de banda no es cuello de botella** | Simplicidad > rendimiento | Sistema local con USB 2.0 @ 480 Mbps |

**Ejemplo concreto:**
- Prototipo educativo (esta unidad): ASCII es mejor porque quieres ver `"500,524,true,false"` en la terminal, no `AA 01 F4 02 0C 01 00 FE`

---

## 4. Diagrama de Flujo de Datos: Actualizado con Protocolo Binario

### Unidad 4 (ASCII)

```
[Micro:bit]
    ↓ (UART @ 115200)
    ↓ "500,524,true,false\n"
[Serial Port]
    ↓
[MicrobitASCIIAdapter]
    ├─ _onChunk(chunk)
    ├─ buf += chunk.toString("utf8")
    ├─ búsqueda: indexOf("\n")
    ├─ parseCsvLine(): split(",")
    └─ onData({ x: 500, y: 524, btnA: true, btnB: false })
    ↓
[bridgeServer.js]
    ├─ broadcast(wss, { type: "microbit", x, y, btnA, btnB, t })
    ↓
[WebSocket JSON]
    └─ Cliente: {type: "microbit", x: 500, y: 524, btnA: true, btnB: false, t: 1744488000}
    ↓
[bridgeClient.js]
    ↓
[sketch.js → FSM → canvas]
```

### Unidad 5 (BINARIO) - NUEVA

```
[Micro:bit]
    ↓ (UART @ 115200, 10 Hz)
    ↓ AA 01 F4 02 0C 01 00 FE (8 bytes)
[Serial Port]
    ↓
[MicrobitBinaryAdapter] ← CAMBIÓ
    ├─ _onChunk(chunk)
    ├─ buf = Buffer.concat([buf, chunk])
    ├─ búsqueda: indexOf(0xAA)
    ├─ verificar checksum: (bytes[1..6]).sum() % 256 === bytes[7]
    ├─ readInt16BE(1) → x = 500
    ├─ readInt16BE(3) → y = 524
    ├─ bytes[5] === 1 → btnA = true
    ├─ bytes[6] === 0 → btnB = false
    └─ onData({ x: 500, y: 524, btnA: true, btnB: false })
    ↓
[bridgeServer.js] ← SIN CAMBIOS
    ├─ broadcast(wss, { type: "microbit", x, y, btnA, btnB, t })
    ↓
[WebSocket JSON] ← SIN CAMBIOS
    └─ Cliente: mismo JSON que antes
    ↓
[bridgeClient.js] ← SIN CAMBIOS
    ↓
[sketch.js → FSM → canvas] ← SIN CAMBIOS
```

---

## 5. Análisis de Componentes

### ¿Qué componentes cambiaron?

| Componente | Estado | Razón |
|-----------|--------|-------|
| **MicrobitBinaryAdapter.js** |  NUEVO | Implementa decodificación binaria |
| **parseCsvLine()** |  NO EXISTE | Reemplazado por `readInt16BE()` |
| **Buffer acumulador** |  EVOLUCIONADO | `string` → `Buffer` (Node.js) |
| **Delimitador de trama** |  EVOLUCIONADO | `\n` → header `0xAA` |
| **Validación** |  EVOLUCIONADO | Rango/tipo → checksum (CRC-8) |

### ¿Qué componentes permanecieron intactos?

| Componente | ¿Por qué? |
|-----------|----------|
| **bridgeServer.js** | Emite idéntico `{type: "microbit", x, y, btnA, btnB}` |
| **bridgeClient.js** | Observa WebSocket, no sabe de serial |
| **sketch.js** | Recibe `{ x, y, btnA, btnB }`, no distingue protocolo |
| **fsm.js** | La máquina de estados es agnóstica a la fuente |
| **BaseAdapter** | Interfaz pura, sin cambios |
| **Contrato JSON en WS** | Idéntico: `{type, x, y, btnA, btnB, t}` |

---

## 6. Visualización del Cambio Arquitectónico

### Unidad 4 vs Unidad 5

```
ANTES (Unidad 4):
┌─────────────────────────────────────────────┐
│ Protocolo ASCII → Adapter ASCII → JSON      │
│                                              │
│ sketch.js ← bridgeClient ← bridgeServer     │
└─────────────────────────────────────────────┘
       (1 adaptador = 1 protocolo)

DESPUÉS (Unidad 5):
┌──────────────────────────────────────────────┐
│ Protocolo Binario → Adapter Binario → JSON  │
│                                               │
│ sketch.js ← bridgeClient ← bridgeServer ───┐│
│                                              ││
│ (adaptador intercambiable, frontend idéntico)
└──────────────────────────────────────────────┘
       (N adaptadores = N protocolos, mismo frontend)
```

### Extensibilidad arquitectónica

La arquitectura es **extensible por composición**, no por modificación. Agregar soporte para un nuevo protocolo (SPI binario, CAN-bus, Bluetooth LE) requiere:

1. ✅ Crear `MicrobitSPIAdapter.js`
2. ✅ Registrarlo en `bridgeServer.js` (2 líneas)
3. ✅ Sin tocar `sketch.js`, `bridgeClient.js`, `fsm.js`

---

## 7. Resumen Ejecutivo

### Ventajas del Enfoque Arquitectónico

| Ventaja | Impacto |
|---------|---------|
| **Cambio de protocolo sin modificar frontend** | Reduce deuda técnica |
| **Reutilización de código en capas superiores** | 0 cambios en lógica de negocio |
| **Fácil testing de adaptadores independientes** | Mock adapters para testing |
| **Escalabilidad** | Agregar nuevos protocolos es trivial |
| **Mantenibilidad** | Cambios localizados al nivel correcto |

### Lecciones Aprendidas

1. **Separación de Responsabilidades**: El adaptador es responsable SOLO de traducción de protocolo
2. **Inversión de Dependencias**: Las capas superiores NO dependen de los detalles de protocolo
3. **Contrato explícito**: `{x, y, btnA, btnB}` es el contrato que permite desacoplamiento
4. **Framing es crítico**: El mecanismo de delimitación (checksum vs newline) es lo más específico al protocolo

