# esphome-homeassistant-lab

Ejemplos de integración de sensores ambientales (temperatura, humedad, presión, CO₂, etc.) y financieros (precio de ETF VWCE XETRA) con ESPHome y Home Assistant usando ESP32.

## Requisitos

### ESPHome

- **Python 3.8+** instalado en tu sistema.
- **pip** (gestor de paquetes de Python).
- Instala ESPHome con:
  ```sh
  pip install esphome
  ```
- Más información y documentación oficial:  
  [https://esphome.io/guides/installing_esphome.html](https://esphome.io/guides/installing_esphome.html)

### Home Assistant

- Puede instalarse en Raspberry Pi, PC, NAS, etc.
- Documentación oficial:  
  [https://www.home-assistant.io/installation/](https://www.home-assistant.io/installation/)

#### Ejemplo de instalación en Raspberry Pi 3 usando Docker

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

## Estructura del proyecto

```
esphome/
  vwce.yaml           # Ejemplo completo de ESPHome con HTTP request
  vwce_dummy.yaml     # Ejemplo ESPHome usando integración Home Assistant
  secrets.yaml.example # Plantilla de secretos (sin datos reales)
homeassistant/
  configuration.yaml  # Configuración principal de Home Assistant
  vwce_sensor.yaml    # Sensor REST de VWCE para incluir
```

---

Cada archivo contiene comentarios explicativos sobre su propósito y uso.

---
# Ejemplos de Configuración ESPHome y Home Assistant

Este repositorio contiene ejemplos prácticos para integrar un ESP32 con ESPHome y Home Assistant, mostrando cómo obtener y visualizar el precio de VWCE desde Yahoo Finance.

## Estructura

- `esphome/`
  - `vwce.yaml`: Ejemplo completo de ESPHome usando petición HTTP directa a Yahoo.
  - `vwce_dummy.yaml`: Ejemplo de ESPHome que consume el sensor de Home Assistant.
  - `secrets.yaml.example`: Plantilla para tus credenciales (no subas tus claves reales).

- `homeassistant/`
  - `configuration.yaml`: Ejemplo de configuración principal mínima.
  - `vwce_sensor.yaml`: Sensor REST de VWCE listo para incluir en Home Assistant.

## Uso
1. Copia `secrets.yaml.example` a `secrets.yaml` y pon tus claves.
2. Incluye `vwce_sensor.yaml` en tu `configuration.yaml` de Home Assistant:
   ```yaml
   sensor: !include homeassistant/vwce_sensor.yaml
   ```
3. Usa los YAML de `esphome/` para flashear tu ESP32 según el método que prefieras.

---

**¡Contribuciones y mejoras son bienvenidas!**
