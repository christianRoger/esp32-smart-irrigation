# ESP32 Smart Irrigation

## Sistema di irrigazione IoT intelligente

Sistema di controllo dell'irrigazione sviluppato su **ESP32-S3**, che integra elettronica embedded, sensori, automazione, connettività remota e un'interfaccia touch interattiva.

Il progetto è stato sviluppato come un sistema IoT completo, in grado di monitorare le condizioni del terreno e dell'ambiente, controllare le pompe di irrigazione e adattare i cicli di irrigazione in base ai dati rilevati e alle informazioni meteorologiche.

> **Nota:** il codice sorgente non è incluso in questo repository. Il progetto è presentato come **portfolio tecnico**, con documentazione dell'architettura, delle funzionalità e delle principali soluzioni implementate.

---

## Panoramica del progetto

Il sistema integra controllo hardware, firmware embedded e connettività IoT in un'unica piattaforma per la gestione automatizzata dell'irrigazione.

Il controller gestisce:

* 💧 Pompe di irrigazione automatiche
* 🌱 Monitoraggio dell'umidità del terreno
* 🌡️ Temperatura e umidità ambientale
* 🚰 Monitoraggio del livello del serbatoio
* 🖥️ Interfaccia TFT touch
* 📡 Connettività Wi-Fi
* 🌐 Web Server integrato
* 📱 Controllo remoto e notifiche tramite Telegram
* 🔗 Comunicazione ESP-NOW
* ☁️ Integrazione con OpenWeatherMap
* 🔄 Aggiornamento firmware OTA
* 🤖 Logica SMART adattiva per l'irrigazione
* 🛡️ Watchdog e meccanismi di sicurezza

---

## Punti di forza tecnici

| Area                | Implementazione                       |
| ------------------- | ------------------------------------- |
| MCU                 | ESP32-S3 Dual-Core                    |
| Firmware            | C++ modulare / Arduino                |
| Sistema real-time   | FreeRTOS                              |
| Display             | ST7789 320×240                        |
| Touch               | XPT2046                               |
| RTC                 | DS3231                                |
| Sensori             | Umidità del terreno / DHT11 / HC-SR04 |
| Comunicazione       | Wi-Fi / ESP-NOW                       |
| Controllo remoto    | Web Server / Telegram                 |
| Cloud               | OpenWeatherMap                        |
| Memoria             | NVS / Preferences                     |
| Aggiornamenti       | OTA                                   |
| Sicurezza           | SHA-256 / CSRF / Rate Limiting        |
| Affidabilità        | Hardware Watchdog                     |
| Logica di controllo | Automatica / Manuale / SMART          |

---

## Aspetti principali del progetto

### 🤖 Logica SMART adattiva

Il sistema utilizza i dati di umidità del terreno e il tempo di funzionamento delle pompe per adattare dinamicamente la durata dei cicli di irrigazione.

La logica analizza principalmente:

* tempo di funzionamento della pompa;
* umidità del terreno prima dell'irrigazione;
* umidità del terreno dopo l'irrigazione.

Sulla base dei dati raccolti, il sistema può modificare la durata dei successivi cicli di irrigazione.

La logica SMART può inoltre utilizzare le informazioni meteorologiche provenienti da **OpenWeatherMap** per sospendere automaticamente l'irrigazione quando è prevista pioggia.

---

### 🌐 Sistema IoT

Il controller integra diversi sistemi di comunicazione:

* Wi-Fi;
* Web Server embedded;
* Telegram;
* ESP-NOW;
* OpenWeatherMap.

Questa architettura permette di combinare controllo locale, gestione remota e comunicazione con dispositivi esterni.

---

### ⚙️ Firmware modulare

Il firmware è stato inizialmente sviluppato come uno sketch monolitico di grandi dimensioni.

Con l'evoluzione del progetto, il software è stato riorganizzato in moduli C++ separati, con responsabilità specifiche per:

* interfaccia grafica;
* gestione hardware;
* irrigazione;
* rete;
* comunicazione cloud;
* ESP-NOW;
* gestione del tempo;
* profili delle piante;
* sistema e sicurezza.

La modularizzazione è stata adottata per migliorare **manutenibilità, leggibilità e possibilità di evoluzione del firmware**.

---

### 🖥️ Interfaccia embedded

Il sistema dispone di un'interfaccia grafica touch sviluppata per il display **ST7789 320×240**.

Le principali schermate comprendono:

* Home
* Acqua / Serbatoio
* Rete
* Meteo
* SMART / AI
* Programmazione
* Impostazioni

L'interfaccia comprende inoltre:

* elementi grafici e animazioni;
* navigazione tramite touch;
* gesture swipe;
* pulsanti touch;
* schermate di allarme;
* tastiera virtuale;
* visualizzazione dello stato del sistema.

---

## Hardware

### Controller principale

* ESP32-S3
* Display TFT ST7789 — 320×240
* Touch resistivo XPT2046
* RTC DS3231

### Sensori

* 2× sensori di umidità del terreno
* DHT11 per temperatura e umidità ambientale
* HC-SR04 per la misura del livello del serbatoio

### Attuatori

* 2× pompe di irrigazione
* 1× relè per il pozzo
* Buzzer
* LED di stato

---

## Software e tecnologie

* **C++**
* **Arduino**
* **ESP32-S3**
* **FreeRTOS**
* **Wi-Fi**
* **ESP-NOW**
* **HTTP / Web Server**
* **Telegram Bot API**
* **OpenWeatherMap API**
* **OTA Firmware Update**
* **NVS / Preferences**
* **JSON**
* **Watchdog**
* **SHA-256**
* **CSRF Protection**
* **Rate Limiting**

---

## Irrigazione SMART

Una delle caratteristiche principali del progetto è la modalità **SMART**, progettata per rendere la gestione dell'irrigazione più adattiva.

Il sistema considera il rapporto tra:

```text
Tempo di irrigazione
        +
Umidità prima dell'irrigazione
        +
Umidità dopo l'irrigazione
        ↓
Analisi della risposta del terreno
        ↓
Adattamento della durata
dei successivi cicli
```

La logica permette al controller di utilizzare il comportamento osservato del sistema per adattare progressivamente i parametri di irrigazione.

In presenza di previsioni di pioggia, il sistema può inoltre sospendere automaticamente i cicli di irrigazione programmati.

---

## Controllo remoto

Il sistema supporta diverse modalità di comunicazione e controllo.

### Web Server

Il Web Server embedded permette di monitorare e configurare il dispositivo direttamente tramite browser.

Le principali funzioni comprendono:

* visualizzazione dello stato del sistema;
* controllo manuale delle pompe;
* programmazione dell'irrigazione;
* configurazione Wi-Fi;
* calibrazione dei sensori;
* gestione dei profili delle piante;
* configurazione della modalità SMART;
* configurazione meteorologica;
* gestione dei log;
* aggiornamento firmware OTA;
* configurazione del sistema;
* factory reset.

### Telegram

L'integrazione con Telegram permette di:

* ricevere notifiche;
* monitorare eventi del sistema;
* inviare comandi da remoto;
* ricevere informazioni sullo stato dell'impianto.

### ESP-NOW

ESP-NOW viene utilizzato per la comunicazione con sensori e dispositivi ESP remoti, permettendo di estendere il sistema con nodi distribuiti.

---

## Affidabilità e sicurezza

Il firmware integra diversi meccanismi progettati per migliorare l'affidabilità e la sicurezza del sistema:

* Hardware Watchdog
* Accesso protetto tramite password
* Hashing della password con SHA-256
* Protezione CSRF
* Rate Limiting
* Protezione degli aggiornamenti OTA
* Factory Reset
* Configurazione persistente tramite NVS
* Gestione delle condizioni anomale del sistema

---

## Architettura

![System Architecture](docs/architecture.svg)

Il sistema segue un'architettura composta da hardware, firmware embedded e servizi esterni.

```text
                    ESP32-S3
                       │
        ┌──────────────┼──────────────┐
        │              │              │
     Sensori        Attuatori      Interfaccia
        │              │              │
   Suolo / DHT11   Pompe / Relè    ST7789 + Touch
   HC-SR04         Buzzer / LED
        │
        └──────────────┬──────────────┘
                       │
                  Logica di controllo
                       │
              ┌────────┼────────┐
              │        │        │
             Wi-Fi   Telegram  ESP-NOW
              │
        OpenWeatherMap
              │
          SMART Logic
```

La comunicazione tra i diversi componenti permette di separare:

* acquisizione dei dati;
* gestione hardware;
* logica di irrigazione;
* interfaccia utente;
* comunicazione di rete;
* servizi cloud;
* gestione della sicurezza.

Per la descrizione dettagliata dell'architettura software, dei moduli e dei flussi di dati:

➡️ **[Documentazione tecnica completa](docs/DOCUMENTAZIONE.md)**

---

## Principali sfide tecniche

Durante lo sviluppo del sistema sono state affrontate diverse problematiche tecniche, tra cui:

* Modularizzazione di un firmware inizialmente sviluppato come sketch monolitico di grandi dimensioni.
* Integrazione di più sensori e attuatori con ESP32-S3.
* Coordinamento tra interfaccia utente, acquisizione dei sensori, controllo dell'irrigazione e servizi di rete.
* Gestione della configurazione persistente tramite NVS.
* Implementazione dell'aggiornamento firmware OTA tramite Web Server integrato.
* Comunicazione remota tramite Wi-Fi, Telegram ed ESP-NOW.
* Calibrazione e gestione dell'acquisizione dei sensori.
* Implementazione di watchdog, autenticazione, protezione CSRF e rate limiting.
* Sviluppo di una logica SMART adattiva basata sul feedback dell'umidità del terreno.
* Gestione di più funzionalità concorrenti all'interno dell'ambiente FreeRTOS.

---

## Architettura hardware e software

Il sistema segue una struttura a più livelli:

```text
┌───────────────────────────────────────┐
│             USER INTERFACE            │
│        TFT Touch / Web / Telegram     │
└───────────────────┬───────────────────┘
                    │
┌───────────────────▼───────────────────┐
│          CONTROL & SMART LOGIC        │
│     Irrigation / Scheduling / AI      │
└───────────────────┬───────────────────┘
                    │
┌───────────────────▼───────────────────┐
│          SENSOR / HARDWARE            │
│ Soil / DHT11 / HC-SR04 / RTC / Relays│
└───────────────────┬───────────────────┘
                    │
┌───────────────────▼───────────────────┐
│              ESP32-S3                 │
│        Embedded Control Platform      │
└───────────────────────────────────────┘
```

Questa separazione permette di mantenere distinti acquisizione dati, logica di controllo, interfaccia utente e comunicazione, facilitando la manutenzione e l'evoluzione del sistema.

---

## Principali aree tecniche

Il progetto dimostra esperienza pratica nelle seguenti aree:

* Sviluppo di sistemi embedded
* Programmazione di microcontrollori
* Programmazione C++
* Integrazione hardware e software
* Acquisizione dati da sensori
* Controllo di attuatori
* Comunicazione SPI e I²C
* Letture ADC e calibrazione dei sensori
* Networking Wi-Fi
* Comunicazione IoT
* Web Server embedded
* Interfacce grafiche touch
* Comunicazione ESP-NOW
* Gestione remota dei dispositivi
* Aggiornamenti firmware OTA
* Gestione di task real-time
* Sicurezza embedded
* Diagnostica del sistema
* Automazione
* Integrazione hardware/software

---

## Project Status

| Parametro            | Valore                       |
| -------------------- | ---------------------------- |
| Piattaforma          | ESP32-S3                     |
| Display              | ST7789 320×240               |
| Touch                | XPT2046                      |
| RTC                  | DS3231                       |
| Ambiente di sviluppo | Arduino IDE                  |
| Firmware             | C++                          |
| Architettura         | Firmware modulare            |
| Real-Time            | FreeRTOS                     |
| Tipologia            | Embedded / IoT / Automazione |

---

## Obiettivo del progetto

Il progetto è stato sviluppato come applicazione pratica di **sistemi embedded, automazione e IoT**, combinando elettronica, firmware e comunicazione di rete in un'unica piattaforma.

L'obiettivo è realizzare un sistema di irrigazione capace non solo di eseguire comandi predefiniti, ma anche di:

* acquisire dati dal campo;
* analizzare le condizioni rilevate;
* controllare autonomamente gli attuatori;
* comunicare con dispositivi remoti;
* fornire interfacce locali e remote;
* utilizzare informazioni meteorologiche;
* adattare il comportamento dell'irrigazione;
* mantenere configurazioni e parametri nel tempo.

---

## Portfolio tecnico

Questo repository rappresenta una sintesi delle competenze applicate nello sviluppo del progetto, tra cui:

**Elettronica · Firmware Embedded · C++ · ESP32 · Automazione · IoT · Networking · Sensoristica · Controllo · Interfacce Touch · Web Server · OTA · Diagnostica**

Il codice sorgente non viene pubblicato; la documentazione è fornita per illustrare l'architettura, le funzionalità, le problematiche affrontate e le principali soluzioni tecniche adottate.

---

## Documentazione

Per approfondire l'architettura e le caratteristiche tecniche del progetto:

- 📐 [Architettura del sistema](docs/ARCHITECTURE.md)
- 📘 [Documentazione tecnica](docs/DOCUMENTATION.md)

La documentazione comprende:

* architettura del sistema;
* hardware e pinout;
* struttura del firmware;
* moduli software;
* comunicazione;
* Web Server;
* Telegram;
* ESP-NOW;
* logica SMART;
* sicurezza;
* OTA;
* gestione dei sensori;
* struttura del progetto.
