# Tırsan Treyler – Hidrolik Dümenlenebilir Low-Bed (SAMİB)
## Safety 4.0 Laboratuvar Dokümantasyonu ve SWOT Analizi

[![Standard: IEC 62443](https://img.shields.io/badge/Standard-IEC%2062443--3--2-blue.svg)](#)
[![Standard: ISO 13849-1](https://img.shields.io/badge/Safety-ISO%2013849--1%20PL%20d%2Fe-red.svg)](#)
[![Regulation: UNECE R155/R156](https://img.shields.io/badge/Automotive-UNECE%20R155%2FR156-green.svg)](#)
[![Domain: Heavy Transport](https://img.shields.io/badge/Industry-Heavy%20Duty%20Low--Bed-orange.svg)](#)

---

## 📌 Proje Özeti

Bu çalışma alanı, **Tırsan Treyler** tarafından üretilen **Hidrolik Dümenlenebilir Low-Bed (SAMİB)** ağır tonajlı taşıma treylerinin, **Safety 4.0 (Fonksiyonel Emniyet + Endüstriyel Siber Güvenlik)** prensipleri doğrultusunda analiz edildiği kapsamlı teknik dokümantasyon setini içerir.

Geleneksel fonksiyonel emniyet (ISO 13849-1 / IEC 62061) yaklaşımları, donanım arızaları ve çevre koşullarına odaklanırken; günümüz telematik, uzaktan kumanda ve hücresel ağlara bağlı akıllı treyler dünyasında, bir emniyet fonksiyonunun en zayıf halkası **siber güvenlik açıkları** haline gelmiştir. Bu dokümantasyon serisi, siber tehditlerin fiziksel emniyeti (makaslama, devrilme, ezilme) nasıl bozabileceğini inceler ve **IEC 62443**, **ISO 13849** ve **UNECE R155/R156** standartlarına tam uyumlu bir savunma derinliği mimarisi sunar.

---

## 🚜 Makine ve Sistem Mimarisi Özeti

Tırsan Hidrolik Dümenlenebilir Low-Bed (SAMİB), gabari dışı ve onlarca tonluk ağır yüklerin (iş makineleri, trafolar, rüzgar türbini bileşenleri vb.) otoyol ve zorlu şantiye ortamlarında taşınması için tasarlanmıştır.

```mermaid
graph TD
    subgraph Dış Dünya & Uzak Erişim
        REMOTE["Servis Teknisyeni / Mühendis"]
        CLOUD["Bulut Filo Yönetim Platformu"]
        TRACTOR["Çekici Araç (ISO 11992 / CAN)"]
    end

    subgraph Telematik & Ağ Geçidi [Zone 3 - DMZ]
        GW["Telematik / Uzak Bakım Gateway (4G/LTE)"]
        RC_RX["Kablosuz Kumanda Alıcısı"]
        OBD["Fiziksel Teşhis Soketi (OBD/J1939)"]
    end

    subgraph Kontrol & Tahrik [Zone 2 - OT]
        STEER_ECU["Ana Dümenleme Kontrol ECU'su"]
        HMI["Treyler Yerel HMI Paneli"]
        HYD_PROP["Hidrolik Oransal Yön Valfleri"]
        PUMP["Hidrolik Güç Ünitesi (HPU)"]
    end

    subgraph Emniyet Yolu / Safety Path [Zone 1 - Safety / SRCS]
        SAFETY_PLC["Safety PLC / Emniyet Kontrolörü"]
        E_STOP["Acil Durdurma (E-Stop) Butonları"]
        DEAD_MAN["Kumanda Dead-Man Anahtarı"]
        LIMIT_SENSORS["Açı ve Basınç Limit Sensörleri"]
        SPEED_LOCK["Bağımsız Tekerlek Hız Sensörleri"]
        SAFETY_VALVES["Çift Kanallı Emniyet Kesme Valfleri (STO Eşdeğeri)"]
        CYLINDERS["Hidrolik Silindirler (Kilitli Çekvalfli)"]
    end

    REMOTE -->|VPN + MFA| GW
    CLOUD <-->|TLS 1.3 Telemetri| GW
    TRACTOR -->|J1939| GW
    
    GW -->|Filtrelenmiş CAN| STEER_ECU
    RC_RX -->|Kumanda Açı Talebi| STEER_ECU
    OBD -->|Fiziksel Kilitli| GW

    STEER_ECU --> HYD_PROP
    HMI --> STEER_ECU
    PUMP --> HYD_PROP
    HYD_PROP --> SAFETY_VALVES

    %% Safety Bağımsız Hattı (Hardwired)
    E_STOP ==>|Kuru Kontak| SAFETY_PLC
    DEAD_MAN ==>|Güvenli RF/Kuru Kontak| SAFETY_PLC
    LIMIT_SENSORS ==>|4-20mA / Çift Kanal| SAFETY_PLC
    SPEED_LOCK ==>|Frekans / Donanımsal| SAFETY_PLC
    
    SAFETY_PLC ==>|Çift Kanal Kesme| SAFETY_VALVES
    SAFETY_VALVES ==>|Hidrolik Akış Kontrolü| CYLINDERS
    SAFETY_PLC -.->|Yalnızca Durum Bilgisi (Read-Only)| STEER_ECU

    classDef safety fill:#ffdddd,stroke:#cc0000,stroke-width:2px;
    classDef ot fill:#e6f3ff,stroke:#0066cc,stroke-width:2px;
    classDef dmz fill:#fff2cc,stroke:#d6b656,stroke-width:2px;
    classDef ext fill:#f5f5f5,stroke:#666666,stroke-width:1px;

    class SAFETY_PLC,E_STOP,DEAD_MAN,LIMIT_SENSORS,SPEED_LOCK,SAFETY_VALVES,CYLINDERS safety;
    class STEER_ECU,HMI,HYD_PROP,PUMP ot;
    class GW,RC_RX,OBD dmz;
    class REMOTE,CLOUD,TRACTOR ext;
```

### Kritik Sistem Bileşenleri:
1. **Normal Kontrol Yolu (Normal Control Path):**
   - Dingil Kontrol ECU'su, yerel operatör paneli (HMI), SAE J1939 CAN-Bus altyapısı, oransal hidrolik yön valfleri ve telematik modülü.
2. **Emniyet Yolu (Safety-Related Control Path - SRCS):**
   - E-Stop acil durdurma butonları, ölü adam (Dead-Man) anahtarı, bağımsız açı/basınç limit sensörleri, bağımsız hız eşiği kilidi (>15-20 km/s hızda manuel manevrayı kilitleyen donanım), Safety PLC ve çift kanallı hidrolik emniyet kesme valfleri (STO - Safe Torque Off / Safe De-energize eşdeğeri).

---

## 📚 Dokümantasyon İndeksi

Aşağıdaki tabloda, projenin tüm laboratuvar föyleri ve stratejik analiz raporu listelenmiştir:

| No | Doküman Adı | Açıklama | Ana Çıktı / Kapsam |
| :---: | :--- | :--- | :--- |
| **01** | [01-Lab1-Safety-Architecture.md](./01-Lab1-Safety-Architecture.md) | **Safety Architecture** | Fiziksel tehlikeler, Safety Function, Normal vs. Safety PLC farkı, PLr/SIL ve dijital/siber bağımlılıklar. |
| **02** | [02-Lab2-IEC62443-Zone-Conduit.md](./02-Lab2-IEC62443-Zone-Conduit.md) | **IEC 62443 Zone & Conduit** | Varlık envanteri, 4 güvenlik bölgesi, geçiş kanalları (conduits) ve güvenli uzak erişim karar matrisi. |
| **03** | [03-Lab3-Threat-Risk-Matrix.md](./03-Lab3-Threat-Risk-Matrix.md) | **Threat & Risk Matrix** | $L \times S \times E$ skorlaması, Risk Register, $S=3$ kuralı ("Kritik Emniyet Bayrağı") ve hafifletme önlemleri. |
| **04** | [04-Lab4-Secure-Remote-Access.md](./04-Lab4-Secure-Remote-Access.md) | **Secure Remote Access** | Mevcut vs. Hedef mimari karşılaştırması, 5 güvenli erişim ilkesi, savunma derinliği ve Functional Safety gereksinimleri. |
| **05** | [05-Lab5-SBOM-CVE-Patch.md](./05-Lab5-SBOM-CVE-Patch.md) | **SBOM, CVE & Patch Management** | CVE-2026-XXXX (CVSS 9.8) analizi, 3 kontrol şartı, HIL test döngüsü, Safety Validation ve telafi edici önlemler. |
| **06** | [06-SWOT-Analizi-Tirsan-LowBed.md](./06-SWOT-Analizi-Tirsan-LowBed.md) | **Safety 4.0 SWOT Analizi** | Tırsan SAMİB treylere özgü Güçlü/Zayıf Yönler, Fırsatlar (CRA, R155/156) ve Tehditler; TOWS stratejileri. |

---

## 🛡️ Standartlar ve Yasal Çerçeve

Bu dokümantasyon seti aşağıdaki uluslararası standartlar ve regülasyonlar referans alınarak kurgulanmıştır:

- **ISO 13849-1 / -2:** Makinelerde Güvenlik - Kumanda Sistemlerinin Güvenlikle İlgili Kısımları (Performance Level - PLr).
- **IEC 62443-3-2 / -3-3 / -4-2:** Endüstriyel İletişim Ağları - Sistem Güvenliği, Bölge & Geçiş Güvenliği ve Cihaz Güvenliği Gereksinimleri.
- **UNECE R155 & R156:** Siber Güvenlik Yönetim Sistemi (CSMS) ve Yazılım Güncelleme Yönetim Sistemi (SUMS).
- **AB Siber Dayanıklılık Yasası (Cyber Resilience Act - CRA):** Dijital unsurlar içeren donanım ve yazılımlarda zorunlu siber güvenlik standartları.

---

*Dokümantasyon Hazırlık Tarihi: Kasım 2026*  
*Kurum: Tırsan Treyler A.Ş. Ar-Ge & Güvenlik Mühendisliği / Safety 4.0 Bootcamp*
