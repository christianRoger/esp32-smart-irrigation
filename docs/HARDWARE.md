# Documentazione Hardware — ESP32-S3 Smart Irrigation

## 1. Panoramica Hardware

Il sistema **ESP32-S3 Smart Irrigation** è stato progettato come piattaforma embedded per il controllo automatizzato di un impianto di irrigazione.

L'hardware integra un microcontrollore ESP32-S3, sensori ambientali e del terreno, attuatori, interfaccia grafica touch, RTC e interfacce di comunicazione.

L'ESP32-S3 rappresenta l'unità centrale di controllo e coordina l'acquisizione dei dati, l'elaborazione delle informazioni e il comando degli attuatori.

---

# 2. Architettura Hardware

```text
                         ┌─────────────────────┐
                         │      ESP32-S3        │
                         │   MCU Dual-Core      │
                         └──────────┬──────────┘
                                    │
          ┌─────────────────────────┼─────────────────────────┐
          │                         │                         │
          ▼                         ▼                         ▼
   ┌─────────────┐           ┌─────────────┐           ┌─────────────┐
   │   SENSORI   │           │  ATTUATORI  │           │     UI      │
   ├─────────────┤           ├─────────────┤           ├─────────────┤
   │ Soil ×2     │           │ Pompa 1      │           │ ST7789      │
   │ DHT11       │           │ Pompa 2      │           │ XPT2046     │
   │ HC-SR04     │           │ Pozzo        │           │ Touch       │
   │ DS3231      │           │ Buzzer       │           │             │
   └─────────────┘           │ LED ×2       │           └─────────────┘
                             └─────────────┘

                         ┌─────────────────────┐
                         │   COMUNICAZIONE     │
                         ├─────────────────────┤
                         │ Wi-Fi               │
                         │ ESP-NOW             │
                         │ Web Server          │
                         │ Telegram            │
                         └─────────────────────┘
```

---

# 3. Microcontrollore

## ESP32-S3

Il controller principale del sistema è un **ESP32-S3 Dual-Core**.

Il microcontrollore gestisce:

* acquisizione dei sensori;
* controllo degli attuatori;
* interfaccia TFT;
* touch screen;
* comunicazione Wi-Fi;
* ESP-NOW;
* Web Server;
* Telegram;
* gestione dell'orario;
* logica di irrigazione;
* modalità SMART;
* memoria persistente;
* watchdog;
* aggiornamento OTA.

L'ESP32-S3 è utilizzato come nodo centrale dell'intero sistema.

---

# 4. Mappa GPIO

La seguente tabella riporta la configurazione hardware utilizzata dal firmware.

|    GPIO | Segnale          | Funzione                        |
| ------: | ---------------- | ------------------------------- |
|  GPIO 1 | `SOIL_PIN1`      | Sensore umidità terreno 1 — ADC |
|  GPIO 2 | `SOIL_PIN2`      | Sensore umidità terreno 2 — ADC |
|  GPIO 4 | `POMPA1`         | Relè pompa 1                    |
|  GPIO 5 | `POMPA2`         | Relè pompa 2                    |
|  GPIO 6 | `RELE_POZZO`     | Relè pozzo                      |
|  GPIO 7 | `BUZZER_PIN`     | Buzzer                          |
|  GPIO 8 | `I2C_SDA`        | SDA — DS3231                    |
|  GPIO 9 | `I2C_SCL`        | SCL — DS3231                    |
| GPIO 10 | `TFT_CS`         | Chip Select ST7789              |
| GPIO 11 | `TFT_MOSI`       | SPI MOSI                        |
| GPIO 12 | `TFT_SCK`        | SPI Clock                       |
| GPIO 13 | `TFT_MISO`       | SPI MISO                        |
| GPIO 14 | `TFT_DC`         | Data / Command ST7789           |
| GPIO 15 | `DHTPIN`         | DHT11                           |
| GPIO 16 | `TRIG_PIN`       | Trigger HC-SR04                 |
| GPIO 17 | `ECHO_PIN`       | Echo HC-SR04                    |
| GPIO 18 | `RESET_WIFI_PIN` | Reset configurazione Wi-Fi      |
| GPIO 20 | `TFT_LED`        | Backlight display               |
| GPIO 38 | `T_CS`           | Chip Select XPT2046             |
| GPIO 39 | `T_IRQ`          | Interrupt touch                 |
| GPIO 40 | `TFT_RST`        | Reset ST7789                    |
| GPIO 47 | `LED1`           | LED di stato 1                  |
| GPIO 48 | `LED2`           | LED di stato 2                  |

---

# 5. Display TFT

## ST7789 — 320×240

Il sistema utilizza un display TFT basato sul controller **ST7789** con risoluzione:

```text
320 × 240 pixel
```

Il display viene utilizzato per:

* visualizzazione dello stato del sistema;
* visualizzazione dei sensori;
* configurazione;
* gestione dell'irrigazione;
* visualizzazione degli allarmi;
* informazioni meteorologiche;
* modalità SMART;
* configurazione della rete.

### Interfaccia

La comunicazione con il display utilizza l'interfaccia **SPI**.

| Segnale   |    GPIO |
| --------- | ------: |
| CS        | GPIO 10 |
| MOSI      | GPIO 11 |
| SCK       | GPIO 12 |
| MISO      | GPIO 13 |
| DC        | GPIO 14 |
| RST       | GPIO 40 |
| Backlight | GPIO 20 |

---

# 6. Touch Screen

## XPT2046

Il display utilizza un controller touch resistivo **XPT2046**.

Il touch screen permette:

* pressione dei pulsanti;
* navigazione tra le schermate;
* gesture swipe;
* configurazione del sistema;
* inserimento di dati tramite tastiera virtuale.

### Collegamenti

| Segnale |    GPIO |
| ------- | ------: |
| CS      | GPIO 38 |
| IRQ     | GPIO 39 |

Il controller touch utilizza inoltre il bus SPI condiviso con il display.

---

# 7. Sensori di umidità del terreno

Il sistema dispone di due ingressi analogici dedicati ai sensori di umidità del terreno.

```text
SOIL_PIN1 → GPIO 1
SOIL_PIN2 → GPIO 2
```

I valori vengono acquisiti tramite ADC dell'ESP32-S3.

La lettura può essere calibrata utilizzando valori di riferimento relativi a:

```text
Terreno secco
       ↓
Valore ADC
       ↓
Elaborazione
       ↓
Percentuale di umidità
       ↓
Logica irrigazione
```

La calibrazione viene memorizzata nella memoria non volatile.

I valori dei sensori vengono utilizzati dalla modalità automatica e dalla logica SMART.

---

# 8. Sensore DHT11

Il **DHT11** viene utilizzato per acquisire:

* temperatura ambientale;
* umidità relativa.

Collegamento:

```text
DHT11 DATA → GPIO 15
```

I dati possono essere utilizzati per:

* visualizzazione sul display;
* monitoraggio ambientale;
* log;
* logica di controllo.

---

# 9. Sensore ultrasonico HC-SR04

Il sensore **HC-SR04** viene utilizzato per il monitoraggio del livello dell'acqua nel serbatoio.

### Collegamenti

| Segnale |    GPIO |
| ------- | ------: |
| TRIG    | GPIO 16 |
| ECHO    | GPIO 17 |

Il principio di funzionamento è basato sulla misura del tempo di ritorno dell'onda ultrasonica.

```text
HC-SR04
   │
   ├── TRIG
   │
   └── ECHO
        │
        ▼
    ESP32-S3
        │
        ▼
Calcolo distanza
        │
        ▼
Livello serbatoio
```

### Nota elettrica

Il segnale `ECHO` dell'HC-SR04 può essere superiore al livello logico massimo ammesso dall'ESP32-S3.

Quando il modulo utilizzato fornisce un'uscita a 5 V, è necessario utilizzare un opportuno **partitore di tensione o level shifter** prima di collegare il segnale al GPIO dell'ESP32-S3.

---

# 10. RTC DS3231

Il modulo **DS3231** viene utilizzato come riferimento temporale locale.

La comunicazione avviene tramite **I²C**.

| Segnale |   GPIO |
| ------- | -----: |
| SDA     | GPIO 8 |
| SCL     | GPIO 9 |

Il DS3231 permette al sistema di mantenere l'orario anche in assenza temporanea della connessione Internet.

Quando il Wi-Fi è disponibile, l'orario può essere inoltre sincronizzato tramite **NTP**.

```text
DS3231
  │
  ▼
ESP32-S3
  │
  ├── Programmazione irrigazione
  ├── Timestamp
  └── Gestione orario
        ▲
        │
       NTP
```

---

# 11. Relè e pompe

Il sistema controlla tre uscite dedicate:

```text
POMPA1
POMPA2
RELE_POZZO
```

### GPIO

| Uscita  |   GPIO |
| ------- | -----: |
| Pompa 1 | GPIO 4 |
| Pompa 2 | GPIO 5 |
| Pozzo   | GPIO 6 |

La logica utilizzata dai relè è:

```text
RELAY_ON  = LOW
RELAY_OFF = HIGH
```

Il firmware verifica le condizioni operative prima di attivare le pompe.

Tra le condizioni considerate possono rientrare:

* modalità manuale;
* programmazione automatica;
* modalità SMART;
* umidità del terreno;
* livello del serbatoio;
* sospensione dell'irrigazione;
* condizioni meteorologiche;
* condizioni di sicurezza.

---

# 12. Buzzer

Il buzzer è collegato a:

```text
GPIO 7
```

Viene utilizzato per fornire indicazioni acustiche relative allo stato del sistema e agli eventi.

Possibili utilizzi:

* conferma di un comando;
* allarmi;
* errori;
* eventi del sistema.

---

# 13. LED di stato

Il sistema utilizza due LED di stato.

| LED  |    GPIO |
| ---- | ------: |
| LED1 | GPIO 47 |
| LED2 | GPIO 48 |

I LED possono essere utilizzati per indicare condizioni operative del dispositivo.

Esempi:

```text
Sistema avviato
Wi-Fi connesso
Irrigazione attiva
Allarme
Errore
```

La gestione effettiva dell'indicazione dipende dalla logica implementata nel firmware.

---

# 14. Pulsante Reset Wi-Fi

È presente un pulsante dedicato al reset della configurazione Wi-Fi.

```text
RESET_WIFI_PIN → GPIO 18
```

Il pulsante permette di avviare la procedura di ripristino delle credenziali Wi-Fi e consentire una nuova configurazione della rete.

---

# 15. Comunicazione SPI

Il bus SPI viene utilizzato principalmente per le periferiche grafiche.

```text
ESP32-S3
   │
   └── SPI
        │
        ├── ST7789
        │
        └── XPT2046
```

Il display e il controller touch condividono il bus SPI, utilizzando linee di Chip Select separate.

---

# 16. Comunicazione I²C

Il bus I²C viene utilizzato dal RTC DS3231.

```text
ESP32-S3
   │
   └── I²C
        │
        └── DS3231
```

Configurazione:

```text
SDA → GPIO 8
SCL → GPIO 9
```

L'architettura può essere estesa in futuro con ulteriori periferiche I²C compatibili, mantenendo le stesse linee di bus e utilizzando indirizzi differenti.

---

# 17. Acquisizione analogica

I sensori di umidità del terreno utilizzano ingressi ADC dell'ESP32-S3.

```text
SOIL SENSOR 1 → GPIO 1 → ADC
SOIL SENSOR 2 → GPIO 2 → ADC
```

Il firmware converte i valori ADC in informazioni utilizzabili dalla logica di irrigazione.

Il processo concettuale è:

```text
Segnale analogico
       ↓
ADC ESP32-S3
       ↓
Valore grezzo
       ↓
Calibrazione
       ↓
Percentuale umidità
       ↓
Decisione irrigazione
```

---

# 18. Alimentazione e separazione dei carichi

Il sistema deve prevedere una corretta separazione tra:

* alimentazione della logica;
* alimentazione dei relè;
* alimentazione delle pompe;
* eventuali carichi ad alta corrente.

L'ESP32-S3 e i circuiti di segnale devono essere protetti dai disturbi generati dai carichi induttivi.

Per l'attivazione delle pompe è consigliabile utilizzare moduli relè o driver adeguatamente dimensionati per tensione, corrente e tipologia del carico.

### Principio generale

```text
             ALIMENTAZIONE
                  │
        ┌─────────┴─────────┐
        │                   │
        ▼                   ▼
   ESP32-S3             POTENZA
   + Sensori            + Pompe
        │                   │
        └────── Relè ───────┘
```

La sezione di potenza deve essere progettata separatamente dalla logica a bassa tensione.

---

# 19. Protezione elettrica

Durante la realizzazione dell'hardware devono essere considerate:

* protezione da sovratensione;
* corretta gestione della massa;
* separazione dei carichi di potenza;
* protezione dei GPIO;
* dimensionamento dei cavi;
* dimensionamento dei relè;
* protezione contro disturbi generati dalle pompe;
* isolamento elettrico quando necessario.

Per carichi a tensione di rete devono essere utilizzati componenti, contenitori e procedure conformi alle normative applicabili.

---

# 20. Schema funzionale dell'hardware

```text
                         ESP32-S3
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
     SENSORI            INTERFACCIA         ATTUATORI
        │                   │                   │
 ┌──────┼──────┐       ┌────┴────┐       ┌────┼────┐
 │      │      │       │         │       │    │    │
Soil   DHT   HC-SR04  ST7789   XPT2046  P1   P2  Pozzo
 │      │      │       │         │
 └──────┴──────┴───────┴─────────┘
                │
                ▼
             ESP32-S3
                │
        ┌───────┼────────┐
        │       │        │
       Wi-Fi  ESP-NOW   I²C
        │                │
        ▼                ▼
   Web/Cloud           DS3231
```

---

# 21. Tabella riassuntiva

| Componente       | Interfaccia | GPIO / Connessione |
| ---------------- | ----------- | ------------------ |
| ESP32-S3         | MCU         | —                  |
| Soil Sensor 1    | ADC         | GPIO 1             |
| Soil Sensor 2    | ADC         | GPIO 2             |
| Pompa 1          | GPIO / Relè | GPIO 4             |
| Pompa 2          | GPIO / Relè | GPIO 5             |
| Pozzo            | GPIO / Relè | GPIO 6             |
| Buzzer           | GPIO        | GPIO 7             |
| DS3231 SDA       | I²C         | GPIO 8             |
| DS3231 SCL       | I²C         | GPIO 9             |
| ST7789 CS        | SPI         | GPIO 10            |
| ST7789 MOSI      | SPI         | GPIO 11            |
| ST7789 SCK       | SPI         | GPIO 12            |
| ST7789 MISO      | SPI         | GPIO 13            |
| ST7789 DC        | GPIO        | GPIO 14            |
| DHT11            | GPIO        | GPIO 15            |
| HC-SR04 TRIG     | GPIO        | GPIO 16            |
| HC-SR04 ECHO     | GPIO        | GPIO 17            |
| Wi-Fi Reset      | GPIO        | GPIO 18            |
| ST7789 Backlight | GPIO        | GPIO 20            |
| XPT2046 CS       | SPI         | GPIO 38            |
| XPT2046 IRQ      | GPIO        | GPIO 39            |
| ST7789 Reset     | GPIO        | GPIO 40            |
| LED1             | GPIO        | GPIO 47            |
| LED2             | GPIO        | GPIO 48            |

---

# 22. Considerazioni progettuali

L'architettura hardware è stata progettata per mantenere separati, per quanto possibile, i livelli:

```text
┌──────────────────────────────┐
│        USER INTERFACE        │
│       TFT + Touch            │
└──────────────┬───────────────┘
               │
┌──────────────▼───────────────┐
│       CONTROL LOGIC          │
│    ESP32-S3 + Firmware       │
└──────────────┬───────────────┘
               │
┌──────────────▼───────────────┐
│       SENSOR LAYER           │
│   ADC / GPIO / I²C / SPI     │
└──────────────┬───────────────┘
               │
┌──────────────▼───────────────┐
│       ACTUATOR LAYER         │
│      Relè / Pompe            │
└──────────────────────────────┘
```

Questa organizzazione permette di separare acquisizione, elaborazione e attuazione, facilitando manutenzione, diagnosi ed eventuali modifiche hardware.

---

# 23. Espandibilità

L'architettura può essere estesa con:

* ulteriori sensori di umidità;
* sensori di pressione;
* sensori di flusso;
* ulteriori zone di irrigazione;
* nodi ESP-NOW;
* display differenti;
* sensori ambientali aggiuntivi;
* sistemi di alimentazione tramite pannello solare;
* batterie e gestione energetica.

La comunicazione ESP-NOW permette inoltre di distribuire parte dell'acquisizione su nodi remoti.

---

# 24. Stato Hardware

**Microcontrollore:** ESP32-S3 Dual-Core
**Display:** ST7789 320×240
**Touch:** XPT2046
**RTC:** DS3231
**Sensori terreno:** 2 × ADC
**Sensore ambientale:** DHT11
**Sensore livello:** HC-SR04
**Attuatori:** 2 × pompe + relè pozzo
**Comunicazione:** Wi-Fi / ESP-NOW
**Interfacce:** SPI / I²C / GPIO / ADC

---

## Conclusione

L'hardware del progetto è stato sviluppato come una piattaforma embedded completa, nella quale **acquisizione, elaborazione, interfaccia e controllo degli attuatori** sono integrati attorno all'ESP32-S3.

La combinazione di sensori analogici e digitali, interfacce SPI/I²C, display touch, comunicazioni wireless e controllo dei carichi permette di realizzare un sistema di irrigazione automatizzato e facilmente espandibile.

