# MNIST Veri Seti ile MLP Tabanlı El Yazısı Rakam Tanıma

**Öğrenci Adı:** Kevser ARSLAN
**Öğrenci Numarası:** 24120205011

## Proje Amacı
Bu projenin amacı, MNIST el yazısı rakam veri seti kullanılarak CNN (Evrişimli Sinir Ağları) kullanılmadan, tamamen Multi-Layer Perceptron (MLP) mimarisine dayalı bir model geliştirmektir. Proje kapsamında modelin eğitilmesi, performansının ölçülmesi ve kendi el yazımla oluşturduğum gerçek dünya verisi üzerinde test edilerek "Distribution Shift" (Veri Dağılımı Kayması) probleminin analiz edilmesi hedeflenmiştir.

## Veri Seti ve Ön İşleme
Projede 28x28 piksel boyutlarında, 0-9 arası gri tonlamalı rakamlardan oluşan MNIST veri seti kullanılmıştır.
- **Eğitim Seti (Train):** 50.000 görsel
- **Doğrulama Seti (Validation):** 10.000 görsel
- **Test Seti:** 10.000 görsel

Uygulanan Ön İşleme (Transform) Adımları: `ToTensor()` ve `Normalize((0.5,), (0.5,))`

## Model Mimarisi ve Hiperparametreler
Model, tamamen tam bağlı (fully connected) katmanlardan oluşan bir yapıdadır:
- **Girdi Katmanı:** 784 (28x28)
- **1. Gizli Katman (Hidden Layer):** 128 Nöron (Aktivasyon: ReLU)
- **2. Gizli Katman (Hidden Layer):** 64 Nöron (Aktivasyon: ReLU)
- **Çıktı Katmanı:** 10 Nöron (0-9 arası sınıflar)

**Eğitim Hiperparametreleri:**
- **Epoch:** 10
- **Batch Size:** 64
- **Optimizer:** Adam
- **Learning Rate:** 0.001
- **Loss Fonksiyonu:** CrossEntropyLoss

## Deney Sonuçları ve Başarı Oranları
Model yaklaşık 24 saniye süren 10 epoch'luk bir eğitim sürecinden geçirilmiş ve aşağıdaki sonuçlar elde edilmiştir:
- **Test Başarısı (Accuracy):** %97+ (Değişkenlik gösterebilir)
- Eğitim süresince Validation Loss değerlerinde stabil bir düşüş, Accuracy değerlerinde ise tutarlı bir artış gözlemlenmiştir (Repo içindeki grafikler incelenebilir).

### Hiperparametre Değişikliklerinin Etkisi (Analiz)
Model geliştirme sürecinde hiperparametrelerin performansa etkisi gözlemlenmiştir:
- **Optimizer Seçimi (SGD vs. Adam):** İlk denemelerde SGD kullanıldığında modelin öğrenme hızı yavaş kalırken, Adam optimizer'a geçildiğinde çok daha hızlı yakınsama (convergence) sağlanmış ve daha düşük epoch sayılarında yüksek doğruluğa ulaşılmıştır.
- **Learning Rate (Öğrenme Oranı):** Öğrenme oranı çok yüksek tutulduğunda loss değerlerinde dalgalanmalar (salınım) yaşanmış, 0.001 değeri ise ezberlemeye (overfitting) yol açmadan en stabil öğrenmeyi sağlamıştır.
- **Batch Size:** 64'lük yığın boyutu, donanım kaynaklarını verimli kullanırken gradient güncellemelerinin de yeterince tutarlı olmasını sağlamıştır.
---

## Kendi El Yazım (24120205011) Üzerinde Modelin Test Edilmesi ve "Distribution Shift" Analizi

Ödevin en öğretici kısmı, modeli kendi el yazımla test ettiğim aşamaydı. Gerçek dünya verisinin (Real-world data), modelin eğitildiği kontrollü MNIST verisinden ne kadar farklı olabileceğini 5 aşamalı bir deneme-yanılma süreciyle bizzat deneyimledim:

**1. Dijital Çizim ve İlk Deneme:**
Rakamları ilk olarak iPad üzerinde siyah zemin üzerine beyaz kalemle çizdim. Görüntüler 28x28 formatına getirilip normalize edilse de model çoğu rakamı yanlış tahmin etti (örneğin 2 ve 4'leri 1 olarak gördü). Bunun sebebi, benim çizgi kalınlığımın ve ortalama (centering) hizamın MNIST veri setindeki standart tarama formatından farklı olmasıydı.

**2. Çizgi Kalınlığını Artırma:**
Modelin çok ince çizgilere tepki veremediğini (çoğunu 7 veya 1 olarak yorumladığını) fark edip daha kalın keçeli kalem stiliyle yeni görseller çizdim. Ancak MLP mimarisi piksel konumlarına (uzamsal dağılıma) çok hassas olduğu için hafif eksen kaymaları bile modelin kafasını karıştırmaya yetti.

**3. Gerçek Kağıt-Kalem ve Format Sorunları:**
Dijital çizimden vazgeçip mermer bir masa üzerinde gerçek beyaz kağıda siyah kalemle rakamları yazdım. Ancak bu kez iki sorunla karşılaştım: 
- Telefon (iPhone) kamerası fotoğrafları HEIC formatında kaydettiği için kod okuyamadı (Dosyalar JPEG formatına dönüştürülerek çözüldü). 
- Beyaz zemin, modelin alıştığı "siyah arka plan" formatına tersti. Koda `1.0 - x` transformasyonu ekleyerek renkleri tersine çevirdim (Invert işlemi). Ancak geniş çekim nedeniyle mermer masa ve kağıt kenarları devasa parlak gürültülere dönüştü.

**4. Siyah Zemin Üzerinde Beyaz Kağıt Testi:**
Model sadece rakama odaklansın diye kağıdı siyah bir zemine koyup çektim. Fakat renkleri tersine çeviren kod parçası yüzünden bu kez dışarıdaki büyük siyah alan bembeyaz bir çerçeveye dönüştü.Model merkezdeki rakamı tamamen kaybetti. Çözüm olarak görselleri tam rakamın etrafından, sıfıra sıfır olacak şekilde kare formatında kırptım.

**5. Ortam Işığı, Gölgeler ve Nihai Çözüm (Thresholding):**
Sıkıca kırpılmış kağıtlarda dahi model bazı rakamlarda zorlandı. Analiz ettiğimde, kağıdın alt kısmına düşen hafif ortam gölgelerinin, renk tersinimi işleminden sonra parlak beyaz tabanlar oluşturduğunu fark ettim. Model (sadece piksellerin parlaklığına baktığı için) bu gölgeleri rakamın bir uzantısı sandı.
- **Çözüm:** Test kodundaki preprocessing adımına `(x > 0.6).float()` şeklinde basit bir eşikleme (thresholding) filtresi ekledim. Bu sayede gri gölgeler zorla silinip sadece en koyu siyah kalem izleri (beyaz piksele dönüştürülerek) bırakıldı.
- **Sonuç:** Bu son veri temizleme müdahalesinin ardından model, **24120205011** olan öğrenci numaramı %100 doğrulukla tahmin etmeyi başardı.

### Proje Dosya Yapısı ve Kullanım
Projeyi çalıştırmak ve test etmek için gerekli dosya hiyerarşisi şu şekildedir:
- `mnist_odev.ipynb`: Modelin eğitildiği, test edildiği ve grafiklerin çizdirildiği ana kod dosyası.
- `el_yazim/`: Öğrenci numarama (24120205011) ait kendi el yazımla oluşturduğum ve preprocessing (thresholding vb.) işlemleri için 28x28 boyutuna uygun hale getirilmiş test görsellerinin (`0.jpg`, `1.jpg`, `2.jpg`, `4.jpg`, `5.jpg`) bulunduğu klasör.

Koddaki son test hücresi çalıştırıldığında, model otomatik olarak `el_yazim/` klasöründeki görselleri okuyacak ve tahminlerini ekrana yazdıracaktır.

### Genel Değerlendirme
Bu proje, yüksek Test doğruluğunun (%97) gerçek dünyada doğrudan başarı anlamına gelmediğini kanıtlamıştır. Model mimarisi ne kadar doğru kurulursa kurulsun, verinin (ışık, gölge, kırpma, merkezleme) modele uygun şekilde ön işlemlerden (preprocessing) geçirilmesinin yapay zeka projelerindeki en kritik aşama olduğunu bizzat tecrübe ederek öğrenmiş oldum.