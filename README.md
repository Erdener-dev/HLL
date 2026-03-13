# HLL
# HyperLogLog — Kardinalite Tahmini (Java)

![Java](https://img.shields.io/badge/Java-17%2B-orange?logo=java) ![Algorithm](https://img.shields.io/badge/Algorithm-HyperLogLog-blue) ![License](https://img.shields.io/badge/License-MIT-green)

> Flajolet et al. (2007) tarafından yayımlanan **"HyperLogLog: the analysis of a near-optimal cardinality estimation algorithm"** makalesinin Java implementasyonu.

---

## 📌 Nedir?

**HyperLogLog (HLL)**, milyonlarca elemandan oluşan bir kümenin kaç farklı (tekil) eleman içerdiğini **sabit bellek** kullanarak yüksek doğrulukla tahmin eden olasılıksal bir veri yapısıdır.

| Yöntem | Bellek | Hata |
|---|---|---|
| Tam sayım (HashSet) | O(n) | %0 |
| HyperLogLog | O(m) — sabit | ~%0.4–%3.25 |

Örneğin `b=14` ile **16.384 byte** bellekte 100 milyonluk bir veri setinin kardinalitesi **~%0.81 hatayla** tahmin edilebilir.

---

## 🧮 Algoritma Özeti

```
1. Her eleman bir hash fonksiyonundan geçirilir → 32-bit tam sayı elde edilir.
2. Hash'in üst b biti → kova indeksi (hangi register güncelleneceğini belirler)
3. Hash'in kalan (32-b) biti → ρ(w): en soldaki 1-bitinin pozisyonu
4. registers[index] = max(registers[index], ρ(w))
5. Tahmin: E = αm · m² · (Σ 2^(-register[i]))⁻¹
```

Küçük ve büyük aralıklar için ek düzeltmeler uygulanır:
- **Küçük aralık:** LinearCounting (`E ≤ 2.5m` ve boş kova varsa)
- **Büyük aralık:** Logaritmik düzeltme (`E > 2³²/30`)

---

## 📐 Teorik Standart Hata

```
σ ≈ 1.04 / √m       (m = 2^b kova sayısı)
```

| b | m (kova) | Bellek | Teorik Hata |
|---|---|---|---|
| 10 | 1.024 | ~1 KB | %3.25 |
| 11 | 2.048 | ~2 KB | %2.30 |
| 12 | 4.096 | ~4 KB | %1.63 |
| 13 | 8.192 | ~8 KB | %1.15 |
| 14 | 16.384 | ~16 KB | %0.81 |
| 16 | 65.536 | ~64 KB | %0.41 |

---

## 🚀 Kullanım

### Derleme ve Çalıştırma

```bash
javac HyperLogLog.java
java HyperLogLog
```

### Temel API

```java
// 1. HLL oluştur (b=14 → ~%0.81 hata)
HyperLogLog hll = new HyperLogLog(14);

// 2. Eleman ekle
hll.add("kullanici_1");
hll.add("kullanici_2");
hll.add("kullanici_1"); // tekrar — etkisiz

// 3. Kardinalite tahmini
long tekil = hll.count();
System.out.println("Tahmini tekil eleman sayısı: " + tekil);

// 4. İki HLL birleştir (A ∪ B)
HyperLogLog hllA = new HyperLogLog(12);
HyperLogLog hllB = new HyperLogLog(12);
// ... elemanlar eklenir ...
HyperLogLog merged = hllA.merge(hllB);
System.out.println("|A ∪ B| tahmini: " + merged.count());

// 5. Teorik hata oranı
System.out.printf("Teorik hata: %.2f%%%n", hll.theoreticalError() * 100);
```

---

## 🧪 Test Sonuçları

Örnek çıktı (`java HyperLogLog`):

```
=== HyperLogLog Test ===

-- Test 1: Temel ekleme --
Gerçek tekil eleman : 4
HLL tahmini         : 4
HyperLogLog{b=11, m=2048, tahminiKardinality=4, teorikHata=2.30%}

-- Test 2: Büyük veri seti (N=200_000) --
Gerçek sayı  : 200.000
HLL tahmini  : 201.847
Gerçek hata  : 0.92%
Teorik hata  : 1.15%

-- Test 3: merge() -- A ∪ B tahmini --
Gerçek |A ∪ B|    : 90000
|A| tahmini        : 60312
|B| tahmini        : 60089
|A ∪ B| tahmini    : 90541

-- Test 4: b artınca hata azalır --
b     m        Teorik σ (%)     Gerçek hata (%)
--------------------------------------------------
4     16       26.00            ...
6     64       13.00            ...
...
16    65536    0.41             ...
```

---

## 🔧 Hash Fonksiyonu

Implementasyonda **FNV-1a + MurmurHash3 finalization** kombinasyonu kullanılmıştır:

- **FNV-1a**: Bayt bazlı hızlı karıştırma (offset: `0x811c9dc5`, prime: `0x01000193`)
- **MurmurHash3 mix**: Avalanche etkisiyle bit dağılımını düzgünleştirir, özellikle ardışık string girdilerinde çarpışmaları minimize eder

Saf FNV-1a tercih edilmemesinin nedeni: `"item_1"`, `"item_2"` gibi düşük entropili girdilerde bit dağılımını bozabilmesi.

---

## 📂 Proje Yapısı

```
.
└── HyperLogLog.java    # Tek dosya implementasyon + testler
```

---

## 📚 Referans

- Flajolet, P., Fusy, É., Gandouet, O., & Meunier, F. (2007). *HyperLogLog: the analysis of a near-optimal cardinality estimation algorithm.* DMTCS Proceedings, AH, 127–146.

---

## 📄 Lisans

MIT
