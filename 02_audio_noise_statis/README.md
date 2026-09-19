# Tugas 2 — Analisis Sinyal Suara & Noise Statis

**Mohd. Musyaffa Alief Athallah — 123140184**
IF25-40305 Sistem Teknologi Multimedia, Week 03
Program Studi Teknik Informatika, Institut Teknologi Sumatera

## Setup Perekaman

| Aspek                   | Keterangan                                                         |
| ----------------------- | ------------------------------------------------------------------ |
| Sumber noise statis     | Kipas angin berdiri, kecepatan 3, putaran dijaga tetap             |
| Jarak ke sumber derau   | Sekitar 70 cm                                                      |
| Perangkat perekam       | iPhone 13                                                          |
| Format asli             | M4A (AAC)                                                          |
| Format setelah konversi | WAV PCM 16-bit, 48 kHz                                             |
| Perintah konversi       | `ffmpeg -i <input> -acodec pcm_s16le -ar 48000 audio_original.wav` |

## Artikel yang Dibacakan

"Kabut asap karhutla menyelimuti Pekanbaru, Disdik perpanjang PJJ untuk siswa"
Sumber: _(https://suarapemerintah.id/2026/09/kabut-asap-karhutla-menyelimuti-pekanbaru-disdik-perpanjang-pjj-untuk-siswa/)_

## Metadata Sinyal

| Besaran              | Nilai                              |
| -------------------- | ---------------------------------- |
| Laju sampel          | 48.000 Hz                          |
| Batas Nyquist        | 24.000 Hz                          |
| Jumlah sampel        | 974.848                            |
| Durasi               | 20,309 detik                       |
| Amplitudo min / maks | -0,921 / +0,949                    |
| Puncak absolut       | 0,949 (-0,45 dBFS), tanpa clipping |
| RMS                  | 0,189 (-14,48 dBFS)                |
| Crest factor         | sekitar 14 dB                      |

## Parameter Analisis

- **Spektrum FFT:** n_fft 8192, window Hann periodik, dirata-ratakan lintas jendela bertumpang tindih 50%. Kalibrasi diuji lebih dulu, nada amplitudo 1,0 berpuncak tepat di 0,0000 dBFS.
- **Spektrogram STFT:** n_fft 2048, hop 512, menghasilkan Δf 23,44 Hz dan jendela 42,67 ms. Matriks 1025 bin × 1905 bingkai.
- **Mel-spektrogram:** 128 bin Mel, konvensi Slaney bawaan librosa.
- **Resampling:** 48 kHz → 8 kHz, faktor desimasi M = 6, batas Nyquist baru 4.000 Hz.

## Ringkasan Temuan

**Karakter noise statis**

- Lantai derau berada di -19,57 dBFS, sedangkan bagian aktif -11,32 dBFS, menyisakan selisih hanya 8,25 dB
- Frekuensi dominan 93,8 Hz (-24,35 dBFS), dengan lima komponen terkuat berdesakan di 58 sampai 100 Hz tanpa membentuk deret harmonik
- Sifatnya broadband aerodinamis dari turbulensi bilah, bukan dengung tonal motor. Tidak ditemukan duri tajam di 50 Hz yang biasanya menandakan induksi jala listrik

**Representasi visual**

- Skala Mel memperbesar porsi daerah vokal sekitar 6× (25,0% bin di bawah 1 kHz lawan 4,2% pada skala linier) sambil memampatkan data 8×
- Trade-off resolusi terbukti, n_fft 256 melebur harmonik menjadi pita tebal sedangkan n_fft 8192 menajamkannya namun mengaburkan transien konsonan

**Aliasing**

- Desimasi naif menambah energi palsu +1,23 dB rata-rata pada 2.500 sampai 4.000 Hz, dengan puncak +6,06 dB di 3.964,8 Hz
- Puncak tersebut terverifikasi berasal dari komponen 4.035,2 Hz, hanya 35,2 Hz di atas batas Nyquist baru, sesuai rumus f_alias = |f − fs|
- Setelah aliasing dan peredaman filter dipisahkan dengan spektrum asli sebagai acuan, bukti aliasing paling bersih justru berada di 1.500 sampai 3.500 Hz (+0,24 dan +0,31 dB) karena filter belum meredam apa pun di sana
- Pada 3.500 sampai 4.000 Hz keduanya bercampur, alias menambah +1,19 dB sementara filter meredam -1,86 dB, sehingga selisih naif terhadap berfilter di pita itu lebih banyak disebabkan peredaman filter daripada aliasing
- Hanya 0,02% energi sinyal asli berada di atas 4.000 Hz, sehingga dampak aliasing pada sinyal wicara tergolong tipis

**Hipotesis yang terbantah**
Dugaan awal bahwa deru kipas ikut menyumbang aliasing tidak terbukti. Kelebihan energi pada pita 2.500 sampai 4.000 Hz berayun dari -0,22 dB (detik 12,96) sampai +8,05 dB (detik 2,75). Karena level deru kipas konstan sepanjang rekaman, penggerak aliasing pastilah konten yang berubah-ubah, yaitu konsonan desis pada suara wicara.

## Keterbatasan

Rekaman berasal dari encoder AAC bawaan iPhone, sehingga pita di atas sekitar 16 kHz sudah terpangkas sebelum konversi ke WAV. Hal ini terlihat sebagai tebing curam pada spektrum FFT dan turut memperkecil energi yang tersedia untuk melipat saat desimasi.

## Daftar Berkas

| Berkas                                   | Keterangan                                |
| ---------------------------------------- | ----------------------------------------- |
| `tugas_audio_noise_statis.ipynb`         | Notebook lengkap beserta seluruh keluaran |
| `tugas_audio_noise_statis_123140184.pdf` | Ekspor PDF notebook                       |
| `audio_original.wav`                     | Rekaman asli, 48 kHz PCM 16-bit           |
| `audio_downsampled_naive.wav`            | Hasil desimasi naif `y[::6]`, 8 kHz       |
| `audio_downsampled_clean.wav`            | Hasil `resample_poly` berfilter, 8 kHz    |

## Lingkungan

Python 3.13 pada virtual environment lokal, dengan numpy 2.5.3, scipy 1.18.1, librosa 1.0.0, soundfile 0.14.0, dan matplotlib 3.11.2.
