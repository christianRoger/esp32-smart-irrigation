# ESP32 Smart Irrigation

## Smart IoT Irrigation System

A smart irrigation controller developed around the **ESP32-S3**, combining embedded electronics, sensors, automation, remote connectivity and an interactive touch interface.

The project was designed as a complete IoT system capable of monitoring environmental conditions, controlling irrigation pumps and adapting irrigation cycles according to soil conditions and weather information.

> **Note:** The source code is not included in this repository. This repository is intended as a technical portfolio and project documentation.

---

## Project Overview

The system combines hardware control, embedded software and IoT connectivity in a single irrigation platform.

The controller manages:

* 💧 Automated irrigation pumps
* 🌱 Soil moisture monitoring
* 🌡️ Temperature and humidity monitoring
* 🚰 Water reservoir level monitoring
* 🖥️ TFT touch-screen interface
* 📡 Wi-Fi connectivity
* 🌐 Embedded Web Server
* 📱 Telegram remote control and notifications
* 🔗 ESP-NOW communication
* ☁️ OpenWeatherMap integration
* 🔄 OTA firmware updates
* 🤖 Adaptive SMART irrigation logic
* 🛡️ Watchdog and security mechanisms

---

## Hardware

### Main Controller

* ESP32-S3
* ST7789 TFT display — 320×240
* XPT2046 resistive touch controller
* DS3231 RTC

### Sensors

* 2× soil moisture sensors
* DHT11 temperature/humidity sensor
* HC-SR04 ultrasonic water-level sensor

### Actuators

* 2× irrigation pumps
* 1× well relay
* Buzzer
* Status LEDs

---

## Software & Technologies

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
* **NVS persistent storage**
* **JSON**
* **Watchdog**
* **SHA-256**
* **CSRF protection**
* **Rate limiting**

---

## SMART Irrigation

One of the main features of the project is the adaptive **SMART irrigation system**.

The controller analyzes the relationship between:

* Pump operating time
* Soil moisture before irrigation
* Soil moisture after irrigation

Based on the collected data, the system can adapt irrigation duration to improve water management.

The system can also take weather information into account and suspend automatic irrigation when rain is expected.

---

## User Interface

The controller features a graphical touch interface developed for the ST7789 display.

Main screens include:

* Home
* Water / Reservoir
* Network
* Weather
* SMART / AI
* Schedule
* Settings

The interface also includes animated elements, alerts, touch navigation and an on-screen keyboard.

---

## Remote Control

The system provides multiple communication methods.

### Web Server

The embedded Web Server allows monitoring and configuration directly from a browser.

Functions include:

* System status
* Manual pump control
* Irrigation scheduling
* Wi-Fi configuration
* Sensor calibration
* Plant profiles
* SMART configuration
* Weather configuration
* System logs
* OTA firmware update

### Telegram

Telegram integration allows remote notifications and commands.

### ESP-NOW

ESP-NOW is used for communication with remote ESP-based sensors or devices.

---

## Reliability & Security

The firmware includes several mechanisms designed to improve reliability and security:

* Hardware watchdog
* Password-protected access
* SHA-256 password hashing
* CSRF protection
* Rate limiting
* OTA protection
* Factory reset
* Persistent configuration using NVS

---

## Architecture

The original firmware was initially developed as a large monolithic sketch.

As the project evolved, the software architecture was reorganized into functional modules covering:

```text
                    ESP32-S3
                       │
        ┌──────────────┼──────────────┐
        │              │              │
     Sensors        Actuators      User Interface
        │              │              │
   Soil / DHT11    Pumps / Relay   ST7789 + Touch
   HC-SR04         Buzzer / LEDs
        │
        └──────────────┬──────────────┘
                       │
                  Control Logic
                       │
              ┌────────┼────────┐
              │        │        │
             Wi-Fi   Telegram  ESP-NOW
              │
        OpenWeatherMap
              │
          SMART Logic
```

---

## Key Engineering Areas

This project demonstrates practical experience in:

* Embedded systems development
* Microcontroller programming
* Electronics and hardware integration
* Sensor acquisition
* Actuator control
* SPI and I²C communication
* ADC measurement and calibration
* Wi-Fi networking
* IoT communication
* Web interfaces
* Remote device management
* OTA firmware updates
* Real-time task management
* Embedded security
* System diagnostics
* Hardware/software integration

---

## Project Status

**Platform:** ESP32-S3
**Display:** ST7789 320×240
**Touch:** XPT2046
**Development environment:** Arduino IDE
**Firmware:** C++
**Project type:** Embedded / IoT / Automation

---

## About

This project was developed as a practical embedded systems project, combining electronics, firmware development, automation and IoT technologies into a complete working platform.

The repository documents the project architecture and technical capabilities without exposing the source code.
