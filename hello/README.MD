### Commit 1 Reflection Notes

Dalam tahap ini, saya telah berhasil mengimplementasikan fungsi `handle_connection` untuk membaca permintaan (request) HTTP yang masuk ke server. Berikut adalah poin-poin penting dari apa yang saya pelajari:
* BufReader: Saya menggunakan `BufReader` untuk membungkus `TcpStream`. Hal ini sangat membantu karena `BufReader` menyediakan cara yang efisien untuk membaca data teks baris demi baris dibandingkan pembacaan byte mentah secara langsung.
* Pemrosesan Baris: Kode menggunakan metode `.lines()` untuk memecah data masuk menjadi baris-baris individual. Saya kemudian menggunakan `.map(|result| result.unwrap())` untuk mengambil nilai string dari setiap baris dan menangani potensi error secara langsung.
* Logika Take While: Bagian `.take_while(|line| !line.is_empty())` sangat krusial dalam protokol HTTP. Karena request HTTP diakhiri dengan baris kosong, program menggunakan baris ini sebagai tanda untuk berhenti membaca header agar tidak terjadi proses menunggu selamanya (*blocking*).
* Pengumpulan Data: Terakhir, `.collect()` menggabungkan semua baris tersebut ke dalam sebuah `Vec<String>`. Saat dijalankan, terminal mencetak detail request seperti metode (GET), path (/), dan versi HTTP, yang membantu saya memahami struktur data yang dikirimkan oleh browser.

### Commit 2 Reflection Notes

Pada tahap ini, saya telah memodifikasi fungsi `handle_connection` agar server tidak hanya membaca *request*, tetapi juga memberikan *respons* berupa file HTML nyata ke browser. Berikut adalah poin-poin penting yang saya pelajari:
* Pemisahan Tampilan dan Logika: Saya menggunakan `fs::read_to_string("hello.html")` untuk membaca konten dari file eksternal. Pendekatan ini jauh lebih rapi dan terstruktur dibandingkan menulis kode HTML secara hardcode di dalam string Rust.
* Struktur Respons HTTP: Saya belajar bahwa untuk mengirimkan respons yang valid, kita harus mengikuti format protokol HTTP secara ketat. Format tersebut meliputi `Status-Line` (misal: `HTTP/1.1 200 OK`), diikuti oleh `Headers`, sebuah baris kosong (`\r\n\r\n`) sebagai pemisah, dan diakhiri dengan `Message Body` (isi file HTML).
* Pentingnya Content-Length**: Menambahkan header `Content-Length` sangat penting untuk memberi tahu browser seberapa besar ukuran data yang akan dikirim. Tanpa ini, browser mungkin tidak tahu kapan proses transfer data selesai.
* Pengiriman dalam Bentuk Byte: Karena koneksi TCP beroperasi pada level byte (bukan teks/string), maka seluruh string respons yang sudah diformat harus dikonversi terlebih dahulu menjadi array of bytes menggunakan metode `.as_bytes()` sebelum dikirimkan melalui `stream.write_all()`.

Berikut adalah *screenshot* hasil halaman web yang berhasil dirender di browser:

![Commit 2 screen capture](/assets/images/commit2.png)

### Commit 3 Reflection Notes

Pada milestone ini, saya menambahkan logika validasi request dan memisahkan respons berdasarkan rute URL. Berikut adalah hal yang saya pelajari, terutama terkait proses *refactoring*:
* Validasi Rute (Routing Dasar): Saya belajar mengambil baris pertama dari request HTTP (`request_line`) untuk mengecek apakah pengguna meminta rute utama (`GET / HTTP/1.1`). Jika benar, server merespons dengan status `200 OK` dan `hello.html`. Jika berbeda, server mengembalikan status `404 NOT FOUND` dan halaman `404.html`.
* Pentingnya Refactoring: Pada awalnya, jika kita menggunakan blok `if-else` mentah, kode untuk membaca file (`fs::read_to_string`) dan mengirim respons (`stream.write_all`) akan ditulis dua kali (satu di blok `if`, satu di blok `else`). Hal ini melanggar prinsip DRY (Don't Repeat Yourself).
* Eksekusi Refactoring: Saya melakukan refactoring dengan membuat variabel tuple `let (status_line, filename)` yang nilainya bergantung pada hasil evaluasi `if-else`. Dengan cara ini, kode pembacaan file dan pengiriman format HTTP cukup ditulis satu kali di bagian akhir fungsi. Ini membuat kode jauh lebih bersih, ringkas, dan mudah di-*maintain* jika nantinya ada tambahan file HTML baru.

![Commit 3 screen capture](/assets/images/commit3.png)