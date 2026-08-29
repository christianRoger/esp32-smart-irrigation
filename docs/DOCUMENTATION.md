# Documentazione Tecnica e Manuale Operativo — ESP32 Smart Irrigation

Guida tecnica e operativa del sistema **ESP32 Smart Irrigation**, con descrizione delle funzionalità disponibili tramite **display TFT touch** e **interfaccia Web**.

Il firmware è eseguito su **ESP32-S3** con display **ST7789 320×240** e touch resistivo **XPT2046**.

> **Nota:** questa documentazione descrive il comportamento e le funzionalità implementate nel firmware del progetto. Il codice sorgente completo non è incluso nel repository pubblico.

---

# 1. Panoramica del sistema

**ESP32 Smart Irrigation** è un sistema embedded/IoT progettato per il controllo e la gestione automatizzata di un impianto di irrigazione.

Il sistema integra:

* acquisizione dati dai sensori;
* controllo di pompe e attuatori;
* monitoraggio del livello del serbatoio;
* gestione dell'orario tramite RTC e NTP;
* interfaccia grafica touch;
* Web Server embedded;
* comunicazione Wi-Fi;
* Telegram;
* ESP-NOW;
* OpenWeatherMap;
* programmazione dell'irrigazione;
* controllo automatico e manuale;
* modalità SMART adattiva;
* aggiornamento firmware OTA;
* registrazione degli eventi;
* meccanismi di sicurezza e affidabilità.

La navigazione principale del display segue il seguente ordine:

```text
STATO SISTEMA
      ↓
POZZO / ACQUA
      ↓
RETE / WIFI
      ↓
PROGRAMMI
      ↓
METEO
      ↓
SMART
      ↓
IMPOSTAZIONI
      ↓
STATO SISTEMA
```

La navigazione può essere effettuata tramite **swipe orizzontale** oppure utilizzando gli elementi della barra di navigazione.

---

# PARTE 1 — DISPLAY TFT TOUCH

## 2. Navigazione generale

Il sistema utilizza un display TFT **ST7789 320×240** con touch resistivo **XPT2046**.

### Swipe

Lo swipe orizzontale permette di passare da una schermata all'altra.

Parametri principali:

```text
SWIPE_THRESHOLD = 60 px
SWIPE_MAX_MS    = 800 ms
```

Lo swipe viene riconosciuto quando il movimento supera la soglia configurata entro il tempo massimo previsto.

### Barra di navigazione

Il tocco sulle icone della barra inferiore permette di accedere direttamente alle principali sezioni del sistema.

### Pulsanti touch

I pulsanti presenti nelle schermate permettono di:

* controllare le pompe;
* modificare configurazioni;
* aprire il tastierino virtuale;
* confermare operazioni;
* modificare modalità operative;
* accedere alle funzioni di sistema.

### Reset Wi-Fi

Un pulsante fisico collegato a:

```text
GPIO18
```

permette di cancellare le credenziali Wi-Fi e riavviare il sistema nella procedura di configurazione della rete.

---

# 3. Schermata STATO SISTEMA

La schermata principale fornisce una panoramica dello stato operativo del sistema.

Sono visualizzati:

* data e ora;
* stato della Pompa 1;
* stato della Pompa 2;
* stato del relè del pozzo;
* umidità del terreno 1;
* umidità del terreno 2;
* temperatura;
* umidità ambientale;
* informazioni sul sistema di irrigazione;
* stato operativo Auto/Manuale;
* informazioni relative al serbatoio;
* eventuali condizioni di allarme.

Quando una pompa è attiva, l'interfaccia può utilizzare animazioni per rappresentare visivamente il funzionamento dell'impianto.

La schermata principale rappresenta quindi il **pannello di controllo locale** del dispositivo.

---

# 4. Schermata POZZO / ACQUA

Questa sezione è dedicata al monitoraggio dell'acqua e del sistema di approvvigionamento.

Sono disponibili:

* livello del serbatoio;
* percentuale stimata dell'acqua disponibile;
* umidità dei due sensori del terreno;
* stato delle pompe;
* stato del pozzo;
* controlli manuali quando consentiti;
* condizioni di sicurezza del serbatoio.

Il livello dell'acqua viene determinato tramite il sensore ultrasonico **HC-SR04**.

Quando il livello scende sotto il limite configurato, il sistema può generare la condizione:

```text
Serbatoio Vuoto
```

e impedire l'attivazione delle pompe per proteggere l'impianto.

---

# 5. Schermata RETE / WIFI

La schermata di rete mostra lo stato della connessione Wi-Fi.

Sono disponibili informazioni quali:

* SSID della rete;
* indirizzo IP;
* intensità del segnale;
* stato della connessione;
* configurazione della rete;
* scansione delle reti disponibili;
* reset della configurazione Wi-Fi.

Il sistema dispone inoltre di un **tastierino virtuale (OSK)** per l'inserimento dei dati di configurazione.

---

# 6. Schermata PROGRAMMI

La sezione **PROGRAMMI** permette di configurare la programmazione temporale dell'irrigazione.

Ogni programma può essere associato a:

```text
Programma
├── Inizio
├── Fine
└── Attivo
```

Le principali funzioni comprendono:

* configurazione degli orari;
* attivazione/disattivazione dei programmi;
* salvataggio della programmazione;
* reset della programmazione;
* reset delle statistiche.

La programmazione viene utilizzata dalla logica automatica del sistema per determinare quando eseguire i cicli di irrigazione.

---

# 7. Schermata METEO

La sezione **METEO** visualizza le informazioni meteorologiche ottenute tramite **OpenWeatherMap**.

Possono essere visualizzati:

* condizioni meteorologiche;
* descrizione del tempo;
* città configurata;
* stato della previsione di pioggia;
* momento dell'ultimo aggiornamento.

La variabile:

```text
pioggiaPrevista
```

viene utilizzata dalla logica di controllo per determinare se l'irrigazione automatica debba essere sospesa.

Flusso semplificato:

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
```

---

# 8. Schermata SMART — Controllo Adattivo

La modalità **SMART** implementa una logica adattiva basata sul feedback dei sensori.

Il sistema considera principalmente:

* durata del funzionamento della pompa;
* umidità del terreno prima dell'irrigazione;
* umidità del terreno dopo l'irrigazione;
* variazione dell'umidità ottenuta.

Il principio generale è:

```text
Umidità iniziale
       ↓
Durata irrigazione
       ↓
Nuova lettura del terreno
       ↓
Calcolo della variazione
       ↓
Aggiornamento dei parametri
       ↓
Ciclo successivo
```

La funzione:

```text
updateLearning()
```

viene utilizzata dalla logica di apprendimento/adattamento del sistema.

### Obiettivo

L'obiettivo è adattare progressivamente la durata dei cicli alle condizioni rilevate dal terreno, evitando di utilizzare esclusivamente tempi fissi.

> **Nota tecnica:** la modalità SMART è una logica adattiva/euristica implementata nel firmware. Non utilizza un modello di machine learning addestrato tramite framework esterni.

---

# 9. Schermata IMPOSTAZIONI

La sezione **IMPOSTAZIONI** raccoglie le principali configurazioni del dispositivo.

Sono disponibili funzioni relative a:

* calibrazione dei sensori del terreno;
* configurazione del serbatoio;
* configurazione dell'orario;
* credenziali di accesso Web;
* configurazione Wi-Fi;
* Telegram;
* OpenWeatherMap;
* modalità SMART;
* aggiornamento OTA;
* configurazioni avanzate;
* fine-corsa;
* controlli di sistema;
* factory reset.

La schermata rappresenta il principale punto di configurazione locale del dispositivo.

---

# 10. Alert e messaggi di sistema

Il display può visualizzare finestre di avviso e messaggi relativi allo stato del sistema.

Tra gli eventi gestiti:

* irrigazione in corso;
* attivazione manuale;
* avvio di un programma;
* fine di un programma;
* serbatoio vuoto;
* terreno secco;
* aggiornamento OTA;
* factory reset;
* condizioni anomale del sistema.

Esempio:

```text
Serbatoio Vuoto
       ↓
Blocco delle pompe
       ↓
Visualizzazione dell'allarme
       ↓
Eventuale notifica remota
```

Quando configurato, determinati eventi possono essere inviati anche tramite Telegram.

---

# 11. Tastierino virtuale — OSK

Il sistema dispone di una **On-Screen Keyboard (OSK)** per l'inserimento dei dati direttamente dal display.

Può essere utilizzata per inserire informazioni come:

* SSID Wi-Fi;
* password Wi-Fi;
* password di accesso;
* Token Telegram;
* Chat ID;
* API Key OpenWeatherMap;
* città meteorologica;
* altri parametri configurabili.

Il tastierino supporta l'inserimento di caratteri e numeri e permette di confermare i dati direttamente dal dispositivo.

---

# PARTE 2 — INTERFACCIA WEB

# 12. Accesso al Web Server

Il sistema dispone di un **Web Server embedded** accessibile tramite browser attraverso la rete Wi-Fi.

Procedura generale:

### 1. Collegamento alla rete

Collegare l'ESP32 alla rete Wi-Fi configurata.

Lo stato della rete può essere verificato dalla schermata:

```text
RETE / WIFI
```

### 2. Recupero dell'indirizzo IP

L'indirizzo IP assegnato al dispositivo viene visualizzato nella schermata di rete.

Esempio:

```text
192.168.1.50
```

### 3. Accesso tramite browser

Inserire nel browser l'indirizzo del dispositivo:

```text
http://192.168.1.50/
```

L'accesso alle funzioni sensibili può richiedere l'autenticazione configurata nel sistema.

---

# 13. Dashboard Web

La dashboard principale permette di monitorare lo stato del sistema attraverso un browser.

Sono visualizzati:

* stato delle pompe;
* stato del pozzo;
* dati dei sensori;
* temperatura;
* umidità ambientale;
* umidità del terreno;
* livello del serbatoio;
* condizioni meteorologiche;
* modalità operativa;
* necessità di irrigazione;
* registro degli eventi del sistema.

La dashboard costituisce l'interfaccia principale per il **monitoraggio remoto**.

---

# 14. Configurazione Web

L'interfaccia Web permette di accedere alle principali funzioni di configurazione.

Le aree principali comprendono:

### 🔐 Credenziali di accesso Web

Gestione della password utilizzata per proteggere le funzioni sensibili.

### 🧠 Modalità SMART autonoma

Attivazione e configurazione della logica SMART.

### 📶 Configurazione Wi-Fi

Gestione delle reti Wi-Fi e delle credenziali.

### 🔧 Configurazione avanzata

Accesso ai parametri avanzati del sistema.

### 🕐 Configurazione manuale dell'ora

Impostazione manuale dell'orologio quando la sincronizzazione NTP non è disponibile.

### 📱 Integrazione Telegram

Configurazione del bot e dei parametri necessari per le notifiche remote.

### ⛅ Meteo intelligente

Configurazione di OpenWeatherMap e dei parametri utilizzati dalla logica meteorologica.

### ⬆️ Aggiornamento firmware

Accesso alla funzione OTA per l'aggiornamento del firmware.

### 🔧 Controlli di sistema

Funzioni di manutenzione e controllo del dispositivo.

### 🏭 Factory Reset

Ripristino delle configurazioni di sistema.

---

# 15. Configurazione iniziale

Una configurazione tipica del sistema può essere eseguita seguendo questi passaggi:

| Passo | Operazione               | Funzione                                     |
| ----: | ------------------------ | -------------------------------------------- |
|     1 | Collegamento Wi-Fi       | `/scanwifi`, `/setwifi`, `/resetwifi`        |
|     2 | Password Web             | `/salvaLogin`                                |
|     3 | Telegram                 | `/setTelegram`                               |
|     4 | Meteo                    | `/setWeather`                                |
|     5 | Programmazione           | `/salvaprog`, `/getprog`                     |
|     6 | Calibrazione terreno     | `/getSoilCal`, `/setSoilCal`                 |
|     7 | Configurazione serbatoio | `/getSerbatoio`, `/setSerbatoio`             |
|     8 | SMART                    | `/setsmart`                                  |
|     9 | Profili piante           | `/getPlants`, `/savePlant`                   |
|    10 | Ora                      | `/synctime`, `/setmanualtime`                |
|    11 | Fine-corsa               | `/togglefdc`                                 |
|    12 | Reset                    | `/factoryreset`, `/resetprog`, `/resetstats` |
|    13 | OTA                      | `/ota`, `/update`                            |
|    14 | Log                      | `/downloadlogs`, `/clearlogs`                |

---

# 16. Endpoint Web

Il Web Server utilizza endpoint HTTP dedicati alle diverse funzioni del sistema.

## Sistema e stato

```text
/
 /logo.png
/status
/ora
/macAddress
/version
```

## Irrigazione

```text
/pompa1
/pompa2
/pozzo
/setauto
/sospensione
/togglefdc
```

## Programmazione

```text
/salvaprog
/getprog
/resetprog
/resetstats
```

## Cloud

```text
/getCloud
/setTelegram
/setWeather
/verifyAdvanced
```

## Wi-Fi

```text
/scanwifi
/setwifi
/resetwifi
```

## Gestione dell'orario

```text
/synctime
/setmanualtime
```

## SMART

```text
/setsmart
```

## Terreno e serbatoio

```text
/getSoilCal
/setSoilCal
/getSerbatoio
/setSerbatoio
```

## Profili delle piante

```text
/getPlants
/savePlant
/deletePlant
/applyPlant
/resetPlants
```

## Sicurezza e manutenzione

```text
/salvaLogin
/factoryreset
```

## OTA

```text
/ota
/update
```

## Log

```text
/downloadlogs
/clearlogs
```

---

# 17. Telegram

L'integrazione Telegram permette di monitorare il sistema e ricevere notifiche da remoto.

A seconda della configurazione, possono essere notificati eventi come:

* accensione della Pompa 1;
* spegnimento della Pompa 1;
* accensione della Pompa 2;
* spegnimento della Pompa 2;
* stato del pozzo;
* serbatoio vuoto;
* terreno secco;
* eventi di programmazione;
* eventi OTA;
* factory reset.

La configurazione del servizio viene gestita tramite l'interfaccia Web.

Le credenziali e i token non vengono pubblicati nel repository.

---

# 18. ESP-NOW

Il sistema supporta la comunicazione tramite **ESP-NOW** con dispositivi ESP remoti.

Questa tecnologia può essere utilizzata per realizzare un'architettura distribuita composta, ad esempio, da:

```text
ESP32-S3 Controller
        │
        ├── ESP-NOW ──► Nodo sensore 1
        │
        ├── ESP-NOW ──► Nodo sensore 2
        │
        └── ESP-NOW ──► Nodo remoto
```

Possibili applicazioni:

* sensori di umidità remoti;
* sensori ambientali;
* nodi distribuiti;
* espansione delle zone di irrigazione;
* acquisizione dati da dispositivi aggiuntivi.

---

# 19. Gestione dei sensori

## 19.1 Sensori di umidità del terreno

Il sistema utilizza due ingressi ADC:

```text
SOIL_PIN1 → GPIO 1
SOIL_PIN2 → GPIO 2
```

I sensori possono essere calibrati utilizzando valori di riferimento per terreno asciutto e terreno bagnato.

La calibrazione viene memorizzata nella memoria non volatile.

---

## 19.2 DHT11

Il DHT11 viene utilizzato per rilevare:

* temperatura;
* umidità relativa dell'ambiente.

Collegamento:

```text
DHT11 → GPIO15
```

---

## 19.3 HC-SR04

Il sensore ultrasonico viene utilizzato per determinare il livello dell'acqua nel serbatoio.

```text
TRIG → GPIO16
ECHO → GPIO17
```

La distanza rilevata viene convertita in un valore utilizzato per determinare lo stato del serbatoio.

> **Nota elettrica:** il segnale ECHO dell'HC-SR04 deve essere adattato al livello logico compatibile con l'ESP32-S3 quando richiesto dal modulo utilizzato.

---

## 19.4 RTC DS3231

Il DS3231 fornisce una sorgente temporale locale tramite I²C.

```text
SDA → GPIO8
SCL → GPIO9
```

Quando disponibile una connessione Internet, il sistema può sincronizzare l'orario tramite NTP.

L'RTC fornisce quindi una base temporale locale anche quando la connessione di rete non è disponibile.

---

# 20. Memoria persistente

Il sistema utilizza **NVS / Preferences** per mantenere le configurazioni anche dopo un riavvio o una perdita di alimentazione.

Possono essere memorizzati:

* calibrazione dei sensori;
* configurazione Wi-Fi;
* configurazione Telegram;
* configurazione OpenWeatherMap;
* programmi di irrigazione;
* parametri SMART;
* configurazione del serbatoio;
* profili delle piante;
* impostazioni del sistema.

---

# 21. Sicurezza

Il firmware implementa diversi meccanismi di protezione.

## Autenticazione

Le funzioni sensibili del Web Server possono essere protette tramite password.

La password viene gestita tramite hash **SHA-256**.

## Protezione CSRF

Gli endpoint sensibili possono utilizzare token CSRF per ridurre il rischio di richieste non autorizzate provenienti da pagine o origini esterne.

## Rate Limiting

Il sistema implementa meccanismi di limitazione delle richieste per ridurre tentativi ripetuti di accesso o utilizzo improprio degli endpoint.

## Factory Reset

Il sistema dispone di una procedura di ripristino delle configurazioni.

Il factory reset permette di cancellare le impostazioni persistenti e riportare il sistema allo stato iniziale previsto dal firmware.

---

# 22. Watchdog e affidabilità

Per migliorare l'affidabilità del sistema viene utilizzato un **Task Watchdog**.

Configurazione principale:

```text
WDT_TIMEOUT = 30 secondi
```

Il watchdog permette di rilevare condizioni in cui le attività monitorate non vengono eseguite correttamente.

In determinate condizioni di blocco può essere eseguito il riavvio del microcontrollore.

---

# 23. OTA — Over The Air Update

Il sistema supporta l'aggiornamento del firmware tramite rete Wi-Fi.

Flusso generale:

```text
Browser
   ↓
Web Server
   ↓
Autenticazione
   ↓
Upload firmware .bin
   ↓
OTA Update
   ↓
Nuova partizione firmware
   ↓
Riavvio ESP32-S3
```

L'aggiornamento permette di installare una nuova versione del firmware senza collegare fisicamente il dispositivo al computer.

Per utilizzare correttamente la funzione OTA è necessaria una configurazione di partizionamento compatibile con l'aggiornamento via rete.

---

# 24. Modalità operative

Il sistema può operare principalmente nelle seguenti modalità:

```text
                 SISTEMA
                    │
          ┌─────────┼─────────┐
          │         │         │
          ▼         ▼         ▼
       MANUALE   AUTOMATICO   SMART
          │         │         │
          ▼         ▼         ▼
       Comando    Programmi   Controllo
       diretto    orari       adattivo
```

## Manuale

L'utente controlla direttamente le pompe e gli attuatori attraverso l'interfaccia disponibile.

## Automatico

Il sistema utilizza i programmi configurati per determinare quando eseguire l'irrigazione.

## SMART

Il sistema utilizza i dati dei sensori e i risultati dei cicli precedenti per adattare il comportamento dell'irrigazione.

---

# 25. Logica di protezione dell'irrigazione

Prima dell'attivazione degli attuatori, il firmware può verificare diverse condizioni operative.

Tra queste:

* livello del serbatoio;
* stato dei sensori;
* modalità operativa;
* programmazione;
* sospensione manuale;
* condizioni meteorologiche;
* stato del sistema;
* eventuali condizioni di sicurezza.

Esempio semplificato:

```text
Richiesta irrigazione
        ↓
Controllo condizioni
        ↓
Serbatoio sufficiente?
        │
     ┌──┴──┐
     │     │
    SI     NO
     │     │
     ▼     ▼
Controllo  BLOCCO
meteo      pompe
     │
     ▼
Pioggia prevista?
     │
  ┌──┴──┐
  │     │
 NO     SI
  │     │
  ▼     ▼
IRRIGA  SOSPENDI
```

---

# 26. Flusso tipico di utilizzo

Una configurazione iniziale del sistema può seguire questo processo:

```text
Prima accensione
       ↓
Configurazione Wi-Fi
       ↓
Accesso Web
       ↓
Impostazione password
       ↓
Configurazione sensori
       ↓
Configurazione serbatoio
       ↓
Configurazione Telegram
       ↓
Configurazione OpenWeatherMap
       ↓
Programmazione irrigazione
       ↓
Calibrazione terreno
       ↓
Attivazione SMART
       ↓
Monitoraggio
       ↓
Irrigazione automatica
```

---

# 27. Monitoraggio e diagnostica

Il sistema fornisce diversi strumenti per il monitoraggio e la diagnostica.

Sono disponibili:

* stato delle pompe;
* stato del pozzo;
* valori dei sensori;
* livello del serbatoio;
* temperatura;
* umidità;
* informazioni di rete;
* indirizzo IP;
* stato operativo;
* registro degli eventi;
* versione firmware;
* indirizzo MAC.

I log possono essere gestiti tramite Web Server utilizzando:

```text
/downloadlogs
/clearlogs
```

---

# 28. Funzioni di manutenzione

Le funzioni di manutenzione includono:

* reset Wi-Fi;
* reset programmazione;
* reset statistiche;
* cancellazione dei log;
* factory reset;
* aggiornamento OTA;
* visualizzazione della versione firmware;
* visualizzazione dell'indirizzo MAC;
* gestione delle configurazioni persistenti.

---

# 29. Architettura funzionale

Il funzionamento generale può essere rappresentato come:

```text
┌──────────────────────┐
│       SENSORI        │
│                      │
│ Terreno              │
│ DHT11                │
│ HC-SR04              │
│ RTC                  │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ ACQUISIZIONE DATI    │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ LOGICA DI CONTROLLO  │
│                      │
│ Automatico           │
│ Manuale              │
│ SMART                │
│ Meteo                │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ DECISIONE            │
│ IRRIGAZIONE          │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ ATTUATORI            │
│                      │
│ Pompa 1              │
│ Pompa 2              │
│ Pozzo                │
└──────────────────────┘
```

Parallelamente, i dati possono essere inviati alle interfacce locali e remote:

```text
                 ┌──► Display TFT
                 │
Sensori → ESP32 ─┼──► Web Server
                 │
                 ├──► Telegram
                 │
                 └──► Logica SMART
```

---

# 30. Tecnologie utilizzate

Il progetto integra diverse tecnologie hardware e software.

| Tecnologia           | Utilizzo                  |
| -------------------- | ------------------------- |
| ESP32-S3             | Controller principale     |
| C++                  | Firmware                  |
| Arduino              | Framework di sviluppo     |
| FreeRTOS             | Gestione delle attività   |
| ST7789               | Display TFT               |
| XPT2046              | Touch resistivo           |
| SPI                  | Comunicazione periferiche |
| I²C                  | Comunicazione RTC         |
| ADC                  | Lettura sensori terreno   |
| DHT11                | Temperatura e umidità     |
| HC-SR04              | Livello serbatoio         |
| DS3231               | RTC                       |
| Wi-Fi                | Connettività              |
| WiFiManager          | Configurazione Wi-Fi      |
| WebServer            | Interfaccia Web           |
| ArduinoJson          | Gestione JSON             |
| HTTPClient           | Comunicazione HTTP        |
| WiFiClientSecure     | HTTPS                     |
| NTPClient            | Sincronizzazione NTP      |
| Preferences          | Memoria NVS               |
| UniversalTelegramBot | Telegram                  |
| ESP-NOW              | Comunicazione tra ESP     |
| Update               | OTA                       |
| esp_task_wdt         | Watchdog                  |
| mbedTLS / SHA-256    | Hashing                   |

La libreria **Espalexa** è stata rimossa dal progetto perché non utilizzata nella versione attuale.

---

# 31. Note sullo sviluppo

Il firmware è stato inizialmente sviluppato come uno sketch monolitico.

Con l'evoluzione del progetto, le funzionalità sono state progressivamente separate in moduli C++ dedicati.

La modularizzazione ha permesso di separare:

```text
Interfaccia
     │
     ▼
Controllo
     │
     ▼
Hardware
     │
     ▼
Comunicazione
     │
     ▼
Servizi di sistema
```

Questo approccio facilita:

* manutenzione;
* debugging;
* diagnostica;
* individuazione degli errori;
* evoluzione del firmware;
* integrazione di nuove funzionalità.

---

# 32. Struttura concettuale del firmware

La struttura logica dei principali moduli è:

```text
app.h
  │
  ├── display.cpp
  ├── hardware.cpp
  ├── irrigation.cpp
  ├── time.cpp
  ├── wifi.cpp
  ├── web.cpp
  ├── cloud.cpp
  ├── espnow.cpp
  ├── plants.cpp
  ├── system.cpp
  │
  └── globals.cpp
```

`app.h` contiene le interfacce condivise, i tipi, le costanti, le dichiarazioni `extern` e i prototipi necessari ai diversi moduli.

`globals.cpp` contiene le definizioni delle variabili globali.

---

# 33. Considerazioni tecniche

Il progetto integra in un'unica piattaforma:

```text
Elettronica
     +
Firmware
     +
Automazione
     +
Networking
     +
Interfaccia utente
     +
Servizi cloud
```

L'architettura permette di utilizzare il sistema sia come controller locale sia come dispositivo IoT con capacità di monitoraggio e controllo remoto.

La combinazione tra sensori, programmazione temporale, dati meteorologici e controllo adattivo permette di realizzare una gestione dell'irrigazione più flessibile rispetto a un semplice temporizzatore.

---

# 34. Stato del sistema

**Platform:** ESP32-S3
**Display:** ST7789 320×240
**Touch:** XPT2046
**RTC:** DS3231
**Firmware:** C++ / Arduino
**Real-Time:** FreeRTOS
**Networking:** Wi-Fi / ESP-NOW
**Remote Control:** Web Server / Telegram
**Weather:** OpenWeatherMap
**Storage:** NVS / Preferences
**Update:** OTA
**Project Type:** Embedded / IoT / Automation

---

# 35. Nota sul repository

Il repository pubblico è stato organizzato come **portfolio tecnico**.

La struttura documentale comprende:

```text
esp32-smart-irrigation/
│
├── README.md
│
├── images/
│   ├── display-running.jpg
│   ├── dashboard-01.jpg
│   ├── dashboard-02.jpg
│   ├── dashboard-03.jpg
│   ├── dashboard-04.jpg
│   ├── dashboard-05.jpg
│   ├── dashboard-06.jpg
│   └── dashboard-07.jpg
│
└── docs/
    ├── ARCHITECTURE.md
    ├── DOCUMENTATION.md
    └── architecture.svg
```

Il firmware completo non viene pubblicato nel repository.

La documentazione è stata resa disponibile per illustrare:

* architettura hardware/software;
* funzionalità del sistema;
* interfacce utente;
* comunicazioni;
* logica di controllo;
* sicurezza;
* affidabilità;
* processo di sviluppo;
* competenze tecniche applicate.

---

## Conclusione

**ESP32 Smart Irrigation** rappresenta un progetto completo di sistemi embedded e IoT, sviluppato integrando hardware, firmware, automazione e comunicazione di rete.

Il sistema dimostra l'integrazione pratica di:

**Sensori → Elaborazione → Controllo → Automazione → Interfaccia → Networking → Monitoraggio**

con particolare attenzione alla modularità del firmware, alla gestione delle condizioni operative e alla possibilità di estendere il sistema con ulteriori sensori, nodi remoti e funzionalità.
