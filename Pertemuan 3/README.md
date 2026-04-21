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
