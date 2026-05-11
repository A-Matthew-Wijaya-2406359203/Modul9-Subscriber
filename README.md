# Jawaban Pertanyaan Tutorial

## a. What is amqp?
AMQP adalah singkatan dari **Advanced Message Queuing Protocol**. Secara sederhana, AMQP itu protokol standar untuk komunikasi antar aplikasi lewat message broker (contohnya RabbitMQ). Dengan AMQP, aplikasi pengirim (producer) bisa mengirim pesan ke broker, lalu aplikasi penerima (consumer) bisa mengambil pesan tersebut tanpa harus terhubung langsung satu sama lain.

Menurut saya, kelebihan AMQP itu ada pada sifatnya yang rapi dan andal: pesan bisa diantrekan, di-routing, dan diproses secara asynchronous. Ini membantu saat membangun sistem terdistribusi supaya antar service tidak saling bergantung secara ketat.

## b. What does it mean? guest:guest@localhost:5672 , what is the first guest, and what is the second guest, and what is localhost:5672 is for?  
Bagian ini biasanya muncul di connection string AMQP, misalnya:
`amqp://guest:guest@localhost:5672`

Penjelasannya:
- **guest** pertama: username untuk login ke broker AMQP.
- **guest** kedua: password dari username tersebut.
- **localhost:5672**: alamat dan port broker yang dituju.
  - **localhost** artinya broker berjalan di komputer yang sama dengan aplikasi kita.
  - **5672** adalah port default AMQP (port standar yang dipakai RabbitMQ untuk koneksi AMQP non-TLS).

Jadi, keseluruhan string itu berarti aplikasi mencoba konek ke broker AMQP lokal, memakai kredensial username `guest` dan password `guest`.

![Slow Subscriber](image.png)
Berdasarkan sceenshot di atas, queued message graphnya meningkat ke 6 secara sesaat sebelum kembali ke 0. Hal ini menunjukan, di satu momen tersebut, RabbitMQ menerima lebih banyak message dari publisher lebih cepat daripada kecepatan subscriber memprosesnya. Namun, setelah subscriber memproses messagenya, queuenya kembali menjadi empty.