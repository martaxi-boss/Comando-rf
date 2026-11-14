# Esquema de montagem — ESP32-C5 + ILI9341/XPT2046 + CC1101

> Fonte técnica de pinos: [Bruce `boards/ESP32-C5-tft/connections.md`](../boards/ESP32-C5-tft/connections.md), [`pins_arduino.h` do upstream](https://github.com/BruceDevices/firmware/blob/1.17/boards/ESP32-C5-tft/pins_arduino.h).
> **Esquema lógico, não layout físico de uma placa específica.** Confirmar a serigrafia e pinout da ESP32-C5 DevKitC-1 recebida antes de ligar cabos.

## Diagrama funcional

```mermaid
flowchart LR
  C5["ESP32-C5 DevKitC-1<br/>3,3 V lógico"]
  TFT["TFT ILI9341<br/>2,8 pol. / SPI"]
  TOUCH["Touch XPT2046<br/>SPI"]
  RF["CC1101 433 MHz<br/>3,3 V"]
  SD["microSD SPI<br/>(opcional)"]
  ANT["Antena 433 MHz"]
  C5 -->|"SPI partilhado<br/>SCK 6 / MOSI 7 / MISO 2"| TFT
  C5 -->|"SPI partilhado<br/>SCK 6 / MOSI 7 / MISO 2"| TOUCH
  C5 -->|"SPI partilhado<br/>SCK 6 / MOSI 7 / MISO 2"| RF
  C5 -->|"SPI partilhado<br/>SCK 6 / MOSI 7 / MISO 2"| SD
  C5 -->|"CS 23 / DC 24 / BL 25"| TFT
  C5 -->|"CS 1"| TOUCH
  C5 -->|"CS 9 / GDO0 8"| RF
  C5 -->|"CS 10"| SD
  RF --- ANT
```

## Ligações elétricas

| Sinal da placa ESP32-C5 | Ligar a TFT ILI9341 | Ligar a Touch XPT2046 | Ligar a CC1101 | microSD opcional |
| --- | --- | --- | --- | --- |
| **GPIO6 — SPI SCK** | SCK / CLK | T_CLK | SCK / SCLK | SCK |
| **GPIO7 — SPI MOSI** | SDI / MOSI | T_DIN | MOSI / SI | MOSI |
| **GPIO2 — SPI MISO** | SDO / MISO (se exposto) | T_DO | MISO / SO | MISO |
| **GPIO23** | TFT_CS | — | — | — |
| **GPIO24** | TFT_DC / RS | — | — | — |
| **GPIO25** | TFT_BL / LED, confirmar circuito do módulo | — | — | — |
| **GPIO1** | — | T_CS | — | — |
| **GPIO9** | — | — | CS / CSN | — |
| **GPIO8** | — | — | GDO0 | — |
| **GPIO10** | — | — | — | CS |
| **3V3 e GND** | Alimentação compatível e GND | 3,3 V e GND se módulo independente | **3,3 V e GND** | alimentação compatível e GND |

**TFT_RESET:** a documentação Bruce indica `C5 RST` para reset do display; no firmware `TFT_RST=-1`. A ligação física depende do módulo escolhido e da topologia do circuito de reset. Não associar um GPIO arbitrário.

**T_IRQ:** não usado pelo pinout Bruce apresentado (`TOUCH_INT=-1`), deixar desligado nesta montagem. **CC1101 GDO2:** não exigido para este pinout. **GPIO0 e GPIO28:** não ligar botões de 3 vias na configuração tátil, salvo projeto explícito e firmware revisto.

## Ordem segura de ligação e teste

1. Desligar USB e qualquer fonte de alimentação.
2. Confirmar **polaridade**, massa comum e compatibilidade dos níveis lógicos. O CC1101 é **3,3 V**, nunca 5 V.
3. Ligar apenas o TFT; testar menus, backlight, toque e orientação.
4. Ligar CC1101 e a **antena da banda correta**; validar deteção/receção sem emitir.
5. Se necessário, acrescentar microSD (CS10) evitando colisões de chip-select.
6. Fixar cabos com isolamento e caixa quando a montagem estiver eletricamente validada.
7. Não fazer transmissões para portões ou outros equipamentos que não sejam próprios ou autorizados.

## Atenções à compra

- Procurar **ILI9341 com XPT2046** efetivo e pinos SPI acessíveis. «2,8 TFT touch» sem referência não basta.
- Procurar CC1101 com módulo de **433 MHz** e antena da mesma banda; verificar a frequência real da central autorizada antes de comprar.
- Evitar comprar o kit **NM-CYD-C5** como substituto direto desta montagem: a placa tem pinout e firmware-alvo diferentes do `esp32-c5-tft` escolhido aqui.
- A montagem em breadboard usa fios curtos e adequados. Não alimentar o rádio pelo pino 5V.

**Estado:** desenho lógico derivado dos pinos oficiais. **Não foi inspecionada nem testada uma montagem física.**
