# ESP32 Dual nRF24L01+ RF Jammer

ESP32-based dual-band RF jammer using two nRF24L01+ modules over HSPI and VSPI.
Features toggle button, status LED, and random channel hopping across the 2.4GHz band.
Custom KiCad PCB design currently in progress.



---

## Hardware necesario

| Componente | Cantidad |
|---|---|
| ESP32 DevKit V1 | 1 |
| nRF24L01+  | 2 |
| Condensador electrolítico 33µF 16V | 2 |
| Condensador cerámico 68nF | 2 |
| Resistencia 10KΩ (pull-up) | 1 |
| Resistencia 220Ω (LED) | 1 |
| LED | 1 |
| Pulsador | 1 |

---

## Pinout

### nRF24L01+ — HSPI
| nRF24 | ESP32 |
|---|---|
| SCK | GPIO14 |
| MISO | GPIO12 |
| MOSI | GPIO13 |
| CSN | GPIO15 |
| CE | GPIO16 |
| VCC | 3.3V |
| GND | GND |

### nRF24L01+ — VSPI
| nRF24 | ESP32 |
|---|---|
| SCK | GPIO18 |
| MISO | GPIO19 |
| MOSI | GPIO23 |
| CSN | GPIO21 |
| CE | GPIO22 |
| VCC | 3.3V |
| GND | GND |

### Otros pines
| Componente | GPIO |
|---|---|
| Botón (pull-up) | GPIO33 |
| LED de estado | GPIO4 |

---

## Librerías necesarias

Instalar desde el gestor de librerías de Arduino IDE:

- [RF24](https://github.com/nRF24/RF24)
- [ezButton](https://arduinogetstarted.com/tutorials/arduino-button-library)


---

## Uso

- **Un pulso** en el botón → activa el jamming (LED encendido)
- **Otro pulso** → desactiva el jamming (LED apagado)

Cuando está activo, los dos módulos nRF24L01+ emiten señal de portadora continua
saltando aleatoriamente entre canales de la banda 2.4GHz (canales 2–79).

---

## PCB

Diseño personalizado en KiCad actualmente **en progreso**.
Los archivos del esquemático y PCB estarán disponibles en este repositorio
cuando el diseño esté terminado.

---

## Créditos y referencias

Este proyecto está basado en el trabajo de
**[smoochiee](https://github.com/smoochiee)** —
tanto para el esquema de conexiones hardware como para las pruebas
iniciales de código:

- [Bluetooth-jammer-esp32](https://github.com/smoochiee/Bluetooth-jammer-esp32)


---

## Licencia

GPL-3.0 — ver [LICENSE](LICENSE)
