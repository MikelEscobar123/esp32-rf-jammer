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
El esquemático está completo y el layout sin enrutar. Los archivos están disponibles por si alguien quiere modificar o continuar el diseño.
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


---

## Código

```
#include "RF24.h"
#include <SPI.h>
#include <ezButton.h>
#include "esp_bt.h"
#include "esp_wifi.h"


SPIClass *sp = nullptr;
SPIClass *hp = nullptr;

RF24 radio(16, 15, 16000000);   //HSPI CAN SET SPI SPEED TO 16000000 BY DEFAULT ITS 10000000
RF24 radio1(22, 21, 16000000);  //VSPI CAN SET SPI SPEED TO 16000000 BY DEFAULT ITS 10000000


//HSPI=SCK = 14, MISO = 12, MOSI = 13, CS = 15 , CE = 16
//VSPI=SCK = 18, MISO =19, MOSI = 23 ,CS =21 ,CE = 22

unsigned int flag = 0;   //HSPI// Flag variable to keep track of direction
unsigned int flagv = 0;  //VSPI// Flag variable to keep track of direction
int ch = 45;    // Variable to store value of ch
int ch1 = 45;   // Variable to store value of ch

ezButton toggleSwitch(33);




void two() {
  if (flagv == 0) {  // If flag is 0, increment ch by 4 and ch1 by 1

    ch1 += 4;
  } else {  // If flag is not 0, decrement ch by 4 and ch1 by 1

    ch1 -= 4;
  }

  if (flag == 0) {  // If flag is 0, increment ch by 4 and ch1 by 1
    ch += 2;

  } else {  // If flag is not 0, decrement ch by 4 and ch1 by 1
    ch -= 2;
  }

  // Check if ch1 is greater than 79 and flag is 0
  if ((ch1 > 79) && (flagv == 0)) {
    flagv = 1;                             // Set flag to 1 to change direction
  } else if ((ch1 < 2) && (flagv == 1)) {  // Check if ch1 is less than 2 and flag is 1
    flagv = 0;                             // Set flag to 0 to change direction
  }

  // Check if ch is greater than 79 and flag is 0
  if ((ch > 79) && (flag == 0)) {
    flag = 1;                            // Set flag to 1 to change direction
  } else if ((ch < 2) && (flag == 1)) {  // Check if ch is less than 2 and flag is 1
    flag = 0;                            // Set flag to 0 to change direction
  }
  radio.setChannel(ch);
  radio1.setChannel(ch1);
  /*Serial.print("SP:");
  Serial.println(ch1);
  Serial.print("\tHP:");
  Serial.println(ch);*/
}


void one() {
  ////RANDOM CHANNEL
  radio1.setChannel(random(80));
  radio.setChannel(random(80));
  delayMicroseconds(random(60));//////REMOVE IF SLOW


 /*  YOU CAN DO -----SWEEP CHANNEL
  for (int i = 0; i < 79; i++) {
    radio.setChannel(i);
*/

}



void setup() {

  Serial.begin(115200);
  esp_bt_controller_deinit();
  esp_wifi_stop();
  esp_wifi_deinit();
  esp_wifi_disconnect();
  toggleSwitch.setDebounceTime(50);

  initHP();
  initSP();
}

void initSP() {
  sp = new SPIClass(VSPI);
  sp->begin();
  if (radio1.begin(sp)) {
    Serial.println("SP Started !!!");
    radio1.setAutoAck(false);
    radio1.stopListening();
    radio1.setRetries(0, 0);
    radio1.setPALevel(RF24_PA_MAX, true);
    radio1.setDataRate(RF24_2MBPS);
    radio1.setCRCLength(RF24_CRC_DISABLED);
    radio1.printPrettyDetails();
    radio1.startConstCarrier(RF24_PA_MAX, ch1);
  } else {
    Serial.println("SP couldn't start !!!");
  }
}
void initHP() {
  hp = new SPIClass(HSPI);
  hp->begin();
  if (radio.begin(hp)) {
    Serial.println("HP Started !!!");
    radio.setAutoAck(false);
    radio.stopListening();
    radio.setRetries(0, 0);
    radio.setPALevel(RF24_PA_MAX, true);
    radio.setDataRate(RF24_2MBPS);
    radio.setCRCLength(RF24_CRC_DISABLED);
    radio.printPrettyDetails();
    radio.startConstCarrier(RF24_PA_MAX, ch);
  } else {
    Serial.println("HP couldn't start !!!");
  }
}

void loop() {


  toggleSwitch.loop();  // MUST call the loop() function first

  if (toggleSwitch.isPressed())
    Serial.println("one");
  if (toggleSwitch.isReleased())
    Serial.println("two");

  int state = toggleSwitch.getState();


  if (state == HIGH)
    two();

  else {
    one();
  }
}
```

## Fotos

<img width="234" height="473" alt="esp32-jammer-fotomontaje" src="https://github.com/user-attachments/assets/fcd12bf5-8540-49e1-aa1f-e1d991bc82d4" />
<img width="671" height="396" alt="image" src="https://github.com/user-attachments/assets/a82a6f68-9faa-4b78-a9a4-034ee24db784" />

