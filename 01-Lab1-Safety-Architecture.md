# LABORATUVAR 1: Safety Architecture & Safety Function Map

**Makine / Sistem:** Tırsan Treyler – Hidrolik Dümenlenebilir Low-Bed (SAMİB)  
**Doküman Kodu:** LAB-01-SAF-ARCH  
**İlgili Standartlar:** ISO 13849-1, IEC 62061, ISO 12100  

---

## 1. Sistem Bağlamı (System Context)

Ağır yük taşımacılığında (iş makineleri, rüzgar kanatları, endüstriyel trafolar vb.) kullanılan bu hidrolik dümenlenebilir low-bed treylerde; 
* Yüksek basınçlı hidrolik silindirler (250–350 bar çalışma basıncı),
* Deveboynu (gooseneck) hidrolik indirme/kaldırma ve ayırma mekanizması,
* Arka dingillerin otonom, çekiciye senkron veya kablosuz uzaktan kumandalı elektro-hidrolik dümenleme (steering) sistemi yer almaktadır.

Sistem iki temel kontrol hattından oluşur:
1. **Normal Kontrol Yolu (Normal Control Path):** Dingil Kontrol ECU'su, operatör HMI paneli, SAE J1939 CAN-Bus veri yolu, hidrolik oransal yön valfleri ve telematik/uzak teşhis modülü.
2. **Emniyet Yolu (Safety-Related Control Path - SRCS):** Acil Durdurma (E-Stop) butonları, kablosuz kumanda Dead-Man anahtarı, bağımsız açı/basınç limit sensörleri, hız eşiği kilitleme mantığı (>15-20 km/s üzerinde manevrayı kilitleyen donanım), Safety PLC / Emniyet Kontrolörü ve Çift Kanallı Hidrolik Emniyet Valfleri (STO - Safe Torque Off eşdeğeri hidrolik kesme).

---

## 2. Temel Emniyet Soruları ve Analizleri

### Soru 1: Bu makinede tehlike nedir? İnsan neyden zarar görür?

Bu treyler platformunda insan sağlığı ve yaşamını doğrudan tehdit eden üç temel fiziksel tehlike bulunmaktadır:

1. **Hidrolik Boşalma ve Ezilme Tehlikesi (Crushing Hazard):**
   Yükleme/boşaltma sırasında hidrolik hortum patlaması, valf sızıntısı veya kontrolsüz basınç düşmesi sonucu deveboynunun veya arka rampaların aniden yere çökmesi. Altında veya çevresinde çalışan operatörler ezilme veya uzuv kaybı riskiyle karşılaşır.
2. **Yüksek Hızda İstemsiz Dingil Dönmesi ve Makaslama/Devrilme (Rollover & Jackknifing Hazard):**
   Otoyol hızlarında (örn. 80 km/s) arka hidrolik dümenleme sisteminin arıza, yazılım hatası veya harici müdahale nedeniyle istemsizce sağa veya sola açı alması. Bu durum çekici ve treylerin derhal makaslamasına, şeritten çıkmasına veya devrilmesine neden olarak ölümcül çoklu araç kazalarına yol açar.
3. **Manevra Sırasında Sıkışma Tehlikesi (Pinching / Trapping Hazard):**
   Dar alanlarda veya şantiye manevralarında uzaktan kumanda ile dingiller yönlendirilirken treyler şasisi ile tekerlek bloğu veya çevredeki sabit yapılar arasına saha personelinin sıkışması.

---

### Soru 2: Safety Function (Emniyet Fonksiyonu) hangisidir?

> **Fonksiyon Tanımı:**  
> **[E-Stop + Hız/Açı Limit Sensörleri + Dead-Man Butonu] $\longrightarrow$ [Safety PLC / Kontrolör] $\longrightarrow$ [Emniyet Selenoid Valfleri (Hidrolik Kilitleme)]**

* **Yüksek Hız Dümenleme Kilitleme:** Çekici veya treyler hızı belirlenen emniyet eşiğini (>15-20 km/s) aştığında arka dingillerin manuel veya uzaktan kumanda ile açı alması donanımsal olarak engellenir; dingiller düz seyir (0° kilit) pozisyonunda hidrolik ve mekanik olarak kilitlenir.
* **Acil Kesme ve Basınç Kilit:** Operatör E-Stop butonuna bastığında, kablosuz kumanda sinyali koptuğunda, kumanda Dead-Man butonu bırakıldığında veya açı limit sensörleri strok sınırını aştığında hidrolik güç derhal kesilir ve pilot kumandalı emniyet çekvalfleri kapatılarak silindirler mevcut pozisyonunda dondurulur.

---

### Soru 3: Safety-Related Control System (SRCS) nerede?

Aşağıdaki mimari diyagramında **kırmızı çift çizgili çerçeve** ile gösterilen bağımsız emniyet hattı SRCS bileşenlerini ifade eder. Bu hat, normal sürüş ve konfor sağlayan CAN-Bus/ECU hattından galvanik ve mantıksal olarak izole edilmiştir.

```mermaid
flowchart TD
    subgraph OperationalPath["Normal Kontrol Yolu (Standart OT)"]
        HMI["Yerel HMI / Panel"]
        TELEMATICS["Telematik & 4G Modem"]
        RC_NORMAL["Kumanda Alıcısı (Yön Talebi)"]
        ECU["Normal Dingil ECU'su"]
        PROP_VALVE["Oransal Hidrolik Valfler"]
    end

    subgraph SafetyPath["Emniyet Yolu (Safety-Related Control System - SRCS)"]
        ESTOP["E-Stop Butonları (Kuru Kontak)"]
        DEADMAN["Kumanda Dead-Man Butonu"]
        SPEED_SENS["Tekerlek Hız Emniyet Sensörü"]
        ANGLE_SENS["Açı/Basınç Emniyet Limit Sensörleri"]
        SAFETY_PLC{"Safety PLC / Emniyet Rölesi"}
        CUT_VALVE["Çift Kanallı Emniyet Kesme Valfi"]
        HYD_LOCK["Pilot Kumandalı Kilit Çekvalfleri"]
    end

    subgraph Actuators["Hidrolik Mekanik Eyleyiciler"]
        PUMP["Hidrolik Pompa / HPU"]
        CYLINDERS["Dümenleme & Gooseneck Silindirleri"]
    end

    %% Normal Akış
    TELEMATICS -.->|CAN J1939| ECU
    HMI -->|CAN| ECU
    RC_NORMAL -->|CAN/PWM| ECU
    ECU -->|Sürüş Sinyali| PROP_VALVE
    PUMP --> PROP_VALVE
    PROP_VALVE --> CUT_VALVE

    %% Safety Akış
    ESTOP ==>|Kanal A & B| SAFETY_PLC
    DEADMAN ==>|Emniyetli RF Girişi| SAFETY_PLC
    SPEED_SENS ==>|Donanımsal Darbe| SAFETY_PLC
    ANGLE_SENS ==>|Çift Kanal Analog| SAFETY_PLC

    SAFETY_PLC ==>|24V Güvenli Enerji Kesme| CUT_VALVE
    SAFETY_PLC ==>|Pilot Basınç Tahliyesi| HYD_LOCK
    CUT_VALVE ==> CYLINDERS
    HYD_LOCK ==> CYLINDERS

    %% Güvenli Durum Geri Beslemesi
    SAFETY_PLC -.->|Yalnızca Teşhis (Read-Only)| ECU

    classDef safe fill:#ffe6e6,stroke:#b30000,stroke-width:2px;
    classDef oper fill:#e6f2ff,stroke:#004080,stroke-width:1px;
    classDef act fill:#f2f2f2,stroke:#333333,stroke-width:1px;

    class ESTOP,DEADMAN,SPEED_SENS,ANGLE_SENS,SAFETY_PLC,CUT_VALVE,HYD_LOCK safe;
    class HMI,TELEMATICS,RC_NORMAL,ECU,PROP_VALVE oper;
    class PUMP,CYLINDERS act;
```

---

### Soru 4: Normal PLC (ECU) ile Safety PLC farkı nedir?

| Özellik | Normal PLC / Dingil Kontrol ECU'su | Safety PLC / Emniyet Kontrolörü |
| :--- | :--- | :--- |
| **Temel Görevi** | Operatör komutlarına göre dingil açılarını hesaplamak, hidrolik pompayı sürmek ve akıcı sürüş/manevra sağlamak. | Hata anında veya limit aşımlarında sistemi öngörülebilir Güvenli Duruma (Safe State) geçirmek. |
| **Donanım Mimarisi** | Tek işlemcili (1oo1) standart otomotiv mikrodenetleyicisi. | Çift işlemcili, çapraz karşılaştırmalı (1oo2D veya 2oo3) yedekli mimari. |
| **Öz-Tanılama (Self-Diagnostics)** | Sınırlı hata kodları (DTC), yazılım çökmelerinde watchdog yeniden başlatması. | Donanım döngü testleri, darbe testi (pulse test), saat senkronizasyonu ve anlık açık devre/kısa devre tespiti. |
| **Hata Davranışı** | Arıza anında son durumu koruyabilir veya rastgele çıkış verebilir. | **Fail-Safe:** Dahili veya harici herhangi bir arızada çıkışları anında enerjisiz bırakarak hidroliği kilitler. |
| **Sertifikasyon** | ISO 16750, standart otomotiv E-Mark. | ISO 13849-1 (Cat 4 / PL e), IEC 61508 / IEC 62061 (SIL 3) bağımsız TÜV sertifikalı. |

---

### Soru 5: PLr / SIL neden var? Bu fonksiyon için neden önemli?

* **Tanım ve Amaç:**  
  ISO 13849-1 standardına göre **Gerekli Performans Seviyesi ($PL_r$)**, bir emniyet fonksiyonunun tehlikeli bir arızaya yol açma olasılığını ($PFH_D$) kabul edilebilir bir sınırın altında tutmak için zorunlu kılınan güvenilirlik metriğidir.
* **Risk Grafiği Parametreleri:**
  - **Şiddet (Severity - S2):** Otoyolda treylerin kontrolsüz açılan dingiller nedeniyle devrilmesi veya rampa çökmesi doğrudan ölüm veya kalıcı ağır yaralanma ile sonuçlanır ($S2$).
  - **Maruz Kalma Sıklığı (Frequency - F2):** Ağır nakliye operasyonlarında araç günlerce seyir halindedir ve operatörler şantiyede rampa etrafında sürekli bulunur ($F2$).
  - **Tehlikeden Kaçınma Olasılığı (Possibility of Avoidance - P2):** 80 km/s hızla seyrederken treylerin istemsiz makaslamasından fiziksel olarak kaçınmak neredeyse imkansızdır ($P2$).
* **Sonuç ($PL_r = d$ veya $e$ / $SIL\ 2$ veya $3$):**  
  Tırsan hidrolik dümenleme ve deveboynu emniyet zincirinde $PL_r = d$ (ve kritik otoyol kilidi için $PL_r = e$) gereklidir. Bu seviye, donanımda tekil arızaların emniyet kaybına yol açmamasını (Kategori 3 veya Kategori 4 mimari) ve emniyet fonksiyonunun yıllık tehlikeli arıza ihtimalinin $10^{-7}$ ile $10^{-8}$ seviyesinde olmasını yasal ve teknik olarak garanti eder.

---

### Soru 6: Safe State (Güvenli Durum) nedir?

Tırsan Hidrolik Dümenlenebilir Low-Bed için Güvenli Durum:
> **Tüm hidrolik yön valflerinin ve ana pompanın enerjisinin kesildiği (de-energized), pilot kumandalı çekvalflerin yay kuvvetiyle kapanarak hidrolik silindirleri mevcut konumunda hapsettiği ve dingillerin mekanik/hidrolik olarak kilitlendiği statik durumdur.**

* **De-energize to Trip Prensibi:** Kablo kopması, güç kaybı, sigorta atması veya yazılım kilitlenmesinde valflere giden 24V enerji kesilir; yay geri dönüşlü valfler hidrolik akışı anında sıfırlar.
* **Mekanik Kilit:** Seyir esnasında emniyet devreye girdiğinde dingiller merkezleme silindirleri veya konik pimler vasıtasıyla 0° düz konumda mekanik olarak kilitlenir.

---

### Soru 7: Safety Function hangi dijital bağımlılıklara sahip?

Geleneksel elektro-mekanik emniyet zincirlerinden farklı olarak, Safety 4.0 kapsamında bu fonksiyon şu dijital varlıklara bağımlıdır:
1. **Safety PLC Firmware ve Kalibrasyon Parametreleri:** Dümenleme açı limitleri, hız eşik değerleri (örn. 15 km/s) ve basınç tavan noktaları flash bellekte tutulur.
2. **SAE J1939 CAN-Bus Hız Verisi:** Çekiciden veya treyler EBS modülünden gelen tekerlek devir / araç hızı mesajları ($PGN\ 65265$ / $Wheel-Based\ Vehicle\ Speed$).
3. **Kablosuz Kumanda Eşleştirme ve Kriptografik Anahtarlar:** Kumanda ile alıcı arasındaki frekans atlamalı (FHSS) ve yuvarlanan kodlu (Rolling Code) yetkilendirme anahtarları.
4. **Servis Teşhis Arayüzü (OBD-II / J1939 Portu):** Kalibrasyon, açı sıfırlama ve hata silme için kullanılan servis yazılımı protokolleri.
5. **Telematik / IoT Gateway Bağlantısı:** Uzaktan arıza teşhisi, konum takibi ve filo yönetimi sağlayan 4G/LTE modem veri hattı.

---

### Soru 8: Siber müdahale hangi noktadan bu fonksiyonu bozar?

* **Saldırı Vektörü:** **Telematik / Uzak Bakım Modülü veya Şifresiz Kablosuz Kumanda Arayüzü.**
* **Saldırı Mekanizması:**
  1. İnternete açık veya zayıf kimlik doğrulamalı telematik modülünü ele geçiren bir saldırgan, treyler CAN-Bus hattına sızar.
  2. Saldırgan, araç 80 km/s hızla otoyolda seyrederken CAN hattına yüksek öncelikli **"Hız = 0 km/s" veya "Park Modu Aktif"** sahte mesaj paketleri enjekte eder ($CAN\ Bus\ Spoofing / Message\ Injection$).
  3. Ana Dümenleme ECU'su aracın durduğunu zannederek hız koruma kilidini serbest bırakır.
  4. Saldırgan aynı hat üzerinden hidrolik oransal valflere **"Tam Sağ Dümenleme Açısı"** komutu basar.
  5. **Sonuç:** Otoyolda tam yükle seyreden treylerin arka dingilleri aniden sağa sapar; treyler makaslar, bariyerleri parçalar ve arkasındaki araçların altına alarak ölümcül bir zincirleme trafik faciasına sebep olur.
