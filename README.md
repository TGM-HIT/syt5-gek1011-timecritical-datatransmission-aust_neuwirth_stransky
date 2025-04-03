# Ampelsteuerung mit SPI-Kommunikation und FreeRTOS

Dieses Projekt erweitert die klassische "Ampel mit LEDs"-Übung um eine SPI-basierte Datenübertragung und erweiterte Fehlerbehandlung. Dabei wird der aktuelle Status der Ampel als 4-Bit-Code an einen SPI-Bus (z. B. zur Weiterverarbeitung durch einen Raspberry Pi) gesendet. Bei fehlerhafter Übertragung oder Portausfall wird automatisch ein Fehlerzustand (Gelb blinkend) aktiviert.

## Projektbeschreibung

- **Kommunikation:** Übertragung der Ampelzustände (Rot, Rot-Gelb, Grün, etc.) über den SPI-Bus mit präzise definierten 4-Bit-Codes.
- **Fehlererkennung:** Bei einem Ausfall des SPI-Bus (keine HIGH-Signale innerhalb von 60 ms) wird das System neu gestartet und der Fehlerzustand (Gelb blinkend) aktiviert.
- **Echtzeitbetrieb:** Einsatz eines Echtzeitbetriebssystems (z. B. FreeRTOS) zur Verwaltung von Tasks, Interrupts und zur Priorisierung zeitkritischer Prozesse.
- **Überwachung:** Ein Check-Task kontrolliert die Signalaktivität, während ein Logic Analyzer zur Überprüfung der Signalzeiten herangezogen wird.
- **Dokumentation:** Alle Konfigurationsdateien, Deployment-Anweisungen und Kommentare im Code werden im README.md dokumentiert.

## Voraussetzungen

- Grundverständnis der Mikrokontroller-Programmierung und des Flashens mittels Toolchains (z. B. pico-template).
- Kenntnisse in der sicheren Verkabelung von Embedded Devices.
- Erfahrung mit Interrupts und deren Priorisierung.
- Einsatz von Single-Board-Computern (z. B. Raspberry Pi) für die Weiterverarbeitung der Ampel-Daten.
- Basiswissen über Real-Time Operating Systems (RTOS), wie FreeRTOS, und deren Einsatz in Embedded Systemen.

## Installation & Deployment

1. **Toolchain einrichten:**  
   Stellen Sie sicher, dass die benötigte Entwicklungsumgebung (z. B. pico-template) korrekt installiert ist und der Mikrocontroller konfiguriert werden kann.

2. **Code-Deployment:**  
   Kompilieren Sie den Code über die Konsole und flashen Sie die erzeugte Firmware auf den Ziel-Mikrokontroller.

3. **Hardware-Setup:**

   - Schließen Sie die externen LEDs für die Ampelsteuerung an.
   - Verbinden Sie den SPI-Bus mit dem entsprechenden Slave-Gerät (z. B. Raspberry Pi).
   - Schließen Sie den Logic Analyzer zur Überwachung der SPI-Datenübertragung an.

4. **Systemstart:**  
   Nach dem Flashen startet das System. Der Check-Task überwacht kontinuierlich das SPI-Bus-Signal und löst im Fehlerfall einen Neustart aus, wobei der Fehlerzustand (Gelb blinkend) aktiviert wird.

## Konfiguration

- **Bit-Codierung der Ampelzustände:**  
  | Status | Code |
  |----------------|-------------|
  | Rot | 1-1-1-0 |
  | Rot-Gelb | 1-1-0-1 |
  | Grün | 0-0-1-0 |
  | Grün blinkend | 0-1-0-1 |
  | Gelb | 1-0-0-0 |
  | Gelb blinkend | 0-0-0-1 |

- **Zeitvorgaben:**  
  Es muss sichergestellt werden, dass die Zeit pro Bit exakt eingehalten wird.

- **Versionierung:**  
  Achten Sie stets auf die korrekte Versionsnummer des Programmes (z. B. _aur-1.2.18_).

## Fehlerbehandlung & Watchdog

- Ein integrierter Watchdog überwacht die Systemaktivität.
- Sollte mehr als 60 ms kein HIGH-Signal am SPI-Bus detektiert werden, erfolgt ein System-Neustart, beginnend mit dem Fehlerzustand (Gelb blinkend).
- Die Priorisierung der Interrupts und Tasks erfolgt über das eingesetzte RTOS (z. B. FreeRTOS).


# Code-Dokumentation: 

**GPIO-Pin Definitionen**

     #define RED_PIN    1
     #define YELLOW_PIN 2
     #define GREEN_PIN  3

Diese Makros definieren die GPIO-Pins, die für die roten, gelben und grünen Ampellichter verwendet werden.

**Task-Prioritäten**

    #define HIGH_PRIORITY 1
    #define LOW_PRIORITY  0

Zwei Prioritätsstufen werden definiert: HIGH_PRIORITY für aktive Zustände und LOW_PRIORITY für inaktive Tasks.


**Ampelzustände**

Jeder Zustand wird durch eine eigene Funktion repräsentiert, die die GPIO-Pins entsprechend setzt.

    Rot: stateRed()

    Rot-Gelb: stateRedYellow()

    Grün: stateGreen()

    Blinkendes Grün: stateGreenBlinking() (grünes Licht blinkt 5-mal)

    Gelb: stateYellow()

**FreeRTOS Task-Funktionen**

Jede Task wird dauerhaft ausgeführt, aber nur aktiv, wenn sie die höchste Priorität hat. Anschließende ist zum Beispiel die redTask angeführt.


    void redTask(void *pvParameters) {
        for (;;) {
            if (uxTaskPriorityGet(NULL) == HIGH_PRIORITY) {
                stateRed();
                vTaskDelay(pdMS_TO_TICKS(STATE_DURATION));
                vTaskPrioritySet(redYellowTaskHandle, HIGH_PRIORITY);
                vTaskPrioritySet(NULL, LOW_PRIORITY);
            } else {
                vTaskDelay(pdMS_TO_TICKS(100));
            }
        }
    }

**Falls die Task höchste Priorität hat:**

- stateRed() wird aufgerufen.

- Die Task wartet 4 Sekunden (STATE_DURATION).

- Die nächste Task (redYellowTaskHandle) wird priorisiert.

- Die eigene Priorität wird gesenkt.

- Falls die Task nicht aktiv ist, wartet sie 100 ms.

**Weitere Tasks**

Alle anderen Tasks folgen einem ähnlichen Ablauf und steuern die jeweiligen Zustände.

Hauptprogramm (main)

    int main() {
        stdio_init_all();

        xTaskCreate(redTask, "Red", 1024, NULL, HIGH_PRIORITY, &redTaskHandle);
        xTaskCreate(redYellowTask, "RedYellow", 1024, NULL, LOW_PRIORITY, &redYellowTaskHandle);
        xTaskCreate(greenTask, "Green", 1024, NULL, LOW_PRIORITY, &greenTaskHandle);
        xTaskCreate(greenBlinkTask, "GreenBlink", 1024, NULL, LOW_PRIORITY, &greenBlinkTaskHandle);
        xTaskCreate(yellowTask, "Yellow", 1024, NULL, LOW_PRIORITY, &yellowTaskHandle);

        vTaskStartScheduler();

        for (;;);
        return 0;
    }

- Initialisiert die UART-Schnittstelle (stdio_init_all()).

- Erstellt alle Tasks mit den entsprechenden Startprioritäten.

- Startet den FreeRTOS Scheduler (vTaskStartScheduler()), der die Aufgabenverwaltung übernimmt



