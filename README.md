# FT Blur — Trend Blur Effect

Tool web ringan & open source untuk merekam video dengan **blur otomatis lewat isyarat tangan**. Semua proses berjalan di perangkatmu (client-side) — tidak ada server, tidak ada upload, privasi terjaga.

## Fitur

- Rekam video langsung dari kamera HP/perangkat (depan/belakang).
- **Deteksi gestur tangan** (lebih dari 1 jari terbuka) → area kamera otomatis di-blur.
- Slider intensitas blur.
- Efek audio yang ikut tergabung ke dalam video saat direkam.
- Preview hasil & download (format `webm`/`mp4` otomatis sesuai dukungan browser).
- Panduan onboarding 3 langkah.

## Cara menjalankan secara lokal

Aplikasi butuh *secure context* (HTTPS atau `localhost`) agar kamera bisa diakses, jadi tidak bisa dibuka langsung lewat `file://`. Jalankan server statis dulu:

```bash
# pakai Node.js
npx serve

# atau pakai Python
python -m http.server 8000
```

Lalu buka `http://localhost:8000` (atau alamat yang ditampilkan `serve`).

## Cara kerja singkat

- Kamera diambil via `getUserMedia`, frame digambar ke `<canvas>`.
- [MediaPipe Tasks Vision `HandLandmarker`](https://ai.google.dev/edge/mediapipe/solutions/vision/hand_landmarker) mendeteksi 21 titik tangan; blur aktif saat **2–3 jari terbuka** (4 jari atau lebih tidak blur).
- Blur dirender via downscale→upscale di `<canvas>` (konsisten di semua browser, termasuk Safari iOS; `ctx.filter` blur tidak dipakai karena tak stabil di Safari).
- Audio efek: **desktop** direkam langsung lewat WebAudio (`MediaStreamAudioDestinationNode`) → `MediaRecorder`. **iOS/Safari** tidak meng-encode track WebAudio, jadi video direkam tanpa audio lalu di-remux: [mp4box.js](https://github.com/gpac/mp4box.js) mendemux sample video + AAC (`bck_sound_blur.m4a`), lalu [mp4-muxer](https://github.com/Vanilagy/mp4-muxer) menyusun ulang jadi **MP4 progresif** (`ftyp/moov/mdat`, non-fragmented). Ini penting: output fragmented (moof) dari mp4box saja bikin audio hilang saat video disimpan ke iOS Photos.
- Aset audio: `bck_sound_blur.mp3` (MP3 asli, untuk monitor live via WebAudio) & `bck_sound_blur.m4a` (AAC, untuk mux mp4box).
- Saat gestur terdeteksi, `<canvas>` diberi filter blur.
- Audio efek diputar via Web Audio API dan track-nya digabung ke output rekaman.
- `MediaRecorder` merekam canvas + audio, hasilnya bisa langsung diunduh.

> Catatan: deteksi gestur memuat MediaPipe dari CDN, jadi butuh koneksi internet. Tanpa internet, kamera & rekam tetap berfungsi tetapi blur otomatis tidak aktif.

## Teknologi

HTML, CSS, JavaScript (vanilla) — tanpa framework, tanpa backend.

## Lisensi

[MIT](LICENSE)
