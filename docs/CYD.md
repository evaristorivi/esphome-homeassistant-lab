# CYD — Cheap Yellow Display (ESP32-2432S028R)

<p align="center">
  <img src="../images/cydpino.png" alt="Pinout ESP32-2432S028R (CYD)" style="width:80%;">
</p>

<p align="center">
  <img src="../images/cyd.jpeg" alt="Parte trasera del CYD con cables CN1" style="width:60%;">
</p>

Guía para los proyectos basados en el **ESP32-2432S028R** (CYD) — una placa todo-en-uno con pantalla **TFT 2.8" a color** (ST7789, 320×240 px), táctil resistivo (XPT2046) e interfaz gráfica **LVGL**.

> **Con la CYD no necesitas el ESP32-C3 Super Mini ni la pantalla OLED** — la CYD ya incluye su propio ESP32 y display integrado. Solo necesitas los sensores I²C y opcionalmente el hub.

---

## Lista de piezas

- [Pantalla Inteligente ESP32 2.8" — TFT 240×320 táctil (ESP32-2432S028R)](https://es.aliexpress.com/item/1005010399042230.html)
- [Sensor SCD40/SCD41 — CO₂, temperatura y humedad I2C](https://es.aliexpress.com/item/1005009897956849.html)
- [AHT20 + BMP280 — temperatura, humedad y presión](https://es.aliexpress.com/item/1005005321276932.html)
- [Módulo hub I2C (expansión de bus)](https://es.aliexpress.com/item/1005002811407142.html) *(opcional pero recomendado)*

---

## Hardware integrado del CYD

La CYD es una placa completa con ESP32-WROOM-32 y estos periféricos:

| Componente | Pines | Descripción |
|---|---|---|
| Display ST7789 | HSPI: CLK=14, MOSI=13, MISO=12, CS=15, DC=2 | Pantalla TFT 320×240 a color |
| Táctil XPT2046 | VSPI: CLK=25, MOSI=32, MISO=39, CS=33, IRQ=36 | Táctil resistivo |
| Backlight | GPIO21 (LEDC PWM) | Brillo regulable por software |
| LED RGB | R=GPIO4, G=GPIO16, B=GPIO17 (activo-bajo) | Indicador de estado |
| Conector CN1 | GND, GPIO22, GPIO27, 3V3 | **Bus I²C para sensores externos** |

> [Pinout completo del CYD — RandomNerdTutorials](https://randomnerdtutorials.com/esp32-cheap-yellow-display-cyd-pinout-esp32-2432s028r/)

---

## Conexión de sensores (conector CN1)

Los sensores se conectan al conector **CN1** del CYD por **I²C** — el mismo protocolo que en el ESP32-C3, solo cambian los pines:

| | ESP32-C3 | CYD (CN1) |
|---|---|---|
| SDA | GPIO4 | **GPIO22** |
| SCL | GPIO5 | **GPIO27** |
| VCC | 3V3 | 3V3 |
| GND | GND | GND |

Los tres sensores van en paralelo al mismo bus (direcciones distintas: SCD40=`0x62`, AHT20=`0x38`, BMP280=`0x77`).

### Con hub I²C (recomendado)

El hub I²C centraliza las conexiones y queda más limpio:

```
CYD CN1                    Hub I²C
  3.3V ────────────────── VCC
  GPIO22 (SDA) ───────── SDA
  GPIO27 (SCL) ───────── SCL
  GND ─────────────────── GND

Hub I²C puertos:
  Puerto 1 ──── SCD40 (CO₂)
  Puerto 2 ──── AHT20+BMP280 (T, H, P)
```

### Sin hub (daisy-chain)

También se pueden cablear en paralelo directamente:

| Pin sensor | Pin CYD (CN1) |
|---|---|
| **VCC** | **3V3** |
| **GND** | **GND** |
| **SDA** | **GPIO22** |
| **SCL** | **GPIO27** |

> ⚠ GPIO21 está ocupado por el backlight. GPIO35 es input-only. Solo GPIO22 y GPIO27 están disponibles para I²C en el CYD.

---

## Interfaz LVGL

Ambos proyectos CYD usan la misma interfaz gráfica con LVGL:

- **Tema oscuro** (fondo `#0D1117`) con tarjetas centrales sombreadas
- **5 páginas** con barra de acento de color único:
  - 🟢 Verde — CO₂
  - 🟠 Naranja — Temperatura
  - 🔵 Azul — Humedad
  - 🟣 Púrpura — Presión
  - 🟡 Dorado — VWCE
- **Valor grande** centrado en Montserrat 48 px — legible a distancia
- **Indicador de calidad del aire** en la página CO₂ (verde/amarillo/naranja/rojo)
- **Navegación táctil:**
  - Deslizar (swipe) izquierda/derecha → página siguiente/anterior con animación
  - Botones ‹ y › en los laterales (zona amplia de toque)
  - 5 puntos indicadores de página activa
  - Navegación circular (`page_wrap: true`)
- **Icono WiFi** — verde = conectado, rojo = desconectado
- **Auto-dim** del backlight tras 30 s sin tocar — se restaura al tocar

<p align="center">
  <img src="../images/cyd2.jpeg" alt="Página CO₂ — 686 ppm" style="width:40%;">
  <img src="../images/cyd5.jpeg" alt="Página Humedad — 59%" style="width:40%;">
</p>
<p align="center">
  <img src="../images/cyd4.jpeg" alt="Página Presión — 1025 hPa" style="width:40%;">
  <img src="../images/cyd3.jpeg" alt="Página VWCE — 148.16 EUR" style="width:40%;">
</p>

---

## Calibración del táctil

Los valores por defecto (`x_min: 280`, `x_max: 3860`, `y_min: 340`, `y_max: 3860`) funcionan para la mayoría de unidades CYD. Si el toque no coincide con la posición:

1. Activa el log a `DEBUG`
2. Toca las cuatro esquinas y anota los valores raw
3. Actualiza `calibration` en el YAML

Si la pantalla aparece invertida, cambia `mirror_x` / `mirror_y` en las secciones `transform` del display y del touchscreen.

### Diferencias respecto al ESP32-C3

| | ESP32-C3 (Proyectos 1–7) | CYD (Proyectos 8–10) |
|---|---|---|
| Placa | ESP32-C3 Super Mini | ESP32-2432S028R (ESP32-WROOM-32) |
| `esp32.board` | `esp32-c3-devkitm-1` | `esp32dev` |
| Framework | `esp-idf` | `esp-idf` |
| Display | SSD1306 128×64 mono I²C | ST7789 320×240 color SPI (HSPI) |
| Táctil | — | XPT2046 resistivo SPI (VSPI) |
| Gráficos | Lambdas C++ (canvas) | **LVGL** (widgets nativos) |
| I²C sensores | GPIO4 (SDA) / GPIO5 (SCL) | GPIO22 (SDA) / GPIO27 (SCL) — CN1 |
| Navegación | Rotación automática o encoder | Swipe + botones táctiles |
| Backlight | — | PWM en GPIO21 con auto-dim |
| LED RGB integrado | — | GPIO4/R, GPIO16/G, GPIO17/B |

---

## Proyectos

### Proyecto 8: CYD con datos desde Home Assistant (`cyd_dummy.yaml`)

Los sensores físicos están conectados al **ESP32-C3** y éste publica los datos a Home Assistant. La CYD los recibe desde HA y los muestra en pantalla. **Requiere que el ESP32-C3 esté activo y conectado a HA.**

| Dato | Origen |
|---|---|
| CO₂, Temperatura, Humedad, Presión | Desde HA (publicados por ESP32-C3) |
| VWCE | Desde HA (sensor REST) |

```sh
esphome run esphome/cyd_dummy.yaml --device COMx
esphome run esphome/cyd_dummy.yaml --device sensors-cyd-dummy.local
```

---

### Proyecto 9: CYD con sensores I²C directos + VWCE desde HA (`cyd_sensors_vwce_dummy.yaml`)

Los sensores van conectados **físicamente al CYD** por el conector CN1. No necesita el ESP32-C3 para los datos ambientales. VWCE sigue viniendo de HA.

| Dato | Origen |
|---|---|
| CO₂, Temperatura, Humedad, Presión | **Sensores I²C en CN1** (directos) |
| VWCE | Desde HA (sensor REST) |

```sh
esphome run esphome/cyd_sensors_vwce_dummy.yaml --device COMx
esphome run esphome/cyd_sensors_vwce_dummy.yaml --device sensors-cyd-vwce-dummy.local
```

---

### Proyecto 10: CYD con sensores I²C directos + VWCE desde Yahoo Finance (`cyd_sensors_vwce.yaml`)

Igual que el Proyecto 9 pero **el ESP32 consulta Yahoo Finance directamente** — no necesita HA para el precio de VWCE.

| Dato | Origen |
|---|---|
| CO₂, Temperatura, Humedad, Presión | **Sensores I²C en CN1** (directos) |
| VWCE | HTTP directo a Yahoo Finance (cada 60 s) |

```sh
esphome run esphome/cyd_sensors_vwce.yaml --device COMx
esphome run esphome/cyd_sensors_vwce.yaml --device sensors-cyd-vwce.local
```

---

### Diferencias entre Proyectos 8, 9 y 10

| | Proyecto 8 (`cyd_dummy`) | Proyecto 9 (`cyd_sensors_vwce_dummy`) | Proyecto 10 (`cyd_sensors_vwce`) |
|---|---|---|---|
| Sensores ambientales | Desde Home Assistant | Físicos I²C en CN1 | Físicos I²C en CN1 |
| Bus I²C | No configurado | GPIO22 (SDA) / GPIO27 (SCL) | GPIO22 (SDA) / GPIO27 (SCL) |
| VWCE | Desde HA | Desde HA | HTTP directo Yahoo Finance |
| Necesita ESP32-C3 activo | Sí | No | No |
| Necesita Home Assistant | Sí | Solo para VWCE | No |
| Interfaz LVGL | Idéntica | Idéntica | Idéntica |
