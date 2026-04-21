# Laporan Praktikum Sistem Mikrokontroler - Pertemuan 3
## Topik: Protokol Komunikasi Serial (UART & I2C)

---

## I. Analisis Komunikasi Serial UART

### 1. Mekanisme Kerja: Dari Perintah Keyboard ke Aksi LED
Proses pengiriman data hingga LED bereaksi mengikuti alur berikut:
* **Input User:** Karakter (seperti `'1'` atau `'0'`) dimasukkan melalui Serial Monitor.
* **Proses Transmisi:** Data dikirimkan secara sekuensial (bit-per-bit) lewat kabel USB menuju pin **RX** Arduino.
* **Buffer Penampung:** Karakter yang diterima disimpan sementara di dalam memori buffer.
* **Logika Program:** Fungsi `Serial.read()` menarik data dari buffer. Jika input adalah `'1'`, Arduino memerintahkan pin digital ke kondisi **HIGH**.
* **Hasil Akhir:** LED menyala karena mendapatkan beda potensial dari pin tersebut.

### 2. Peran Vital `Serial.available()`
Instruksi ini berfungsi untuk memvalidasi keberadaan data di buffer sebelum proses pembacaan dilakukan.
* **Risiko Tanpa Pengecekan:** Tanpa baris ini, Arduino akan terus mengeksekusi `Serial.read()`. Karena fungsi tersebut menghasilkan nilai `-1` saat kosong, program akan terus-menerus menjalankan logika *error* atau mencetak pesan "perintah tidak dikenal" tanpa henti.

### 3. Modifikasi Fitur: Mode Blink (Input '2')
Berikut adalah implementasi kode agar LED dapat berkedip secara otomatis tanpa menghentikan pembacaan serial:

```cpp
char dataMasuk;
bool modeBlink = false;
unsigned long prevTime = 0;
const long jeda = 500; 
int statusLampu = LOW;

void loop() {
  if (Serial.available() > 0) {
    dataMasuk = Serial.read();
    if (dataMasuk == '1') { modeBlink = false; digitalWrite(8, HIGH); }
    else if (dataMasuk == '0') { modeBlink = false; digitalWrite(8, LOW); }
    else if (dataMasuk == '2') { modeBlink = true; }
  }

  if (modeBlink) {
    unsigned long currentTime = millis();
    if (currentTime - prevTime >= jeda) {
      prevTime = currentTime;
      statusLampu = !statusLampu;
      digitalWrite(8, statusLampu);
    }
  }
}
```
4. Perbedaan delay() vs millis()
delay() (Blocking): Menghentikan seluruh aktivitas prosesor. Jika perintah baru dikirim saat jeda berlangsung, Arduino tidak akan merespons sampai waktu habis.
millis() (Non-blocking): Memungkinkan multitasking. Arduino tetap bisa memantau input serial sambil menghitung waktu kedipan secara bersamaan.

## II. Analisis Komunikasi I2C

## 1. Prinsip Master-Slave pada LCD
Komunikasi ini menggunakan dua jalur utama untuk menggerakkan layar LCD:
SDA (Serial Data): Jalur untuk mengirimkan informasi teks atau perintah visual dari Arduino ke alamat khusus LCD (misal 0x27).
SCL (Serial Clock): Sinyal sinkronisasi agar pengiriman bit data berjalan tepat waktu.

## 2. Efek Pemasangan Potensiometer Terbalik
Potensiometer memiliki tiga pin (VCC, Ground, dan Wiper).
Analisis: Jika pin VCC dan Ground tertukar, secara elektrik tidak merusak komponen. Namun, orientasi pembacaan nilai akan terbalik; putaran yang seharusnya menaikkan nilai ADC justru akan menurunkannya.

## 3. Integrasi Sistem: Monitoring ADC via Serial & I2C
Kode berikut menampilkan nilai sensor ke Serial Monitor sekaligus visualisasi bar graph di LCD:

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);
const int pPot = A0;

void setup() {
  Serial.begin(9600);
  lcd.init();
  lcd.backlight();
}

void loop() {
  int valADC = analogRead(pPot);
  float tegangan = valADC * (5.0 / 1023.0);
  int prsen = map(valADC, 0, 1023, 0, 100);
  int bar = map(valADC, 0, 1023, 0, 16);

  Serial.print("ADC: "); Serial.println(valADC);
  Serial.print("V: "); Serial.print(tegangan); Serial.println("V");

  lcd.setCursor(0, 0);
  lcd.print("ADC:"); lcd.print(valADC); lcd.print(" "); lcd.print(prsen); lcd.print("%");

  lcd.setCursor(0, 1);
  for (int i = 0; i < 16; i++) {
    if (i < bar) lcd.print((char)255);
    else lcd.print(" ");
  }
  delay(500);
}
```
### Tabel Pengamatan Konversi ADC

| Nilai ADC | Tegangan (Volt) | Persentase (%) |
| :---: | :---: | :---: |
| 1 | 0.005 V | 0% |
| 21 | 0.10 V | 2% |
| 49 | 0.24 V | 4% |
| 74 | 0.36 V | 7% |
| 96 | 0.47 V | 9% |
