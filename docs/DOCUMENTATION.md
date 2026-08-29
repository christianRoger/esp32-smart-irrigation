# Documentazione Tecnica — ESP32 Smart Irrigation

## 1. Panoramica del progetto

**ESP32 Smart Irrigation** è un sistema IoT per il controllo e la gestione automatizzata dell'irrigazione, sviluppato attorno a un microcontrollore **ESP32-S3**.

Il sistema integra controllo hardware, acquisizione dati da sensori, interfaccia grafica touch, connettività di rete e logiche di irrigazione adattive.

L'obiettivo del progetto è realizzare un sistema in grado di:

* monitorare l'umidità del terreno;
* controllare automaticamente le pompe di irrigazione;
* monitorare il livello dell'acqua nel serbatoio;
* acquisire temperatura e umidità ambientale;
* gestire programmi di irrigazione;
* fornire un'interfaccia locale tramite display touch;
* consentire il controllo remoto tramite Web Server e Telegram;
* comunicare con dispositivi remoti tramite ESP-NOW;
* utilizzare dati meteorologici per ottimizzare l'irrigazione;
* aggiornare il firmware tramite OTA;
* adattare automaticamente la durata dell'irrigazione attraverso la logica SMART.

> **Nota:** il codice sorgente non è incluso nel repository. Il progetto è pubblicato come portfolio tecnico per documentare architettura, funzionalità, integrazione hardware/software e competenze sviluppate.

---

# 2. Architettura del sistema

Il sistema è composto da tre aree principali:

1. **Hardware e periferiche**
2. **Firmware embedded ESP32-S3**
3. **Servizi e dispositivi esterni**

L'ESP32-S3 rappresenta il nodo centrale del sistema e coordina l'acquisizione dei dati, la logica di controllo, l'interfaccia utente e le comunicazioni.

![System Architecture](architecture.svg)

---

## 2.1 Architettura di deployment

### Hardware / periferiche

Il sistema hardware comprende:

* TFT ST7789 320×240;
* touch resistivo XPT2046;
* sensori di umidità del terreno;
* DHT11;
* sensore ultrasonico HC-SR04;
* RTC DS3231;
* relè per le pompe;
* relè del pozzo;
* buzzer;
* LED di stato;
* pulsante per il reset della configurazione Wi-Fi.

### ESP32-S3

Il microcontrollore esegue il firmware principale e gestisce:

* acquisizione dei sensori;
* controllo degli attuatori;
* interfaccia grafica;
* gestione della rete;
* Web Server;
* Telegram;
* ESP-NOW;
* logica SMART;
* gestione della configurazione;
* sicurezza;
* watchdog.

### Servizi esterni

Il sistema può comunicare con:

* browser Web;
* Telegram;
* OpenWeatherMap;
* dispositivi ESP-NOW remoti;
* rete Wi-Fi / Internet.

La configurazione persistente viene memorizzata nella **NVS (Non-Volatile Storage)** dell'ESP32.

---

# 3. Hardware e pinout

## 3.1 Tabella di collegamento

| Segnale          | GPIO | Funzione                  |
| ---------------- | ---: | ------------------------- |
| `POMPA1`         |    4 | Relè pompa 1              |
| `POMPA2`         |    5 | Relè pompa 2              |
| `RELE_POZZO`     |    6 | Relè pompa/pozzo          |
| `BUZZER_PIN`     |    7 | Buzzer                    |
| `DHTPIN`         |   15 | Sensore DHT11             |
| `TRIG_PIN`       |   16 | Trigger HC-SR04           |
| `ECHO_PIN`       |   17 | Echo HC-SR04              |
| `RESET_WIFI_PIN` |   18 | Pulsante reset Wi-Fi      |
| `I2C_SDA`        |    8 | SDA — RTC DS3231          |
| `I2C_SCL`        |    9 | SCL — RTC DS3231          |
| `TFT_CS`         |   10 | Chip Select ST7789        |
| `TFT_MOSI`       |   11 | SPI MOSI                  |
| `TFT_SCK`        |   12 | SPI Clock                 |
| `TFT_MISO`       |   13 | SPI MISO                  |
| `TFT_DC`         |   14 | Data/Command ST7789       |
| `SOIL_PIN1`      |    1 | Sensore umidità terreno 1 |
| `SOIL_PIN2`      |    2 | Sensore umidità terreno 2 |
| `TFT_RST`        |   40 | Reset ST7789              |
| `TFT_LED`        |   20 | Backlight display         |
| `T_CS`           |   38 | Chip Select touch XPT2046 |
| `T_IRQ`          |   39 | Interrupt touch           |
| `LED1`           |   47 | LED di stato 1            |
| `LED2`           |   48 | LED di stato 2            |

### Note elettriche

I relè utilizzano logica:

```text
RELAY_ON  = LOW
RELAY_OFF = HIGH
```

L'ingresso `ECHO_PIN` del sensore HC-SR04 deve essere adattato al livello logico compatibile con ESP32-S3, utilizzando un opportuno partitore di tensione quando necessario.

---

# 4. Specifiche principali

| Parametro                     | Valore                |
| ----------------------------- | --------------------- |
| Microcontrollore              | ESP32-S3 Dual-Core    |
| Display                       | ST7789                |
| Risoluzione                   | 320×240               |
| Touch                         | XPT2046 resistivo     |
| Sensori terreno               | 2 × ADC               |
| Temperatura/umidità           | DHT11                 |
| Livello serbatoio             | HC-SR04               |
| RTC                           | DS3231                |
| Comunicazione locale          | Wi-Fi / ESP-NOW       |
| Controllo remoto              | Web Server / Telegram |
| Servizio meteorologico        | OpenWeatherMap        |
| Storage                       | NVS / Preferences     |
| Aggiornamento                 | OTA                   |
| Watchdog                      | 30 secondi            |
| Numero massimo profili piante | 100                   |

---

# 5. Struttura software

Il firmware è stato inizialmente sviluppato come un unico sketch monolitico.

Con l'evoluzione del progetto, il firmware è stato riorganizzato in moduli funzionali separati, con l'obiettivo di migliorare:

* leggibilità;
* manutenzione;
* separazione delle responsabilità;
* riutilizzabilità;
* debugging;
* gestione delle dipendenze.

La struttura logica del firmware è organizzata intorno a un header comune e a moduli specializzati.

## 5.1 Moduli principali

| Modulo           | Responsabilità                                    |
| ---------------- | ------------------------------------------------- |
| `app.h`          | Tipi globali, include, define, extern e prototipi |
| `main.ino`       | `setup()` e `loop()`                              |
| `globals.cpp`    | Definizione delle variabili globali               |
| `display.cpp`    | UI, display, touch e tastiera virtuale            |
| `hardware.cpp`   | I/O, sensori e attuatori                          |
| `irrigation.cpp` | Logica di irrigazione e modalità SMART            |
| `time.cpp`       | RTC, NTP e gestione dell'orario                   |
| `wifi.cpp`       | Wi-Fi e WiFiManager                               |
| `web.cpp`        | Web Server ed endpoint HTTP                       |
| `cloud.cpp`      | OpenWeatherMap e configurazione cloud             |
| `espnow.cpp`     | Comunicazione ESP-NOW                             |
| `plants.cpp`     | Profili delle piante                              |
| `system.cpp`     | Watchdog, sicurezza, log e funzioni di sistema    |

---

# 6. Organizzazione dei moduli

## `app.h`

`app.h` rappresenta il punto comune di riferimento del firmware.

Contiene:

* librerie;
* costanti;
* macro;
* strutture dati;
* dichiarazioni `extern`;
* prototipi delle funzioni.

L'utilizzo di un header comune permette ai diversi moduli di condividere in modo controllato tipi, variabili e interfacce.

---

## `globals.cpp`

Contiene le definizioni effettive delle variabili globali dichiarate tramite `extern` in `app.h`.

Questo approccio evita definizioni multiple durante il linking e centralizza lo stato globale del sistema.

---

## `display.cpp`

Gestisce l'interfaccia grafica del dispositivo.

Responsabilità principali:

* rendering delle schermate;
* gestione touch;
* navigazione;
* swipe;
* pulsanti;
* animazioni;
* schermate di configurazione;
* messaggi di errore;
* schermate di allarme;
* tastiera virtuale.

---

## `hardware.cpp`

Gestisce l'interazione diretta con l'hardware.

Comprende:

* relè;
* pompe;
* buzzer;
* LED;
* DHT11;
* sensori;
* letture GPIO;
* sensore ultrasonico.

---

## `irrigation.cpp`

Contiene la logica principale del sistema di irrigazione.

Gestisce:

* pompe;
* relè del pozzo;
* lettura dei sensori del terreno;
* livello del serbatoio;
* programmazione;
* modalità automatica;
* modalità manuale;
* sospensione dell'irrigazione;
* calibrazione dei sensori;
* modalità SMART.

---

## `time.cpp`

Gestisce il sistema temporale:

* RTC DS3231;
* sincronizzazione NTP;
* impostazione manuale dell'orario;
* gestione del tempo di sistema;
* formattazione dell'orario.

Il modulo contiene inoltre alcune funzioni relative alla gestione dei comandi Telegram.

---

## `wifi.cpp`

Gestisce:

* connessione Wi-Fi;
* configurazione tramite WiFiManager;
* scansione delle reti;
* stato della connessione;
* reset delle credenziali Wi-Fi.

---

## `web.cpp`

Implementa il Web Server embedded.

Il browser può essere utilizzato per:

* monitoraggio;
* configurazione;
* controllo manuale;
* gestione dell'irrigazione;
* configurazione Wi-Fi;
* calibrazione;
* gestione delle piante;
* configurazione SMART;
* gestione dei log;
* aggiornamento OTA.

---

## `cloud.cpp`

Gestisce l'integrazione con servizi esterni.

In particolare:

* OpenWeatherMap;
* configurazione Telegram;
* configurazione meteo;
* salvataggio delle credenziali nella NVS;
* aggiornamento delle informazioni meteorologiche.

---

## `espnow.cpp`

Implementa la comunicazione tramite **ESP-NOW**.

Il modulo permette di ricevere dati da dispositivi ESP32 remoti senza richiedere una connessione TCP/IP tradizionale tra i dispositivi.

---

## `plants.cpp`

Gestisce i profili delle piante.

Il sistema supporta fino a:

```text
MAX_PLANTS = 100
```

Ogni profilo può essere utilizzato per associare parametri specifici alla gestione dell'irrigazione.

---

## `system.cpp`

Contiene le funzionalità di sistema:

* watchdog;
* sicurezza;
* rate limiting;
* CSRF;
* log;
* factory reset;
* funzioni di inizializzazione;
* gestione delle task;
* funzioni di supporto.

---

# 7. Interfaccia utente

L'interfaccia grafica è sviluppata per il display **ST7789 320×240** con touch resistivo **XPT2046**.

Le principali schermate sono:

* **Home**
* **Acqua / Serbatoio**
* **Rete**
* **Meteo**
* **SMART / AI**
* **Programmazione**
* **Impostazioni**

La navigazione utilizza:

* pulsanti touch;
* gesture swipe;
* tastiera virtuale;
* schermate di configurazione;
* messaggi di stato;
* schermate di allarme.

### Gestione degli swipe

Parametri principali:

```text
SWIPE_THRESHOLD = 60 px
SWIPE_MAX_MS    = 800 ms
```

---

# 8. Sistema di irrigazione

Il sistema può operare in modalità:

* **Automatica**
* **Manuale**
* **SMART**

## 8.1 Controllo automatico

La modalità automatica utilizza programmi configurabili dall'utente.

La struttura principale del programma è:

```text
Programma
├── Inizio
├── Fine
└── Attivo
```

Il sistema verifica le condizioni configurate e determina quando avviare il ciclo di irrigazione.

---

# 9. Modalità SMART

La modalità **SMART** rappresenta una delle principali características tecniche del progetto.

Il sistema utilizza una logica adattiva basata sui dati raccolti durante i cicli di irrigazione.

La relazione analizzata comprende:

```text
Tempo di funzionamento della pompa
            ↓
Umidità prima dell'irrigazione
            ↓
Ciclo di irrigazione
            ↓
Umidità dopo l'irrigazione
```

Il sistema può quindi stimare il guadagno di umidità ottenuto rispetto al tempo di funzionamento della pompa.

I dati vengono utilizzati per adattare la durata dei cicli successivi.

### Vantaggi

La logica SMART permette di:

* ridurre irrigazioni eccessive;
* adattare il funzionamento alle condizioni reali del terreno;
* migliorare la gestione dell'acqua;
* utilizzare dati storici per modificare il comportamento del sistema.

> La modalità SMART è una logica adattiva/euristica implementata a livello firmware e non rappresenta un modello di machine learning addestrato tramite framework esterni.

---

# 10. Gestione meteorologica

Il sistema può utilizzare **OpenWeatherMap** per ottenere informazioni sulle condizioni meteorologiche.

Uno degli utilizzi principali è la previsione della pioggia.

Il flusso logico è:

```text
OpenWeatherMap
       ↓
Previsione meteorologica
       ↓
Pioggia prevista?
       ↓
      Sì
       ↓
Sospensione irrigazione automatica
```

Questo permette di evitare, quando previsto dalle condizioni configurate, l'attivazione di cicli di irrigazione prima di una precipitazione.

---

# 11. Sensori

## 11.1 Sensori di umidità del terreno

Sono presenti due ingressi ADC dedicati:

```text
SOIL_PIN1 → GPIO 1
SOIL_PIN2 → GPIO 2
```

I sensori possono essere calibrati attraverso valori di riferimento per terreno secco e umido.

La calibrazione viene salvata nella memoria non volatile.

---

## 11.2 DHT11

Il DHT11 viene utilizzato per acquisire:

* temperatura;
* umidità relativa dell'ambiente.

Collegamento:

```text
DHT11 → GPIO 15
```

---

## 11.3 HC-SR04

Il sensore ultrasonico viene utilizzato per determinare il livello dell'acqua nel serbatoio.

```text
TRIG → GPIO 16
ECHO → GPIO 17
```

Il sistema utilizza la distanza misurata per determinare lo stato del serbatoio e generare eventuali condizioni di allarme.

---

## 11.4 RTC DS3231

Il DS3231 fornisce una sorgente temporale locale tramite I²C.

```text
SDA → GPIO 8
SCL → GPIO 9
```

Il sistema può inoltre sincronizzare l'orario tramite NTP quando è disponibile la connessione Wi-Fi.

---

# 12. Web Server

Il firmware include un Web Server embedded accessibile tramite browser.

Tra le principali funzioni disponibili:

| Endpoint         | Funzione                     |
| ---------------- | ---------------------------- |
| `/`              | Pagina principale            |
| `/status`        | Stato del sistema            |
| `/ora`           | Orario                       |
| `/salvaprog`     | Salvataggio programmazione   |
| `/getprog`       | Lettura programmazione       |
| `/pompa1`        | Controllo pompa 1            |
| `/pompa2`        | Controllo pompa 2            |
| `/pozzo`         | Controllo relè pozzo         |
| `/setauto`       | Modalità automatica          |
| `/sospensione`   | Sospensione                  |
| `/setwifi`       | Configurazione Wi-Fi         |
| `/scanwifi`      | Scansione reti               |
| `/resetwifi`     | Reset Wi-Fi                  |
| `/setsmart`      | Modalità SMART               |
| `/synctime`      | Sincronizzazione orario      |
| `/setmanualtime` | Impostazione manuale         |
| `/getSoilCal`    | Lettura calibrazione terreno |
| `/setSoilCal`    | Salvataggio calibrazione     |
| `/getPlants`     | Lettura piante               |
| `/savePlant`     | Salvataggio pianta           |
| `/deletePlant`   | Eliminazione pianta          |
| `/applyPlant`    | Applicazione profilo         |
| `/resetPlants`   | Reset profili                |
| `/ota`           | Pagina OTA                   |
| `/update`        | Upload firmware              |
| `/downloadlogs`  | Download log                 |
| `/clearlogs`     | Cancellazione log            |
| `/factoryreset`  | Factory reset                |
| `/version`       | Versione firmware            |
| `/macAddress`    | Indirizzo MAC                |

---

# 13. Telegram

L'integrazione Telegram permette di utilizzare il dispositivo anche da remoto.

Funzionalità previste:

* notifiche di eventi;
* notifiche relative alle pompe;
* notifiche relative al serbatoio;
* comandi remoti;
* configurazione del bot.

Le informazioni necessarie vengono memorizzate nella configurazione cloud.

Le credenziali non vengono pubblicate nel repository.

---

# 14. ESP-NOW

ESP-NOW viene utilizzato per la comunicazione con dispositivi ESP remoti.

Possibili applicazioni:

* sensori di umidità remoti;
* nodi ambientali;
* dispositivi ESP aggiuntivi;
* acquisizione distribuita dei dati.

Il firmware gestisce la ricezione dei dati attraverso callback dedicate.

---

# 15. Memoria persistente

Il sistema utilizza la **NVS (Non-Volatile Storage)** tramite `Preferences`.

La memoria viene utilizzata per conservare configurazioni che devono rimanere disponibili anche dopo il riavvio del dispositivo.

Esempi:

* calibrazione dei sensori;
* configurazione Wi-Fi;
* configurazione Telegram;
* configurazione OpenWeatherMap;
* programmazione;
* parametri SMART;
* impostazioni del sistema;
* profili delle piante.

---

# 16. Sicurezza

Il firmware implementa diversi meccanismi di protezione.

## Autenticazione

L'accesso alle funzioni sensibili può essere protetto tramite password.

La password viene gestita utilizzando un hash **SHA-256**.

---

## CSRF Protection

Gli endpoint sensibili possono utilizzare meccanismi di protezione contro richieste non autorizzate tramite **CSRF token**.

---

## Rate Limiting

Il firmware implementa un sistema di limitazione delle richieste per ridurre tentativi ripetuti di accesso o abuso degli endpoint.

---

## Factory Reset

Il sistema dispone di una procedura di ripristino alle impostazioni di fabbrica.

Il factory reset permette di cancellare le configurazioni persistenti e riportare il dispositivo allo stato iniziale.

---

# 17. Watchdog

Per aumentare l'affidabilità del sistema viene utilizzato un **Task Watchdog**.

Configurazione principale:

```text
WDT_TIMEOUT = 30 secondi
```

In caso di blocco o mancata esecuzione corretta delle attività monitorate, il watchdog può provocare il riavvio del microcontrollore.

---

# 18. OTA — Over The Air Update

Il firmware supporta l'aggiornamento tramite rete Wi-Fi.

Il processo utilizza:

```text
Browser
   ↓
Web Server
   ↓
Upload firmware
   ↓
OTA Update
   ↓
Nuova partizione firmware
   ↓
Riavvio ESP32-S3
```

La funzione OTA è protetta dalle funzionalità di autenticazione del sistema.

Per utilizzare correttamente l'aggiornamento è consigliata una configurazione di partizionamento compatibile con OTA.

---

# 19. FreeRTOS

L'ESP32-S3 utilizza il framework **FreeRTOS** per la gestione delle attività concorrenti.

Le principali attività comprendono:

```text
┌──────────────────────────────┐
│         ESP32-S3             │
├──────────────────────────────┤
│ UI / Touch                   │
│                              │
│ Sensors Task                 │
│                              │
│ Irrigation Control           │
│                              │
│ Web Server / Network         │
│                              │
│ Watchdog                     │
└──────────────────────────────┘
```

La separazione delle attività consente di mantenere indipendenti le principali funzioni del sistema e di ridurre il rischio che un singolo componente blocchi l'intero firmware.

---

# 20. Flusso principale dei dati

Il flusso generale del sistema può essere rappresentato come:

```text
Sensori
   │
   ▼
Acquisizione dati
   │
   ▼
Elaborazione firmware
   │
   ├──────────────► Display
   │
   ├──────────────► Web Server
   │
   ├──────────────► Telegram
   │
   └──────────────► Logica SMART
                         │
                         ▼
                 Decisione irrigazione
                         │
                         ▼
                  Pompe / Relè
```

Le informazioni meteorologiche possono influenzare la decisione finale:

```text
OpenWeatherMap
      │
      ▼
Previsione pioggia
      │
      ▼
Logica SMART / Automatica
      │
      ▼
Irrigazione consentita?
```

---

# 21. Stati operativi

Il sistema può gestire diversi stati operativi:

```text
                    ┌─────────────┐
                    │   SISTEMA   │
                    │   AVVIATO   │
                    └──────┬──────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          MANUALE       AUTOMATICO      SMART
             │             │             │
             ▼             ▼             ▼
          Comando       Programma     Adattamento
          diretto       irrigazione   dinamico
```

A questi stati si aggiungono condizioni di sicurezza, come:

* serbatoio vuoto;
* sospensione manuale;
* fine-corsa;
* errore di sistema;
* perdita di condizioni necessarie all'irrigazione.

---

# 22. Allarmi e protezioni operative

Il sistema dispone di schermate dedicate agli eventi anomali.

Tra gli stati gestiti:

* irrigazione in corso;
* attivazione manuale;
* attivazione programmata;
* serbatoio vuoto;
* terreno secco;
* aggiornamento OTA;
* reset del sistema.

Gli allarmi vengono visualizzati localmente e, quando configurato, possono essere notificati anche tramite Telegram.

---

# 23. Librerie e tecnologie

Il progetto utilizza o integra le seguenti tecnologie:

| Libreria / tecnologia | Utilizzo                   |
| --------------------- | -------------------------- |
| Arduino GFX Library   | Driver display ST7789      |
| XPT2046_Touchscreen   | Touch resistivo            |
| SPI                   | Comunicazione SPI          |
| Wire                  | Comunicazione I²C          |
| ArduinoJson           | Parsing JSON               |
| DHT                   | Sensore DHT11              |
| HTTPClient            | Comunicazione HTTP         |
| WiFi                  | Connettività Wi-Fi         |
| WiFiClientSecure      | HTTPS                      |
| NTPClient             | Sincronizzazione NTP       |
| Preferences           | NVS                        |
| PubSubClient          | MQTT opzionale             |
| UniversalTelegramBot  | Telegram                   |
| WebServer             | Web Server embedded        |
| Update                | OTA                        |
| WiFiManager           | Configurazione Wi-Fi       |
| RTClib                | RTC DS3231                 |
| ESP-NOW               | Comunicazione peer-to-peer |
| esp_task_wdt          | Watchdog                   |
| mbedTLS / SHA-256     | Hash password              |

La libreria **Espalexa** è stata rimossa dal progetto perché non utilizzata nella versione attuale.

---

# 24. Struttura del progetto

La struttura concettuale del progetto è:

```text
esp32-smart-irrigation/
│
├── README.md
│
├── docs/
│   ├── architecture.svg
│   └── DOCUMENTAZIONE.md
│
└── images/
    └── ...
```

Il repository pubblico contiene la documentazione e gli elementi necessari per presentare il progetto come portfolio tecnico.

Il firmware completo non viene pubblicato.

---

# 25. Ambiente di sviluppo

Il progetto è stato sviluppato utilizzando:

```text
Microcontrollore : ESP32-S3
Framework        : Arduino
Linguaggio       : C++
IDE              : Arduino IDE
RTOS             : FreeRTOS
Core ESP32       : 2.0.17
```

Per una eventuale ricostruzione del firmware originale, è necessario utilizzare una configurazione compatibile con il core ESP32 e con le librerie indicate nella documentazione.

---

# 26. Modularizzazione del firmware

Il firmware originale era costituito da uno sketch monolitico di grandi dimensioni.

La successiva modularizzazione ha separato le funzionalità in componenti indipendenti.

La struttura risultante segue il principio:

```text
                app.h
                  │
        ┌─────────┼─────────┐
        │         │         │
        ▼         ▼         ▼
       UI       Control    Network
        │         │         │
   display.cpp irrigation.cpp web.cpp
              │
              ▼
         hardware.cpp
              │
              ▼
           ESP32-S3
```

Questo approccio facilita:

* manutenzione;
* debugging;
* estensione del firmware;
* individuazione degli errori;
* separazione delle responsabilità;
* gestione delle dipendenze.

---

# 27. Processo di sviluppo

Il progetto è stato sviluppato attraverso un processo iterativo:

```text
Prototipo hardware
       ↓
Acquisizione sensori
       ↓
Controllo attuatori
       ↓
Interfaccia locale
       ↓
Connettività Wi-Fi
       ↓
Web Server
       ↓
Comunicazione remota
       ↓
Automazione
       ↓
Logica SMART
       ↓
Sicurezza e affidabilità
       ↓
Modularizzazione firmware
```

Questo approccio ha permesso di evolvere il sistema da un semplice controller hardware a una piattaforma IoT completa.

---

# 28. Competenze tecniche dimostrate

Il progetto rappresenta un'integrazione di diverse aree dell'ingegneria elettronica e dello sviluppo embedded.

### Embedded Systems

* Programmazione C/C++;
* ESP32-S3;
* FreeRTOS;
* gestione memoria;
* gestione task;
* watchdog.

### Elettronica

* GPIO;
* ADC;
* relè;
* sensori analogici;
* sensori digitali;
* SPI;
* I²C;
* integrazione periferiche.

### IoT

* Wi-Fi;
* Web Server;
* REST-like HTTP endpoints;
* Telegram;
* ESP-NOW;
* OpenWeatherMap;
* OTA.

### Software Architecture

* modularizzazione;
* separazione delle responsabilità;
* gestione delle dipendenze;
* strutture dati;
* configurazione persistente;
* gestione dello stato.

### Sicurezza

* SHA-256;
* autenticazione;
* CSRF protection;
* rate limiting;
* protezione OTA.

### Automazione

* controllo automatico;
* programmazione;
* feedback da sensori;
* logica adattiva;
* gestione delle condizioni ambientali.

---

# 29. Possibili sviluppi futuri

L'architettura del progetto permette ulteriori evoluzioni, tra cui:

* dashboard Web più avanzata;
* gestione di un numero maggiore di zone di irrigazione;
* sensori remoti aggiuntivi;
* storico dei dati;
* grafici di consumo dell'acqua;
* integrazione con ulteriori piattaforme IoT;
* algoritmi predittivi più avanzati;
* gestione energetica;
* alimentazione tramite pannello solare;
* espansione del sistema ESP-NOW;
* database remoto per lo storico.

---

# 30. Conclusioni

**ESP32 Smart Irrigation** è un progetto embedded/IoT completo che integra elettronica, firmware, automazione e comunicazione di rete.

Il progetto dimostra la capacità di sviluppare un sistema partendo dall'integrazione hardware fino alla gestione software di alto livello, includendo:

* acquisizione dati;
* controllo attuatori;
* interfaccia utente;
* networking;
* controllo remoto;
* automazione;
* logica adattiva;
* sicurezza;
* aggiornamento OTA;
* gestione real-time.

L'architettura modulare permette inoltre di mantenere il firmware organizzato e di facilitarne l'evoluzione futura.

---

## Stato del progetto

**Platform:** ESP32-S3
**Display:** ST7789 320×240
**Touch:** XPT2046
**Firmware:** C++ / Arduino
**Real-Time:** FreeRTOS
**Project Type:** Embedded / IoT / Automation
**Repository:** Technical Portfolio
