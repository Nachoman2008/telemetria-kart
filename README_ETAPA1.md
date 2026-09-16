# ETAPA 1: Sensor Hall A3144 - Medición de Velocidad

## Objetivo
Validar que el ESP32 puede contar correctamente los pulsos del sensor Hall y calcular la velocidad del kart.

---

## Componentes Necesarios

| Componente | Especificación | Cantidad |
|---|---|---|
| ESP32 | DevKit o compatible | 1 |
| Sensor Hall | A3144 o similar | 1 |
| Imán | Pequeño permanente | 1 |
| Cable dupont | M-M o M-H | 3 |
| Fuente | USB o 5V externo | 1 |

---

## Conexiones

```
Sensor Hall A3144:
├── VCC (pin 1)  → 5V ESP32
├── GND (pin 2)  → GND ESP32
└── OUT (pin 3)  → GPIO 4 ESP32 ⚠️ CONFIGURABLE

Imán:
└── Montar en la rueda del kart
```

**Nota:** El sensor Hall tiene 3 pines. Ver datasheet para identificar correctamente VCC, GND y OUT.

---

## Paso a Paso

### 1. Descargar Arduino IDE
- Descargar desde: https://www.arduino.cc/en/software
- Instalar soporte para ESP32: Archivo → Preferencias → Gestor de URLs Adicionales
  - Agregar: `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`

### 2. Crear carpeta del proyecto
```bash
mkdir telemetria_kart
cd telemetria_kart
```

### 3. Copiar archivos
- `config.h` → Carpeta del proyecto
- `hall_sensor.h` → Carpeta del proyecto
- `Etapa1_HallSensor.ino` → Carpeta del proyecto (con el mismo nombre)

Estructura:
```
telemetria_kart/
├── config.h
├── hall_sensor.h
└── Etapa1_HallSensor.ino
```

### 4. Configurar Arduino IDE
- Abrir `Etapa1_HallSensor.ino` en Arduino IDE
- Seleccionar placa: Herramientas → Placa → ESP32 → "ESP32 Dev Module"
- Seleccionar puerto: Herramientas → Puerto → (puerto COM del ESP32)
- Verificar velocidad serial: 115200

### 5. Verificar y cargar
- Click en ✓ (Verificar)
- Si no hay errores, click en → (Subir)

---

## Pruebas

### Test 1: Sistema sin movimiento
**Esperado:**
```
========================================
TELEMETRÍA KART - ETAPA 1: SENSOR HALL
========================================

[HALL] Inicializando sensor Hall...
[HALL] Sensor conectado en GPIO: 4
[HALL] Circunferencia de rueda: 1.55 m
[HALL] Pulsos por revolución: 1
[SETUP] Sistema listo. Esperando pulsos...

[DATOS] Pulsos: 0 | RPM: 0.00 | Velocidad: 0.00 km/h | Int.Pulso: 0 µs
[DATOS] Pulsos: 0 | RPM: 0.00 | Velocidad: 0.00 km/h | Int.Pulso: 0 µs
```

✅ **Pasa si:** El contador de pulsos es 0 y no hay incrementos.

---

### Test 2: Acercar y alejar un imán manualmente
**Esperado:**
```
[DATOS] Pulsos: 1 | RPM: X.XX | Velocidad: X.XX km/h | Int.Pulso: 1000000 µs
[DATOS] Pulsos: 2 | RPM: Y.YY | Velocidad: Y.YY km/h | Int.Pulso: 950000 µs
[DATOS] Pulsos: 3 | RPM: Z.ZZ | Velocidad: Z.ZZ km/h | Int.Pulso: 980000 µs
```

✅ **Pasa si:**
- El contador de pulsos incrementa
- RPM y velocidad aumentan al mover más rápido el imán
- El intervalo de pulso se actualiza

---

### Test 3: Rotación controlada de la rueda
**Esperado:**
- Si rotas la rueda ~1 vuelta/segundo = ~1 pulso/segundo
- Velocidad debería ser: (1 pulso/s × 1,55 m × 3,6) / 1 = **5.58 km/h**

✅ **Pasa si:** La velocidad mostrada es cercana a 5.58 km/h

---

### Test 4: Movimiento rápido del kart
**Esperado:**
- A 30 km/h: pulsos deben aumentar proporcionalmente
- RPM y velocidad deben ser estables (sin grandes saltos)

Cálculo: 
- 30 km/h = (pulsos/s × 1,55 × 3,6) / 1
- pulsos/s = 30 / (1,55 × 3,6) ≈ **5.38 pulsos/segundo**

✅ **Pasa si:** Ves ~5-6 pulsos/segundo a velocidad de 30 km/h

---

## Resolución de Problemas

### Problema: Contador de pulsos no incrementa
**Causa probable:**
1. ❌ Sensor Hall no conectado correctamente
2. ❌ GPIO incorrecto en config.h
3. ❌ Imán muy débil o alejado
4. ❌ Sensor conectado a tierra

**Solución:**
- Verificar conexiones con multímetro (continuidad)
- Cambiar GPIO en `#define HALL_SENSOR_PIN` a otro (ej: 5, 12, 13)
- Probar con un imán más fuerte
- Leer voltaje en pin OUT con multímetro

---

### Problema: Contador incrementa pero velocidad es 0
**Causa probable:**
- Filtro de velocidad muy agresivo
- Tiempo insuficiente entre pulsos

**Solución:**
- Aumentar `SPEED_STABILITY_FILTER` en config.h a 0.9 o 1.0 temporalmente
- Reducir `MIN_SPEED_THRESHOLD` a 0.01

---

### Problema: Velocidad muy inestable (salta mucho)
**Causa probable:**
- Rebotes del sensor (pulsos falsos)
- Filtro de estabilidad débil

**Solución:**
- Aumentar tiempo de debounce en `hallSensorISR()` (línea 35: cambiar 5000 a 10000 µs)
- Reducir `SPEED_STABILITY_FILTER` a 0.5 o 0.3

---

### Problema: "error: undefined reference to `setup()'
**Causa probable:**
- El archivo .ino no está en la carpeta correcta

**Solución:**
- El archivo debe estar en una carpeta con el mismo nombre:
  ```
  Etapa1_HallSensor/
  └── Etapa1_HallSensor.ino
  ```

---

## Validación de la Etapa 1

✅ **La Etapa 1 está completa cuando:**

1. El sensor Hall detecta pulsos correctamente
2. El contador de pulsos incrementa con cada paso del imán
3. RPM se calcula y muestra valores lógicos
4. Velocidad en km/h es calculada correctamente
5. No hay errores de compilación ni en ejecución
6. Los valores son estables (sin saltos excesivos)

---

## Siguientes Pasos

Cuando la ETAPA 1 esté validada, pasaremos a:
- **ETAPA 2:** INA219 - Voltaje y corriente
- **ETAPA 3:** DS18B20 - Temperatura
- **ETAPA 4:** GPS - Posición
- **ETAPA 5:** LoRa - Comunicación inalámbrica

---

## Preguntas Frecuentes

**P: ¿Por qué usar interrupción?**
R: Porque el sensor Hall puede generar pulsos rápidos. Con interrupción no perdemos ninguno, incluso si el loop está ocupado.

**P: ¿Por qué filtro de estabilidad?**
R: Porque con un imán puede haber rebotes. El filtro suaviza las variaciones rápidas manteniendo cambios reales.

**P: ¿Puedo usar múltiples imanes?**
R: Sí. Cambiar `PULSES_PER_REVOLUTION` en config.h. Si hay 2 imanes, poner 2. Las fórmulas se ajustan automáticamente.

**P: ¿Qué sensor Hall alternativo puedo usar?**
R: A3144 (normalmente cerrado), TLE4935 (normalmente abierto), o cualquier sensor digital Hall.

---

## Documentación de referencia

- **ESP32 GPIO:** https://randomnerds.com/esp32-pinout-reference-Which-GPIO-pins-to-use/
- **Arduino attachInterrupt:** https://www.arduino.cc/reference/en/language/functions/external-interrupts/attachinterrupt/
- **Sensor Hall A3144:** https://datasheetspdf.com/pdf-file/940-A3144/

