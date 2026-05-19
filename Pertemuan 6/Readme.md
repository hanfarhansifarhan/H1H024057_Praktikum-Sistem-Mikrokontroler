# Percobaan 6A — External Interrupt

## 1. Jelaskan proses bagaimana tombol dapat mengubah kondisi LED menggunakan interrupt!

Saat tombol ditekan, pin interrupt pada Arduino akan mendeteksi perubahan sinyal dari HIGH menjadi LOW karena memakai mode `FALLING`. Interrupt kemudian menghentikan sementara program utama dan menjalankan ISR (*Interrupt Service Routine*). Di dalam ISR, variabel `ledState` diubah sehingga kondisi LED dapat berubah menjadi ON atau OFF. Setelah ISR selesai dijalankan, program kembali melanjutkan loop utama.

## 2. Apa fungsi `attachInterrupt()` pada program tersebut?

Fungsi `attachInterrupt()` dipakai untuk menghubungkan pin interrupt dengan ISR yang akan dijalankan ketika terjadi kondisi tertentu. Fungsi ini juga menentukan mode interrupt seperti `RISING`, `FALLING`, `CHANGE`, ataupun `LOW`.

Contoh:

```cpp
attachInterrupt(digitalPinToInterrupt(2), tombolInterrupt, FALLING);
```

## 3. Mengapa pada ISR tidak disarankan menggunakan `delay()` dan `Serial.print()`?

Karena selama ISR berjalan, program utama akan berhenti sementara. Jika menggunakan `delay()` atau `Serial.print()`, ISR menjadi lebih lama sehingga sistem bisa menjadi lambat, kurang responsif, atau menyebabkan interrupt lain tidak terbaca.

## 4. Apa fungsi keyword `volatile` pada variabel `ledState`?

Keyword `volatile` digunakan supaya compiler selalu membaca nilai terbaru dari variabel karena nilainya bisa berubah sewaktu-waktu di dalam ISR. Tanpa `volatile`, compiler dapat menyimpan nilai lama sehingga program tidak berjalan dengan semestinya.

## 5. Modifikasi Program Menggunakan Mode Interrupt Lain

### a. Mode RISING

**Analisis**

Mode `RISING` aktif ketika sinyal berubah dari LOW ke HIGH. Pada tombol dengan `INPUT_PULLUP`, interrupt biasanya terjadi saat tombol dilepas sehingga LED berubah kondisi ketika tombol sudah dilepas.

### b. Mode CHANGE

**Analisis**

Mode `CHANGE` aktif setiap terjadi perubahan sinyal HIGH ke LOW maupun LOW ke HIGH. Akibatnya LED bisa berubah dua kali saat tombol ditekan dan dilepas sehingga LED terlihat lebih sensitif.

### c. Mode LOW

**Analisis**

Mode `LOW` akan terus menjalankan ISR selama pin berada dalam kondisi LOW. Akibatnya LED dapat berkedip sangat cepat atau berubah terus selama tombol ditekan.

# Percobaan 6B — Timer Menggunakan millis()

## 1. Jelaskan bagaimana fungsi `millis()` bekerja pada program tersebut!

Fungsi `millis()` menghitung waktu sejak Arduino dinyalakan dalam satuan milidetik. Program kemudian membandingkan waktu sekarang dengan waktu sebelumnya untuk menentukan kapan LED harus berubah kondisi.

## 2. Apa perbedaan utama antara `delay()` dan `millis()`?

`delay()` menghentikan seluruh program selama waktu tertentu, sedangkan `millis()` memungkinkan program tetap berjalan sambil menghitung waktu sehingga penggunaannya lebih efisien.

## 3. Mengapa metode `millis()` disebut non-blocking?

Karena `millis()` tidak menghentikan jalannya program utama. Arduino tetap dapat menjalankan proses lain sambil menghitung interval waktu.

## 4. Modifikasi Program Dua LED Tanpa delay()

**Analisis**

Program menggunakan dua timer berbeda dengan `millis()` sehingga kedua LED dapat berkedip dengan interval berbeda tanpa memakai `delay()`. LED pertama berkedip setiap 1 detik dan LED kedua setiap 500 ms.
