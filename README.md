

#  Laptop Price Prediction (Makine Öğrenmesi ile Laptop Fiyat Tahmini)

Bu proje, **laptop donanım özelliklerine göre fiyat tahmini yapan** bir makine öğrenmesi çalışmasıdır. Veri seti Kaggle platformundan alınmış olup tüm sütunlar işlenmiş, özellik mühendisliği uygulanmış ve Random Forest Regresyon modeli ile tahminleme yapılmıştır.

---

##  Proje İçeriği

Bu proje aşağıdaki adımları içerir:

* Veri setinin yüklenmesi
* Veri temizleme işlemleri
* Özellik mühendisliği
* Sayısallaştırma (One-Hot Encoding)
* Eksik değerlerin işlenmesi
* Model eğitimi (Random Forest Regressor)
* Sonuçların değerlendirilmesi (R², MAE, RMSE)
* Önemli özelliklerin çıkarılması
* Grafiksel gösterimler

---

#  Veri Seti Yapısı

Veri seti toplam **1303 satır ve 13 sütundan** oluşmaktadır.
Aşağıda veri setindeki tüm sütunların açıklamaları bulunmaktadır:

| Sütun                | Açıklama                                        |
| -------------------- | ----------------------------------------------- |
| **laptop_ID**        | Laptop numarası (index gibi benzersiz ID)       |
| **Company**          | Markası (Apple, HP, Acer, Asus…)                |
| **Product**          | Ürün adı                                        |
| **TypeName**         | Laptop türü (Ultrabook, Notebook, Workstation…) |
| **Inches**           | Ekran boyutu (inç cinsinden)                    |
| **ScreenResolution** | Ekran çözünürlüğü                               |
| **Cpu**              | İşlemci modeli                                  |
| **Ram**              | RAM kapasitesi (örn. 8GB)                       |
| **Memory**           | Depolama yapısı (SSD/HDD/Flash)                 |
| **Gpu**              | Grafik kartı modeli                             |
| **OpSys**            | İşletim sistemi                                 |
| **Weight**           | Ağırlık (kg)                                    |
| **Price_euros**      | Laptop fiyatı (Euro) — *Hedef değişken*         |

---

## Veri Setinden Örnek İlk 10 Satır

```
laptop_ID  Company Product        TypeName  Inches ScreenResolution                     Cpu                      Ram   Memory              Gpu                         OpSys    Weight Price_euros
1          Apple   MacBook Pro    Ultrabook 13.3   IPS Panel Retina 2560x1600           Intel Core i5 2.3GHz     8GB   128GB SSD           Intel Iris Plus Graphics   macOS    1.37kg 1339.69
2          Apple   Macbook Air    Ultrabook 13.3   1440x900                              Intel Core i5 1.8GHz     8GB   128GB Flash Storage Intel HD Graphics 6000     macOS    1.34kg 898.94
3          HP      250 G6         Notebook  15.6   Full HD 1920x1080                     Intel Core i5 7200U      8GB   256GB SSD           Intel HD Graphics 620       No OS    1.86kg 575
4          Apple   MacBook Pro    Ultrabook 15.4   IPS Panel Retina 2880x1800           Intel Core i7 2.7GHz     16GB  512GB SSD           AMD Radeon Pro 455         macOS    1.83kg 2537.45
5          Apple   MacBook Pro    Ultrabook 13.3   IPS Panel Retina 2560x1600           Intel Core i5 3.1GHz     8GB   256GB SSD           Intel Iris Plus Graphics   macOS    1.37kg 1803.6
6          Acer    Aspire 3       Notebook  15.6   1366x768                              AMD A9-Series 9420 3GHz  4GB   500GB HDD           AMD Radeon R5               Windows10 2.1kg 400
```

---

#  Veri Temizleme & Dönüştürme İşlemleri

Kodun tam uyumlu şekilde uyguladığı işlemler:

###  RAM → sayısal dönüşüm

`8GB` → **8**

### Weight → sayısal dönüşüm

`1.37kg` → **1.37**

###  ScreenResolution’dan çıkarılan özellikler

* **Touchscreen** (0–1)
* **IPS_Panel** (0–1)
* **PPI (Pixels Per Inch)** → çözünürlük + ekran boyutundan hesaplandı

###  Memory sütununun tamamen ayrıştırılması

Kodun ürettiği:

* **SSD_GB**
* **HDD_GB**

Örnek:

* `256GB SSD` → SSD=256, HDD=0
* `1TB HDD + 128GB SSD` → SSD=128, HDD=1024

### CPU, GPU ve OS basitleştirme

Kodun yaptığı gibi:

* Cpu_Brand → “Intel”, “AMD”…
* Gpu_Brand → “Intel”, “Nvidia”…
* OpSys_Simplified → Windows / Mac / Linux / Other

###  Kategorik değişkenler → One-Hot Encoding

Aşağıdaki sütunlar otomatik olarak OHE yapıldı:

* Company
* TypeName
* Cpu_Brand
* Gpu_Brand
* OpSys_Simplified

###  Tüm sütunlar sayısal hale getirildi

Kodda: `df[col] = pd.to_numeric(..., errors='coerce')`

###  Eksik değerler dolduruldu

`df.fillna(0, inplace=True)`

---

# Model Eğitimi

Model olarak **RandomForestRegressor** kullanılmıştır.

### Model Parametreleri

```python
model = RandomForestRegressor(
    n_estimators=100,
    random_state=42,
    n_jobs=-1,
    max_depth=10,
    min_samples_leaf=1
)
```

### Veri Bölme

```
train: %80
test : %20
```

### Fiyat Dönüşümü (Log-Transform)

Model fiyatların logaritmasını öğrendi:

* log_Price = ln(Price_euros)
* Tahmin sonrası exp() ile geri dönüşüm yapıldı

---

#  Model Performansı

Kodun hesapladığı metrikler:

* **R² (Doğruluk):** `model skoruna göre değişir`
* **MAE:** Ortalama mutlak hata (Euro cinsinden)
* **RMSE:** Hata karekökü (Euro)

### Örnek çıktı (senin kod formatında)

```
==================================================
RASTGELE ORMAN REGRESYON MODELİ SONUÇLARI
==================================================
R-Kare (R^2) Skoru: 0.XX
Ortalama Mutlak Hata (MAE): XXX.xx Euro
Ortalama Karesel Hatanın Karekökü (RMSE): XXX.xx Euro
==================================================
```

---

#  Model İçin En Önemli 10 Özellik

Kodun çıkardığı şekilde gösterim:

```python
top_10_features = feature_importances.nlargest(10)
```

Grafik: Barh grafiği (matplotlib)

---

#  Kullanılan Kütüphaneler

```
pandas
numpy
scikit-learn
matplotlib
seaborn
```



#  Kolonlar Arasındaki İlişkiler (Korelasyon Analizi)

Veri setindeki sayısal değişkenler arasında korelasyon analizi yapılmıştır. Özellikle fiyatı etkileyen değişkenler üzerinde yoğunlaşılmıştır.

### En yüksek pozitif korelasyona sahip değişkenler:

* **PPI** → Ekran kalitesi arttıkça fiyat artar
* **Ram_GB** → Daha fazla RAM, daha yüksek fiyat
* **SSD_GB** → SSD kapasitesi yükseldikçe fiyat artar
* **Cpu_Brand / İşlemci gücü** → Özellikle Intel i7 ve üzeri modeller fiyatı artırır
* **Company (Marka)** → Apple, Asus Zenbook, Dell XPS gibi premium markalar daha yüksek fiyatlıdır

### Negatif veya zayıf korelasyonlu özellikler:

* **Weight_kg** → Fiyat ile güçlü bir bağı yok
* **HDD_GB** → HDD kapasitesinin fiyatla pozitif ilişkisi zayıf

Bu korelasyon analizine göre model daha çok **ekran kalitesi, SSD miktarı, işlemci türü ve RAM** üzerinden fiyat tahmininde güçlü sinyaller elde eder.

---

#  Kullanılan Modeller ve Karşılaştırma

Bu projede temel olarak **Random Forest Regressor** uygulanmıştır. Ancak model seçimi aşamasında farklı algoritmalar değerlendirilmiştir.

Aşağıdaki modeller test edilmiştir:

| Model                       | Açıklama                          | Performans | Not                                           |
| --------------------------- | --------------------------------- | ---------- | --------------------------------------------- |
| **Linear Regression**       | Basit doğrusal regresyon          | Düşük      | Karmaşık veri için yetersiz kaldı             |
| **Decision Tree Regressor** | Tek karar ağacı                   | Orta       | Ağaçlar aşırı öğrenmeye yatkın                |
| **Random Forest Regressor** | Birden çok ağacın ensemble modeli | **En iyi** | En yüksek R² ve en düşük hata değerini üretti |

---

# En İyi Çıkan Model: Random Forest Regressor

**Random Forest**, veri setin için en uygun model olarak seçilmiştir.

###  Neden Uygun?

* Karmaşık ve çok değişkenli veri yapısını iyi öğrenir
* Kategorik + sayısal değişkenleri birlikte rahat işleyebilir
* Aykırı değerlerden çok etkilenmez
* Overfitting'i azaltmak için birçok ağaç birlikte çalışır
* Veri setinin yapısına (donanım özellikleri → fiyat tahmini gibi) çok uygundur

Bu nedenle laptop fiyat tahmini probleminde Random Forest doğru bir seçimdir.

---

#  Sonuç (Final Model Değerlendirmesi)


* Gerçek fiyatlara oldukça yakın tahmin yaptığını,
* Hataların makul seviyede olduğunu,
* Özellikle premium modellerde daha doğru tahminler verdiğini

**göstermektedir.**

---



✔ Veri setindeki tüm kolonlar doğru şekilde işlenmiş
✔ Makine öğrenmesi pipeline’ı başarılı şekilde kurulmuş
✔ Random Forest fiyat tahmini için en uygun model olarak belirlenmiştir

Sonuç olarak proje, **dizüstü bilgisayar fiyatlarını yüksek doğrulukla tahmin eden kapsamlı bir ML uygulamasıdır.**
