# WEF Subscription Dosyaları Karşılaştırması

## Mevcut dosyalar

| Dosya | İçerik | Fark |
|-------|--------|------|
| **wef_subscription.xml** | QueryList only, Security + System, **yorumlu** | Ana referans; her EventID için açıklama var |
| **wef_subscription_minimal.xml** | QueryList only, Security + System, **yorumsuz** | Aynı Select/Suppress sorguları, sadece yorum yok |
| **wef_subscription_with_sql.xml** | **Tam subscription** (CDATA + ContentFormat, Locale, AllowedSource…) + MSSQL Application log | Tek fark: ek olarak `<Select Path="Application">*[System[Provider[@Name='MSSQLSERVER'] ...]]]</Select>` ve Subscription wrapper |

## Sorgu eşleşmesi

- **wef_subscription.xml** ile **wef_subscription_minimal.xml** aynı 23 Select + 1 Suppress içerir; tek fark yorumlar ve girinti.
- **wef_subscription_with_sql.xml** içindeki QueryList, yukarıdaki QueryList’e **artı** MSSQL Application log Select’i eklenmiş hâliyle aynıdır.

## Path özeti

| Path | Açıklama |
|------|----------|
| Security | 4627, 4703–4705, 4720, 4722–4735, 4737–4739, 4741–4767, 4780–4782, 4793–4794, 4798–4799, 5376–5377, 1102, 4698–4702, 4616, 4624–4626, 4634/4647/4649/4672/4675, 4774–4779, 4800–4803, 4964, 5378, 4688, 4689; Suppress S-1-5-18 |
| System | Service Control Manager: 7022–7026, 7031–7034, 7040, 7045 |
| Application | (sadece _with_sql): MSSQLSERVER provider |

## Zaman penceresi

Tüm Select’lerde `TimeCreated[timediff(@SystemTime) <= 86400000]` kullanılıyor → **son 24 saat** (86400000 ms).

---

## DHCP-Operational subscription (ayrı dosya)

- **wef_subscription_dhcp_operational.xml**: Sadece **Microsoft-Windows-DHCP-Server/Operational** kanalı.
- **Kaynak**: Applications and Services Logs > Microsoft > Windows > DHCP-Server > Operational.
- **XPath**: Kanalın tüm event’leri, son 24 saat (`timediff <= 86400000`).
- **Not**: Bazı sunucularda channel adı farklı casing ile görünebilir (örn. `Microsoft-Windows-Dhcp-Server/Operational`). Çalışmazsa Event Viewer’da loga sağ tıklayıp Properties > General > **Full Name** değerini kopyalayıp `<Query Path="...">` ve `<Select Path="...">` içinde kullanın.
