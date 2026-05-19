
# Percobaan 6A — External Interrupt

## 1. Jelaskan bagaimana tombol bisa mengubah kondisi LED menggunakan interrupt!

Ketika tombol ditekan, pin interrupt pada Arduino akan membaca perubahan sinyal dari HIGH ke LOW karena memakai mode `FALLING`. Setelah perubahan terdeteksi, Arduino langsung menjalankan ISR (*Interrupt Service Routine*) dan menghentikan sementara program utama. Di dalam ISR tersebut, nilai variabel `ledState` dibalik sehingga LED dapat berubah kondisi menjadi menyala atau mati. Setelah ISR selesai dijalankan, program kembali ke proses utama.

## 2. Apa kegunaan `attachInterrupt()` pada program?

Fungsi `attachInterrupt()` dipakai untuk menghubungkan pin interrupt dengan fungsi ISR yang akan dijalankan saat kondisi tertentu terjadi. Selain itu, fungsi ini juga menentukan jenis interrupt yang digunakan seperti `RISING`, `FALLING`, `CHANGE`, maupun `LOW`.

Contoh:

```cpp
attachInterrupt(digitalPinToInterrupt(2), tombolInterrupt, FALLING);
```

## 3. Kenapa `delay()` dan `Serial.print()` tidak dianjurkan di dalam ISR?

Karena saat ISR berjalan, program utama akan berhenti sementara. Jika di dalam ISR menggunakan `delay()` atau `Serial.print()`, proses interrupt menjadi lebih lama sehingga performa sistem bisa menurun, respon menjadi lambat, bahkan interrupt lain dapat terlewat.

## 4. Apa fungsi keyword `volatile` pada variabel `ledState`?

Keyword `volatile` digunakan supaya compiler selalu mengambil nilai terbaru dari variabel tersebut karena nilainya dapat berubah sewaktu-waktu di dalam ISR. Jika tidak memakai `volatile`, compiler bisa saja menggunakan nilai lama yang tersimpan sehingga hasil program menjadi tidak sesuai.

## 5. Modifikasi Program Menggunakan Mode Interrupt Lain

### a. Mode RISING

**Analisis**

Mode `RISING` akan aktif ketika sinyal berubah dari LOW menjadi HIGH. Pada penggunaan `INPUT_PULLUP`, interrupt biasanya terjadi saat tombol dilepas sehingga perubahan kondisi LED terjadi ketika tombol sudah tidak ditekan.

### b. Mode CHANGE

**Analisis**

Mode `CHANGE` bekerja setiap ada perubahan sinyal, baik dari HIGH ke LOW maupun LOW ke HIGH. Karena itu LED bisa berubah kondisi dua kali, yaitu saat tombol ditekan dan saat dilepas, sehingga terlihat lebih sensitif.

### c. Mode LOW

**Analisis**

Mode `LOW` akan terus menjalankan ISR selama kondisi pin berada di LOW. Akibatnya LED bisa berubah sangat cepat atau terlihat berkedip terus selama tombol masih ditekan.

# Percobaan 6B — Timer Menggunakan millis()

## 1. Jelaskan cara kerja fungsi `millis()` pada program!

Fungsi `millis()` digunakan untuk menghitung lama waktu sejak Arduino mulai dinyalakan dalam satuan milidetik. Pada program, nilai waktu sekarang dibandingkan dengan waktu sebelumnya untuk menentukan kapan LED harus berubah kondisi.

## 2. Apa perbedaan utama `delay()` dan `millis()`?

`delay()` akan menghentikan sementara seluruh proses program sesuai waktu yang ditentukan. Sedangkan `millis()` memungkinkan program tetap berjalan sambil menghitung waktu sehingga lebih fleksibel dan efisien.

## 3. Mengapa `millis()` disebut metode non-blocking?

Karena penggunaan `millis()` tidak menghentikan jalannya program utama. Arduino masih bisa menjalankan proses lain sambil tetap menghitung interval waktu.

## 4. Modifikasi Program Dua LED Tanpa delay()

**Analisis**

Program memakai dua timer berbeda menggunakan `millis()` sehingga kedua LED dapat berkedip dengan jeda waktu yang berbeda tanpa perlu memakai `delay()`. LED pertama berkedip setiap 1 detik, sedangkan LED kedua berkedip setiap 500 milidetik.
