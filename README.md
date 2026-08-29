# ESP32 Smart Irrigation

## Sistema di irrigazione IoT intelligente

Sistema di controllo dell'irrigazione sviluppato su **ESP32-S3**, progettato per integrare elettronica embedded, sensori, automazione, connettività di rete, controllo remoto e interfaccia touch.

Il progetto combina acquisizione dei dati, controllo degli attuatori e logiche automatiche per realizzare un sistema di irrigazione capace di adattare il proprio funzionamento alle condizioni del terreno e alle informazioni meteorologiche.

> **Nota:** il codice sorgente non è incluso nel repository. Il progetto è pubblicato come **portfolio tecnico**, con documentazione dell'architettura, delle funzionalità e delle principali soluzioni tecniche adottate.

---

## Panoramica del progetto

Il sistema integra hardware, firmware e servizi IoT all'interno di un'unica piattaforma embedded.

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

| Area                   | Implementazione                       |
| ---------------------- | ------------------------------------- |
| MCU                    | ESP32-S3 Dual-Core                    |
| Firmware               | C++ modulare / Arduino                |
| Sistema real-time      | FreeRTOS                              |
| Display                | ST7789 320×240                        |
| Touch                  | XPT2046                               |
| RTC                    | DS3231                                |
| Sensori                | Umidità del terreno / DHT11 / HC-SR04 |
| Comunicazione          | Wi-Fi / ESP-NOW                       |
| Controllo remoto       | Web Server / Telegram                 |
| Servizio meteorologico | OpenWeatherMap                        |
| Memoria                | NVS / Preferences                     |
| Aggiornamenti          | OTA                                   |
| Sicurezza              | SHA-256 / CSRF / Rate Limiting        |
| Affidabilità           | Hardware Watchdog                     |
| Logica di controllo    | Automatica / Manuale / SMART          |

---

## Aspetti principali del progetto

### 🤖 Logica SMART adattiva

La modalità **SMART** utilizza i dati rilevati dai sensori e le informazioni raccolte durante i cicli di irrigazione per rendere il controllo più adattivo.

Il sistema considera principalmente:

* durata del funzionamento della pompa;
* umidità del terreno prima dell'irrigazione;
* umidità del terreno dopo l'irrigazione;
* variazione dell'umidità ottenuta.

Sulla base di queste informazioni, la durata dei cicli può essere adattata per migliorare la gestione dell'acqua.

La logica SMART può inoltre utilizzare le informazioni meteorologiche provenienti da **OpenWeatherMap** per evitare o sospendere l'irrigazione automatica quando è prevista pioggia.

---

### ⚙️ Firmware modulare

Il firmware è stato inizialmente sviluppato come uno sketch monolitico.

Con l'evoluzione del progetto, l'architettura è stata riorganizzata in moduli C++ separati, ciascuno dedicato a una specifica responsabilità.

Questa organizzazione facilita:

* manutenzione;
* diagnostica;
* debugging;
* evoluzione del firmware;
* integrazione di nuove funzionalità;
* separazione tra hardware, controllo e comunicazione.

---

### 🖥️ Interfaccia embedded

Il sistema dispone di un'interfaccia grafica touch basata su **ST7789 320×240** e **XPT2046**.

L'interfaccia permette di:

* visualizzare lo stato del sistema;
* controllare manualmente le pompe;
* configurare l'irrigazione;
* visualizzare i dati dei sensori;
* consultare le informazioni meteorologiche;
* configurare la modalità SMART;
* gestire le impostazioni del sistema.

Sono inoltre presenti gesture touch, animazioni, schermate di allarme e tastiera virtuale.

---

### 🌐 Sistema IoT

Il controller integra diverse tecnologie di comunicazione:

* Wi-Fi;
* Web Server;
* Telegram;
* ESP-NOW;
* OpenWeatherMap.

Questa architettura permette di gestire il dispositivo sia localmente tramite display e browser, sia da remoto tramite servizi di rete.

---

## Hardware

### Controller principale

* ESP32-S3 Dual-Core
* Display TFT ST7789 — 320×240
* Touch resistivo XPT2046
* RTC DS3231

### Sensori

* 2× sensori di umidità del terreno
* DHT11 per temperatura e umidità
* HC-SR04 per il rilevamento del livello dell'acqua

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

## Interfaccia utente

L'interfaccia grafica è progettata per permettere il controllo locale del sistema direttamente dal dispositivo.

Le principali schermate comprendono:

* Home
* Acqua / Serbatoio
* Rete
* Meteo
* SMART / AI
* Programmazione
* Impostazioni

Funzionalità dell'interfaccia:

* navigazione touch;
* gesture swipe;
* pulsanti touch;
* animazioni;
* notifiche e allarmi;
* tastiera virtuale;
* visualizzazione dello stato del sistema.

---

## Controllo remoto

### Web Server

Il Web Server integrato permette di accedere al sistema tramite browser attraverso la rete Wi-Fi.

Le principali funzioni comprendono:

* visualizzazione dello stato del sistema;
* controllo manuale delle pompe;
* programmazione dell'irrigazione;
* configurazione Wi-Fi;
* calibrazione dei sensori;
* gestione dei profili delle piante;
* configurazione SMART;
* configurazione meteorologica;
* gestione dei log;
* aggiornamento firmware OTA;
* configurazione e reset del sistema.

### Telegram

L'integrazione con Telegram permette di:

* ricevere notifiche;
* monitorare il sistema;
* inviare comandi da remoto;
* ricevere informazioni sullo stato del dispositivo.

### ESP-NOW

ESP-NOW viene utilizzato per la comunicazione con dispositivi ESP remoti.

Questa funzionalità permette di estendere il sistema attraverso nodi distribuiti, ad esempio sensori remoti.

---

## Affidabilità e sicurezza

Il firmware integra diversi meccanismi per migliorare l'affidabilità e la sicurezza del sistema.

### Sicurezza

* autenticazione tramite password;
* hashing SHA-256;
* protezione CSRF;
* rate limiting;
* protezione delle operazioni sensibili;
* protezione dell'aggiornamento OTA.

### Affidabilità

* Hardware Watchdog;
* gestione delle condizioni anomale;
* factory reset;
* configurazione persistente;
* monitoraggio dei sensori;
* gestione del livello del serbatoio.

---

## Architettura del sistema

Il firmware utilizza un'architettura modulare che separa le principali responsabilità del sistema:

**Acquisizione → Elaborazione → Decisione → Azionamento → Monitoraggio**

![Architettura del sistema](docs/architecture.svg)

Per una descrizione dettagliata dei moduli software, dei flussi di dati, dell'esecuzione runtime e delle interazioni tra hardware e firmware:

👉 [Visualizza l'architettura completa](docs/ARCHITECTURE.md)

---

## 🖥️ Interfaccia Web

Il sistema dispone di un'interfaccia Web integrata progettata per consentire il monitoraggio, la configurazione e il controllo remoto dell'impianto di irrigazione.

L'interfaccia è accessibile tramite browser attraverso la rete Wi-Fi del dispositivo.

### Dashboard principale

La dashboard principale fornisce una panoramica dello stato del sistema in tempo reale.

Sono visualizzati:

* stato delle pompe;
* stato dei sensori;
* condizioni meteorologiche;
* livello del serbatoio;
* temperatura;
* umidità ambientale;
* umidità del terreno;
* necessità di irrigazione;
* registro degli eventi di sistema.

![Dashboard principale](images/dashboard-01.png)

---

### Monitoraggio dei sensori

La schermata dei sensori permette di visualizzare i valori rilevati dai dispositivi collegati al controller.

![Monitoraggio sensori](images/dashboard-02.png)

---

### Gestione delle piante

La sezione dedicata alle piante permette di utilizzare profili preconfigurati per associare parametri specifici alla gestione dell'irrigazione.

![Gestione piante](images/dashboard-03.png)

---

### Controllo delle pompe

La schermata delle pompe permette di monitorare e gestire gli attuatori dell'impianto.

![Controllo pompe](images/dashboard-04.png)

---

### Monitoraggio del serbatoio

La sezione dedicata al serbatoio permette di visualizzare il livello dell'acqua e lo stato del sistema di approvvigionamento.

![Monitoraggio serbatoio](images/dashboard-05.png)

---

### Programmazione dell'irrigazione

Il sistema permette di configurare programmi di irrigazione basati su orari e condizioni operative.

![Programmazione irrigazione](images/dashboard-06.png)

---

### Configurazione del sistema

La schermata di configurazione centralizza le principali impostazioni del sistema.

Sono disponibili:

* 🔐 Credenziali di accesso Web
* 🧠 Modalità Smart autonoma
* 📶 Configurazione Wi-Fi
* 🔧 Configurazione avanzata
* 🕐 Configurazione manuale dell'ora
* 📱 Integrazione Telegram
* ⛅ Meteo intelligente tramite OpenWeather
* ⬆️ Aggiornamento firmware OTA
* 🔧 Controlli di sistema
* 🏭 Factory Reset

![Configurazione sistema](images/dashboard-07.png)

---

### Identificazione del sistema

Le interfacce riportano l'identificazione del progetto:

**© 2026 TECH3D SYSTEM**
**Progettato da Christian R. Scarparo**

---

## Principali aree tecniche

Il progetto dimostra esperienza pratica nelle seguenti aree:

* Sviluppo di sistemi embedded
* Programmazione C++
* Programmazione di microcontrollori
* Integrazione hardware/software
* Acquisizione dati da sensori
* Controllo di attuatori
* Comunicazione SPI e I²C
* Letture ADC e calibrazione
* Display e interfacce touch
* Networking Wi-Fi
* Comunicazione ESP-NOW
* Web Server embedded
* API e servizi cloud
* Telegram Bot
* Aggiornamenti firmware OTA
* FreeRTOS
* Memoria non volatile
* Sicurezza embedded
* Watchdog e gestione degli errori
* Automazione
* Diagnostica
* Progettazione di sistemi IoT

---

## Principali sfide tecniche

Durante lo sviluppo sono state affrontate diverse problematiche tecniche, tra cui:

* modularizzazione di un firmware inizialmente monolitico;
* integrazione di più sensori e attuatori con ESP32-S3;
* coordinamento tra interfaccia utente, acquisizione dei sensori, controllo dell'irrigazione e servizi di rete;
* gestione della configurazione persistente tramite NVS;
* implementazione dell'aggiornamento firmware OTA;
* comunicazione remota tramite Wi-Fi, Telegram ed ESP-NOW;
* calibrazione dei sensori;
* gestione delle condizioni anomale;
* implementazione di watchdog e meccanismi di sicurezza;
* sviluppo di una logica SMART basata sul feedback dell'umidità del terreno.

---

## Documentazione tecnica

Il repository contiene documentazione separata per facilitare la consultazione del progetto.

### Architettura

👉 [ARCHITECTURE.md](docs/ARCHITECTURE.md)

Documentazione dell'architettura hardware e software, dei moduli firmware, dei flussi di dati, delle attività runtime e delle comunicazioni tra i componenti.

### Documentazione tecnica

👉 [DOCUMENTATION.md](docs/DOCUMENTATION.md)

Documentazione tecnica dettagliata relativa all'hardware, alle funzionalità, alla configurazione e agli aspetti implementativi del sistema.

### Diagramma dell'architettura

👉 [architecture.svg](docs/architecture.svg)

Diagramma visuale dell'architettura complessiva del sistema.

---

## Struttura del repository

```text
esp32-smart-irrigation/
│
├── README.md
│
├── docs/
│   ├── ARCHITECTURE.md
│   ├── DOCUMENTATION.md
│   └── architecture.svg
│
└── images/
    ├── dashboard-01.png
    ├── dashboard-02.png
    ├── dashboard-03.png
    ├── dashboard-04.png
    ├── dashboard-05.png
    ├── dashboard-06.png
    └── dashboard-07.png
```

Il repository non contiene il firmware sorgente. La struttura è stata organizzata come **portfolio tecnico**, con l'obiettivo di presentare il progetto, la sua architettura e le competenze tecniche utilizzate nello sviluppo.

---

## Stato del progetto

**Piattaforma:** ESP32-S3
**Display:** ST7789 320×240
**Touch:** XPT2046
**RTC:** DS3231
**Ambiente di sviluppo:** Arduino IDE
**Firmware:** C++
**Architettura:** Firmware modulare
**Comunicazione:** Wi-Fi / ESP-NOW / Telegram
**Tipologia:** Embedded / IoT / Automazione

---

## Obiettivo del progetto

Il progetto è stato sviluppato come applicazione pratica di **sistemi embedded, automazione e IoT**, combinando elettronica, firmware e comunicazione di rete in un'unica piattaforma.

L'obiettivo è realizzare un sistema di irrigazione capace di:

* acquisire dati dal terreno e dall'ambiente;
* elaborare le informazioni ricevute;
* controllare autonomamente gli attuatori;
* gestire cicli di irrigazione automatici e manuali;
* adattare la durata dell'irrigazione;
* utilizzare informazioni meteorologiche;
* comunicare con dispositivi remoti;
* offrire interfacce locali e remote;
* mantenere configurazioni e parametri nel tempo;
* aggiornare il firmware senza collegamento fisico al dispositivo.

---

## Portfolio tecnico

Questo repository rappresenta una sintesi delle competenze applicate nello sviluppo del progetto:

**Elettronica → Firmware → Automazione → IoT → Networking → Interfaccia utente → Integrazione hardware/software**

Il codice sorgente non viene pubblicato. La documentazione è fornita per illustrare l'architettura, le funzionalità e le principali soluzioni tecniche adottate durante lo sviluppo.
