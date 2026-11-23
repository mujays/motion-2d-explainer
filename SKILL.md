---
name: motion-2d-explainer
description: Bikin video motion explainer 2D flat-vector untuk topik apa pun dari voice over dan/atau naskah (VO ditranskrip offline dengan Whisper base via onnxruntime). Animasi kaya (aliran, gerak, transformasi, hitungan), highlight caption menyala saat kata diucapkan, latar mengikuti topik, konsep desain tetap. Format bisa diatur.
---

# Motion 2D Explainer (flat-vector, topik apa pun, caption + SFX)

Kamu tidak menggambar frame sendiri. Kamu menulis SATU spec JSON lalu merendernya dengan mesin `explainer.py`.

**Konsep desain yang selalu sama (jangan diubah):**
- ilustrasi vektor flat: isi warna solid + satu bayangan + satu highlight, garis tepi membulat 2–3 px;
- satu take kamera tanpa potongan, pindah bahasan lewat pan/zoom, dan "menyelam" untuk level mikro;
- caption 2–4 kata dengan satu kata kunci yang menyala kuning TEPAT saat diucapkan;
- animasi yang hidup: proses ditunjukkan sebagai proses (mengalir, bergerak, berubah, tumbuh, menghitung);
- label "│NAMA" dengan garis penunjuk, kartu perbandingan, angka besar, cincin pulse;
- judul hook di awal, outro takeaway + CTA, SFX otomatis.

**Yang mengikuti topik:** latar/setting, palet objek, dan apa yang digambar. Planet → `space`, masak → ruangan
dapur di latar `peach`, kesehatan → `body`, teknik → `blueprint`, pelajaran anak → `kids` atau `paper`, dan
seterusnya. Objek yang tidak ada di katalog digambar sebagai SVG dengan gaya yang sama.

Balas dalam bahasa user (default Bahasa Indonesia), singkat dan praktis.
Output: MP4 (default 9:16, 720x1280, 30 fps) + SRT + spec.json + preset.json.
Buat task list. Kerjakan langsung; satu-satunya pertanyaan yang boleh adalah konfirmasi format (langkah 1) dan naskah
kalau tidak bisa didapat.

## 0. Siapkan mesin (sekali per percakapan)

1. `P=<scratchpad>/motion2d-<slug>`; `mkdir -p $P/job`.
2. Mesin render ada di folder `engine/` di dalam skill ini. Cari foldernya:
   `find / -path "*motion-2d-explainer*" -name explainer.py 2>/dev/null | head -1`, lalu salin folder `engine` yang
   memuat file itu: `cp -r <folder skill>/engine $P/`.
   Contoh spec (WAJIB baca minimal satu sebelum menulis spec): `engine/examples/fotosintesis.json`
   (contoh utama: kaya animasi, stream/move/transformasi/zoom mikro), `roti.json` (latar terang, ruangan),
   `tatasurya.json` (luar angkasa), `preset_contoh.json`.
3. Kalau `explainer.py` tidak ditemukan: minta user mengunggah ulang skill dari
   https://github.com/mujays/motion-2d-explainer (Code → Download ZIP). Jangan menulis ulang mesin sendiri.
4. Mesin = `E=$P/engine/explainer.py`. Baca `SPEC_REFERENCE.md` sampai habis sebelum menulis spec (latar, token warna,
   katalog objek dan prop, aturan SVG, aksi, layout per rasio, aturan gaya).
5. Butuh Python 3 + numpy, Pillow, opencv (`cv2`), ffmpeg. Kalau `cv2` tidak ada:
   `pip install --break-system-packages opencv-python-headless`.

## 1. FORMAT DULU, sebelum generate apa pun

User bisa menempel blok seperti ini (urutan bebas, boleh sebagian, boleh bahasa Indonesia/Inggris):

```
FORMAT VIDEO
Rasio      : 9:16            (9:16 | 16:9 | 1:1 | 4:5 | portrait | landscape | square)
Kualitas   : 720p            (720p | 1080p)
FPS        : 30              (24 | 30 | 60)
Subtitle   : ya              (ya | tidak)
  Posisi   : bawah           (bawah | tengah | atas)
  Kata     : 3               (1–5 kata per tampilan)
  Highlight: ya              (ya = kata kunci menyala saat diucapkan | karaoke | statis | tidak)
  Ukuran   : sedang          (kecil | sedang | besar)
  Kapital  : tidak           (HURUF KAPITAL: ya | tidak)
Judul      : ya              (hook di awal)
Outro      : ya              (takeaway di akhir)
CTA        : FOLLOW UNTUK LAINNYA   (teks sendiri | tidak)
SFX        : ya, normal      (ya | tidak ; pelan | normal | keras)
Ambience   : ya              (loop hujan/angin kalau adegannya cocok)
Watermark  : tidak           (@akun + kanan-atas/kiri-atas/kanan-bawah/kiri-bawah | tidak)
Latar      : otomatis        (otomatis | nama preset, mis. paper, kids, space, body, blueprint | warna "#hex")
Bahasa     : Indonesia       (Indonesia | Inggris)
Input      : audio+naskah    (audio | audio+naskah | audio+srt | naskah | topik)
```

Aturan:
- **Ada blok** → baca, isi yang kosong dengan default di atas, tampilkan pengaturan final dalam SATU baris ringkas,
  lanjut tanpa bertanya.
- **Tidak ada blok** → tampilkan pengaturan default sebagai daftar pendek, tanya SEKALI: "Pakai pengaturan ini, atau
  ada yang mau diubah?" (pakai AskUserQuestion kalau ada). Lanjut setelah dikonfirmasi atau dijawab "default". Kalau
  user jelas tidak sedang menunggu, pakai default dan sebutkan.
- **User melampirkan preset JSON** (mis. `preset_channel.json`) → itulah pengaturannya.
- Simpan pengaturan final ke `$P/job/preset.json` (objek `format`). Pakai lagi untuk video berikutnya di percakapan
  ini kecuali user mengubahnya.

Pemetaan ke `"format"` di spec:

| Blok | spec |
|---|---|
| Rasio | `format.aspect`: "9:16", "16:9", "1:1", "4:5", "3:4" |
| Kualitas | `format.quality`: 720 atau 1080 |
| FPS | `format.fps` |
| Subtitle tidak | `format.captions: false` |
| Detail subtitle | `format.captions: {"show": true, "position": "bottom"\|"middle"\|"top", "words": 3, "highlight": true, "mode": "spoken"\|"karaoke"\|"static", "size": "kecil"\|"sedang"\|"besar", "uppercase": false}` (Highlight ya → mode spoken, karaoke → karaoke, statis → static, tidak → highlight false) |
| Judul / Outro | `format.title`, `format.outro` (true/false) |
| CTA | `format.cta: "TEKS"`; `false` = tanpa tombol |
| SFX | `format.sfx`, `format.sfx_volume_db`: pelan −6, normal 0, keras +3 |
| Ambience | `format.ambience` true/false |
| Watermark | `format.watermark: "@akun"`, `format.watermark_pos`: top-right\|top-left\|bottom-right\|bottom-left |
| Latar | otomatis = kamu pilih `spec.background` sesuai topik; nama preset atau `{"top":"#hex","bottom":"#hex","pattern":"dots"}` = pakai itu (simpan juga di `format.background` supaya ikut preset channel) |
| Bahasa | `spec.lang` = "id"/"en"; label, judul, outro ditulis dalam bahasa itu |

Walau subtitle mati, mesin tetap menulis `video.srt`; berikan ke user untuk caption platform.

## 2. Input → mode

Simpan file user ke `$P/job/` dengan nama sederhana (`voice.mp3`, `captions.srt`). Konversi format audio lain ke mp3
dengan ffmpeg.

- **Ada audio voice over (dengan atau tanpa naskah)** → SELALU transkrip dulu dengan **Whisper "base"** (model
  OpenAI, versi ONNX dari proyek sherpa-onnx). Model berjalan offline di sini lewat `onnxruntime`
  (`engine/whisper_ort.py`), tanpa torch dan tanpa paket sherpa-onnx:
  ```bash
  pip install --break-system-packages -q onnxruntime
  mkdir -p $P/asr && curl -sSL https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/sherpa-onnx-whisper-base.tar.bz2 | tar xj -C $P/asr
  python3 $E $P/job/voice.mp3 --transcribe $P/asr/sherpa-onnx-whisper-base --lang id -o $P/job/transcript.txt \
          [--compare $P/job/naskah.txt]
  ```
  `--lang` = bahasa VO (`id`, `en`, … atau `auto`). Kalau user memberi naskah, simpan dulu sebagai `naskah.txt` dan
  pakai `--compare`: mesin menampilkan persentase kecocokan kata dan daftar bedanya.
  Cara memakai hasilnya:
  - **Ada naskah** → teks caption tetap dari naskah (ejaan benar). Baca daftar beda:
    salah dengar Whisper (mis. "Suria"/"surya", "Panus"/"Venus", kata nempel/terpisah) → abaikan.
    Kalimat/frasa yang benar-benar tidak diucapkan, ditambah, atau diganti di VO → sesuaikan naskah dengan yang
    DIUCAPKAN (ejaan tetap rapi) supaya sinkronnya pas, lalu beri tahu user bagian mana yang berbeda.
  - **Tanpa naskah** → transkrip jadi naskah setelah kamu koreksi: ejaan, nama, istilah, angka, tanda baca akhir
    kalimat (jeda kalimat dipakai untuk sinkron). Tunjukkan hasil koreksinya ke user dalam satu blok singkat.
  Hasil akhirnya jadi `"script"` + `"audio"`: caption dan beat sinkron otomatis, suara ikut dimix.
  Whisper base cepat (±10–15 detik untuk VO 30–50 detik) tapi untuk bahasa Indonesia sering salah eja nama/istilah;
  kalau transkrip tanpa naskah terlalu berantakan, pakai `sherpa-onnx-whisper-small` (±640 MB, lebih akurat) dengan
  perintah yang sama. Kalau download model gagal, minta naskah atau SRT. Jangan pernah mengarang transkrip.
- **Audio + SRT** (mis. auto-caption CapCut) → `"audio"` + `"srt"`, tetap isi `"script"` dengan teks polos.
- **Naskah saja** → timing diperkirakan; video punya SFX tapi tanpa suara. Sebutkan ini, dan tawarkan sinkron ulang
  setelah user mengunggah VO dari naskah yang persis sama.
- **Topik saja** → tulis naskah dulu: durasi sesuai permintaan (default 60–90 detik, ±140–200 kata), dibuka pertanyaan
  yang bikin penasaran, menjelaskan langkah demi langkah dengan contoh/angka konkret, ditutup takeaway satu kalimat.
  Tampilkan, lalu lanjut kecuali user ingin mengubah.

## 3. Rancang spec (`$P/job/spec.json`)

1. Masukkan objek `"format"` dari langkah 1. Isi `"design_aspect"` dengan rasio yang kamu rancang.
2. **Pilih setting untuk topiknya** (referensi §1b): `background` preset atau warna kustom, ditambah `room` untuk
   adegan dalam ruangan. Pilih yang paling mudah dipahami penonton dan cocok dengan target (anak-anak → `kids`/`paper`/
   `peach`; serius/teknis → `studio`/`blueprint`/`dark`).
3. **Tulis beat sheet dulu** (di kepala atau di file kerja, tidak perlu ditunjukkan ke user): tabel
   `kalimat naskah → apa yang BERGERAK di layar`. Setiap kalimat minimal satu animasi objek (bukan cuma kamera/label):
   - aliran/proses → `stream` (air naik, gas masuk/keluar, sinar, darah, listrik, uang, data);
   - benda berpindah → `move` (lewat `path` yang mulus, `orient` untuk kendaraan/hewan);
   - A berubah jadi B → `move` keduanya ke satu titik, `hide`, lalu `show` B + `flash`;
   - tumbuh/membesar/memanas → `set` (`scale`, `w`, `hot`, `lit`, `highlight`);
   - benda disebut namanya → `emphasize` (punch) atau `glow`/`pulse` pada kata itu;
   - sebab → akibat → `arrow`; angka → `count`; perbandingan → `card`; salah/hidup → `wiggle`;
   - kemunculan → `show` dengan `anim` yang bervariasi (grow, slide-*, drop, spin, wipe, pop).
   Aturan lengkap: referensi §4b. Target: tidak ada jeda > 3 detik tanpa animasi objek, ≥ 0,5 beat animasi/detik,
   minimal 3 jenis dari stream / move / arrow / emphasize / wiggle / count.
4. Rancang SATU dunia yang memuat semua langkah, ditata untuk rasio yang dipilih. Pakai objek bawaan dan `prop` kalau
   cocok; gambar sisanya sebagai `svg` dengan gaya rumah (referensi §3b: isi flat, satu bayangan, satu highlight,
   `stroke="outline"`). Objek besar dan jelas (subjek utama ±40–60 % lebar layar). Sembunyikan (`hidden`) yang baru
   muncul belakangan; siapkan detail mikro dengan `min_zoom` (sel, stomata, molekul) untuk adegan zoom 5–6.
5. Rute kamera: mulai lebar, zoom ke tiap langkah, "menyelam" untuk level mikro, akhiri lebar. Beat dijangkarkan ke
   kata yang disalin persis dari naskah, berurutan; taruh `emphasize`/`glow` tepat di kata benda yang disebut.
   Label 1–2 kata dalam bahasa narasi, maksimal 2 di layar.
6. `meta.title` = hook 2 baris pendek; `meta.outro` = takeaway 2 baris pendek; `meta.cta` = teks CTA.
7. `highlight` = 15–35 kata kunci topik (satu per potongan caption; menyala kuning saat kata itu diucapkan). `ambience` hanya kalau hujan/angin tampil hampir sepanjang video.
8. Jangan gambar logo merek asli atau karakter terkenal; buat versi orisinal yang generik.

## 4. Cek → pratinjau → render

```bash
python3 $E $P/job/spec.json --check     # perbaiki SEMUA peringatan "!" (anchor, id, laporan gerak/motion)
python3 $E $P/job/spec.json --preview 1,5,10,15,20,25,30,40,50   # -> $P/job/spec_preview.png
```
Pratinjau minimal 1 frame per kalimat. Buka `spec_preview.png` dengan Read dan PERIKSA: latar cocok dengan topik?
subjek besar dan jelas? di tiap frame ada sesuatu yang sedang bergerak/berubah (aliran, benda berpindah, objek baru)?
SVG terbaca jelas dan bergaya flat yang sama? ada yang tertutup caption/judul? label keluar layar atau bertumpuk?
Perbaiki lalu pratinjau lagi (maks 2 putaran). Lalu:
```bash
python3 $E $P/job/spec.json -o $P/job/video.mp4
```
Waktu render ±0,5× durasi video di 720p (±1× di 1080p atau 60 fps). Video panjang: `--part 1/3`, `--part 2/3`,
`--part 3/3`, lalu `--finalize 3` (dengan `-o` yang sama).
Versi rasio lain dari video yang sama ("buat juga versi 16:9"): salin spec, ubah hanya `format.aspect`, biarkan
`design_aspect` seperti semula; mesin otomatis menyesuaikan kamera. Pratinjau, ubah beat kamera hanya kalau perlu.

## 5. Serahkan

Kalau ada audio, cek sinkron sebelum menyerahkan: `--timing` menampilkan waktu tiap potongan caption; awal tiap
kalimat harus jatuh di awal suara kalimat itu.
Kirim dengan SendUserFile: `video.mp4` (utama), `video.srt`, `spec.json`, `preset.json`. Satu–dua kalimat saja:
durasi, format (mis. "9:16 · 1080p · latar kids · subtitle 3 kata"), mode, adegan. Tawarkan revisi.

## 6. Revisi

Ubah hanya yang diminta (latar, objek, beat kamera, label, satu pengaturan format, judul…), jalankan `--check`, render
ulang. Sisanya tetap identik supaya seri konsisten. Edit kode mesin hanya kalau error, lalu beri tahu user apa yang
diubah.
