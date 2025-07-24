# 🛡️ Raspberry Pi + Pi-hole + DNSCrypt + Tailscale Kurulum Rehberi

Bu rehber, ASUS DSL-N16 modem üzerinden Raspberry Pi'ye sabit IP atamasından başlayarak, Pi-hole reklam engelleme, DNSCrypt ile şifreleme ve Tailscale ile güvenli uzak erişim kurulumu ve yapılandırmasını **adım adım**, **görsellerle** ve **detaylı** şekilde anlatır.

---

## 🔌 1. ASUS DSL-N16 Modem Üzerinden Raspberry Pi’ye Sabit IP ve DNS Ataması

Raspberry Pi’nin ağda her zaman aynı IP adresini alması ve tüm cihazların DNS sorgularını Pi-hole’a yönlendirmesi için aşağıdaki adımları takip edin.

### 1.1. Modeme Giriş

1. Web tarayıcınızdan `http://192.168.1.1` adresine gidin.  
2. Kullanıcı adı ve şifrenizi girin (varsayılan: `admin/admin`).  
3. Giriş yaptıktan sonra aşağıdaki menüye erişin.

### 1.2. Yerel Ağ Ayarları

- Sol menüden **Gelişmiş Ayar > Yerel Ağ** sekmesine tıklayın.  
- **DHCP Sunucusu Etkinleştirilsin mi?** seçeneği **Evet** olmalı.

### 1.3. IP Rezervasyonu (Sabit IP Atama)

- Aynı sayfada **IP Rezervasyonu** veya **Adres Rezervasyonu** bölümüne gidin.  
- Raspberry Pi’nin MAC adresini girin. (Raspberry Pi’de `ip a` komutuyla bulunur.)  
- Sabit IP olarak örneğin `192.168.1.50` verin.

| Alan         | Açıklama                  |
|--------------|---------------------------|
| MAC Adresi   | Raspberry Pi’nin MAC adresi|
| IP Adresi    | Sabit IP (örnek: 192.168.1.50) |

### 1.4. DNS Ayarları

- Aynı sayfada **DNS ve WINS Sunucusu Ayarları** bölümünde:  
  - **DNS Sunucusu 1:** Raspberry Pi’nin sabit IP adresi (`192.168.1.50`)  
  - **DNS Sunucusu 2:** Alternatif DNS (örnek: `1.1.1.1`)

### 1.5. Kaydetme ve Modem Yeniden Başlatma

- Ayarları kaydedin ve modemi yeniden başlatın.  
- Raspberry Pi’ye sabit IP atandığını doğrulayın.

---

### 🖼️ Görsel Referans

![ASUS DSL-N16 DHCP Ayarı](https://i.hizliresim.com/n2gcr5q.jpg)

---

## 📦 2. Raspberry Pi’ya SSH ile Bağlanma

```bash
ssh pi@192.168.1.50
````

Varsayılan parola `raspberry` olabilir. Güvenlik için değiştirin.

---

## 🔄 3. Raspberry Pi OS Güncelleme

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 🕳️ 4. Pi-hole Kurulumu

```bash
curl -sSL https://install.pi-hole.net | bash
```

* Kurulumda “upstream DNS” sağlayıcısı seçebilirsiniz, sonrasında DNSCrypt ile değiştireceğiz.

---

## 🔐 5. DNSCrypt Proxy Kurulumu

### 5.1. Debian SID Deposunu Ekle

```bash
echo 'deb http://deb.debian.org/debian sid main' | sudo tee /etc/apt/sources.list.d/sid.list
```

### 5.2. Pinleme Ayarları

```bash
sudo tee /etc/apt/preferences.d/99-sid-repository-pinning > /dev/null <<EOF
Package: *
Pin: release a=unstable
Pin-Priority: -10
EOF
```

### 5.3. Kurulum

```bash
sudo apt update
sudo apt install -t sid dnscrypt-proxy -y
```

---

## ⚙️ 6. DNSCrypt Yapılandırması

### 6.1. Dinleme IP ve Port Ayarı

```bash
sudo nano /usr/lib/systemd/system/dnscrypt-proxy.socket
```

`ListenStream=127.0.0.1:5053` olarak ayarlayın.

### 6.2. DNS Sağlayıcıları Seçimi

DNSCrypt Proxy, birçok farklı DNS sağlayıcısını destekler. Bu sağlayıcılar, farklı güvenlik, hız ve gizlilik özelliklerine sahiptir. İhtiyacınıza göre tercih edebilir veya birden fazla sağlayıcıyı listeleyerek yedekli ve dağıtılmış sorgulama yapabilirsiniz.

DNS Sağlayıcı Listesine Erişim
Güncel ve desteklenen DNS sağlayıcılarının tam listesine aşağıdaki adresten ulaşabilirsiniz:

🌐 https://dnscrypt.info/

Buradan size uygun olan, güvenlik ve gizlilik kriterlerinize uyan DNS sağlayıcısını seçebilirsiniz.

Örnek: cloudflare-security Sağlayıcısını Kullanma
Benim tercihim cloudflare-security oldu; bu sağlayıcı DNS over HTTPS (DoH) desteği ile güvenli bağlantı sağlar ve reklam, kötü amaçlı site engelleme özellikleri bulunur.

```bash
sudo nano /etc/dnscrypt-proxy/dnscrypt-proxy.toml
```

Örnek:

```toml
server_names = ['cloudflare-security']
```

---

## 🔗 7. Pi-hole’a DNSCrypt Proxy Tanıtımı

```bash
sudo pihole-FTL --config dns.upstreams '["127.0.0.1#5053"]'
```

---

## ♻️ 8. Servisleri Yeniden Başlatma

```bash
sudo systemctl restart dnscrypt-proxy.socket
sudo systemctl restart dnscrypt-proxy.service
sudo systemctl restart pihole-FTL.service
```

---

## 📊 9. Servis Durumlarını Kontrol Etme

```bash
sudo systemctl status dnscrypt-proxy.socket
sudo systemctl status dnscrypt-proxy.service
sudo systemctl status pihole-FTL.service
```

---

## 🌍 10. Tailscale Kurulumu ve DNS Yönetimi

### 10.1. Tailscale Kurulumu

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

### 10.2. Tailscale ile Oturum Açma

```bash
sudo tailscale up --accept-dns=false
```

* Terminalde çıkan linki tarayıcıda açın ve Raspberry Pi’yi hesabınıza bağlayın.

### 10.3. Tailscale DNS Ayarları (Web Arayüzü)

* Tailscale admin paneline giriş:
  [https://login.tailscale.com/admin/dns](https://login.tailscale.com/admin/dns)

* **Nameservers** kısmına Raspberry Pi’nin Tailscale IP’sini yazın (örn: `100.x.y.z`)

* **MagicDNS** aktif olsun

* **Override local DNS** seçeneğini işaretleyin

* **Global Nameserver** olarak bu DNS sunucusunu seçin (tüm cihazlar otomatik kullanacak)

---

### 🧪 Tailscale DNS Testi

Herhangi bir cihazda terminal açıp:

```bash
nslookup doubleclick.net
```

Çıktı IP’si `0.0.0.0` ise Pi-hole reklam engellemesi çalışıyor demektir.

---

### 🤝 Tailscale Invite ile Ağ Paylaşımı

1. [https://login.tailscale.com/admin/invite](https://login.tailscale.com/admin/invite) adresine gidin.
2. "Invite user" ile kullanıcı e-posta adresi ekleyin.
3. Kullanıcı Tailscale uygulamasını indirip giriş yaptıktan sonra ağınıza otomatik bağlanır.

---

## ✅ Sonuç

* Pi-hole merkezi reklam engelleyici olarak çalışır.
* DNS trafiği DNSCrypt ile şifrelenir.
* Tailscale ile güvenli, port yönlendirmesiz uzaktan erişim sağlanır.
* Tüm cihazlar Pi-hole DNS üzerinden filtrelenir.
* ASUS DSL-N16 modem üzerinde sabit IP ve DNS ayarları görselli ve adım adım uygulanmıştır.

---

## 🔗 Kaynaklar

* [Pi-hole](https://pi-hole.net)
* [DNSCrypt Proxy](https://github.com/DNSCrypt/dnscrypt-proxy)
* [Tailscale](https://tailscale.com)
* [ASUS DSL-N16 Kullanıcı Kılavuzu](https://www.asus.com/Networking-IoT-Servers/Modems-Routers/DSL-N16/HelpDesk_Manual/)

---
