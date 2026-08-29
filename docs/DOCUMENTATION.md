# Documentazione Tecnica — ESP32 Smart Irrigation

## 1. Panoramica del progetto

**ESP32 Smart Irrigation** è un sistema embedded/IoT per il controllo e la gestione automatizzata dell'irrigazione, sviluppato attorno a un microcontrollore **ESP32-S3**.

Il sistema integra acquisizione dati da sensori, controllo degli attuatori, interfaccia grafica touch, connettività di rete, comunicazione remota e logiche adattive per la gestione dell'irrigazione.

L'obiettivo del progetto è realizzare un sistema in grado di:

* monitorare l'umidità del terreno;
* controllare automaticamente le pompe di irrigazione;
* monitorare il livello dell'acqua nel serbatoio;
* acquisire temperatura e umidità ambientale;
* gestire programmi di irrigazione;
* fornire un'interfaccia locale tramite display touch;
* consentire il controllo remoto tramite Web Server e Telegram;
* comunicare con dispositivi remoti tramite ESP-NOW;
* utilizzare dati meteorologici per supportare le decisioni di irrigazione;
* aggiornare il firmware tramite OTA;
* adattare la durata dei cicli di irrigazione attraverso la logica SMART.

> **Nota:** il codice sorgente completo non è incluso nel repository. Il progetto è pubblicato come **portfolio tecnico**, con l'obiettivo di documentare l'architettura, le funzionalità, l'integrazione hardware/software e le principali soluzioni tecniche adottate.

---

# 2. Architettura del sistema

L'architettura è organizzata in tre livelli principali:

1. **Hardware e periferiche**
2. **Firmware embedded ESP32-S3**
3. **Servizi e dispositivi esterni**

L'ESP32-S3 rappresenta l'unità centrale del sistema e coordina l'acquisizione dei dati, l'elaborazione della logica di controllo, l'interfaccia utente e le comunicazioni.

![System Architecture](architecture.svg)

---

## 2.1 Architettura di deployment

### Hardware e periferiche

Il sistema comprende:

* display TFT ST7789 320×240;
* touch resistivo XPT2046;
* due sensori di umidità del terreno;
* sensore DHT11;
* sensore ultrasonico HC-SR04;
* RTC DS3231;
* relè per le pompe;
* relè per il pozzo;
* buzzer;
* LED di stato;
* pulsante dedicato al reset della configurazione Wi-Fi.

### ESP32-S3

Il microcontrollore esegue il firmware principale e gestisce:

* acquisizione dei sensori;
* controllo degli attuatori;
* interfaccia grafica;
* gestione della rete;
* Web Server embedded;
* comunicazione Telegram;
* comunicazione ESP-NOW;
* logica SMART;
* gestione della configurazione;
* sicurezza;
* watchdog.

### Servizi e dispositivi esterni

Il sistema può comunicare con:

* browser Web;
* Telegram;
* OpenWeatherMap;
* dispositivi ESP-NOW remoti;
* rete Wi-Fi e Internet.

La configurazione persistente viene memorizzata nella **NVS (Non-Volatile Storage)** dell'ESP32.

---

# 3. Hardware e pinout

## 3.1 Tabella dei collegamenti

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

I relè utilizzano una logica **active-low**:

```text
RELAY_ON  = LOW
RELAY_OFF = HIGH
```

L'uscita `ECHO` dell'HC-SR04 deve essere adattata al livello logico compatibile con l'ESP32-S3. Quando necessario, è quindi richiesto un opportuno partitore di tensione.

---

# 4. Specifiche principali

| Parametro              | Valore                |
| ---------------------- | --------------------- |
| Microcontrollore       | ESP32-S3 Dual-Core    |
| Display                | ST7789                |
| Risoluzione            | 320×240               |
| Touch                  | XPT2046 resistivo     |
| Sensori terreno        | 2 × ADC               |
| Temperatura / umidità  | DHT11                 |
| Livello serbatoio      | HC-SR04               |
| RTC                    | DS3231                |
| Comunicazione locale   | Wi-Fi / ESP-NOW       |
| Controllo remoto       | Web Server / Telegram |
| Servizio meteorologico | OpenWeatherMap        |
| Storage                | NVS / Preferences     |
| Aggiornamento firmware | OTA                   |
| Watchdog               | 30 secondi            |
| Profili piante         | Fino a 100            |

---

# 5. Architettura software

Il firmware è stato inizialmente sviluppato come uno sketch monolitico di grandi dimensioni.

Con l'evoluzione del progetto, il software è stato progressivamente riorganizzato in moduli funzionali separati, con l'obiettivo di migliorare:

* leggibilità;
* manutenzione;
* separazione delle responsabilità;
* riutilizzabilità;
* debugging;
* gestione delle dipendenze;
* possibilità di estensione futura.

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
| `cloud.cpp`      | OpenWeatherMap e configurazione dei servizi cloud |
| `espnow.cpp`     | Comunicazione ESP-NOW                             |
| `plants.cpp`     | Gestione dei profili delle piante                 |
| `system.cpp`     | Watchdog, sicurezza, log e funzioni di sistema    |

---

# 6. Organizzazione dei moduli

## `app.h`

`app.h` rappresenta l'header comune dell'architettura software.

Contiene:

* librerie;
* costanti;
* macro;
* strutture dati;
* dichiarazioni `extern`;
* prototipi delle funzioni.

L'utilizzo di un header comune consente ai diversi moduli di condividere in modo controllato tipi, variabili e interfacce.

---

## `globals.cpp`

Contiene le definizioni effettive delle variabili globali dichiarate tramite `extern` in `app.h`.

Questo approccio evita definizioni multiple durante la fase di linking e centralizza lo stato globale del sistema.

---

## `display.cpp`

Gestisce l'interfaccia grafica locale del dispositivo.

Responsabilità principali:

* rendering delle schermate;
* gestione del touch;
* navigazione;
* gesture swipe;
* pulsanti;
* animazioni;
* schermate di configurazione;
* messaggi di errore;
* schermate di allarme;
* tastiera virtuale.

---

## `hardware.cpp`

Gestisce l'interazione diretta con le periferiche hardware.

Comprende:

* relè;
* pompe;
* buzzer;
* LED;
* DHT11;
* sensori di umidità;
* sensore ultrasonico;
* letture GPIO;
* acquisizione dei segnali analogici.

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

Il modulo supporta inoltre le funzionalità temporali utilizzate dalle altre parti del firmware.

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

* monitoraggio del sistema;
* configurazione;
* controllo manuale;
* gestione dell'irrigazione;
* configurazione Wi-Fi;
* calibrazione;
* gestione delle piante;
* configurazione SMART;
* gestione dei log;
* aggiornamento OTA;
* reset del sistema.

---

## `cloud.cpp`

Gestisce l'integrazione con i servizi esterni.

In particolare:

* OpenWeatherMap;
* configurazione dei servizi cloud;
* configurazione meteorologica;
* gestione delle informazioni meteorologiche;
* gestione delle configurazioni persistenti correlate.

Le credenziali e i parametri sensibili non vengono pubblicati nel repository.

---

## `espnow.cpp`

Implementa la comunicazione tramite **ESP-NOW**.

Il modulo permette di ricevere dati da dispositivi ESP remoti senza richiedere una connessione TCP/IP tradizionale tra i dispositivi.

Questa architettura permette di estendere il sistema con nodi distribuiti.

---

## `plants.cpp`

Gestisce i profili delle piante.

Il sistema supporta fino a:

```text
MAX_PLANTS = 100
```

I profili possono essere utilizzati per associare parametri specifici alla gestione dell'irrigazione.

---

## `system.cpp`

Contiene le principali funzionalità di sistema:

* watchdog;
* sicurezza;
* rate limiting;
* protezione CSRF;
* logging;
* factory reset;
* inizializzazione del sistema;
* gestione delle attività;
* funzioni di supporto.

---

# 7. Interfaccia utente

L'interfaccia grafica è sviluppata per il display **ST7789 320×240** con touch resistivo **XPT2046**.

Le principali schermate comprendono:

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

### Gestione delle gesture

Parametri principali:

```text
SWIPE_THRESHOLD = 60 px
SWIPE_MAX_MS    = 800 ms
```

---

# 8. Sistema di irrigazione

Il sistema supporta tre modalità operative principali:

* **Automatica**
* **Manuale**
* **SMART**

## 8.1 Modalità automatica

La modalità automatica utilizza programmi configurabili dall'utente.

La struttura principale del programma comprende:

```text
Programma
├── Inizio
├── Fine
└── Attivo
```

Il firmware verifica periodicamente le condizioni configurate e determina quando avviare un ciclo di irrigazione.

---

## 8.2 Modalità manuale

La modalità manuale permette all'utente di comandare direttamente le pompe attraverso l'interfaccia locale o il Web Server.

Questa modalità è utile principalmente per:

* test degli attuatori;
* manutenzione;
* verifica dell'impianto;
* attivazione manuale dell'irrigazione.

---

# 9. Modalità SMART

La modalità **SMART** rappresenta una delle principali caratteristiche tecniche del progetto.

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
            ↓
Variazione dell'umidità
```

Il sistema può quindi stimare il rapporto tra il tempo di funzionamento della pompa e la variazione dell'umidità del terreno.

Queste informazioni vengono utilizzate per adattare la durata dei cicli successivi.

### Obiettivi della logica SMART

La logica SMART è progettata per:

* ridurre irrigazioni non necessarie;
* adattare il funzionamento alle condizioni reali del terreno;
* migliorare la gestione dell'acqua;
* utilizzare i dati dei cicli precedenti per modificare il comportamento del sistema.

> **Nota:** la modalità SMART è una logica adattiva/euristica implementata direttamente nel firmware. Non utilizza un modello di machine learning addestrato tramite framework esterni.

---

# 10. Gestione meteorologica

Il sistema può utilizzare **OpenWeatherMap** per acquisire informazioni meteorologiche.

Uno degli utilizzi principali è la valutazione della previsione di pioggia.

Il flusso logico è:

```text
OpenWeatherMap
       ↓
Dati meteorologici
       ↓
Previsione di pioggia
       ↓
Logica di irrigazione
       ↓
Irrigazione consentita?
       │
       ├── Sì → Avvio ciclo
       │
       └── No → Sospensione
```

In presenza delle condizioni configurate, la previsione di pioggia può quindi impedire l'avvio di un ciclo automatico.

---

# 11. Gestione dei sensori

## 11.1 Sensori di umidità del terreno

Sono presenti due ingressi ADC dedicati:

```text
SOIL_PIN1 → GPIO 1
SOIL_PIN2 → GPIO 2
```

I sensori possono essere calibrati utilizzando valori di riferimento per terreno asciutto e terreno umido.

I parametri di calibrazione vengono salvati nella memoria non volatile.

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

La distanza misurata viene utilizzata dal firmware per determinare lo stato del serbatoio e gestire eventuali condizioni di allarme.

---

## 11.4 RTC DS3231

Il DS3231 fornisce una sorgente temporale locale tramite I²C.

```text
SDA → GPIO 8
SCL → GPIO 9
```

Quando la connessione Wi-Fi è disponibile, il sistema può inoltre sincronizzare l'orario tramite NTP.

---

# 12. Web Server

Il firmware include un Web Server embedded accessibile tramite browser attraverso la rete Wi-Fi.

Le principali funzionalità includono:

* monitoraggio dello stato;
* controllo manuale delle pompe;
* configurazione dell'irrigazione;
* gestione della programmazione;
* configurazione Wi-Fi;
* calibrazione dei sensori;
* gestione dei profili delle piante;
* configurazione SMART;
* configurazione meteorologica;
* gestione dei log;
* aggiornamento OTA;
* reset e configurazione del sistema.

## 12.1 Endpoint principali

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
| `/sospensione`   | Sospensione irrigazione      |
| `/setwifi`       | Configurazione Wi-Fi         |
| `/scanwifi`      | Scansione reti               |
| `/resetwifi`     | Reset Wi-Fi                  |
| `/setsmart`      | Configurazione SMART         |
| `/synctime`      | Sincronizzazione orario      |
| `/setmanualtime` | Impostazione manuale         |
| `/getSoilCal`    | Lettura calibrazione terreno |
| `/setSoilCal`    | Salvataggio calibrazione     |
| `/getPlants`     | Lettura profili piante       |
| `/savePlant`     | Salvataggio profilo          |
| `/deletePlant`   | Eliminazione profilo         |
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

L'integrazione Telegram permette di interagire con il sistema anche da remoto.

Le funzionalità comprendono:

* notifiche degli eventi;
* notifiche relative alle pompe;
* notifiche relative al serbatoio;
* comandi remoti;
* comunicazione con il dispositivo;
* gestione della configurazione del bot.

Le informazioni necessarie per il funzionamento del servizio vengono gestite nella configurazione del sistema.

Le credenziali non vengono pubblicate nel repository.

---

# 14. ESP-NOW

ESP-NOW viene utilizzato per la comunicazione diretta con dispositivi ESP remoti.

Possibili applicazioni:

* sensori di umidità remoti;
* nodi ambientali;
* dispositivi ESP aggiuntivi;
* acquisizione distribuita dei dati.

Questa tecnologia consente di estendere il sistema con dispositivi periferici senza richiedere necessariamente una connessione IP tra i nodi.

---

# 15. Memoria persistente

Il sistema utilizza **NVS (Non-Volatile Storage)** attraverso la libreria `Preferences`.

La memoria non volatile viene utilizzata per conservare le configurazioni che devono rimanere disponibili anche dopo un riavvio o una perdita di alimentazione.

Tra i dati memorizzabili rientrano:

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

Il firmware integra diversi meccanismi di protezione.

## 16.1 Autenticazione

L'accesso alle funzionalità sensibili può essere protetto tramite password.

La password viene gestita mediante hashing **SHA-256**, evitando di memorizzarla direttamente in formato leggibile.

---

## 16.2 Protezione CSRF

Gli endpoint sensibili possono utilizzare token **CSRF (Cross-Site Request Forgery)** per impedire richieste non autorizzate provenienti da contesti esterni.

---

## 16.3 Rate Limiting

Il firmware implementa meccanismi di limitazione delle richieste per ridurre tentativi ripetuti di accesso o utilizzi anomali degli endpoint.

---

## 16.4 Factory Reset

Il sistema dispone di una procedura di ripristino alle impostazioni di fabbrica.

Il factory reset permette di cancellare le configurazioni persistenti e riportare il sistema allo stato iniziale.

---

# 17. Watchdog

Per aumentare l'affidabilità del sistema viene utilizzato un **Task Watchdog**.

Configurazione principale:

```text
WDT_TIMEOUT = 30 secondi
```

Il watchdog consente di rilevare condizioni in cui le attività monitorate non vengono eseguite correttamente.

In caso di blocco non recuperabile, il meccanismo può provocare il riavvio del microcontrollore.

---

# 18. OTA — Over The Air Update

Il firmware supporta l'aggiornamento tramite rete Wi-Fi.

Il processo può essere rappresentato come:

```text
Browser
   ↓
Web Server
   ↓
Upload firmware
   ↓
OTA Update
   ↓
Scrittura firmware
   ↓
Riavvio ESP32-S3
```

Questa funzionalità consente di aggiornare il firmware senza collegare fisicamente l'ESP32-S3 al computer.

L'aggiornamento OTA richiede una configurazione di partizionamento compatibile con il meccanismo OTA dell'ESP32.

---

# 19. FreeRTOS e gestione delle attività

L'ESP32-S3 utilizza **FreeRTOS** come sistema operativo real-time integrato nel framework ESP32.

Le principali attività del firmware comprendono:

```text
┌──────────────────────────────┐
│           ESP32-S3           │
├──────────────────────────────┤
│ UI / Touch                   │
│                              │
│ Acquisizione sensori         │
│                              │
│ Controllo irrigazione        │
│                              │
│ Web Server / Networking      │
│                              │
│ Comunicazione remota         │
│                              │
│ Watchdog                     │
└──────────────────────────────┘
```

La gestione concorrente delle attività permette di mantenere separate le principali funzioni del sistema e di ridurre il rischio che una singola operazione blocchi l'intero firmware.

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
Dati meteorologici
      │
      ▼
Logica automatica / SMART
      │
      ▼
Irrigazione consentita?
      │
      ├── Sì → Attivazione
      │
      └── No → Sospensione
```

---

# 21. Stati operativi

Il sistema gestisce tre modalità operative principali:

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

A queste modalità si aggiungono condizioni operative e di sicurezza, come:

* serbatoio vuoto;
* sospensione manuale;
* condizioni non idonee all'irrigazione;
* errore di sistema;
* perdita di condizioni necessarie all'attivazione delle pompe.

---

# 22. Allarmi e protezioni operative

Il sistema dispone di schermate dedicate alla visualizzazione degli eventi anomali e dello stato operativo.

Tra gli stati gestiti rientrano:

* irrigazione in corso;
* attivazione manuale;
* attivazione programmata;
* serbatoio vuoto;
* terreno secco;
* aggiornamento OTA;
* reset del sistema;
* condizioni anomale dei sensori.

Quando configurato, il sistema può inoltre inviare notifiche tramite Telegram.

---

# 23. Librerie e tecnologie

Il progetto utilizza o integra le seguenti librerie e tecnologie:

| Libreria / tecnologia | Utilizzo                            |
| --------------------- | ----------------------------------- |
| Arduino GFX Library   | Driver display ST7789               |
| XPT2046_Touchscreen   | Touch resistivo                     |
| SPI                   | Comunicazione SPI                   |
| Wire                  | Comunicazione I²C                   |
| ArduinoJson           | Parsing e generazione JSON          |
| DHT                   | Gestione DHT11                      |
| HTTPClient            | Comunicazioni HTTP                  |
| WiFi                  | Connettività Wi-Fi                  |
| WiFiClientSecure      | Comunicazioni HTTPS                 |
| NTPClient             | Sincronizzazione NTP                |
| Preferences           | Storage NVS                         |
| PubSubClient          | Supporto MQTT                       |
| UniversalTelegramBot  | Comunicazione Telegram              |
| WebServer             | Web Server embedded                 |
| Update                | Aggiornamento OTA                   |
| WiFiManager           | Configurazione Wi-Fi                |
| RTClib                | Gestione RTC DS3231                 |
| ESP-NOW               | Comunicazione wireless peer-to-peer |
| esp_task_wdt          | Task Watchdog                       |
| mbedTLS / SHA-256     | Hashing delle password              |

La libreria **Espalexa** è stata rimossa dal progetto perché non utilizzata nella versione attuale.

---

# 24. Struttura del repository

La struttura concettuale del repository è:

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
    └── ...
```

Il repository pubblico contiene la documentazione tecnica e gli elementi necessari alla presentazione del progetto come portfolio.

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

La ricostruzione del firmware originale richiede una configurazione compatibile con il core ESP32 e con le librerie indicate nella documentazione.

---

# 26. Modularizzazione del firmware

Il firmware originale era costituito da uno sketch monolitico di grandi dimensioni.

La successiva modularizzazione ha separato le principali funzionalità in componenti distinti.

La struttura concettuale risultante è:

```text
                    app.h
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
       UI           Control       Network
        │             │             │
 display.cpp    irrigation.cpp    web.cpp
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
Acquisizione dei sensori
       ↓
Controllo degli attuatori
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
Modularizzazione del firmware
```

Questo processo ha permesso di evolvere il sistema da un controller hardware iniziale a una piattaforma embedded/IoT più completa e strutturata.

---

# 28. Competenze tecniche dimostrate

Il progetto integra diverse aree dell'elettronica, della programmazione embedded e dell'automazione.

## Embedded Systems

* Programmazione C/C++;
* ESP32-S3;
* FreeRTOS;
* gestione della memoria;
* gestione delle attività;
* watchdog;
* gestione delle periferiche.

## Elettronica

* GPIO;
* ADC;
* relè;
* sensori analogici;
* sensori digitali;
* SPI;
* I²C;
* integrazione hardware;
* calibrazione dei sensori.

## IoT e Networking

* Wi-Fi;
* Web Server embedded;
* endpoint HTTP;
* Telegram;
* ESP-NOW;
* OpenWeatherMap;
* OTA.

## Software Architecture

* modularizzazione del firmware;
* separazione delle responsabilità;
* gestione delle dipendenze;
* strutture dati;
* configurazione persistente;
* gestione dello stato del sistema.

## Sicurezza

* SHA-256;
* autenticazione;
* CSRF protection;
* rate limiting;
* protezione delle funzionalità OTA.

## Automazione

* controllo automatico;
* programmazione;
* feedback dei sensori;
* logica adattiva;
* gestione delle condizioni ambientali.

---

# 29. Possibili sviluppi futuri

L'architettura attuale permette ulteriori evoluzioni, tra cui:

* dashboard Web più avanzata;
* gestione di un numero maggiore di zone di irrigazione;
* sensori remoti aggiuntivi;
* storico dei dati;
* grafici relativi ai consumi d'acqua;
* integrazione con ulteriori piattaforme IoT;
* algoritmi predittivi più avanzati;
* gestione energetica;
* alimentazione tramite pannello solare;
* espansione della rete ESP-NOW;
* database remoto per lo storico dei dati.

---

# 30. Conclusioni

**ESP32 Smart Irrigation** è un progetto embedded/IoT completo che integra elettronica, firmware, automazione e comunicazione di rete.

Il progetto dimostra la capacità di sviluppare un sistema partendo dall'integrazione hardware fino alla gestione software di alto livello, includendo:

* acquisizione dei dati;
* controllo degli attuatori;
* interfaccia utente;
* networking;
* controllo remoto;
* automazione;
* logica adattiva;
* sicurezza;
* aggiornamento OTA;
* gestione real-time;
* persistenza della configurazione.

L'architettura modulare consente inoltre di mantenere il firmware organizzato, facilitando la manutenzione, il debugging e l'introduzione di nuove funzionalità.

---

# Stato del progetto

**Piattaforma:** ESP32-S3
**Display:** ST7789 320×240
**Touch:** XPT2046
**Firmware:** C++ / Arduino
**Real-Time:** FreeRTOS
**Architettura:** Firmware modulare
**Tipologia:** Embedded / IoT / Automazione
**Repository:** Portfolio tecnico
