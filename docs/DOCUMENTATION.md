# Documentazione tecnica — ESP32 Smart Irrigation

## Irrigatore ESP32-S3 — ST7789 V3.1

Firmware di controllo per un sistema di irrigazione intelligente basato su **ESP32-S3**, con display TFT touch **ST7789 320×240**, sensori del terreno e ambientali, controllo di pompe e relè, connettività IoT e logica di irrigazione adattiva.

Il sistema integra interfaccia locale, Web Server embedded, controllo remoto tramite Telegram, comunicazione ESP-NOW, sincronizzazione dell'orologio, aggiornamenti OTA e gestione persistente della configurazione.

> **Nota:** il codice sorgente non è incluso nel repository. Questa documentazione descrive l'architettura, le funzionalità e le principali scelte tecniche del progetto a scopo di portfolio.

---

# 1. Panoramica del sistema

Il dispositivo è progettato come **controller di irrigazione IoT autonomo**, capace di acquisire informazioni dal terreno e dall'ambiente e di utilizzare tali dati per gestire automaticamente l'irrigazione.

Le principali funzionalità comprendono:

* Display TFT ST7789 con touch resistivo XPT2046.
* Interfaccia grafica con navigazione touch.
* Controllo di 2 pompe di irrigazione.
* Controllo di 1 relè per il pozzo.
* Monitoraggio di 2 sensori di umidità del terreno.
* Monitoraggio del livello del serbatoio tramite HC-SR04.
* Acquisizione di temperatura e umidità tramite DHT11.
* RTC DS3231 con sincronizzazione NTP.
* Programmazione automatica dei cicli di irrigazione.
* Modalità SMART con logica adattiva.
* Integrazione con OpenWeatherMap.
* Web Server embedded per configurazione e controllo.
* Controllo remoto tramite Telegram.
* Comunicazione ESP-NOW con dispositivi remoti.
* Aggiornamento firmware tramite OTA.
* Memorizzazione persistente dei parametri tramite NVS.
* Watchdog hardware per la gestione dei blocchi del sistema.
* Meccanismi di autenticazione e protezione degli endpoint.

---

# 2. Hardware e pinout

## 2.1 Controller

**Microcontrollore:** ESP32-S3

Il microcontrollore gestisce l'acquisizione dei sensori, il controllo degli attuatori, l'interfaccia utente, le comunicazioni di rete e la logica di irrigazione.

## 2.2 Mappa dei GPIO

| Segnale          | GPIO | Funzione                              |
| ---------------- | ---: | ------------------------------------- |
| `POMPA1`         |    4 | Relè pompa 1                          |
| `POMPA2`         |    5 | Relè pompa 2                          |
| `RELE_POZZO`     |    6 | Relè del pozzo                        |
| `BUZZER_PIN`     |    7 | Buzzer                                |
| `DHTPIN`         |   15 | DHT11                                 |
| `TRIG_PIN`       |   16 | Trigger HC-SR04                       |
| `ECHO_PIN`       |   17 | Echo HC-SR04                          |
| `RESET_WIFI_PIN` |   18 | Pulsante reset Wi-Fi                  |
| `I2C_SDA`        |    8 | I²C SDA — DS3231                      |
| `I2C_SCL`        |    9 | I²C SCL — DS3231                      |
| `TFT_CS`         |   10 | Chip Select ST7789                    |
| `TFT_MOSI`       |   11 | SPI MOSI                              |
| `TFT_SCK`        |   12 | SPI Clock                             |
| `TFT_MISO`       |   13 | SPI MISO                              |
| `TFT_DC`         |   14 | Data / Command ST7789                 |
| `SOIL_PIN1`      |    1 | Sensore umidità terreno 1             |
| `SOIL_PIN2`      |    2 | Sensore umidità terreno 2             |
| `TFT_RST`        |   40 | Reset ST7789                          |
| `TFT_LED`        |   20 | Backlight display                     |
| `T_CS`           |   38 | Chip Select XPT2046                   |
| `T_IRQ`          |   39 | Interrupt touch                       |
| `LED1`           |   47 | LED di stato 1                        |
| `LED2`           |    — | LED di stato 2 — GPIO non specificato |

### Parametri principali

```text
RELAY_ON = LOW
RELAY_OFF = HIGH

WDT_TIMEOUT = 30 s

SCR_W = 320
SCR_H = 240

MAX_PLANTS = 100
MAX_TG_MSGS = 8

SWIPE_THRESHOLD = 60 px
SWIPE_MAX_MS = 800 ms
```

> **Nota hardware:** il segnale `ECHO` dell'HC-SR04 può raggiungere livelli di 5 V. Deve essere utilizzato un opportuno adattamento di livello/divisore di tensione prima del collegamento all'ESP32-S3.

---

# 3. Librerie e tecnologie

Il firmware utilizza le seguenti librerie e componenti software:

| Libreria / componente  | Utilizzo                              |
| ---------------------- | ------------------------------------- |
| `Arduino_GFX_Library`  | Driver display ST7789                 |
| `XPT2046_Touchscreen`  | Touch resistivo                       |
| `SPI`                  | Comunicazione SPI                     |
| `Wire`                 | Comunicazione I²C                     |
| `ArduinoJson`          | Parsing e gestione dati JSON          |
| `DHT`                  | Sensore DHT11                         |
| `HTTPClient`           | Comunicazioni HTTP                    |
| `WiFi`                 | Connettività Wi-Fi                    |
| `WiFiClientSecure`     | Comunicazioni HTTPS                   |
| `NTPClient`            | Sincronizzazione dell'orario          |
| `WiFiUdp`              | Comunicazione UDP per NTP             |
| `time.h`               | Gestione dell'orologio di sistema     |
| `Preferences`          | Memorizzazione NVS                    |
| `PubSubClient`         | MQTT, utilizzo opzionale              |
| `UniversalTelegramBot` | Telegram Bot                          |
| `WebServer`            | Web Server embedded                   |
| `Update`               | Aggiornamento OTA                     |
| `esp_ota_ops`          | Gestione partizioni OTA               |
| `WiFiManager`          | Configurazione Wi-Fi / captive portal |
| `RTClib`               | RTC DS3231                            |
| `esp_now`              | Comunicazione ESP-NOW                 |
| `esp_task_wdt`         | Task Watchdog                         |
| `nvs_flash`            | Gestione memoria NVS                  |
| `mbedtls/sha256`       | Hash SHA-256                          |

La libreria **Espalexa** è stata rimossa perché l'integrazione Alexa non viene utilizzata nella versione attuale.

---

# 4. Architettura software

Il firmware è stato inizialmente sviluppato come uno sketch monolitico di grandi dimensioni.

Per migliorare manutenzione, leggibilità e separazione delle responsabilità, il codice è stato successivamente organizzato in moduli C++ distinti.

Lo sketch principale si trova nella directory:

```text
main/
```

## 4.1 Struttura dei moduli

| File             | Responsabilità                                             |
| ---------------- | ---------------------------------------------------------- |
| `app.h`          | Include, define, tipi globali, `extern` e prototipi        |
| `main.ino`       | `setup()` e `loop()`                                       |
| `globals.cpp`    | Definizione delle variabili globali                        |
| `display.cpp`    | Display, touch, UI, animazioni e tastiera                  |
| `hardware.cpp`   | I/O hardware, DHT, buzzer, LED e ultrasonico               |
| `irrigation.cpp` | Logica irrigazione, pompe, sensori, programmazione e SMART |
| `time.cpp`       | RTC, NTP e gestione dell'orario                            |
| `wifi.cpp`       | Wi-Fi e WiFiManager                                        |
| `web.cpp`        | Web Server ed endpoint HTTP                                |
| `cloud.cpp`      | OpenWeatherMap e configurazione cloud                      |
| `espnow.cpp`     | Comunicazione ESP-NOW                                      |
| `plants.cpp`     | Profili delle piante                                       |
| `system.cpp`     | Watchdog, sicurezza, log, reset e funzioni di sistema      |

Il modulo `telegram.cpp` non viene utilizzato nella versione attuale. Le funzioni Telegram sono state integrate nei moduli `time.cpp`, `cloud.cpp` e `web.cpp` secondo la loro responsabilità.

---

# 5. Tipi e strutture globali

Il firmware utilizza diverse strutture dati condivise tra i moduli.

## `Programma`

Rappresenta una finestra temporale di irrigazione.

```text
Programma {
    int inizio;
    int fine;
    bool attivo;
}
```

## `PlantProfile`

Contiene i parametri relativi al profilo di una pianta.

Il sistema supporta fino a:

```text
MAX_PLANTS = 100
```

## `SystemTime`

Gestisce lo stato dell'orologio del sistema, integrando RTC e sincronizzazione NTP.

## `WiFiState`

Contiene lo stato e i parametri relativi alla connessione Wi-Fi.

## `CloudConfig`

Gestisce le configurazioni relative ai servizi cloud:

```text
tgToken
tgChat
tgChat2
tgChat3
tgEnabled

owmKey
owmCity
owmEnabled
```

Questi parametri vengono memorizzati nella NVS.

## `LoginGuard`

Gestisce autenticazione, password, rate limiting e protezione delle richieste.

## `SwipeState`

Gestisce il rilevamento delle gesture swipe dell'interfaccia touch.

---

# 6. Interfaccia utente

L'interfaccia grafica viene gestita principalmente da `display.cpp`.

Il sistema utilizza:

* Display ST7789 320×240.
* Touch resistivo XPT2046.
* Navigazione tramite pulsanti touch.
* Gesture swipe.
* Tastiera virtuale.
* Schermate di stato.
* Animazioni.
* Messaggi di allarme.

## 6.1 Schermate principali

Le principali schermate del sistema sono:

* `Main`
* `Water`
* `Network`
* `Meteo`
* `AI`
* `Schedule`
* `Settings`

La schermata attiva viene gestita tramite l'enum `DisplayScreen`.

## 6.2 Sistema di navigazione

La navigazione touch supporta gesture swipe.

Parametri:

```text
SWIPE_THRESHOLD = 60 px
SWIPE_MAX_MS = 800 ms
```

Il sistema include inoltre schermate dedicate agli eventi di allarme.

Tra queste:

```text
drawAlertIrrigazione
drawAlertManuale
drawAlertProgramma
drawAlertSerbatoioVuoto
drawAlertTerrenoSecco
drawAlertOTA
drawAlertResetting
```

---

# 7. Gestione dell'irrigazione

La logica principale dell'irrigazione è implementata in:

```text
irrigation.cpp
```

Il modulo gestisce:

* pompe;
* relè del pozzo;
* sensori di umidità;
* livello del serbatoio;
* programmazione;
* modalità automatica;
* modalità manuale;
* modalità SMART;
* calibrazione dei sensori;
* condizioni di sicurezza.

## 7.1 Controllo delle pompe

Il sistema dispone di due pompe:

```text
POMPA1
POMPA2
```

e di un relè per il pozzo:

```text
RELE_POZZO
```

La logica dei relè utilizza:

```text
RELAY_ON  = LOW
RELAY_OFF = HIGH
```

---

# 8. Sensori

## 8.1 Umidità del terreno

Il sistema utilizza due ingressi ADC:

```text
SOIL_PIN1 = GPIO 1
SOIL_PIN2 = GPIO 2
```

I valori possono essere calibrati e memorizzati nella NVS.

La calibrazione viene gestita attraverso:

```text
loadSoilCalibration()
saveSoilCalibration()
```

## 8.2 DHT11

Il DHT11 viene utilizzato per rilevare:

* temperatura ambientale;
* umidità relativa.

Pin:

```text
DHTPIN = GPIO 15
```

## 8.3 HC-SR04

Il sensore ultrasonico viene utilizzato per determinare il livello del serbatoio.

```text
TRIG_PIN = GPIO 16
ECHO_PIN = GPIO 17
```

Il sistema utilizza questa informazione per evitare condizioni di irrigazione con serbatoio vuoto o insufficiente.

## 8.4 RTC DS3231

Il DS3231 fornisce un riferimento temporale locale tramite bus I²C.

```text
I2C_SDA = GPIO 8
I2C_SCL = GPIO 9
```

L'orologio può inoltre essere sincronizzato tramite NTP.

---

# 9. Modalità SMART

La modalità **SMART** rappresenta uno degli elementi principali del progetto.

L'obiettivo è rendere la durata dell'irrigazione adattiva invece di utilizzare esclusivamente tempi fissi.

Il sistema analizza la relazione tra:

```text
Tempo di funzionamento della pompa
            ↓
Umidità prima dell'irrigazione
            ↓
Ciclo di irrigazione
            ↓
Umidità dopo l'irrigazione
```

La funzione:

```text
updateLearning(...)
```

utilizza i dati raccolti per stimare il guadagno di umidità ottenuto in relazione al tempo di funzionamento della pompa.

La configurazione del modello SMART viene mantenuta tramite:

```text
loadSmartConfig()
saveSmartConfig()
```

Questo permette al sistema di conservare i parametri appresi anche dopo un riavvio.

---

# 10. Integrazione meteorologica

Il modulo:

```text
cloud.cpp
```

gestisce l'integrazione con **OpenWeatherMap**.

La configurazione comprende:

```text
owmKey
owmCity
owmEnabled
```

Il sistema può utilizzare le informazioni meteorologiche per determinare se è prevista pioggia.

Quando viene rilevata una previsione compatibile con la sospensione dell'irrigazione, la logica SMART può evitare o sospendere il ciclo automatico.

La variabile:

```text
pioggiaPrevista
```

viene utilizzata dalla logica di controllo.

È inoltre disponibile:

```text
weatherForceUpdate
```

per forzare l'aggiornamento delle informazioni meteorologiche.

---

# 11. Connettività Wi-Fi

La gestione Wi-Fi è implementata tramite:

```text
wifi.cpp
```

e utilizza **WiFiManager** per semplificare la configurazione iniziale.

## Funzioni principali

* Configurazione della rete Wi-Fi.
* Captive portal.
* Scansione delle reti disponibili.
* Monitoraggio dello stato della connessione.
* Reset delle credenziali.
* Riconnessione.

Il pulsante:

```text
RESET_WIFI_PIN = GPIO 18
```

permette di ripristinare la configurazione Wi-Fi e riavviare il processo di configurazione.

---

# 12. Web Server embedded

Il Web Server è implementato in:

```text
web.cpp
```

e permette di accedere al sistema tramite browser.

Le funzionalità comprendono:

* monitoraggio;
* controllo manuale;
* configurazione;
* programmazione;
* gestione Wi-Fi;
* calibrazione;
* gestione delle piante;
* configurazione SMART;
* configurazione cloud;
* log;
* OTA;
* reset del sistema.

## 12.1 Endpoint principali

| Endpoint          | Funzione                         |
| ----------------- | -------------------------------- |
| `/`               | Pagina principale                |
| `/logo.png`       | Logo                             |
| `/status`         | Stato del sistema in JSON        |
| `/ora`            | Ora corrente                     |
| `/salvaprog`      | Salvataggio programmazione       |
| `/getprog`        | Lettura programmazione           |
| `/getCloud`       | Configurazione cloud             |
| `/setTelegram`    | Configurazione Telegram          |
| `/setWeather`     | Configurazione meteo             |
| `/salvaLogin`     | Configurazione password          |
| `/pompa1`         | Controllo pompa 1                |
| `/pompa2`         | Controllo pompa 2                |
| `/pozzo`          | Controllo relè pozzo             |
| `/setauto`        | Modalità automatica              |
| `/sospensione`    | Sospensione irrigazione          |
| `/setwifi`        | Configurazione Wi-Fi             |
| `/scanwifi`       | Scansione reti Wi-Fi             |
| `/resetwifi`      | Reset Wi-Fi                      |
| `/togglefdc`      | Fine-corsa                       |
| `/resetprog`      | Reset programmazione             |
| `/resetstats`     | Reset statistiche                |
| `/synctime`       | Sincronizzazione orario          |
| `/setmanualtime`  | Impostazione manuale dell'orario |
| `/setsmart`       | Configurazione SMART             |
| `/factoryreset`   | Factory reset                    |
| `/getSoilCal`     | Lettura calibrazione terreno     |
| `/setSoilCal`     | Salvataggio calibrazione terreno |
| `/getSerbatoio`   | Lettura livello serbatoio        |
| `/setSerbatoio`   | Configurazione serbatoio         |
| `/verifyAdvanced` | Verifica configurazione avanzata |
| `/ota`            | Pagina aggiornamento OTA         |
| `/update`         | Upload firmware                  |
| `/downloadlogs`   | Download log                     |
| `/clearlogs`      | Cancellazione log                |
| `/getPlants`      | Lettura profili piante           |
| `/savePlant`      | Salvataggio profilo pianta       |
| `/deletePlant`    | Eliminazione profilo pianta      |
| `/applyPlant`     | Applicazione profilo pianta      |
| `/resetPlants`    | Reset profili piante             |
| `/macAddress`     | MAC address                      |
| `/version`        | Versione firmware                |

---

# 13. Telegram

L'integrazione Telegram utilizza:

```text
UniversalTelegramBot
```

La configurazione viene mantenuta tramite `CloudConfig`.

Sono supportati:

* token del bot;
* chat principale;
* chat aggiuntive;
* abilitazione/disabilitazione del servizio.

Il sistema può inviare notifiche relative a:

* attivazione delle pompe;
* eventi di irrigazione;
* allarmi;
* livello del serbatoio;
* condizioni anomale;
* eventi del sistema.

Sono inoltre disponibili comandi remoti per alcune funzioni del controller.

---

# 14. ESP-NOW

Il modulo:

```text
espnow.cpp
```

gestisce la comunicazione **ESP-NOW** con dispositivi ESP remoti.

Questa tecnologia permette di creare nodi distribuiti per:

* sensori remoti;
* acquisizione dati;
* comunicazione punto-punto;
* estensione del sistema.

La funzione `OnDataRecv` utilizza una gestione condizionale della firma per mantenere la compatibilità con differenti versioni dell'ESP32 Arduino Core / ESP-IDF.

---

# 15. Memorizzazione persistente

La configurazione del sistema viene memorizzata utilizzando:

```text
Preferences
NVS
```

Questo permette di mantenere i parametri anche dopo il riavvio o la perdita di alimentazione.

Tra i dati persistenti possono essere inclusi:

* configurazione Wi-Fi;
* configurazione cloud;
* credenziali;
* calibrazione sensori;
* configurazione SMART;
* programmazione;
* profili delle piante;
* parametri di sistema.

---

# 16. Sicurezza

Il firmware integra diversi meccanismi di protezione.

## Autenticazione

L'accesso alle funzioni protette può essere gestito tramite password.

La password viene trattata utilizzando:

```text
SHA-256
```

tramite:

```text
mbedtls/sha256
```

## CSRF Protection

Gli endpoint sensibili sono protetti contro richieste non autorizzate tramite meccanismi CSRF.

## Rate Limiting

Il sistema limita il numero di tentativi/richieste in determinate operazioni sensibili.

## Factory Reset

La funzione:

```text
/factoryreset
```

permette di ripristinare la configurazione del sistema ai valori predefiniti.

---

# 17. Watchdog e affidabilità

Il firmware utilizza il Task Watchdog dell'ESP32-S3.

Configurazione:

```text
WDT_TIMEOUT = 30 s
```

Il watchdog permette di rilevare condizioni di blocco del firmware e riavviare il sistema quando necessario.

Questo è particolarmente importante in un dispositivo embedded che deve poter funzionare autonomamente per lunghi periodi.

---

# 18. OTA — Over The Air

Il sistema supporta l'aggiornamento del firmware tramite rete Wi-Fi.

Il processo utilizza:

```text
Update
esp_ota_ops
```

e permette di caricare un nuovo firmware direttamente tramite il Web Server.

Endpoint principali:

```text
/ota
/update
```

L'accesso alla funzione OTA è protetto tramite autenticazione.

Il sistema utilizza una partizione compatibile con aggiornamenti OTA.

---

# 19. Gestione del tempo

Il sistema combina:

* RTC DS3231;
* NTP;
* orologio di sistema.

Il modulo:

```text
time.cpp
```

gestisce:

* lettura dell'orario;
* sincronizzazione NTP;
* impostazione manuale;
* formattazione dell'orario;
* gestione del riferimento temporale utilizzato dalla programmazione.

La sincronizzazione permette al controller di mantenere un riferimento temporale corretto anche dopo periodi di funzionamento prolungato.

---

# 20. Gestione delle piante

Il modulo:

```text
plants.cpp
```

gestisce i profili delle piante.

Il sistema supporta:

```text
MAX_PLANTS = 100
```

I profili possono essere:

* creati;
* modificati;
* eliminati;
* applicati;
* ripristinati.

La gestione dei profili permette di associare parametri differenti alle diverse esigenze di irrigazione.

---

# 21. Task e gestione real-time

Il firmware utilizza l'architettura real-time dell'ESP32-S3 tramite **FreeRTOS**.

Le principali attività comprendono:

```text
Interfaccia / Touch
        │
        ├── Acquisizione sensori
        │
        ├── Controllo irrigazione
        │
        ├── Web Server
        │
        ├── Telegram
        │
        └── Gestione sistema
```

Una task dedicata ai sensori gestisce l'acquisizione di:

* DHT11;
* sensori di umidità;
* HC-SR04.

I dati acquisiti vengono utilizzati dalla logica di controllo e dalle interfacce locali e remote.

---

# 22. Flusso di funzionamento

Il funzionamento generale può essere riassunto nel seguente ciclo:

```text
Sensori
   │
   ▼
Acquisizione dati
   │
   ▼
Analisi condizioni
   │
   ├───────────────┐
   │               │
   ▼               ▼
Programmazione   SMART Logic
   │               │
   └───────┬───────┘
           ▼
    Decisione irrigazione
           │
           ▼
      Controllo pompe
           │
           ▼
     Nuova lettura sensori
           │
           ▼
      Feedback SMART
```

Il sistema può quindi utilizzare un ciclo di **feedback**, confrontando le condizioni del terreno prima e dopo l'irrigazione.

---

# 23. Struttura del progetto

La struttura prevista per il progetto è:

```text
esp32-smart-irrigation/

├── README.md
│
└── docs/
    ├── ARCHITECTURE.md
    ├── architecture.svg
    └── DOCUMENTATION.md
```

La documentazione tecnica fa riferimento all'architettura del firmware originale, mentre il codice sorgente non viene pubblicato nel repository.

---

# 24. Struttura del firmware

La struttura interna del firmware è organizzata come segue:

```text
main/
│
├── app.h
├── main.ino
├── globals.cpp
│
├── display.cpp
├── hardware.cpp
├── irrigation.cpp
├── time.cpp
│
├── wifi.cpp
├── web.cpp
├── cloud.cpp
├── espnow.cpp
│
├── plants.cpp
├── system.cpp
│
├── logo.h
├── LogoData.h
└── WebUI.h
```

### `app.h`

Rappresenta il punto di condivisione principale tra i moduli.

Contiene:

* include;
* define;
* strutture;
* tipi;
* dichiarazioni `extern`;
* prototipi delle funzioni.

### `globals.cpp`

Contiene le definizioni effettive delle variabili globali dichiarate tramite `extern`.

Questo permette di evitare definizioni multiple durante il linking.

---

# 25. Modularizzazione del firmware

La modularizzazione è stata effettuata partendo da un firmware monolitico di circa **8.600 righe**.

Il processo ha separato il firmware in moduli funzionali.

L'obiettivo principale era:

* ridurre la complessità;
* migliorare la leggibilità;
* separare le responsabilità;
* facilitare la manutenzione;
* semplificare il debugging;
* rendere più semplice l'evoluzione futura del progetto.

La struttura modulare permette, ad esempio, di intervenire sulla gestione della rete senza modificare direttamente la logica principale dell'irrigazione.

---

# 26. Processo di modularizzazione

La separazione del firmware originale è stata supportata da uno script Python:

```text
split2.py
```

Lo script:

1. legge il firmware monolitico originale;
2. identifica le sezioni tramite banner;
3. individua le funzioni;
4. assegna le funzioni ai moduli appropriati;
5. genera il file `app.h`;
6. gestisce le dichiarazioni `extern`;
7. genera i prototipi delle funzioni;
8. gestisce la compatibilità della callback ESP-NOW.

L'ordine delle regole di routing è importante per evitare sovrapposizioni tra sezioni.

---

# 27. Compatibilità ESP32 Core

La gestione della callback ESP-NOW utilizza una struttura condizionale per supportare differenti versioni dell'ESP32 Arduino Core / ESP-IDF.

Questo permette di mantenere la compatibilità con ambienti basati su differenti API della callback `OnDataRecv`.

La versione di riferimento utilizzata durante lo sviluppo è:

```text
ESP32 Arduino Core 2.0.17
```

---

# 28. Ambiente di compilazione

Il progetto è stato sviluppato utilizzando:

```text
Arduino IDE
```

Scheda:

```text
ESP32S3 Dev Module
```

Core:

```text
ESP32 Arduino Core 2.0.17
```

## Procedura

1. Aprire `main/main.ino` nell'Arduino IDE.
2. Selezionare la scheda ESP32-S3 appropriata.
3. Verificare la configurazione delle partizioni.
4. Compilare il progetto.
5. Caricare il firmware sulla scheda.

Per utilizzare l'aggiornamento OTA è consigliata una configurazione delle partizioni compatibile con OTA.

---

# 29. Build e cache

Durante lo sviluppo con Arduino IDE, una modifica alla struttura dei moduli può occasionalmente causare problemi legati a oggetti compilati precedentemente.

Un esempio è un errore di linker relativo a:

```text
multiple definition
```

In questi casi può essere necessario eliminare la cache di compilazione di Arduino e ricompilare completamente il progetto.

Su Windows, la cache può trovarsi in una directory simile a:

```text
C:\Users\<USER>\AppData\Local\arduino\sketches\
```

La pulizia della cache forza una nuova compilazione dei moduli.

---

# 30. Considerazioni progettuali

Il progetto combina diversi aspetti dell'ingegneria embedded:

```text
Hardware
   │
   ├── Sensori
   ├── Attuatori
   ├── Display
   └── RTC
        │
        ▼
ESP32-S3
        │
        ├── Firmware C++
        ├── FreeRTOS
        ├── Control Logic
        └── SMART Logic
        │
        ▼
Connectivity
        │
        ├── Wi-Fi
        ├── Web Server
        ├── Telegram
        ├── ESP-NOW
        └── OpenWeatherMap
```

L'architettura è stata progettata per integrare acquisizione dati, controllo, interfaccia e comunicazione mantenendo separate le principali responsabilità software.

---

# 31. Sintesi tecnica

Il progetto rappresenta un sistema embedded completo basato su ESP32-S3, con integrazione tra:

* elettronica;
* firmware C++;
* acquisizione sensori;
* controllo attuatori;
* interfaccia touch;
* networking;
* IoT;
* automazione;
* gestione real-time;
* sicurezza;
* aggiornamento remoto;
* logica adattiva.

La combinazione di **controllo locale, accesso remoto, acquisizione dei dati e logica SMART** permette di utilizzare il sistema come piattaforma sperimentale per applicazioni di automazione e IoT.

---

## Stato del progetto

**Piattaforma:** ESP32-S3
**Display:** ST7789 320×240
**Touch:** XPT2046
**RTC:** DS3231
**Ambiente:** Arduino IDE
**Firmware:** C++
**Real-Time:** FreeRTOS
**Networking:** Wi-Fi / ESP-NOW
**Cloud:** OpenWeatherMap
**Remote Control:** Web Server / Telegram
**Update:** OTA
**Storage:** NVS / Preferences
**Tipologia:** Embedded / IoT / Automazione

---

## Portfolio

Questo documento fa parte della documentazione tecnica del progetto **ESP32 Smart Irrigation**.

Il repository è stato realizzato come **portfolio tecnico**, con l'obiettivo di mostrare l'architettura, le funzionalità e le competenze applicate nello sviluppo del sistema, senza pubblicare il firmware sorgente completo.

