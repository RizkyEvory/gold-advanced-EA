# 💎 M4DI Gold Advanced EA

# Telegram : t.me/madiganzz

<div align="center">

![Version](https://img.shields.io/badge/version-1.0-gold?style=for-the-badge)
![Platform](https://img.shields.io/badge/platform-MetaTrader%205-blue?style=for-the-badge)
![Symbol](https://img.shields.io/badge/symbol-XAUUSD-yellow?style=for-the-badge)
![Timeframe](https://img.shields.io/badge/timeframe-M5-orange?style=for-the-badge)
![Risk](https://img.shields.io/badge/risk--managed-ATR%20Based-green?style=for-the-badge)

**EA Premium untuk Akun Kecil — Follow Trend + Konfirmasi Candle + Scaling In**

*Dikembangkan oleh* **M4DI~UciH4** | [github.com/RizkyEvory](https://github.com/RizkyEvory)

---

> *"Trade smart, not hard. Risiko terukur, profit konsisten."*

</div>

---

## 📋 Daftar Isi

- [Tentang EA](#-tentang-ea)
- [Perbandingan dengan HedgeGrid](#-perbandingan-dengan-hedgegrid)
- [Cara Kerja](#-cara-kerja)
- [Persyaratan Sistem](#-persyaratan-sistem)
- [Instalasi](#-instalasi)
- [Parameter Input](#️-parameter-input)
- [Dashboard Real-Time](#-dashboard-real-time)
- [Strategi Trading Detail](#-strategi-trading-detail)
- [Sistem Manajemen Risiko](#-sistem-manajemen-risiko)
- [Kalkulasi Lot Otomatis](#-kalkulasi-lot-otomatis)
- [Backtesting & Optimasi](#-backtesting--optimasi)
- [Rekomendasi Pengaturan](#-rekomendasi-pengaturan)
- [FAQ](#-faq)
- [Peringatan Risiko](#️-peringatan-risiko)

---

## 🚀 Tentang EA

**M4DI Gold Advanced v1.0** adalah Expert Advisor MetaTrader 5 generasi lanjutan yang dirancang khusus untuk **akun kecil** dengan fokus utama pada **manajemen risiko yang ketat**. Berbeda dari EA grid agresif, EA ini menggunakan pendekatan yang lebih terstruktur:

| Fitur Unggulan | Deskripsi |
|----------------|-----------|
| 💰 **Auto Lot Sizing** | Lot dihitung otomatis berdasarkan % risiko dari balance |
| 📊 **Dual Indicator** | EMA untuk trend + ATR untuk volatilitas & jarak SL/TP |
| 🔍 **Candle Confirmation** | Entry hanya setelah konfirmasi candle valid |
| 📈 **Scaling In** | Tambah posisi saat pullback — mengikuti trend yang lebih kuat |
| 🛑 **Smart Stop Loss** | SL berbasis ATR yang adaptif terhadap volatilitas pasar |
| 🔄 **Breakeven + Trailing** | Otomatis pindah SL ke titik aman lalu trailing mengikuti harga |
| 🚫 **Spread Filter** | Blokir entry saat spread terlalu lebar (news, sesi sepi) |
| 🎛️ **Risk Slider** | Ubah % risiko real-time via tombol di dashboard |

---

## ⚖️ Perbandingan dengan HedgeGrid

| Aspek | Gold Advanced (ini) | Gold HedgeGrid |
|-------|-------------------|----------------|
| **Stop Loss** | ✅ Ada (ATR-based) | ❌ Tidak ada |
| **Lot Sizing** | Auto % risiko balance | Manual / Multiplier |
| **Strategi** | Follow trend + scaling | Grid + hedging |
| **Cocok untuk** | Akun kecil, konservatif | Akun modal lebih besar |
| **Indikator** | EMA + ATR | SuperTrend + ATR |
| **Drawdown** | Lebih terkontrol | Potensi drawdown besar |
| **Trailing** | Trailing SL | Trailing TP |
| **Breakeven** | ✅ Ada | ❌ Tidak ada |

---

## 🧠 Cara Kerja

### Alur Eksekusi Utama

```
OnTick() dipanggil setiap tick
│
├── 1. Cek Balance Minimum
│       └── Balance < InpMinAccountUSD → Stop
│
├── 2. Update Indikator
│       ├── ATR (14 periode) → Volatilitas
│       └── EMA (10 periode) → Arah trend
│
├── 3. Update Info Posisi
│       ├── Hitung buyCount, sellCount
│       ├── Hitung totalRisk (dalam USD)
│       └── Hitung totalProfit floating
│
├── 4. Manajemen Posisi Aktif (setiap tick)
│       ├── Breakeven: SL → titik entry jika profit ≥ BEAfter × ATR
│       └── Trailing: SL ikuti harga jika profit ≥ TrailStart × ATR
│
├── 5. Deteksi Bar Baru (M5)
│       ├── Update arah trend (EMA + filter ATR)
│       └── CheckForSignals()
│               ├── Cek spread, jumlah posisi, total risiko
│               ├── Sinyal baru → Buka posisi pertama
│               └── Kondisi scaling in → Buka posisi tambahan (lot × 0.5)
│
└── 6. Update Dashboard (timer + per tick)
```

### Logika Deteksi Trend

```
Diambil dari bar yang sudah ditutup (index 1) — anti-repaint

  Close[1] > EMA[1] + (TrendFilter × ATR)  →  BULLISH ▲
  Close[1] < EMA[1] − (TrendFilter × ATR)  →  BEARISH ▼
  Di antara keduanya (zona netral)          →  Pertahankan arah sebelumnya
```

> Filter ATR berfungsi sebagai "noise filter" — mencegah sinyal palsu di market choppy.

### Logika Entry Signal

```
BULLISH → Sinyal BUY jika:
  ✔ Candle[1] adalah candle BULLISH (Close > Open)
  ✔ (Opsional) Candle[2] juga BULLISH (konfirmasi 2 candle)
  ✔ Close[1] berada DI ATAS EMA
  ✔ Spread ≤ Threshold

BEARISH → Sinyal SELL jika:
  ✔ Candle[1] adalah candle BEARISH (Close < Open)
  ✔ (Opsional) Candle[2] juga BEARISH (konfirmasi 2 candle)
  ✔ Close[1] berada DI BAWAH EMA
  ✔ Spread ≤ Threshold
```

### Logika Scaling In

```
Scaling BUY (tambah posisi buy):
  Kondisi: Ada posisi BUY aktif
         + Harga turun (pullback) ≥ ScalingPullback × ATR dari harga entry terakhir
         + Harga masih DI ATAS EMA (trend tetap valid)
  Lot: 50% dari lot utama

Scaling SELL (tambah posisi sell):
  Kondisi: Ada posisi SELL aktif
         + Harga naik (pullback) ≥ ScalingPullback × ATR dari harga entry terakhir
         + Harga masih DI BAWAH EMA (trend tetap valid)
  Lot: 50% dari lot utama
```

### Logika Breakeven & Trailing

```
BREAKEVEN (aktif jika InpUseBreakeven = true):
  BUY:  Jika (Bid − EntryPrice) ≥ BEAfter × ATR
        → SL dipindah ke EntryPrice + Spread (titik impas)
  SELL: Jika (EntryPrice − Ask) ≥ BEAfter × ATR
        → SL dipindah ke EntryPrice − Spread (titik impas)

TRAILING STOP (aktif jika InpUseTrailing = true):
  BUY:  Jika profit ≥ TrailStart × ATR
        → SL baru = Bid − (TrailOffset × ATR)  [hanya bergerak naik]
  SELL: Jika profit ≥ TrailStart × ATR
        → SL baru = Ask + (TrailOffset × ATR)  [hanya bergerak turun]
```

---

## 💻 Persyaratan Sistem

| Komponen | Spesifikasi |
|----------|-------------|
| **Platform** | MetaTrader 5 (Build 3000+) |
| **Symbol** | XAUUSD (dioptimasi) |
| **Timeframe** | M5 (rekomendasi utama) |
| **Tipe Akun** | Standard / ECN / Raw Spread |
| **Modal Minimum** | $100 (InpMinAccountUSD) |
| **Leverage** | 1:100 atau lebih |
| **Koneksi** | VPS stabil, latency rendah |

> ✅ EA ini **kompatibel dengan akun netting maupun hedging**, karena tidak membuka posisi dua arah sekaligus.

---

## 📥 Instalasi

### Langkah 1 — Salin File EA

```
Salin file  M4DI_Gold_Advanced.mq5
ke folder:  [MT5 Data Folder]\MQL5\Experts\
```

Untuk menemukan folder MT5:
`MT5 → File → Open Data Folder → MQL5 → Experts`

### Langkah 2 — Kompilasi

1. Buka **MetaEditor** (tekan `F4` di MT5)
2. Buka file `M4DI_Gold_Advanced.mq5`
3. Tekan `F7` untuk kompilasi
4. Pastikan output: **0 error(s), 0 warning(s)**

### Langkah 3 — Pasang ke Chart

1. Buka chart **XAUUSD, M5**
2. Navigator → Expert Advisors → Drag EA ke chart
3. Atur parameter sesuai kebutuhan
4. Centang **"Allow Algo Trading"**
5. Klik **OK**

### Langkah 4 — Verifikasi

Setelah EA aktif, cek tab **Experts** di MT5 — seharusnya muncul:

```
M4DI Gold Advanced EA berhasil diinisialisasi
Simbol: XAUUSD
Timeframe: M5
Magic Number: 20240319
Risiko per trade: 2.0%
```

---

## ⚙️ Parameter Input

### 💰 Manajemen Risiko

| Parameter | Default | Range | Deskripsi |
|-----------|---------|-------|-----------|
| `InpRiskPercent` | `2.0` | 0.1–10 | Risiko per trade sebagai % dari balance |
| `InpMaxRiskPerPos` | `5.0` | 1–20 | Batas total risiko semua posisi aktif (%) |
| `InpMaxPositions` | `3` | 1–10 | Maksimum posisi terbuka bersamaan |
| `InpMinAccountUSD` | `100.0` | >0 | EA berhenti trading jika balance di bawah nilai ini |
| `InpUseAutoLot` | `true` | — | Aktifkan kalkulasi lot otomatis berbasis risiko |
| `InpFixedLot` | `0.01` | >0 | Lot tetap (hanya dipakai jika AutoLot = false) |

---

### 📐 Parameter Strategi

| Parameter | Default | Deskripsi |
|-----------|---------|-----------|
| `InpATRPeriod` | `14` | Periode ATR untuk mengukur volatilitas |
| `InpATRMultiplierSL` | `1.5` | SL ditempatkan 1.5× ATR dari entry |
| `InpATRMultiplierTP` | `3.0` | TP ditempatkan 3.0× ATR dari entry (Risk:Reward = 1:2) |
| `InpTrendPeriod` | `10` | Periode EMA untuk filter trend |
| `InpTrendFilter` | `0.3` | Zona netral = ±0.3 ATR dari EMA (anti-whipsaw) |
| `InpConfirmationCandles` | `1` | Jumlah candle konfirmasi (1 atau 2) |
| `InpScalingPullback` | `0.5` | Jarak minimum pullback untuk scaling in (dalam ATR) |

**Rasio Risk:Reward bawaan:**
```
SL = 1.5 × ATR
TP = 3.0 × ATR
R:R = 1 : 2  →  Cukup menang 34% trades untuk break even
```

---

### 🔄 Trailing & Breakeven

| Parameter | Default | Deskripsi |
|-----------|---------|-----------|
| `InpUseTrailing` | `true` | Aktifkan Trailing Stop |
| `InpTrailStart` | `1.0` | Trailing aktif setelah profit ≥ 1.0× ATR |
| `InpTrailOffset` | `0.5` | Jarak SL trailing dari harga = 0.5× ATR |
| `InpUseBreakeven` | `true` | Aktifkan fitur Breakeven |
| `InpBEAfter` | `0.8` | Breakeven aktif setelah profit ≥ 0.8× ATR |

**Urutan aktivasi proteksi:**
```
Profit > 0.8 ATR  →  SL pindah ke Breakeven (Entry ± Spread)
Profit > 1.0 ATR  →  Trailing Stop mulai aktif
```

---

### 🔒 Filter & Keamanan

| Parameter | Default | Deskripsi |
|-----------|---------|-----------|
| `InpUseSpreadFilter` | `true` | Aktifkan filter spread |
| `InpSpreadThreshold` | `40.0` | Batas spread maksimum (poin). Entry diblokir jika spread > nilai ini |
| `InpMaxSlippage` | `5` | Toleransi slippage saat eksekusi order (poin) |
| `InpMagicNumber` | `20240319` | ID unik EA ini |

> 💡 Spread 40 poin ≈ 4 pip untuk XAUUSD. Selama sesi London/NY aktif, spread biasanya 1–3 pip.

---

### 🎨 Dashboard

| Parameter | Default | Deskripsi |
|-----------|---------|-----------|
| `InpShowDashboard` | `true` | Tampilkan/sembunyikan dashboard |
| `InpDashPosition` | `"Kiri"` | Posisi dashboard: `"Kiri"` atau `"Kanan"` |
| `InpDashTransp` | `85` | Tingkat transparansi background dashboard (0–100) |

---

## 📊 Dashboard Real-Time

Dashboard diperbarui setiap **1 detik** via `OnTimer()` dan setiap tick.

```
┌──────────────────────────────┐
│  💎 M4DI GOLD ADVANCED       │
│        v1.0 Premium          │
├──────────────────────────────┤
│  Balance   1,250.00 USD      │
│  Equity    1,263.45 USD      │
│  Profit    +13.45 USD        │
├──────────────────────────────┤
│  Spread    12.3 / 40.0  ✅   │
│  ATR       8.72              │
│  Trend     ▲ BULLISH         │
├──────────────────────────────┤
│  Posisi    2 / 3             │
│  Buy/Sell  2 / 0             │
│  Lot       0.02              │
│  Tot.Risk  3.2% / 5.0%  ✅   │
├──────────────────────────────┤
│  Risiko: [−] 2.0% [+]        │
├──────────────────────────────┤
│  [TUTUP BUY] [TUTUP SELL]   │
│  [      TUTUP SEMUA      ]  │
├──────────────────────────────┤
│  Status: AKTIF               │
└──────────────────────────────┘
```

### Tombol Kontrol Dashboard

| Tombol | Fungsi |
|--------|--------|
| `[−]` | Kurangi risiko per trade sebesar 0.5% (min: 0.5%) |
| `[+]` | Tambah risiko per trade sebesar 0.5% (maks: 5.0%) |
| `[TUTUP BUY]` | Tutup semua posisi BUY milik EA ini |
| `[TUTUP SELL]` | Tutup semua posisi SELL milik EA ini |
| `[TUTUP SEMUA]` | Tutup semua posisi BUY + SELL sekaligus |

### Kode Warna Status

| Warna | Status | Kondisi |
|-------|--------|---------|
| 🟢 Hijau | `AKTIF` | EA berjalan normal, siap entry |
| 🔴 Merah | `BALANCE RENDAH` | Balance < `InpMinAccountUSD` |
| 🟠 Oranye | `SPREAD TINGGI` | Spread melebihi `InpSpreadThreshold` |
| 🟡 Kuning | `POSISI PENUH` | Jumlah posisi = `InpMaxPositions` |

---

## 📐 Strategi Trading Detail

### Fase 1 — Deteksi Trend (Per Bar Baru)

Setiap bar M5 baru terbentuk, EA:
1. Membaca nilai EMA dan ATR dari bar yang sudah **closed** (index 1) — memastikan tidak ada repaint
2. Membandingkan posisi Close terhadap EMA ± zona filter
3. Trend dipertahankan selama harga berada di zona netral (anti-whipsaw)

### Fase 2 — Konfirmasi Sinyal Entry

Setelah trend teridentifikasi, EA mencari konfirmasi berupa:
- Candle bullish/bearish yang jelas (body nyata)
- Harga di sisi yang benar relative terhadap EMA
- Spread dalam batas aman

### Fase 3 — Eksekusi & Sizing

Lot dihitung secara dinamis setiap kali sinyal muncul (berdasarkan balance terkini). SL dan TP dihitung dari ATR real-time, bukan nilai fixed.

### Fase 4 — Scaling In (Piramida Posisi)

Jika kondisi terpenuhi (pullback valid, trend masih berlaku), EA menambah posisi searah dengan lot **50% lebih kecil** dari posisi utama — teknik klasik untuk memaksimalkan keuntungan saat trend kuat.

### Fase 5 — Proteksi Profit (Per Tick)

Setiap tick, EA memeriksa semua posisi aktif:
1. **Breakeven** diaktifkan pertama — melindungi modal saat profit awal
2. **Trailing Stop** mengambil alih — mengunci profit semakin besar seiring harga bergerak

---

## 🛡️ Sistem Manajemen Risiko

### Berlapis dan Komprehensif

```
Layer 1 — Balance Check
  └── Trading berhenti jika Balance < InpMinAccountUSD

Layer 2 — Spread Filter
  └── Entry diblokir jika Spread > InpSpreadThreshold

Layer 3 — Position Limit
  └── Maks InpMaxPositions posisi terbuka bersamaan

Layer 4 — Total Risk Cap
  └── Entry baru diblokir jika totalRisk ≥ InpMaxRiskPerPos × Balance

Layer 5 — Auto Stop Loss (ATR-based)
  └── Setiap posisi punya SL = EntryPrice ± (InpATRMultiplierSL × ATR)

Layer 6 — Breakeven
  └── SL otomatis pindah ke titik impas saat profit awal tercapai

Layer 7 — Trailing Stop
  └── SL mengikuti harga untuk mengunci profit maksimal
```

### Mengapa Ini Lebih Aman dari EA Grid?

| Skenario | Gold Advanced | Gold HedgeGrid |
|----------|--------------|----------------|
| Harga turun 100 pip | SL hit, rugi terbatas | Posisi tetap terbuka, floating loss terus |
| Spread melebar 8 pip | Entry diblokir | Entry mungkin dilakukan |
| Balance habis 80% | EA berhenti | EA tetap jalan |
| Trend berbalik | SL hit, cari sinyal baru | Grid terus menambah posisi |

---

## 🔢 Kalkulasi Lot Otomatis

Formula yang digunakan EA untuk menghitung lot:

```
RiskAmount (USD)  =  Balance × RiskPercent / 100
SL Distance (pts) =  (ATRMultiplierSL × ATR) / PointValue
ValuePerPoint     =  TickValue / (TickSize / PointValue)
Lot               =  RiskAmount / (SL Distance × ValuePerPoint)
```

**Contoh nyata (Balance $500, Risk 2%, ATR = 8.5):**
```
RiskAmount        =  500 × 2% = $10
SL Distance (pts) =  (1.5 × 8.5) / 0.00001 = 1,275,000 pts
ValuePerPoint     =  ~$0.00001 per 0.01 lot
Lot               ≈  0.01 lot

→ Jika SL kena, kerugian ≈ $10 (2% dari $500)
```

---

## 🔬 Backtesting & Optimasi

### Pengaturan Strategy Tester

| Setting | Nilai |
|---------|-------|
| **Symbol** | XAUUSD |
| **Timeframe** | M5 |
| **Model** | Every Tick Based on Real Ticks |
| **Spread** | Current (atau Fixed 15–20 untuk XAUUSD) |
| **Deposit** | $1,000 (representatif) |
| **Leverage** | 1:100 |
| **Periode Test** | Minimal 6 bulan |

### Parameter yang Direkomendasikan untuk Optimasi

Urutan prioritas:

1. `InpATRMultiplierSL` — range: 1.0–2.5, step: 0.25
2. `InpATRMultiplierTP` — range: 2.0–5.0, step: 0.5
3. `InpTrendFilter` — range: 0.1–0.8, step: 0.1
4. `InpTrailStart` / `InpBEAfter` — range: 0.5–2.0, step: 0.25
5. `InpScalingPullback` — range: 0.3–1.0, step: 0.1

### Target Hasil Backtest yang Baik

| Metrik | Target |
|--------|--------|
| Profit Factor | > 1.5 |
| Max Drawdown | < 20% |
| Win Rate | > 40% (R:R 1:2 memungkinkan ini) |
| Recovery Factor | > 2.0 |

---

## 💡 Rekomendasi Pengaturan

### 🟢 Setup Ultra-Konservatif (Pemula / Akun Mini $100–$300)

```
InpRiskPercent       = 1.0
InpMaxRiskPerPos     = 3.0
InpMaxPositions      = 2
InpMinAccountUSD     = 100.0
InpATRMultiplierSL   = 2.0
InpATRMultiplierTP   = 4.0
InpConfirmationCandles = 2
InpScalingPullback   = 0.8
InpSpreadThreshold   = 30.0
```

### 🟡 Setup Standar (Akun $300–$1000)

```
InpRiskPercent       = 2.0
InpMaxRiskPerPos     = 5.0
InpMaxPositions      = 3
InpMinAccountUSD     = 100.0
InpATRMultiplierSL   = 1.5
InpATRMultiplierTP   = 3.0
InpConfirmationCandles = 1
InpScalingPullback   = 0.5
InpSpreadThreshold   = 40.0
```

### 🔴 Setup Agresif (Akun $1000+ / Berpengalaman)

```
InpRiskPercent       = 3.0
InpMaxRiskPerPos     = 8.0
InpMaxPositions      = 5
InpMinAccountUSD     = 200.0
InpATRMultiplierSL   = 1.2
InpATRMultiplierTP   = 2.5
InpConfirmationCandles = 1
InpScalingPullback   = 0.3
InpSpreadThreshold   = 50.0
```

---

## ❓ FAQ

**Q: Apa bedanya EA ini dengan Gold HedgeGrid?**
> A: EA ini memiliki Stop Loss dan manajemen risiko berbasis persentase balance — jauh lebih aman untuk akun kecil. HedgeGrid lebih cocok untuk trader berpengalaman dengan modal lebih besar yang memahami risiko tanpa SL.

**Q: Kenapa EA tidak entry meski trend sudah jelas?**
> A: Ada beberapa penyebab: (1) spread terlalu lebar, (2) total risiko sudah mencapai `InpMaxRiskPerPos`, (3) jumlah posisi sudah mencapai `InpMaxPositions`, atau (4) candle konfirmasi belum valid. Cek tab Experts untuk detail.

**Q: Apakah bisa dipakai di pair lain selain XAUUSD?**
> A: Bisa, tapi perlu penyesuaian `InpSpreadThreshold` dan mungkin `InpATRMultiplierSL/TP` karena karakteristik volatilitas setiap pair berbeda.

**Q: Apa itu "Zona Netral" pada deteksi trend?**
> A: Zona antara `EMA − (TrendFilter × ATR)` dan `EMA + (TrendFilter × ATR)`. Saat harga di zona ini, EA tidak mengubah arah trend — mencegah sinyal palsu saat market sideways.

**Q: Berapa jumlah posisi maksimum yang aman?**
> A: Untuk akun kecil ($100–$500), rekomendasinya 2–3 posisi. Lebih dari 3 posisi perlu modal lebih besar agar risk per posisi tidak terlalu kecil akibat pembatasan `InpMaxRiskPerPos`.

**Q: Kenapa ada parameter `InpDashTransp`?**
> A: Untuk mengatur transparansi background dashboard agar tidak menutupi candle chart di belakangnya. Nilai 85 = 85% opaque (cukup solid namun tidak terlalu mengganggu).

**Q: Bisa running bersamaan dengan Gold HedgeGrid?**
> A: Bisa, asalkan Magic Number berbeda. Gunakan `20240319` untuk Advanced dan `20240318` untuk HedgeGrid (sudah berbeda secara default).

---

## ⚡ Changelog

### v1.0 (Current)
- ✅ Auto lot sizing berbasis % risiko balance
- ✅ Dual indicator: EMA (trend) + ATR (volatilitas)
- ✅ Konfirmasi candle 1 atau 2 bar
- ✅ Scaling in saat pullback dengan validasi EMA
- ✅ Breakeven otomatis berbasis ATR
- ✅ Trailing Stop berbasis ATR
- ✅ Spread filter dengan threshold yang bisa diatur
- ✅ Dashboard interaktif dengan risk slider
- ✅ Tombol close per tipe (BUY/SELL/ALL)
- ✅ Kompatibel akun netting & hedging

---

## 🛡️ Peringatan Risiko

```
╔══════════════════════════════════════════════════════════════╗
║                    ⚠️  PERINGATAN RISIKO  ⚠️                ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  Meskipun EA ini dilengkapi Stop Loss dan manajemen risiko   ║
║  yang lebih ketat, trading forex/gold tetap mengandung       ║
║  risiko kerugian yang signifikan, termasuk seluruh modal.    ║
║                                                              ║
║  • Selalu gunakan modal yang siap untuk hilang               ║
║  • Backtest & forward test sebelum live trading              ║
║  • Pantau EA secara berkala, jangan dibiarkan tanpa monitor  ║
║  • Saat rilis berita besar (NFP, FOMC), pertimbangkan        ║
║    untuk mematikan EA sementara                              ║
║  • Leverage tinggi memperbesar risiko dan potensi profit     ║
║                                                              ║
║  PAST PERFORMANCE DOES NOT GUARANTEE FUTURE RESULTS.        ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 👤 Author

**M4DI~UciH4**
- GitHub: [github.com/RizkyEvory](https://github.com/RizkyEvory)
- Magic Number Default: `20240319`

---

<div align="center">

**💎 Trade Smart. Manage Risk. Stay Consistent. 💎**

*"Risk comes from not knowing what you're doing."*
— Warren Buffett

</div>
