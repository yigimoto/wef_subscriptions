# WEF Subscription’da Toplanan Event’ler

Tüm Select’lerde **zaman penceresi**: son **24 saat** (`TimeCreated[timediff(@SystemTime) <= 86400000]` = 86400000 ms).

---

## 1. Security (Güvenlik)

| Event ID | Açıklama |
|----------|----------|
| **4627** | Group membership bilgisi |
| **4703, 4704, 4705** | Kullanıcı hakkı ayarlandı / atandı / kaldırıldı |
| **4720** | Kullanıcı hesabı oluşturuldu |
| **4722–4735** | Hesap etkinleştirme, şifre değişimi/sıfırlama, hesap devre dışı/silme, güvenlik grupları (global/local) oluşturma/değişiklik/silme, üye ekleme/çıkarma, SID History |
| **4737–4739** | Global group değişikliği, kullanıcı hesabı değişikliği, domain policy değişikliği |
| **4741–4767** | Bilgisayar hesabı oluşturma/değişiklik/silme; güvenlik-disabled gruplar; universal gruplar; grup tipi değişikliği; SID History; hesap kilidi açma |
| **4780–4782** | Admin grubu üyelerinde ACL, hesap adı değişikliği, parola hash erişimi |
| **4793, 4794** | Password Policy API çağrısı, DSRM admin parolası denemesi |
| **4798, 4799** | Yerel grup üyelikleri listelendi |
| **5376, 5377** | Credential Manager yedeklendi / geri yüklendi |
| **1102** | Security log dosyası temizlendi (Microsoft-Windows-Eventlog, Level=4) |
| **4698–4702** | Zamanlanmış görev oluşturuldu / silindi / etkinleştirildi / devre dışı bırakıldı / güncellendi |
| **4616** | Sistem saati değiştirildi |
| **4624–4626** | Başarılı / başarısız oturum açma, User/Device claims |
| **4634, 4647, 4649, 4672, 4675** | Oturum kapatma, kullanıcı logoff, replay saldırısı, özel ayrıcalık ataması, SID filtreleme |
| **4774–4779** | Hesap logon için eşlendi, credential doğrulama, oturum yeniden bağlanma/ayrılma |
| **4800–4803** | İş istasyonu kilitlendi / açıldı, ekran koruyucu başladı / bitti |
| **4964** | Özel gruplar yeni logon atandı |
| **5378** | Credentials delegation policy tarafından reddedildi |
| **4688** | Process oluşturuldu |
| **4689** | Process sonlandı |

### Security – hariç tutulan (Suppress)

| Koşul | Açıklama |
|--------|----------|
| **EventData[Data[1]="S-1-5-18"]** | LOCAL SYSTEM hesabı (S-1-5-18) event’leri toplanmaz; gürültü azaltma |

---

## 2. System (Sistem)

**Provider:** `Service Control Manager`

| Event ID | Açıklama |
|----------|----------|
| **7022** | Servis başlarken takıldı |
| **7023** | Servis hata ile sonlandı |
| **7024** | Servis özel hata ile sonlandı |
| **7026** | Boot/system-start sürücü(leri) yüklenemedi |
| **7031** | Servis beklenmedik şekilde sonlandı |
| **7032** | Servis beklenmedik şekilde sonlandı (kod 7032) |
| **7034** | Servis beklenmedik şekilde sonlandı (kod 7034) |
| **7040** | Servis başlangıç türü değiştirildi |
| **7045** | Servis yüklendi (yeni servis kuruldu) |

---

## 3. Application (Uygulama) – MSSQL

**Dosyalar:** `wef_subscription_with_sql.xml`, `wef_subscription_base_sql_dhcpo.xml`

| Provider | Açıklama |
|----------|----------|
| **MSSQLSERVER** | SQL Server Application log event’leri (tüm EventID’ler, son 24 saat) |

---

## 4. Microsoft-Windows-DHCP-Server/Operational (DHCP Operational)

**Dosyalar:** `wef_subscription_dhcp_operational.xml`, `wef_subscription_base_sql_dhcpo.xml`

**Kanal:** Applications and Services Logs → Microsoft → Windows → DHCP-Server → Operational

| Kapsam | Event ID aralıkları (özet) |
|--------|----------------------------|
| **IPv4 scope / reservation** | 70–128 (scope/reservation yapılandırma, DNS/NAP, filter, policy) |
| **IPv6** | 130–166 (scope, reservation, stateless, server option) |
| **Policy / failover** | 20220–20316 (policy, failover relationship, auth, BINDING-UPDATE vb.) |

Tüm Operational kanalı event’leri toplanır (filtre: son 24 saat).

---

## Özet tablo

| Log / Kanal | Event / kapsam | Hangi subscription dosyalarında |
|-------------|----------------|----------------------------------|
| **Security** | Yukarıdaki EventID’ler (Suppress: S-1-5-18) | Tüm base / minimal / with_sql / base_sql_dhcpo |
| **System** | Service Control Manager 7022–7034, 7040, 7045 | Tüm base / minimal / with_sql / base_sql_dhcpo |
| **Application** | MSSQLSERVER | with_sql, base_sql_dhcpo |
| **Microsoft-Windows-DHCP-Server/Operational** | Tüm event’ler (70–166, 20220–20316 vb.) | dhcp_operational, base_sql_dhcpo |
