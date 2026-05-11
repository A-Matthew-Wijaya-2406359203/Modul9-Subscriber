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

![Lots Subscriber 1](image-1.png)
![Lots Subscriber 2](image-2.png)

Pada percobaan ini, saya menjalankan RabbitMQ bersama dengan tiga instance subscriber dan satu publisher. Berdasarkan dashboard RabbitMQ, jumlah queued messages sempat naik sampai sekitar 5 message, lalu kembali lagi menjadi 0. Hal ini terjadi karena publisher mengirim beberapa message ke RabbitMQ, sehingga dalam waktu singkat message tersebut sempat menunggu di dalam queue.

Namun, karena ada tiga subscriber yang berjalan secara bersamaan, message dapat dikonsumsi dengan lebih cepat. Itulah mengapa jumlah queued messages akhirnya kembali menjadi 0. Spike pada grafik menunjukkan bahwa ada message yang masuk ke queue, sedangkan nilai akhir 0 menunjukkan bahwa semua message sudah berhasil diproses oleh subscriber.

Dari output terminal, setiap subscriber menerima data `UserCreatedEventMessage` yang berbeda, seperti message untuk Amir, Budi, Cica, Dira, dan Emir. Hal ini menunjukkan bahwa RabbitMQ membagi message ke beberapa subscriber yang aktif. Karena semua subscriber mendengarkan queue yang sama, RabbitMQ tidak mengirim setiap message ke semua subscriber. Sebaliknya, RabbitMQ membagi beban kerja ke consumer yang tersedia. Ini berguna karena semakin banyak subscriber yang aktif, semakin cepat juga message dapat diproses.

## Refleksi

Hasil percobaan ini menunjukkan salah satu manfaat penting dari event-driven architecture, yaitu publisher dan subscriber tidak perlu saling memanggil secara langsung. Publisher hanya perlu mengirim message ke RabbitMQ, lalu RabbitMQ yang bertanggung jawab mengatur pengiriman message ke subscriber. Dengan cara ini, sistem menjadi lebih fleksibel karena jumlah subscriber dapat ditambah tanpa perlu mengubah kode publisher.

## Hal yang Bisa Ditingkatkan

Setelah melihat kode publisher dan subscriber, ada beberapa hal yang menurut saya bisa ditingkatkan.

Pertama, pada kode subscriber terdapat infinite loop kosong:
Loop ini memang membuat program tetap berjalan, tetapi kurang efisien karena CPU terus menjalankan loop tanpa melakukan pekerjaan yang jelas. Akan lebih baik jika program menggunakan mekanisme blocking yang lebih tepat, atau setidaknya menggunakan sleep agar tidak membuang resource.

Kedua, kode masih menggunakan .unwrap() saat membuat listener. Cara ini sederhana, tetapi jika RabbitMQ belum berjalan atau koneksi gagal, program akan langsung panic. Akan lebih baik jika error ditangani dengan lebih jelas, misalnya dengan menampilkan pesan bahwa RabbitMQ belum aktif atau koneksi tidak berhasil.

Secara keseluruhan, percobaan ini menunjukkan bahwa RabbitMQ dapat membagi message ke beberapa subscriber dan membantu proses komunikasi asynchronous antar komponen. Dengan menjalankan minimal tiga subscriber, pemrosesan message menjadi lebih cepat dan queue tidak menumpuk terlalu lama.

## Bonus: Running RabbitMQ on Cloud using Railway
![RabbitMQ di Railway](image-3.png)

Pada bagian bonus ini, RabbitMQ tidak dijalankan di local machine menggunakan Docker Desktop, tetapi dijalankan di cloud menggunakan Railway. Publisher dan subscriber tetap dijalankan dari WSL di komputer lokal, tetapi keduanya terhubung ke RabbitMQ yang berjalan di Railway melalui koneksi AMQP.

Di sisi terminal, subscriber berhasil menerima beberapa `UserCreatedEventMessage` yang dikirim oleh publisher. Message yang diterima berisi beberapa data user seperti Amir, Budi, Cica, Dira, dan Emir. Ini membuktikan bahwa publisher berhasil mengirim message ke RabbitMQ di Railway, lalu subscriber berhasil mengambil dan memproses message tersebut dari queue.

Pada dashboard RabbitMQ juga terlihat terdapat `Connections: 1` dan `Queues: 2`. Jumlah connection menunjukkan bahwa ada aplikasi yang sedang terhubung ke RabbitMQ, yaitu subscriber yang masih berjalan dan mendengarkan message. Sementara itu, queue terbentuk karena subscriber membuat queue untuk menerima message dari publisher. Setelah publisher dijalankan, message langsung diterima oleh subscriber, sehingga tidak terlihat penumpukan message yang lama pada queue.

Dari percobaan ini, saya memahami bahwa message broker tidak harus berjalan di mesin yang sama dengan publisher dan subscriber. RabbitMQ dapat dijalankan di cloud, sedangkan publisher dan subscriber dapat berjalan dari komputer lokal selama connection string diarahkan ke alamat broker yang benar. Dengan pendekatan ini, arsitektur menjadi lebih fleksibel karena setiap komponen dapat berjalan di environment yang berbeda.