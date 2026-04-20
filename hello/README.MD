### Commit 1 Reflection Notes

Dalam tahap ini, saya telah berhasil mengimplementasikan fungsi `handle_connection` untuk membaca permintaan (request) HTTP yang masuk ke server. Berikut adalah poin-poin penting dari apa yang saya pelajari:

* BufReader: Saya menggunakan `BufReader` untuk membungkus `TcpStream`. Hal ini sangat membantu karena `BufReader` menyediakan cara yang efisien untuk membaca data teks baris demi baris dibandingkan pembacaan byte mentah secara langsung.
* Pemrosesan Baris: Kode menggunakan metode `.lines()` untuk memecah data masuk menjadi baris-baris individual. Saya kemudian menggunakan `.map(|result| result.unwrap())` untuk mengambil nilai string dari setiap baris dan menangani potensi error secara langsung.
* Logika Take While: Bagian `.take_while(|line| !line.is_empty())` sangat krusial dalam protokol HTTP. Karena request HTTP diakhiri dengan baris kosong, program menggunakan baris ini sebagai tanda untuk berhenti membaca header agar tidak terjadi proses menunggu selamanya (*blocking*).
* Pengumpulan Data: Terakhir, `.collect()` menggabungkan semua baris tersebut ke dalam sebuah `Vec<String>`. Saat dijalankan, terminal mencetak detail request seperti metode (GET), path (/), dan versi HTTP, yang membantu saya memahami struktur data yang dikirimkan oleh browser.