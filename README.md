# BambangShop Receiver App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create SubscriberRequest model struct.`
    -   [x] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [x] Commit: `Implement add function in Notification repository.`
    -   [x] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Commit: `Implement receive_notification function in Notification service.`
    -   [x] Commit: `Implement receive function in Notification controller.`
    -   [x] Commit: `Implement list_messages function in Notification service.`
    -   [x] Commit: `Implement list function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1

## 1. Mengapa menggunakan `RwLock<>` dan bukan `Mutex<>`?

Dalam implementasi ini, saya menggunakan `RwLock<>` untuk menyinkronkan akses terhadap `Vec` yang berisi daftar notifikasi.

Alasan utama pemilihan `RwLock<>` dibandingkan `Mutex<>` adalah karena perbedaan cara keduanya menangani akses thread:

- `RwLock<>` (Read-Write Lock) memungkinkan **banyak thread melakukan pembacaan (read) secara bersamaan**, namun hanya mengizinkan **satu thread untuk menulis (write)** pada satu waktu.
- `Mutex<>` hanya mengizinkan **satu thread mengakses data pada satu waktu**, baik untuk operasi baca maupun tulis.

Dalam konteks aplikasi ini, operasi membaca (seperti menampilkan daftar notifikasi melalui request GET) terjadi jauh lebih sering dibandingkan operasi menulis (menambahkan notifikasi baru).

Dengan menggunakan `RwLock<>`, multiple request pembacaan dapat diproses secara paralel tanpa harus saling menunggu. Hal ini meningkatkan performa aplikasi secara signifikan.

Sebaliknya, jika menggunakan `Mutex<>`, setiap operasi baca tetap harus dilakukan secara bergantian, meskipun tidak ada proses penulisan yang sedang berlangsung. Hal ini dapat menyebabkan bottleneck dan menurunkan efisiensi sistem.

---

## 2. Mengapa Rust tidak mengizinkan mutasi langsung pada variabel `static` seperti Java?

Rust tidak mengizinkan mutasi langsung pada variabel `static` karena bahasa ini menerapkan prinsip **memory safety** yang ketat tanpa menggunakan garbage collector.

Variabel `static` bersifat global dan dapat diakses oleh banyak thread secara bersamaan. Jika mutasi diperbolehkan tanpa mekanisme pengamanan, maka berpotensi terjadi **data race**, yaitu kondisi di mana beberapa thread mengakses dan memodifikasi data yang sama secara bersamaan tanpa sinkronisasi.

Data race dapat menyebabkan **undefined behavior**, yang sangat berbahaya karena hasil eksekusi program menjadi tidak dapat diprediksi.

Untuk mencegah hal tersebut, Rust mewajibkan penggunaan mekanisme sinkronisasi eksplisit seperti:
- `Mutex<>`
- `RwLock<>`
- atau struktur data thread-safe lainnya

Selain itu, penggunaan `lazy_static` memungkinkan saya untuk mendefinisikan variabel `static` yang kompleks dengan inisialisasi yang dilakukan saat pertama kali digunakan. Meskipun demikian, Rust tetap mengharuskan data tersebut dibungkus dalam mekanisme sinkronisasi agar tetap aman digunakan dalam konteks multi-threading.

Dengan pendekatan ini, Rust memastikan bahwa akses terhadap data global tetap aman, terkontrol, dan bebas dari data race.

#### Reflection Subscriber-2

## 1. Eksplorasi di luar langkah tutorial (src/lib.rs)

Ya, saya telah mengeksplorasi file `src/lib.rs` di luar langkah yang dijelaskan dalam tutorial.

File ini berfungsi sebagai pusat definisi fungsi dan konstanta yang dapat digunakan di seluruh crate, seperti:
- `APP_CONFIG`
- `REQWEST_CLIENT`
- `Result`
- `compose_error_response`

Dalam implementasinya, `src/lib.rs` berperan sebagai **core utilities** dari aplikasi. Saya juga mengamati bahwa konfigurasi aplikasi dimuat dari environment variable menggunakan `lazy_static`, sehingga hanya diinisialisasi sekali saat pertama kali diakses.

Selain itu, HTTP client (`reqwest`) juga diinisialisasi satu kali sebagai **singleton**, sehingga dapat digunakan secara efisien oleh seluruh bagian aplikasi tanpa perlu membuat instance baru berulang kali.

Pendekatan ini merupakan contoh penerapan **Singleton Pattern** yang membantu meningkatkan efisiensi dan konsistensi dalam pengelolaan resource.

---

## 2. Peran Observer Pattern dalam memudahkan penambahan subscriber baru

Observer Pattern memudahkan penambahan subscriber baru karena tidak memerlukan perubahan pada sisi publisher (Main app).

Untuk menambahkan subscriber baru, saya hanya perlu:
1. Menjalankan instance baru dari Receiver app (dengan port dan nama yang berbeda).
2. Melakukan subscribe ke tipe produk tertentu melalui endpoint yang tersedia.

Setelah itu, Main app akan secara otomatis:
- Menyimpan subscriber tersebut dalam daftar
- Mengirimkan notifikasi setiap kali terjadi event yang relevan

Namun, saya juga mengamati bahwa jika terdapat lebih dari satu instance Main app, maka akan muncul tantangan baru. Hal ini disebabkan karena setiap instance memiliki state `SUBSCRIBERS` yang disimpan secara in-memory dan tidak saling terhubung.

Akibatnya:
- Subscriber yang terdaftar di satu instance tidak akan dikenali oleh instance lain
- Notifikasi tidak terkirim secara konsisten

Untuk mengatasi hal ini, diperlukan mekanisme berbagi state, seperti menggunakan **database eksternal** atau sistem penyimpanan terpusat lainnya.

---

## 3. Penggunaan Tests dan dokumentasi di Postman

Saya telah mengeksplorasi fitur **Tests** pada Postman untuk meningkatkan proses pengujian API.

Postman menyediakan kemampuan untuk menambahkan script berbasis JavaScript yang dapat digunakan untuk:
- Memvalidasi status code (misalnya memastikan response bernilai `200 OK`)
- Memastikan field tertentu terdapat dalam response body
- Melakukan pengecekan otomatis terhadap format data

Dengan adanya fitur ini, proses testing menjadi lebih efisien karena tidak perlu lagi melakukan pengecekan secara manual setiap kali endpoint dipanggil.

Dalam konteks proyek kelompok, penggunaan Postman Tests sangat membantu untuk:
- Menjaga konsistensi API
- Memastikan tidak terjadi regression setelah perubahan kode
- Mempermudah kolaborasi antar anggota tim