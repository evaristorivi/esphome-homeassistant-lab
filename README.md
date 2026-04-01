# esphome-homeassistant-lab

<p align="center">
  <img src="images/portada.png" alt="Portada del proyecto" style="width:90%;">
</p>

Ejemplos prácticos para integrar un **ESP32-C3** con **ESPHome** y **Home Assistant**, mostrando cómo obtener y visualizar datos ambientales (temperatura, humedad, presión, CO₂) y financieros (precio del ETF VWCE en XETRA) en una pantalla OLED.

---

## Lista de piezas

A continuación se muestra una lista de módulos y componentes compatibles, con enlaces de ejemplo para su compra:

- [Placa de desarrollo ESP32-C3 Super Mini, 16 pines tipo C](https://es.aliexpress.com/item/1005009014269977.html)
- [Pantalla OLED 0,96" I2C 128x64 SSD1306](https://es.aliexpress.com/item/1005006365881525.html)
- [Sensor SCD40/SCD41 CO₂, temperatura y humedad I2C](https://es.aliexpress.com/item/1005009897956849.html)
- [AHT20 + BMP280 — temperatura, humedad y presión del aire](https://es.aliexpress.com/item/1005005321276932.html)
- [Módulo hub I2C (expansión de bus)](https://es.aliexpress.com/item/1005002811407142.html)
- [Placa de pruebas sin soldadura, 400 puntos](https://es.aliexpress.com/item/1005010426826947.html)
- [TP4056 — módulo cargador de batería LiPo con protección](https://es.aliexpress.com/item/1005009887303870.html)

> **Nota sobre la batería:** El módulo TP4056 aparece en la lista porque inicialmente se planteó alimentar el proyecto con una batería LiPo de 3,7V (tipo dron). Funciona, pero no es la solución óptima: una LiPo recién cargada entrega ~4,2V (aceptable), pero conforme se descarga el voltaje baja progresivamente. Al alimentar el ESP32 directamente por el pin 3V3, a partir de cierto nivel de descarga la tensión es insuficiente para un funcionamiento fiable. La solución correcta sería interponer un *boost converter* que mantenga una salida estable de 3,3V independientemente del nivel de la batería. Finalmente se optó por simplificar y construir el proyecto **sin batería**, alimentado directamente por USB. La lista se mantiene por si alguien quiere explorar esa vía.

> Puedes usar módulos equivalentes compatibles con ESP32 y ESPHome.

---

## Esquema de conexión

Todos los módulos se comunican mediante el bus **I2C**, que solo requiere 4 cables:

<p align="center">
  <img src="images/esquema.webp" alt="Pinout ESP32-C3 Super Mini" style="width:80%;">
</p>

| Pin módulo | Pin ESP32-C3 Super Mini |
|-----------|------------------------|
| **VCC** | **3V3** |
| **GND** | **GND** |
| **SDA** | **GPIO4** |
| **SCL** | **GPIO5** |

> Si usas el hub I2C, conecta **un único par SDA/SCL** desde el ESP32 al hub y desde ahí reparte al resto de módulos. VCC y GND se pueden llevar en paralelo directamente desde el ESP32.

> Los pines GPIO4 (SDA) y GPIO5 (SCL) están definidos en `esphome/vwce.yaml` y `esphome/vwce_dummy.yaml`. Si cambias los pines físicos, actualízalos también en el YAML.

---

## Estructura del proyecto

```
esphome/
  vwce.yaml             # ESP32 obtiene el precio de VWCE directamente de Yahoo Finance
  vwce_dummy.yaml       # ESP32 recibe el precio desde un sensor REST de Home Assistant
  secrets.yaml.example  # Plantilla de credenciales (no subas tus claves reales)
homeassistant/
  configuration.yaml    # Configuración principal mínima de Home Assistant
  vwce_sensor.yaml      # Sensor REST de VWCE listo para incluir
images/
  portada.png           # Imagen de portada
  esquema.webp          # Pinout del ESP32-C3 Super Mini
  1.png                 # Ejemplo de uso en pantalla
```

### ¿Cuál YAML usar?

| | `vwce.yaml` | `vwce_dummy.yaml` |
|---|---|---|
| **Quién obtiene el precio** | El propio ESP32 (petición HTTP a Yahoo) | Home Assistant (sensor REST) |
| **Requiere Home Assistant activo** | No (funciona autónomo) | Sí |
| **Ideal cuando...** | Quieres un dispositivo independiente | Ya tienes HA y quieres compartir el dato con varios ESP32 |

Cada archivo contiene comentarios explicativos sobre su propósito y uso.

---

## Requisitos

### ESPHome

- **Python 3.8+** y **pip** instalados en tu sistema.
- Instala ESPHome con:
  ```sh
  pip install esphome
  ```
- Documentación oficial: [https://esphome.io/guides/installing_esphome.html](https://esphome.io/guides/installing_esphome.html)

### Home Assistant

- Puede instalarse en Raspberry Pi, PC, NAS, máquina virtual, etc.
- Documentación oficial: [https://www.home-assistant.io/installation/](https://www.home-assistant.io/installation/)

#### Ejemplo de instalación en Raspberry Pi 4 con Docker

```sh
docker pull ghcr.io/home-assistant/raspberrypi4-homeassistant:stable

docker run -d \
  --name homeassistant \
  --restart=unless-stopped \
  -v /root/homeassistant:/config \
  -e TZ=Europe/Madrid \
  --network=host \
  ghcr.io/home-assistant/raspberrypi4-homeassistant:stable
```

> Cambia `/root/homeassistant` por la ruta donde quieras guardar la configuración.

---

## Puesta en marcha

### 1. Configura las credenciales

Copia `esphome/secrets.yaml.example` a `esphome/secrets.yaml` y rellena tus datos:

```yaml
wifi_ssid: "TuRedWiFi"
wifi_password: "TuContraseña"
ota_password: "una_clave_ota"
api_encryption_key: "clave_base64_de_32_bytes"
```

### 2. (Opcional) Configura Home Assistant

Si usas `vwce_dummy.yaml`, incluye el sensor REST en tu `configuration.yaml`:

```yaml
sensor: !include homeassistant/vwce_sensor.yaml
```

### 3. Compila y flashea el ESP32

**Primera vez — por USB:**

Pon el ESP32 en modo flasheo (mantén pulsado BOOT al conectar el USB, luego suelta RESET) y ejecuta:

```sh
esphome run esphome/vwce.yaml --device COMx
```

Reemplaza `COMx` por el puerto serie de tu ESP32 (por ejemplo `COM3` en Windows o `/dev/ttyUSB0` en Linux).

**Siguientes veces — por OTA/WiFi:**

Una vez flasheado y conectado a la red, las actualizaciones se hacen por WiFi:

```sh
esphome run esphome/vwce.yaml --device vwce-ticker.local
```

Reemplaza `vwce-ticker.local` por el nombre mDNS o la IP de tu ESP32 (por ejemplo `192.168.1.106`).

> La primera vez siempre debe ser por USB. A partir de ahí OTA permite actualizar sin tocar el cable.

---

<p align="center">
  <img src="images/1.png" alt="Ejemplo de uso en pantalla" style="width:50%;">
</p>

---

**¡Contribuciones y mejoras son bienvenidas!**
