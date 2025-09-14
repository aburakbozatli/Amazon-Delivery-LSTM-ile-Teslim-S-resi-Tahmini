# 🚚 Amazon Delivery — LSTM ile Teslim Süresi Tahmini

Bu proje, **Amazon Delivery** veri seti kullanılarak günlük teslim sürelerinin **LSTM (Long Short-Term Memory)** ağlarıyla tahmin edilmesini amaçlamaktadır.  
Çalışmada:  
- **Tek değişkenli (univariate)**  
- **Çok değişkenli (multivariate)** LSTM modelleri uygulanmış  
- Basit bir **Naive benchmark** modeliyle kıyaslama yapılmıştır.  

---

## 📂 Veri Seti

- **Kaynak:** Kaggle — Amazon Delivery Dataset  
- **Toplam Satır:** 43.632  
- **Özellikler:** Sipariş tarihi/saati, teslim süresi (dakika), hava durumu, trafik, araç tipi vb.  
- **Hedef Değişken:** `Delivery_Time` (dakika cinsinden)  

### 🔧 Ön İşleme Adımları
- `Order_Date` + `Order_Time` → `order_dt` (datetime birleştirme)  
- Günlük ortalama teslim süresi çıkarıldı (55 günlük seri)  
- Kategorik değişkenler → **one-hot encoding**  
- Eksik değerler → **interpolasyon yöntemi** ile tamamlandı  

---

## ⚙️ Yöntem

### 1️⃣ Univariate LSTM  
- **Girdi:** Son 14 günün teslim süreleri  
- **Çıktı:** Ertesi gün teslim süresi  
- **Mimari:** `LSTM(64)` → `Dense(1)`  

### 2️⃣ Multivariate LSTM  
- **Girdi:** Teslim süresi + takvim öznitelikleri + kategorik dummy değişkenler  
- **Çıktı:** Ertesi gün teslim süresi  
- **Mimari:** `LSTM(64, return_sequences=True)` → `Dropout(0.2)` → `LSTM(32)` → `Dense(1)`  

### 3️⃣ Naive Benchmark  
- **Kural:** “Yarın = Bugün”  

### 4️⃣ Forecast (7 Gün İleri)  
- Takvim öznitelikleri (`dow`, `hour`) ileriye taşındı  
- Hava/trafik gibi bilinmeyen değişkenler için **hold-last** varsayımı kullanıldı  

---

## 📊 Sonuçlar

| Model        | MAE (dk) | RMSE (dk) |
|--------------|----------|-----------|
| Naive        | 14.3     | 16.1      |
| Univariate   | 15.3     | 15.3      |
| Multivariate | 14.8     | 15.0      |

### 🔎 Yorumlar
- **Univariate LSTM** → Trend ve genel yapıyı yakaladı fakat kısa veri sebebiyle sınırlı kaldı.  
- **Multivariate LSTM** → Ek özniteliklerle Naive’den daha iyi sonuç verdi.  
- **Forecast (7 gün)** → 123.8 dk → 124.9 dk aralığında, düzenli artış öngördü.  

---

## 📈 Görselleştirmeler

- Gerçek vs Tahmin grafikleri  
- 7 günlük forecast çizimi  
- preds_univariate  
- preds_multivariate  
- future_7day  

---

## 🔑 Öne Çıkan Noktalar

- Veri kısa (55 gün) → Modelin öğrenme kapasitesi sınırlı.  
- **LSTM + EarlyStopping + ReduceLROnPlateau** → Overfitting engellendi.  
- Takvim öznitelikleri forecast doğruluğunu artırdı.  
- Daha uzun veriyle ve eksojen değişkenlerin tahminiyle performans geliştirilebilir.  
- Model, küçük veri setinde bile LSTM’in sekans bağımlılıklarını yakalayabildiğini gösterdi.  

---
