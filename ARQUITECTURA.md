# ARQUITECTURA DEL PROYECTO DE TELEMETRÍA

## Estructura General

```
KART (Transmisor)
├── Sensores
│   ├── Hall A3144 (velocidad)
│   ├── INA219 (voltaje/corriente)
│   ├── DS18B20 (temperatura)
│   └── GPS NEO-7M (posición)
├── ESP32 Microcontrolador
│   ├── Lectura de sensores
│   ├── Procesamiento de datos
│   └── Creación de paquetes
└── LoRa SX1278 (433 MHz)
    └── Transmisión inalámbrica


ESTACIÓN BASE (Receptor)
├── LoRa SX1278 (433 MHz)
├── ESP32 Microcontrolador
│   ├── Recepción de paquetes
│   ├── Decodificación de datos
│   └── Envío a PC por Serial
└── Puerto Serial USB
    └── PC (Aplicación de monitoreo)
```

---

## Organización de Archivos

```
telemetria-kart/
│
├── config.h                          [Constantes globales configurables]
│
├── Módulos de Sensores
│   ├── hall_sensor.h                 [Sensor Hall A3144 + velocidad]
│   ├── ina219_sensor.h               [INA219 - voltaje/corriente]
│   ├── temperature_sensor.h          [DS18B20 - temperatura]
│   └── gps_sensor.h                  [GPS NEO-7M - posición]
│
├── Módulo de Comunicación
│   └── lora_module.h                 [SX1278 - LoRa 433 MHz]
│
├── Módulo de Datos
│   └── telemetry_packet.h            [Estructura y serialización de datos]
│
├── Programas por Etapa
│   ├── Etapa1_HallSensor.ino         [Test: Sensor Hall]
│   ├── Etapa2_INA219.ino             [Test: INA219]
│   ├── Etapa3_Temperature.ino        [Test: DS18B20]
│   ├── Etapa4_GPS.ino                [Test: GPS]
│   ├── Etapa5_LoRa.ino               [Test: LoRa básico]
│   ├── Etapa6_TelemetryComplete.ino  [Programa completo del kart]
│   ├── Etapa7_LoRaReceiver.ino       [Receptor en estación base]
│   └── Etapa8_SerialBridge.ino       [Puente Serial a PC]
│
├── Guías
│   ├── README_ETAPA1.md              [Esta guía]
│   ├── README_ETAPA2.md              [Próxima]
│   ├── ARQUITECTURA.md               [Este archivo]
│   └── PINES_CONEXIONES.md           [Esquema completo]
│
└── Datos
    └── ejemplos_paquetes.txt         [Formato de datos para debuggin]
```

---

## Flujo de Datos

### Transmisor (Kart)

```
1. LECTURA DE SENSORES (cada ~50-100ms)
   ├── Hall:        contador de pulsos → RPM → velocidad km/h
   ├── INA219:      voltaje (V) + corriente (mA) → potencia (W)
   ├── DS18B20:     temperatura (°C)
   └── GPS:         latitud, longitud, validez

2. PROCESAMIENTO
   ├── Filtrado de datos ruidosos
   ├── Validación de rangos
   ├── Detección de errores

3. EMPAQUETAMIENTO
   ├── Estructura binaria eficiente
   ├── Checksum o CRC para validación
   └── Número de paquete (secuencia)

4. TRANSMISIÓN LoRa
   └── Envío cada ~200-500ms
```

### Receptor (Estación Base)

```
1. RECEPCIÓN LoRa
   ├── Esperar paquete
   ├── Validar checksum
   └── Decodificar datos

2. INTERPRETACIÓN
   └── Extraer variables

3. ENVÍO SERIAL
   ├── Formato texto o binario
   └── Envío a PC a 115200 baud

4. APLICACIÓN PC
   └── Visualización en tiempo real
```

---

## Módulos Principales

### 1. Sensor Hall (`hall_sensor.h`)
- **Entrada:** Interrupción en GPIO
- **Salida:** RPM, velocidad km/h
- **Uso:** En el kart
- **No bloquea:** Usa ISR + cálculo no-bloqueante

### 2. INA219 (`ina219_sensor.h`)
- **Entrada:** I2C (SDA/SCL)
- **Salida:** Voltaje (V), corriente (mA), potencia (W)
- **Uso:** En el kart
- **Bloquea:** Mínimamente (lectura I2C rápida)

### 3. Temperatura (`temperature_sensor.h`)
- **Entrada:** OneWire (GPIO)
- **Salida:** Temperatura (°C)
- **Uso:** En el kart
- **Bloquea:** Mínimamente

### 4. GPS (`gps_sensor.h`)
- **Entrada:** Serial UART (RX/TX)
- **Salida:** Latitud, longitud, validez
- **Uso:** En el kart
- **No bloquea:** Lectura asincrónica por serial

### 5. LoRa (`lora_module.h`)
- **Entrada:** SPI (MOSI/MISO/CLK) + pines de control
- **Salida:** Envío/recepción de datos
- **Uso:** En ambos (kart + estación)
- **No bloquea:** Puede enviar/recibir sin bloquear

### 6. Paquete de telemetría (`telemetry_packet.h`)
- **Función:** Serializar y deserializar datos
- **Uso:** En ambos (kart para crear, receptor para decodificar)

---

## Flujo de Compilación

### Transmisor (Kart)

Seleccionar programa según etapa:
```
config.h + hall_sensor.h → Etapa1_HallSensor.ino ✓ Verificar → Subir
config.h + ina219_sensor.h → Etapa2_INA219.ino ✓ Verificar → Subir
... (y así para cada etapa)
```

### Receptor (Estación Base)

```
config.h + lora_module.h → Etapa7_LoRaReceiver.ino ✓ Verificar → Subir
```

---

## Configuración de Pines

### ESP32 - Disponibilidad de GPIO

```
USOS RECOMENDADOS:
GPIO 4     → Sensor Hall
GPIO 5     → CS (Chip Select) LoRa
GPIO 12    → MOSI SPI
GPIO 13    → MISO SPI
GPIO 14    → CLK SPI
GPIO 15    → LoRa Reset
GPIO 25    → LoRa IRQ
GPIO 21    → SDA I2C (INA219)
GPIO 22    → SCL I2C (INA219)
GPIO 16    → RX GPS
GPIO 17    → TX GPS
GPIO 2     → DS18B20 OneWire

(La configuración exacta se detallará en cada módulo)
```

---

## Consideraciones de Rendimiento

### Timing

```
SENSOR HALL:          Cada pulso (rápido, ISR)
CÁLCULO VELOCIDAD:    Cada 100 ms
INA219:               Cada 200-500 ms
TEMPERATURA:          Cada 1 segundo
GPS:                  Cada 200 ms (lectura UART asincrónica)
PAQUETE LoRa:         Cada 500 ms
SERIAL DEBUG:         Cada 500 ms
```

### Sin bloqueos

Todas las operaciones deben evitar `delay()` largo. Usar:
- `millis()` para timing no-bloqueante
- ISR para eventos críticos
- Lectura asincrónica de UART para GPS

---

## Validación por Etapa

| Etapa | Componente | Validación |
|---|---|---|
| 1 | Sensor Hall | Contador incrementa, velocidad correcta |
| 2 | INA219 | Voltaje y corriente dentro de rango |
| 3 | DS18B20 | Temperatura lectible |
| 4 | GPS | Lat/Long válidas, posición correcta |
| 5 | LoRa básico | Transmisión y recepción funcionan |
| 6 | Todos los sensores | Datos completos cada 500ms |
| 7 | LoRa + Serial | Datos llegan a PC |
| 8 | Aplicación PC | Visualización en tiempo real |

---

## Variables Globales Clave

```c
// Velocidad
float current_speed_kmh
float filtered_speed_kmh
float current_rpm

// Potencia
float voltage_v
float current_ma
float power_w

// Temperatura
float temperature_c

// GPS
float latitude
float longitude
bool gps_valid
float gps_speed

// LoRa
uint32_t packet_number
bool lora_connected
```

---

## Restricciones del Proyecto

✅ **Debe:**
- Funcionar a < 30 km/h
- Transmitir cada ~500ms (mín)
- Operar sin delay() bloqueante
- Detectar errores de sensores
- Ser modular y testeable por etapas

❌ **NO debe:**
- Perder pulsos del sensor Hall
- Enviar datos corruptos por LoRa
- Bloquear si un sensor falla
- Mostrar información inconsistente

---

## Próximos Pasos

1. ✅ Validar ETAPA 1 (Sensor Hall)
2. ⏳ ETAPA 2 (INA219)
3. ⏳ ETAPA 3 (DS18B20)
4. ⏳ ETAPA 4 (GPS)
5. ⏳ ETAPA 5 (LoRa básico)
6. ⏳ ETAPA 6 (Telemetría completa)
7. ⏳ ETAPA 7 (Receptor)
8. ⏳ ETAPA 8 (Integración con PC)

