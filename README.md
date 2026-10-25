# Su İçilebilirliği Sınıflandırması
## Makine Öğrenmesi Proje Raporu

### 1. Projenin Amacı

Bu projenin amacı, su örneklerine ait fizikokimyasal özellikleri kullanarak `Potability` hedef değişkenini sınıflandırmak ve farklı makine öğrenmesi algoritmalarının performanslarını karşılaştırmaktır.

Proje kapsamında veri analizi, eksik değerlerin işlenmesi, model eğitimi, hiperparametre optimizasyonu, performans değerlendirmesi, özellik öneminin incelenmesi ve karar eşiği analizi gerçekleştirilmiştir.

### 2. Veri Seti

Veri seti toplam **3.276 kayıt** ve dokuz girdi değişkeninden oluşmaktadır.

Girdi değişkenleri:

- `ph`
- `Hardness`
- `Solids`
- `Chloramines`
- `Sulfate`
- `Conductivity`
- `Organic_carbon`
- `Trihalomethanes`
- `Turbidity`

Hedef değişken `Potability` olup iki sınıftan oluşmaktadır:

| Sınıf | Kayıt sayısı | Oran |
|---|---:|---:|
| 0 | 1.998 | %60,99 |
| 1 | 1.278 | %39,01 |
| Toplam | 3.276 | %100 |

Veri seti, `stratify=y` kullanılarak %80 eğitim ve %20 test verisi şeklinde ayrılmıştır. Eğitim verisinde 2.620, test verisinde 656 kayıt bulunmaktadır. Bölme işlemi `random_state=42` ile tekrarlanabilir hâle getirilmiştir.

### 3. Veri Ön İşleme

Veri analizi sonucunda üç sütunda eksik değerler belirlenmiştir:

| Değişken | Eksik değer sayısı |
|---|---:|
| `ph` | 491 |
| `Sulfate` | 781 |
| `Trihalomethanes` | 162 |

Eksik değerler, `SimpleImputer(strategy="median")` kullanılarak medyan değerleriyle doldurulmuştur. Veri setinde tekrarlanan satır bulunmamıştır.

Logistic Regression modeli için `StandardScaler` ile ölçeklendirme uygulanmıştır. Random Forest ve Gradient Boosting modellerinde ölçeklendirme kullanılmamıştır.

### 4. Kullanılan Algoritmalar

**Logistic Regression:** Doğrusal sınıflandırma yaklaşımıdır. Veriler arasındaki doğrusal ilişkileri kullanarak sınıf tahmini yapar.

**Random Forest:** Birden fazla karar ağacının tahminlerini birleştiren topluluk öğrenmesi algoritmasıdır. Hiperparametre aramasıyla `max_depth`, `min_samples_leaf`, `min_samples_split`, `max_features` ve `n_estimators` parametreleri değerlendirilmiştir.

**Gradient Boosting:** Karar ağaçlarını sıralı olarak eğiterek önceki aşamalardaki hataları azaltmaya çalışan topluluk öğrenmesi algoritmasıdır. `n_estimators`, `learning_rate` ve `max_depth` parametreleri GridSearchCV ile karşılaştırılmıştır.

### 5. Model Değerlendirme Yöntemi

Modellerin performansı aşağıdaki metriklerle ölçülmüştür:

- **Accuracy:** Bütün tahminler içinde doğru tahminlerin oranı.
- **Precision:** Modelin pozitif sınıf olarak tahmin ettiği örneklerin ne kadarının gerçekten pozitif olduğu.
- **Recall:** Gerçek pozitif örneklerin ne kadarının yakalandığı.
- **F1-score:** Precision ve recall arasındaki dengeyi ölçen metrik.

Random Forest ve Gradient Boosting hiperparametre aramalarında beş katlı çapraz doğrulama kullanılmıştır. Random Forest için seçim ölçütü F1-score olmuştur.

### 6. Algoritmaların Karşılaştırılması

Aşağıdaki sonuçlar test verisinde, varsayılan 0,5 karar eşiğiyle elde edilmiştir.

| Model | Accuracy | Precision (1) | Recall (1) | F1 (1) |
|---|---:|---:|---:|---:|
| Random Forest | **0,671** | **0,661** | **0,320** | **0,432** |
| Gradient Boosting | 0,654 | 0,614 | 0,305 | 0,407 |
| Logistic Regression | 0,610 | 0,000 | 0,000 | 0,000 |

Random Forest en yüksek test doğruluğuna ve pozitif sınıf F1-score değerine ulaşmıştır. Gradient Boosting sonuçları Random Forest'a yakın olmakla birlikte daha düşük kalmıştır.

Logistic Regression, test verisindeki tüm örnekleri sınıf 0 olarak tahmin etmiştir. Bu nedenle sınıf 1 için precision, recall ve F1-score değerleri sıfır olmuştur. Yaklaşık %61 doğruluk elde etmesi, çoğunluk sınıfının veri setindeki oranına yakındır.

### 7. Random Forest Hiperparametre Optimizasyonu

Random Forest için önce daha geniş bir hiperparametre araması planlanmış, hesaplama süresini azaltmak amacıyla arama alanı daraltılmıştır.

Son aramada en iyi bulunan parametreler:

| Parametre | Seçilen değer |
|---|---|
| `max_depth` | `None` |
| `min_samples_leaf` | `1` |
| `min_samples_split` | `5` |
| `max_features` | `sqrt` |
| `n_estimators` | `100` |

Bu kombinasyonun beş katlı çapraz doğrulamadaki ortalama F1-score değeri **0,467** olmuştur.

Test verisindeki sonuçlar:

- Accuracy: **0,671**
- Precision: **0,661**
- Recall: **0,320**
- F1-score: **0,432**

### 8. Karar Eşiği Analizi

Random Forest modelinde sınıf 1 için karar eşiği incelenmiştir. Varsayılan eşik 0,5 iken sınıf 1 recall değeri düşüktür. Eşik düşürüldüğünde daha fazla örnek sınıf 1 olarak tahmin edilmiş; recall ve F1 artarken accuracy azalmıştır.

Eğitim verisindeki çapraz doğrulama olasılıklarında elde edilen sonuçlardan bazıları:

| Eşik | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|
| 0,300 | 0,522 | 0,441 | 0,848 | **0,581** |
| 0,350 | 0,573 | 0,470 | 0,734 | 0,573 |
| 0,400 | 0,635 | 0,529 | 0,599 | 0,562 |
| 0,450 | 0,671 | 0,597 | 0,481 | 0,533 |

F1-score açısından en iyi eşik, denenen değerler arasında 0,300 olmuştur.

Bu eşik test verisinde uygulandığında:

| Metrik | Sonuç |
|---|---:|
| Accuracy | 0,532 |
| Precision | 0,446 |
| Recall | 0,820 |
| F1-score | 0,578 |

Karar eşiği 0,5'ten 0,3'e düşürüldüğünde recall %32'den %82'ye yükselmiştir. Ancak accuracy %67,1'den %53,2'ye düşmüştür. Bu sonuç, model kararlarında precision, recall ve accuracy arasında bir ödünleşim bulunduğunu göstermektedir.

Eşik seçimi eğitim verisindeki çapraz doğrulama tahminlerine dayandırılmıştır. Bununla birlikte hiperparametre seçimi ve eşik belirleme aşamalarının tamamen bağımsız çapraz doğrulama içinde yapılması, daha katı bir değerlendirme için tercih edilebilir.

### 9. Özellik Önemleri

Random Forest modelinde elde edilen özellik önemleri aşağıdaki şekilde sıralanmıştır:

| Değişken | Özellik önemi |
|---|---:|
| `ph` | 0,129 |
| `Sulfate` | 0,128 |
| `Hardness` | 0,122 |
| `Chloramines` | 0,116 |
| `Solids` | 0,112 |
| `Turbidity` | 0,100 |
| `Conductivity` | 0,100 |
| `Organic_carbon` | 0,097 |
| `Trihalomethanes` | 0,096 |

Bu sonuçlarda `ph`, `Sulfate` ve `Hardness` en yüksek özellik önemlerine sahiptir. Ancak bu değerler değişkenlerin nedensel etkisini göstermez; yalnızca kullanılan Random Forest modelinin tahminlerinde göreli önemlerini yansıtır.

### 10. Sonuç ve Değerlendirme

Bu çalışmada üç sınıflandırma algoritması uygulanmış ve aynı test veri setinde karşılaştırılmıştır. Mevcut deneyler arasında Random Forest en başarılı model olmuştur. Bununla birlikte varsayılan karar eşiğinde sınıf 1 için recall değerinin 0,320 olması, modelin bu sınıfa ait örneklerin önemli bir bölümünü kaçırdığını göstermektedir.

Karar eşiği analizi, recall ve F1 değerlerinin artırılabildiğini ancak bunun yanlış pozitifleri artırarak doğruluğu düşürebildiğini göstermiştir. Dolayısıyla en uygun model ve karar eşiği, uygulamanın önceliğine göre seçilmelidir.

Sonuçlar, bu veri seti ve mevcut deney düzeni için geçerlidir. Daha yüksek performans elde etmek için veri kalitesinin, eksik değerlerin ele alınış biçiminin, alternatif özelliklerin ve ek algoritmaların araştırılması önerilmektedir.

**Sınırlılık:** Bu çalışma bir makine öğrenmesi sınıflandırma deneyidir. Model tahminleri tek başına suyun güvenli biçimde içilebilir olduğunu belirlemek için kullanılamaz; bunun için ilgili standartlara uygun laboratuvar ölçümleri ve uzman değerlendirmesi gerekir.
