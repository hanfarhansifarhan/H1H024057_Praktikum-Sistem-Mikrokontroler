# Jawaban Praktikum Modul 4

## Percobaan 1 : ADC dan Kontrol Motor Servo

### Jawaban Pertanyaan

### 1. Apa fungsi pembacaan analog pada Arduino?

Fungsi `analogRead()` digunakan untuk membaca tegangan analog dari pin input Arduino dan mengubahnya menjadi data digital. Nilai yang dihasilkan berada pada rentang `0–1023` karena ADC pada Arduino Uno memiliki resolusi 10-bit.

Contoh:
- `0` → tegangan 0 Volt
- `1023` → tegangan 5 Volt

---

### 2. Mengapa diperlukan fungsi `map()`?

Fungsi `map()` digunakan untuk menyesuaikan rentang nilai input dengan kebutuhan output tertentu.

Contoh penerapan pada percobaan:
- Nilai ADC : `0–1023`
- Sudut servo : `0–180`

Tanpa fungsi pemetaan, data ADC tidak dapat langsung digunakan untuk menentukan posisi servo secara akurat.

---

### 3. Modifikasi Program — Rentang Servo 20° sampai 160°

Berikut contoh kode untuk membatasi gerakan servo:

```cpp
#include <Servo.h>

Servo myServo;

void setup() {
  myServo.attach(9);
}

void loop() {
  int nilaiADC = analogRead(A0);

  int sudut = map(nilaiADC, 0, 1023, 20, 160);

  myServo.write(sudut);

  delay(15);
}
```

---

### Penjelasan

Program di atas membaca nilai analog dari potensiometer kemudian mengubahnya menjadi sudut servo menggunakan fungsi `map()`.

#### Langkah kerja:
1. Arduino membaca nilai analog dari potensiometer.
2. Data ADC dikonversi ke rentang sudut servo.
3. Servo bergerak sesuai posisi potensiometer.
4. Delay kecil diberikan agar pergerakan servo lebih stabil.

#### Hasil Pengamatan:
- Servo dapat bergerak secara halus.
- Gerakan servo terbatas hanya antara 20° hingga 160°.
- Sistem bekerja sesuai dengan perubahan posisi potensiometer.

---

# Percobaan 2 : PWM untuk Kontrol LED

## Jawaban Pertanyaan

### 1. Jelaskan fungsi PWM pada Arduino

PWM (*Pulse Width Modulation*) digunakan untuk menghasilkan sinyal digital yang menyerupai keluaran analog melalui pengaturan duty cycle.

Pada Arduino, PWM dimanfaatkan untuk:
- Mengatur tingkat kecerahan LED
- Mengontrol kecepatan motor
- Mengatur besar daya output tertentu

Pin PWM pada Arduino Uno biasanya ditandai simbol `~`.

---

### 2. Hubungan ADC dan PWM

Pada percobaan ini:
- ADC membaca input analog dari potensiometer.
- PWM menghasilkan output berupa intensitas cahaya LED.

Karena:
- ADC memiliki resolusi 10-bit → `0–1023`
- PWM memiliki resolusi 8-bit → `0–255`

Maka digunakan fungsi `map()` agar data ADC sesuai dengan kebutuhan PWM.

---

### Tabel Perbandingan Resolusi

| Fitur | Resolusi | Rentang |
|------|-----------|----------|
| ADC | 10-bit | 0 – 1023 |
| PWM | 8-bit | 0 – 255 |

---

### 3. Modifikasi Program — LED Menyala 30% sampai 90%

```cpp
void setup() {
  pinMode(9, OUTPUT);
}

void loop() {
  int sensor = analogRead(A0);

  int pwm = map(sensor, 0, 1023, 77, 230);

  analogWrite(9, pwm);

  delay(10);
}
```

---

### Penjelasan

Program membaca nilai analog dari potensiometer kemudian mengubahnya menjadi nilai PWM untuk LED.

#### Keterangan:
- Nilai PWM minimum `77` ≈ 30%
- Nilai PWM maksimum `230` ≈ 90%

#### Hasil:
- LED tidak pernah benar-benar mati.
- Intensitas cahaya berubah secara bertahap.
- Perubahan cahaya terlihat lebih stabil dan halus.

---

# Kesimpulan

Berdasarkan hasil praktikum, Arduino mampu:
- Membaca sinyal analog menggunakan ADC.
- Mengubah data analog menjadi kontrol aktuator.
- Menghasilkan sinyal PWM untuk pengaturan output.

Penggunaan fungsi `map()` sangat membantu dalam menyesuaikan perbedaan resolusi antara ADC dan PWM sehingga sistem dapat bekerja dengan lebih efektif dan presisi.

---
