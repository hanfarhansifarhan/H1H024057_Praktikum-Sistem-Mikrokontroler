# Praktikum FreeRTOS pada Arduino

## Pertanyaan 5A

### 1. Apakah ketiga task dijalankan secara bersamaan atau bergantian?

Pada board mikrokontroler dengan satu core seperti Arduino Uno (ATMega328P), seluruh task tidak benar-benar berjalan paralel. Task dieksekusi secara bergiliran dengan perpindahan yang sangat cepat sehingga tampak seperti berjalan bersamaan.

#### Mekanisme yang digunakan:

* **Context Switching**
  Kernel FreeRTOS menyimpan kondisi task yang sedang berjalan lalu berpindah ke task lain.

* **Time Slicing**
  Setiap task memperoleh alokasi waktu tertentu untuk menggunakan CPU. Ketika task menjalankan `vTaskDelay()`, task akan masuk ke status *blocked* sehingga task lain dapat diproses.

* **Preemptive Scheduling**
  Task yang memiliki prioritas lebih tinggi dapat menghentikan sementara task dengan prioritas lebih rendah agar sistem real-time tetap berjalan dengan baik.

---

### 2. Bagaimana cara menambahkan task baru?

#### Langkah-langkah:

1. **Membuat fungsi task baru**

```cpp
void TaskBaru(void *pvParameters) {
  for (;;) {
    // kode task
  }
}
```

2. **Menambahkan task di fungsi `setup()`**

```cpp
xTaskCreate(TaskBaru, "Task4", 128, NULL, 1, NULL);
```

3. **Mengatur parameter task**

Beberapa parameter yang perlu ditentukan:

* Nama fungsi task
* Nama task
* Ukuran stack
* Parameter task (`NULL`)
* Prioritas task
* Handle task

4. **Menuliskan logika program**

Tambahkan instruksi sesuai kebutuhan dan gunakan `vTaskDelay()` agar task tidak menggunakan CPU secara terus-menerus.

---

### 3. Modifikasi Program Menggunakan Potensiometer

Program berikut memanfaatkan dua task pada FreeRTOS, yaitu task untuk membaca nilai potensiometer dan task untuk mengatur kecepatan kedipan LED.

```cpp
#include <Arduino_FreeRTOS.h>

// ===== DEKLARASI TASK =====
void TaskBlink(void *pvParameters);
void TaskSensor(void *pvParameters);

// ===== VARIABEL GLOBAL =====
int delayLed = 200;

void setup() {
  Serial.begin(9600);
  pinMode(8, OUTPUT);

  xTaskCreate(TaskSensor, "Sensor", 128, NULL, 1, NULL);
  xTaskCreate(TaskBlink, "Blink", 128, NULL, 1, NULL);

  vTaskStartScheduler();
}

void loop() {
  // Tidak digunakan karena FreeRTOS memakai scheduler
}

void TaskSensor(void *pvParameters) {
  while (1) {
    int sensorValue = analogRead(A0);

    delayLed = map(sensorValue, 0, 1023, 50, 1000);

    Serial.print("Nilai Potensio = ");
    Serial.print(sensorValue);
    Serial.print(" | Delay LED = ");
    Serial.println(delayLed);

    vTaskDelay(100 / portTICK_PERIOD_MS);
  }
}

void TaskBlink(void *pvParameters) {
  while (1) {
    digitalWrite(8, HIGH);
    vTaskDelay(delayLed / portTICK_PERIOD_MS);

    digitalWrite(8, LOW);
    vTaskDelay(delayLed / portTICK_PERIOD_MS);
  }
}
```

### Penjelasan Program

| Bagian          | Keterangan                                                           |
| --------------- | -------------------------------------------------------------------- |
| Input           | Potensiometer terhubung ke pin `A0`                                  |
| Variabel Global | `delayLed` digunakan untuk menyimpan nilai delay LED                 |
| `TaskSensor`    | Membaca nilai analog dari potensiometer lalu memperbarui nilai delay |
| `TaskBlink`     | Mengatur kedipan LED berdasarkan nilai delay                         |

### Analisis Hasil

Ketika potensiometer diputar, nilai analog yang terbaca akan berubah dari 0 hingga 1023. Nilai tersebut kemudian dipetakan menjadi delay LED antara 50 ms sampai 1000 ms.

Akibatnya, kecepatan LED berkedip berubah sesuai posisi potensiometer.

### Contoh Output Serial Monitor

```txt
Nilai Potensio = 512 | Delay LED = 525
Nilai Potensio = 300 | Delay LED = 343
```

### Hasil Percobaan

* Kecepatan kedipan LED berubah mengikuti nilai potensiometer
* FreeRTOS mampu menjalankan task sensor dan LED secara concurrent
* Tidak terjadi konflik data karena hanya satu task yang memodifikasi variabel `delayLed`

---

## Pertanyaan 5B

### 1. Apakah kedua task berjalan secara bersamaan?

Kedua task dijalankan secara concurrent atau bergantian sangat cepat. Arduino Uno hanya memiliki satu inti prosesor sehingga task tidak benar-benar berjalan paralel.

Scheduler FreeRTOS bertugas mengatur perpindahan task menggunakan metode *time slicing*.

* Task `read_data` mengirim data ke Queue menggunakan `xQueueSend()`
* Task `display` menerima data dari Queue memakai `xQueueReceive()`
* Saat salah satu task berada pada kondisi *blocked*, scheduler akan menjalankan task lain yang siap dieksekusi

---

### 2. Apakah program dapat mengalami race condition?

Program ini tidak mengalami race condition karena pertukaran data antar task dilakukan melalui Queue.

Queue pada FreeRTOS menyediakan mekanisme komunikasi yang aman sehingga data dapat dikirim dan diterima tanpa bentrok.

* `read_data` hanya mengirim data ke Queue
* `display` hanya membaca data dari Queue
* Tidak ada penggunaan variabel global yang diakses secara bersamaan

> Race condition biasanya muncul ketika dua atau lebih task mengakses data yang sama tanpa sinkronisasi seperti Queue, Semaphore, atau Mutex.

---

### Program Pembacaan Sensor DHT22 Menggunakan Queue

```cpp
#include <Arduino_FreeRTOS.h>
#include <queue.h>
#include <DHT.h>

// ===== DHT SETUP =====
#define DHTPIN 2
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);

// ===== STRUCT =====
struct readings {
  float temp;
  float h;
};

// ===== QUEUE =====
QueueHandle_t my_queue;

void setup() {
  Serial.begin(9600);
  dht.begin();

  my_queue = xQueueCreate(1, sizeof(struct readings));

  xTaskCreate(read_data, "read sensors", 128, NULL, 1, NULL);
  xTaskCreate(display, "display", 128, NULL, 1, NULL);
}

void loop() {
  // Tidak digunakan karena scheduler FreeRTOS bekerja otomatis
}

void read_data(void *pvParameters) {
  struct readings x;

  for (;;) {
    x.temp = dht.readTemperature();
    x.h = dht.readHumidity();

    xQueueSend(my_queue, &x, portMAX_DELAY);

    vTaskDelay(1000 / portTICK_PERIOD_MS);
  }
}

void display(void *pvParameters) {
  struct readings x;

  for (;;) {
    if (xQueueReceive(my_queue, &x, portMAX_DELAY) == pdPASS) {
      Serial.print("Temperature = ");
      Serial.print(x.temp);
      Serial.println(" C");

      Serial.print("Humidity = ");
      Serial.print(x.h);
      Serial.println(" %");
    }
  }
}
```

### Contoh Output Serial Monitor

```txt
Temperature = 29.40 C
Humidity = 71.20 %

Temperature = 29.50 C
Humidity = 70.90 %
```

### Hasil Percobaan

* Nilai suhu dan kelembapan berubah mengikuti kondisi lingkungan
* Queue berhasil digunakan sebagai media komunikasi antar task
* FreeRTOS dapat menjalankan task pembacaan sensor dan tampilan data tanpa konflik akses
