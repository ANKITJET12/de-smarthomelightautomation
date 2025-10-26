IoT Smart Home Automation System

1. Introduction

This report details the design and implementation of a 4-channel Smart Home Automation System. The project's core purpose is to provide remote control over four independent electrical appliances using a standard smartphone.

By leveraging the Internet of Things (IoT), this system bridges the gap between household devices and the internet. It uses an ESP32 microcontroller as the "brain" to connect to a WiFi network and communicate with the Blynk cloud platform. A user can then use the simple Blynk mobile application from anywhere in the world to turn their home lights, fans, or other appliances on or off, with the system's status updated in real-time.

This project serves as a low-cost, scalable, and practical foundation for a full-scale smart home.

2. Uniqueness

While many IoT relay controllers exist, this project's design is unique due to its emphasis on system stability and robustness.

The primary challenge in many DIY IoT projects is power instability. The ESP32's WiFi radio and the 4-channel relay module are both power-hungry components. When they draw power from a single source, it often causes voltage drops ("brownouts") that lead to the microcontroller crashing and getting stuck in a permanent reset loop.

This project's unique design feature is the isolation of power supplies.

The ESP32 "brain" is powered by one dedicated 5V source.

The 4-channel relay "muscle" is powered by a second, separate 5V source.

This "Separate Power Supply" architecture, combined with a "Common Ground" connection, ensures that the relay module's high-current operations can never interfere with or crash the sensitive ESP32. This makes the system exceptionally reliable, which is a critical and often-overlooked feature for a smart home device.

3. Components

A. Hardware Components

1x ESP32 Dev Board: The main microcontroller with built-in WiFi and Bluetooth, serving as the "brain" of the project.

1x 4-Channel 5V Relay Module: The "muscle" of the project. Each relay is an electrically-operated switch that can safely turn a high-voltage appliance on or off.

1x Solderless Breadboard: Used as a central point for wiring the low-voltage components.

Assorted Jumper Wires (Male-to-Female): Used to connect the ESP32 and relay module to the breadboard and each other.

Power Supply 1 (for ESP32): One 5V (1A or 2A) phone charger with a micro-USB cable.

Power Supply 2 (for Relay Module): One 5V (1A or 2A) phone charger with a spare, cut USB cable (exposing the Red/+5V and Black/GND wires).

B. Software and Services

Arduino IDE: The software environment used to write and upload the C++ code to the ESP32.

Blynk IoT Platform (Mobile App): The cloud service and mobile app used to create the user-friendly control dashboard.

Required Libraries:

WiFi.h (for connecting to the internet)

BlynkSimpleEsp32.h (for communicating with the Blynk server)

4. Working

The project's operation is divided into three parts: the stable hardware setup, the software logic, and the user-control flow.

A. Hardware Setup (The Stable Power Design)

To prevent the ESP32 from crashing, the power is isolated.

ESP32 Power: The ESP32 is powered independently by its own USB cable connected to Phone Charger #1.

Relay Module Power: The "extra" USB cable is plugged into Phone Charger #2. Its Red (+5V) wire is connected to the breadboard's red positive rail, and its Black (GND) wire is connected to the blue negative rail. The relay module's VCC and GND pins are then connected to these rails.

Common Ground (Critical): A single jumper wire connects a GND pin from the ESP32 to the blue negative rail on the breadboard. This ensures both independent circuits share the same "ground" reference, which is essential for the signal wires to work.

Signal Connections: The ESP32's GPIO pins are connected to the relay module's trigger pins.

GPIO 13 → IN1

GPIO 12 → IN2

GPIO 14 → IN3

GPIO 27 → IN4

B. Software Logic

The provided Arduino code is designed for maximum stability.

Boot & WiFi: On startup, the ESP32's first priority is to connect to the user's WiFi network (Redmi Note 13 Pro 5G). Software fixes are included to lower the WiFi's power draw (setTxPower) and disable the sensitive brownout detector, further preventing crashes.

Blynk Connection: Once on WiFi, the ESP32 connects to the Blynk server using the unique Auth Token.

Initialization (The BLYNK_CONNECTED function): As soon as the ESP32 is online, it runs this function one time. This function initializes the four GPIO pins as outputs and immediately sets them all to HIGH. Since the relays are "active-low," this HIGH signal turns all four relays OFF, preventing them from being stuck on at startup.

Listen for Commands: The ESP32 enters the loop(), where it constantly runs Blynk.run(). This function listens for new commands from the Blynk app.

C. User Control Flow

The user opens the Blynk app on their phone. The app shows the ESP32 is "Online."

The user taps the button for "Relay 2" (Virtual Pin V2).

The app sends a "V2=1" (ON) command to the Blynk server.

The server instantly forwards this command to the ESP32.

The BLYNK_WRITE(V2) function in the code is triggered.

This function takes the 1 (ON) state and runs digitalWrite(relay2_pin, !1), which sends a 0 (LOW) signal to GPIO 12.

The relay module receives the LOW signal on IN2, activates its electromagnet, and the relay "clicks."

This "click" closes the internal switch, connecting the COM and NO terminals, which would turn on the user's connected light bulb or appliance.

Simultaneously, the small red indicator LED for Relay 2 on the module turns on, providing a safe visual confirmation that the circuit is working.

When the user taps the button again, the app sends a "V2=0" (OFF) command, and the ESP32 sends a HIGH signal, turning the relay off.
