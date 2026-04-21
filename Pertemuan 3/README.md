# Laporan Praktikum — UART & I2C Communication

---

## IV. Pertanyaan Praktikum

---

## Bagian A: UART (Serial Communication)

### 1. Alur Input Keyboard hingga LED Bereaksi

| Tahapan | Penjelasan |
|--------|-----------|
| Input | User mengetik '1' atau '0' di Serial Monitor |
| Kirim | Data dikirim via USB ke Arduino (UART) |
| Buffer | Data ditampung di Serial Buffer |
| Proses | Arduino membaca dengan `Serial.read()` |
| Output | LED menyala (1) / mati (0) |

---

### 2. Fungsi Serial.available()

Digunakan untuk mengecek apakah ada data masuk sebelum dibaca.

Jika tidak digunakan:
- Serial.read() bisa membaca -1
- Program bisa error atau spam output

---

### 3. Mode Blink (Input '2')

```cpp
char data;
bool isBlinking = false;
unsigned long previousMillis = 0;
const long interval = 500;
int ledState = LOW;

void loop() {
  if (Serial.available() > 0) {
    data = Serial.read();

    if (data == '1') {
      isBlinking = false;
      digitalWrite(8, HIGH);
    }
    else if (data == '0') {
      isBlinking = false;
      digitalWrite(8, LOW);
    }
    else if (data == '2') {
      isBlinking = true;
    }
  }

  if (isBlinking) {
    unsigned long currentMillis = millis();
    if (currentMillis - previousMillis >= interval) {
      previousMillis = currentMillis;
      ledState = !ledState;
      digitalWrite(8, ledState);
    }
  }
}
4. Delay vs Millis

delay()

Program berhenti sementara
Tidak responsif

millis()

Tidak menghentikan program
Tetap bisa menerima input
Bagian B: I2C Communication
1. Cara Kerja I2C
Jalur	Fungsi
SCL	Clock
SDA	Data

Arduino = Master
LCD = Slave (0x27)

2. Potensiometer
3 pin: VCC, GND, Signal
Jika terbalik:
Tidak rusak
Hanya arah berubah
3. Program UART + I2C
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);
const int pinPot = A0;

void setup() {
  Serial.begin(9600);
  lcd.init();
  lcd.backlight();
}

void loop() {
  int adcVal = analogRead(pinPot);
  float voltage = adcVal * (5.0 / 1023.0);
  int percent = map(adcVal, 0, 1023, 0, 100);
  int barLength = map(adcVal, 0, 1023, 0, 16);

  Serial.print("ADC: ");
  Serial.println(adcVal);

  Serial.print("Volt: ");
  Serial.print(voltage);
  Serial.println(" V");

  Serial.print("Persen: ");
  Serial.print(percent);
  Serial.println("%");

  Serial.println("-----");

  lcd.setCursor(0, 0);
  lcd.print("ADC:");
  lcd.print(adcVal);
  lcd.print(" ");
  lcd.print(percent);
  lcd.print("%");

  lcd.setCursor(0, 1);
  for (int i = 0; i < 16; i++) {
    if (i < barLength) lcd.print((char)255);
    else lcd.print(" ");
  }

  delay(500);
}
Data Pengamatan
ADC	Volt (V)	Persen (%)
1	0.005	0%
21	0.10	2%
49	0.24	4%
74	0.36	7%
96	0.47	9%
