OverTheWire: Bandit — Level 0 to 5
Dokumentasi progres menyelesaikan level awal Bandit wargame. Fokus tiap writeup: command yang dipakai, konsep di baliknya, dan kenapa itu bekerja — bukan sekadar jawaban.

Level 0 → 1
Goal: Login ke server via SSH menggunakan kredensial yang diberikan, cari password untuk level berikutnya.
Command:
ssh bandit0@bandit.labs.overthewire.org -p 2220
cat readme

Konsep:
	•	SSH (Secure Shell) adalah protokol untuk remote login terenkripsi ke sistem lain lewat jaringan. Port default SSH adalah 22, tapi Bandit pakai port custom 2220 — ini kebiasaan umum di server production untuk mengurangi automated brute-force scan yang biasanya menyasar port default.
	•	cat adalah command dasar untuk menampilkan isi file ke stdout (terminal). File readme di home directory user bandit0 menyimpan password untuk level 1.
Password ditemukan di: ~/readme (home directory default user).

Level 1 → 2
Goal: Password disimpan di file bernama - (tanda dash/minus).
Command:
cat ./-

Konsep:
	•	Nama file - itu masalah karena di shell, dash di awal argumen biasanya diartikan sebagai flag/option, bukan nama file. Kalau langsung ketik cat -, shell akan menunggu input dari stdin (bukan baca file), karena - sering dipakai sebagai konvensi "baca dari stdin" di banyak command Unix.
	•	Solusinya: pakai path eksplisit ./- (menandakan "file bernama dash di direktori saat ini"), sehingga shell tidak salah interpretasi sebagai flag.

Level 2 → 3
Goal: Password disimpan di file dengan nama mengandung spasi: spaces in this filename.
Command:
cat "spaces in this filename"

Konsep:
	•	Shell (bash) menggunakan spasi sebagai delimiter/pemisah argumen. Kalau nama file punya spasi dan tidak di-escape, shell akan salah membaca satu nama file sebagai beberapa argumen terpisah.
	•	Solusi: bungkus nama file dengan tanda kutip ("..."), atau alternatif lain pakai backslash sebelum tiap spasi: cat spaces\ in\ this\ filename.

Level 3 → 4
Goal: Password ada di salah satu file dalam direktori inhere, tapi file-nya hidden (dimulai dengan titik).
Command:
cd inhere
ls -la
cat .hidden

Konsep:
	•	Di Linux, file/folder yang namanya diawali titik (.) dianggap hidden oleh konvensi shell — bukan fitur keamanan sungguhan, cuma default ls yang menyembunyikannya dari tampilan biasa.
	•	Flag -a (all) di ls menampilkan semua file termasuk yang hidden. Flag -l menampilkan format long listing (permission, owner, size, dll).

Level 4 → 5
Goal: Password ada di salah satu file dalam inhere, hanya satu file yang human-readable (isinya teks biasa), sisanya berupa data biner/non-teks.
Command:
cd inhere
file ./-file*

Konsep:
	•	Command file mendeteksi tipe konten sebuah file dengan membaca beberapa byte pertama (disebut magic number atau file signature), bukan cuma mengandalkan ekstensi nama file (karena Linux tidak bergantung pada ekstensi seperti Windows).
	•	Output file akan menunjukkan tipe seperti ASCII text untuk file yang bisa dibaca manusia, versus data untuk file biner. Ini mempercepat pencarian tanpa harus cat satu-satu file yang isinya garbage/non-printable characters.

Level 5 → 6
Goal: Password ada di file dalam direktori inhere dengan kriteria spesifik: human-readable, ukuran tepat 1033 bytes, dan tidak executable.
Command:
find inhere -type f -size 1033c ! -executable
cat inhere/<nama_file_hasil>

Konsep:
	•	find adalah tool pencarian file berdasarkan kriteria (bukan cuma nama, tapi bisa size, permission, waktu modifikasi, tipe, dll) — jauh lebih powerful daripada ls untuk direktori dengan banyak file/subfolder.
	•	-size 1033c artinya ukuran file persis 1033 bytes (suffix c = character/byte, beda dengan k untuk kilobyte).
	•	! -executable artinya negasi (exclude) file yang punya execute permission — filter tambahan untuk mempersempit hasil pencarian sesuai kriteria soal.

Refleksi
Level 0-5 ini fokus ke navigasi dasar dan quirks shell (spasi, dash, hidden file) plus pengenalan find sebagai tool pencarian berbasis kriteria. Konsep yang paling penting dipahami di sini bukan command-nya doang, tapi kenapa shell berperilaku seperti itu (parsing argumen, konvensi hidden file, magic number pada file) — ini fondasi yang bakal kepake terus di level-level lanjutan dan real-world enumeration.
