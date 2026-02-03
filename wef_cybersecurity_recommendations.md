# Siber Güvenlik Bakış Açısıyla WEF Subscription Önerileri

Mevcut subscription güçlü bir taban sunuyor (logon, hesap/grup değişiklikleri, process create/terminate, servis, credential manager, scheduled task). Aşağıdaki eklemeler tehdit tespiti, credential abuse, lateral movement ve persistence tespiti için önerilir.

---

## 1. Öncelik: Yüksek (hemen eklenmeli)

### 1.1 Kerberos (Security)

**Eksik Event ID’ler:** 4768, 4769, 4770, 4771

| Event ID | Açıklama | Neden önemli |
|----------|----------|----------------|
| **4768** | Kerberos TGT (Ticket Granting Ticket) istendi | Golden Ticket, Pass-the-Ticket, normal oturum açma davranışı |
| **4769** | Kerberos servis bileti istendi / verildi | Silver Ticket, lateral movement, servis hesabı abuse |
| **4770** | Kerberos bileti yenilendi | Uzun süreli oturum, persistence |
| **4771** | Kerberos pre-authentication başarısız | Parola brute force, credential stuffing tespiti |

**Kullanım:** Golden/Silver ticket, Kerberos delegation abuse, brute force tespiti.

---

### 1.2 NTLM (Security)

**Eksik Event ID:** 4776

| Event ID | Açıklama | Neden önemli |
|----------|----------|----------------|
| **4776** | DC, NTLM kimlik doğrulaması yaptı | Pass-the-Hash, NTLM relay, legacy auth kullanımı; mümkünse NTLM’i kapatıp sadece Kerberos’a geçilmesi önerilir, geçiş sürecinde mutlaka loglanmalı |

**Kullanım:** Credential theft/relay, legacy auth izleme.

---

### 1.3 Hesap kilidi (Security)

**Eksik Event ID:** 4740

| Event ID | Açıklama | Neden önemli |
|----------|----------|----------------|
| **4740** | Hesap kilitlendi | Brute force / parola saldırısı tespiti, account lockout politikası izleme |

---

### 1.4 PowerShell (Script Block / Operational)

**Kanal:** `Microsoft-Windows-PowerShell/Operational`

| Event ID | Açıklama | Neden önemli |
|----------|----------|----------------|
| **4103** | Module logging (komutlar) | PowerShell ile çalıştırılan komutlar |
| **4104** | Script Block Logging | Çalıştırılan script blokları; malware, living-off-the-land, encoded komut tespiti |

**Not:** PowerShell Script Block Logging Group Policy ile etkinleştirilmelidir. Açıldıktan sonra bu kanal çok veri üretir; SIEM’de filtreleme/limit gerekebilir.

---

### 1.5 Windows Defender / Microsoft Defender (Operational)

**Kanal:** `Microsoft-Windows-Windows Defender/Operational`  
*(Windows 10/11 ve Server 2016+ için: `Microsoft-Windows-Windows Defender/Operational`; bazen `Microsoft-Windows-Security-Mitigations/KernelMode` vb. ile birlikte kullanılır.)*

| Event ID | Açıklama | Neden önemli |
|----------|----------|----------------|
| **1116, 1117** | Kötü amaçlı yazılım tespit edildi / eylem alındı | EDR/SIEM ile tehdit doğrulama, containment |
| **1119** | Kötü amaçlı yazılım tespit edildi (detection) | Tespit zamanı, dosya, kullanıcı |
| **1120** | Kötü amaçlı yazılım temizlendi / karantinaya alındı | Response takibi |

**Kullanım:** Endpoint tespitleri ile logon/process/network event’lerini ilişkilendirme.

---

## 2. Öncelik: Orta (planlı eklenmeli)

### 2.1 Windows Firewall (Security / Operational)

**Kanal:** `Microsoft-Windows-Windows Firewall With Advanced Security/Firewall`

| Event ID | Açıklama | Neden önemli |
|----------|----------|----------------|
| **2004** | Firewall açıldı | Policy değişikliği, kapatma denemesi sonrası |
| **2033** | Kural eklenmiş / değiştirilmiş | Yetkisiz kural değişikliği |
| Bağlantı reddedildi / izin verildi (ilgili Event ID’ler) | Firewall drop/allow | Lateral movement, C2, port tarama; çok gürültülü olabilir, filtre gerekebilir |

---

### 2.2 RDP / Terminal Services (Operational)

**Kanal:** `Microsoft-Windows-TerminalServices-LocalSessionManager/Operational`

| Event ID | Açıklama | Neden önemli |
|----------|----------|----------------|
| **21** | Uzak masaüstü oturumu başarıyla başlatıldı | RDP girişleri, jump server / admin erişimi |
| **22** | Oturum girişi (shell start) | Oturum aktivitesi |
| **25** | Yeniden bağlanma | Session hijack / persistence riski |

4624/4625 ile birlikte RDP kaynaklı oturumları ayırt etmek için kullanılır.

---

### 2.3 DNS (Sunucu / İstemci)

- **Sunucu (DNS Server role varsa):**  
  `Microsoft-Windows-DNS-Server/Audit` veya Operational — sorgu/yanıt, zone transfer, dinamik güncelleme.  
  C2, data exfiltration, DNS tunneling tespiti için değerlendirilebilir.
- **İstemci:**  
  `Microsoft-Windows-DNS-Client/Operational` — çok yüksek hacim; önce sunucu tarafı tercih edilebilir.

---

### 2.4 WMI Activity (Operational)

**Kanal:** `Microsoft-Windows-WMI-Activity/Operational`

| Event ID | Açıklama | Neden önemli |
|----------|----------|----------------|
| **5858** | WMI aktivitesi | WMI ile lateral movement, persistence (örn. WMI event subscription), script tabanlı saldırılar |

---

### 2.5 Print / Spooler (Operational) – İhtiyaca göre

**Kanal:** `Microsoft-Windows-PrintService/Operational`  
PrintNightmare ve spooler abuse senaryoları için driver yükleme / yazdırma event’leri değerlendirilebilir. Ortamda yazdırma sunucusu kritikse eklenmesi mantıklı.

---

## 3. Öncelik: Düşük / Koşullu

### 3.1 Sysmon (kurulu ise)

Sysmon ayrı bir bileşen olarak kurulur; event’ler **Application** veya **Microsoft-Windows-Sysmon/Operational** kanalına yazılır. Kuruluysa:

- Process Create (1), Network Connect (3), File Create (11), Pipe (17), Remote Thread (8) vb. WEF ile toplanmalı.
- Mevcut 4688/4689’a ek olarak hash, parent process, komut satırı detayı sağlar.

### 3.2 Certificate Services

PKI/CA kullanılıyorsa: sertifika verme/iptal, template değişikliği event’leri (Security veya ilgili CA log’ları) yetkisiz sertifika kullanımı ve golden certificate tespiti için eklenebilir.

### 3.3 Object Access (Registry / Dosya)

Belirli registry anahtarları veya dosya yolları için “Object Access” denetimi açılırsa (örn. persistence konumu, LSASS erişimi ile ilişkili yollar), bu event’ler Security log’da 4663, 4656 vb. olarak gelir. Çok gürültülü olabileceği için hedefli anahtarlar seçilmeli.

---

## 4. Özet: Eklenmesi Önerilen (öncelik sırasıyla)

| Öncelik | Kaynak | Event / Kanal | Amaç |
|---------|--------|---------------|------|
| **Yüksek** | Security | 4768, 4769, 4770, 4771 | Kerberos abuse, golden/silver ticket, brute force |
| **Yüksek** | Security | 4776 | NTLM / Pass-the-Hash izleme |
| **Yüksek** | Security | 4740 | Hesap kilidi / brute force |
| **Yüksek** | Microsoft-Windows-PowerShell/Operational | 4103, 4104 | PowerShell kötüye kullanım, script block |
| **Yüksek** | Microsoft-Windows-Windows Defender/Operational | 1116, 1117, 1119, 1120 | AV/EDR tespitleri |
| **Orta** | Microsoft-Windows-Windows Firewall…/Firewall | 2004, 2033 (+ gerekirse drop/allow) | Firewall policy ve bağlantı |
| **Orta** | TerminalServices-LocalSessionManager/Operational | 21, 22, 25 | RDP oturumları |
| **Orta** | Microsoft-Windows-WMI-Activity/Operational | 5858 | WMI abuse |
| **Orta** | DNS Server (varsa) | İlgili audit/operational | DNS abuse, C2 |
| **Düşük** | Sysmon (kuruluysa) | Operasional kanal | Process/network/dosya detayı |
| **Düşük** | Print / Certificate / Object Access | İhtiyaca göre | Persistence, PKI, hedefli erişim |

---

## 5. Dikkat Edilmesi Gerekenler

- **PowerShell 4104:** Çok yüksek hacim; önce pilot (birkaç kritik sunucu) ile açılması, SIEM’de saklama ve filtreleme planlanması önerilir.
- **Firewall “connection” event’leri:** Tüm drop/allow çok gürültülü olabilir; önce policy değişiklikleri (2004, 2033), gerekirse belirli port/hedef filtreleri ile connection log’ları eklenebilir.
- **Suppress S-1-5-18:** Mevcut subscription’da SYSTEM event’leri bastırılıyor; Kerberos/NTLM’de servis hesapları (SPN) önemli olduğu için sadece “noise” olan SYSTEM logon’ları bastırılmaya devam edilip, 4768/4769/4776 gibi event’lerde servis hesabı analizi SIEM’de yapılabilir.
- **Saklama ve Gizlilik:** Eklenen kanallar (özellikle PowerShell, DNS) kişisel veri içerebilir; mevcut log saklama ve KVKK/gizlilik politikasına uyum gözden geçirilmelidir.

Bu liste, mevcut WEF subscription’ınızı siber güvenlik odaklı genişletmek için kullanılabilir; ilk adım olarak Security’e 4768, 4769, 4770, 4771, 4776, 4740 eklenmesi en yüksek getiriyi sağlar.
