# TDS-Water-Quality-Monitor-
# Built a basic Water Quality Monitoring System using a TDS Sensor 💧  This project measures the TDS level in water to estimate water quality using Arduino and a TDS sensor.  Components used: • Arduino Uno • TDS Sensor • LCD Display
#upload this code in arduino uno ide

#include <Wire.h>
#include <LiquidCrystal_I2C.h>

#define TdsSensorPin A0       // Pin connected to the TDS sensor
#define VREF 5.0              // Analog reference voltage
#define SCOUNT 30             // Number of samples

int analogBuffer[SCOUNT];     
int analogBufferTemp[SCOUNT];
int analogBufferIndex = 0;

float averageVoltage = 0;
float tdsValue = 0;
float temperature = 25.0;
float x = 0;

LiquidCrystal_I2C lcd(0x27, 16, 2);   // LCD address 0x27, 16 columns, 2 rows

// Function Prototype
int getMedianNum(int bArray[], int iFilterLen);

void setup() {
    lcd.init();
    lcd.backlight();

    lcd.setCursor(0, 0);
    lcd.print("TDS Meter");

    delay(2000);

    Serial.begin(9600);
}

void loop() {

    static unsigned long analogSampleTimepoint = millis();

    // Sample analog values every 40 ms
    if (millis() - analogSampleTimepoint > 40U) {

        analogSampleTimepoint = millis();

        analogBuffer[analogBufferIndex] = analogRead(TdsSensorPin);

        analogBufferIndex++;

        if (analogBufferIndex == SCOUNT) {
            analogBufferIndex = 0;
        }
    }

    static unsigned long printTimepoint = millis();

    // Print TDS value every 800 ms
    if (millis() - printTimepoint > 800U) {

        printTimepoint = millis();

        // Copy buffer values
        for (int i = 0; i < SCOUNT; i++) {
            analogBufferTemp[i] = analogBuffer[i];
        }

        // Calculate average voltage
        averageVoltage = getMedianNum(analogBufferTemp, SCOUNT) 
                         * (float)VREF / 1024.0;

        // Temperature compensation
        float compensationCoefficient = 1.0 + 0.02 * (temperature - 25.0);

        float compensationVoltage = averageVoltage / compensationCoefficient;

        // TDS calculation
        tdsValue = (133.42 * compensationVoltage * compensationVoltage * compensationVoltage
                   - 255.86 * compensationVoltage * compensationVoltage
                   + 857.39 * compensationVoltage) * 0.5;

        // Calibration
        x = 2642.86 * tdsValue + 123.07;

        // Display on LCD
        lcd.clear();

        lcd.setCursor(0, 0);
        lcd.print("TDS: ");

        lcd.print(x, 2);
        lcd.print(" ppm");

        // Serial Monitor Output
        Serial.print("TDS Value: ");
        Serial.print(x, 2);
        Serial.println(" ppm");
    }
}

// Median Filtering Function
int getMedianNum(int bArray[], int iFilterLen) {

    int bTab[iFilterLen];

    for (int i = 0; i < iFilterLen; i++) {
        bTab[i] = bArray[i];
    }

    int i, j, bTemp;

    // Bubble sort
    for (j = 0; j < iFilterLen - 1; j++) {

        for (i = 0; i < iFilterLen - j - 1; i++) {

            if (bTab[i] > bTab[i + 1]) {

                bTemp = bTab[i];
                bTab[i] = bTab[i + 1];
                bTab[i + 1] = bTemp;
            }
        }
    }

    // Return median value
    if ((iFilterLen & 1) > 0) {

        bTemp = bTab[(iFilterLen - 1) / 2];

    } else {

        bTemp = (bTab[iFilterLen / 2] + bTab[iFilterLen / 2 - 1]) / 2;
    }

    return bTemp;
}
