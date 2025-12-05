

# Makine Öğrenmesi ile Laptop Fiyat Tahmini

Bu projem, **laptop donanım özelliklerine göre fiyat tahmini yapan bir makine öğrenmesi dersi çalışmasıdır**.  
Veri seti Kaggle platformundan alınmış, tüm sütunlar işlenmiş, özellik mühendisliği uygulanmış ve **Random Forest Regressor** modeli ile tahmin yapılmıştır.  

---

## Proje Hedefi

- Kullanıcıların laptop fiyatlarını tahmin edebilmek.  
- Donanım özelliklerinden fiyat öngörüsü yapabilmek.  
- Makine öğrenmesi teknikleri ile en doğru tahmin modelini seçebilmek amaçlarım arasındadır.  

---


#  Veri Seti Yapısı

Veri seti toplam **1303 satır ve 13 sütundan** oluşmaktadır.
Aşağıda veri setindeki tüm sütunların açıklamaları bulunmaktadır:

| Sütun                | Açıklama                                        |
| -------------------- | ----------------------------------------------- |
| **laptop_ID**        | Laptop numarası                                 |
| **Company**          | Markası                                         |
| **Product**          | Ürün adı                                        |
| **TypeName**         | Laptop türü                                     |
| **Inches**           | Ekran boyutu (inç cinsinden)                    |
| **ScreenResolution** | Ekran çözünürlüğü                               |
| **Cpu**              | İşlemci modeli                                  |
| **Ram**              | RAM kapasitesi                                  |
| **Memory**           | Depolama yapısı                                 |
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


---

## Veri Temizleme ve Özellik Mühendisliği

1. **Sayısal Dönüşümler**  
   - `Ram` ve `Weight` sütunları sayısallaştırdım  
   - `Memory`sütünü   SSD ve HDD kapasiteleri olarak ayırarak GB cinsinden yazdırdım.  

 
2. **CPU, GPU ve OS Basitleştirme**  
   - İşlemci ve GPU markaları ayrıştırdım (`Intel`, `AMD`, `Nvidia` vb.).  
   - İşletim sistemi basitleştirdim (`Windows`, `Mac`, `Linux`, `Other`).  

3. **Kategorik Verilerin Sayısallaştırılması**  
   - `Company`, `TypeName`, `Cpu_Brand`, `Gpu_Brand`, `OpSys_Simplified` → One-Hot Encoding  

4. **Eksik Değerlerin Doldurulması**  
   - Tüm NaN değerler 0 ile doldurdum.  

5. **Hedef Değişkenin Log Dönüşümü**  
   - Fiyatların logaritması alındı (`log_Price`) → aşırı değerlerin etkisi azalttım.  

---

## Modelleme

### Kullanılan Modeller

1. **Random Forest Regressor (Seçilen Model)**  
   - Ensemble (çok ağaçlı) bir  modeldir 
   - Karmaşık veri yapısında daah verimlidir
   - Kategorik + sayısal verilerle uyumluludur  
   - Aykırı değerlerden kolay  etkilenmez  
   - Overfitting’i azaltmak için çoklu ağaç yapısı kullanır  

2. **Polynomial Regression (Derece 2)**  
   - Fiyat ile özellikler arasındaki **non-lineer ilişkileri** modellemeye çalışır  
   - Random Forest’a kıyasla daha düşük R² ve yüksek hata değerleri  ürettiği için secilmedim

3. **Diğer Modeller (Deneme Amaçlı)**  
   - **Linear Regression**: Basit doğrusal model → karmaşık ve çok değişkenli veri için yetersiz kaldı
    

**Sonuç:** Random Forest en iyi performansı verdiği için radndom forest algoritmasını seçtim  

---

## Model Performansı

| Model | R² Skoru | MAE (Euro) | RMSE (Euro) |
|-------|----------|------------|-------------|
| Random Forest Regressor | Yüksek | Düşük | Düşük |
| Polynomial Regression | Orta | Orta | Orta |
| Linear Regression | Düşük | Yüksek | Yüksek |


- Log dönüşümü tersine çevrilerek fiyatlar **orijinal Euro ölçeğinde** karşılaştırıldı.  

---

###  Neden Random Forest Algoritması  Uygun?

* Karmaşık ve çok değişkenli veri yapısını iyi öğrenir
* Kategorik + sayısal değişkenleri birlikte rahat işleyebilir
* Aykırı değerlerden çok etkilenmez
* Overfitting'i azaltmak için birçok ağaç birlikte çalışır
* Veri setinin yapısına (donanım özellikleri → fiyat tahmini gibi) çok uygundur

Bu nedenle laptop fiyat tahmini probleminde Random Forest doğru bir seçimdir.

---

## Özellik Önem Düzeyi

- Random Forest modeli, fiyat tahmininde **en önemli 10 özelliği** belirledi:  
  - PPI, Ram_GB, SSD_GB, Cpu_Brand, Company vb.  
- Görselleştirme ile en önemli özellikler bar chart olarak gösterildi.  

---

## Kullanılan Kütüphaneler

- **pandas, numpy** → veri işleme  
- **scikit-learn** → makine öğrenmesi modelleri ve metrikler  
- **matplotlib, seaborn** → görselleştirme  


# Korelasyon Analizi (Correlation)
Değişkenlerin fiyatını  incelendiğimde:

 Güçlü Pozitif İlişki:

RAM & SSD: Kapasite arttıkça fiyat doğru orantılı olarak artıyor.

Çözünürlük (PPI): Yüksek çözünürlüklü ekranlar daha pahalı modellerde bulunuyor.

 Zayıf/Negatif İlişki:

HDD: Eski teknoloji HDD disklerin boyutu artsa bile fiyata etkisi SSD kadar yüksek değildir.

Ağırlık: Ağırlık ile fiyat arasında doğrudan net bir çizgi yoktur; hem ucuz ağır laptoplar hem de pahalı ağır (Gaming) laptoplar mevcuttur.
## Sonuç

- Random Forest modeli, laptop fiyatlarını tahmin etmek için en uygun yöntemdir.  
- Özellikle premium laptop modellerinde tahmin doğruluğu yüksektir.  
- Polinom ve lineer regresyon, karmaşık veri yapısı nedeniyle yeterli performansı veremedi.  
- Model, ekran kalitesi, RAM, SSD kapasitesi ve CPU gücü gibi temel donanım özelliklerinden fiyat tahmininde güçlü sinyaller elde eder.
- <img width="989" height="590" alt="indir" src="https://github.com/user-attachments/assets/4c4438f4-c015-4bdb-8cc5-74426e7c0682" />
<img width="1260" height="312" alt="Ekran görüntüsü 2025-11-24 211020" src="https://github.com/user-attachments/assets/5f29de97-eda2-4432-962c-a72c31682767" />





