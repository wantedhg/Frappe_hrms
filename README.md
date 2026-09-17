# Frappe HR - Kolay Kurulum

Frappe HR; çalışan yönetimi, izin, devam takibi, performans, işe alım ve bordro modülleri içeren açık kaynaklı bir insan kaynakları uygulamasıdır.

Bu depo, Frappe HR'nin Docker ile kolay kurulumu için hazırlanmıştır.

## Özellikler

- Çalışan yönetimi
- Departman ve pozisyon yönetimi
- İzin ve onay süreçleri
- Puantaj ve devam takibi
- Performans değerlendirme
- İşe alım
- Maaş yapıları
- Bordro hesaplama
- Vergi ve kesinti tanımları
- ERPNext entegrasyonu
- Kullanıcı rolleri ve yetkileri
- Yerel ağ üzerinden kullanım

> Frappe HR'nin bordro motoru genel amaçlıdır. Türkiye SGK, vergi, bordro ve resmi bildirim süreçleri için ayrıca mevzuat kontrolü ve muhasebe uzmanı doğrulaması gerekir.

---

## Gereksinimler

### Linux / Kali Linux

- Docker Engine
- Docker Compose
- Git
- En az 4 GB RAM
- En az 10 GB boş disk alanı

### Windows

Windows üzerinde Docker Desktop kurulmalıdır.

Docker Desktop kurulduktan sonra PowerShell veya Git Bash kullanılabilir.

---

## Linux / Kali Linux kurulumu

Docker kurulu değilse:

```bash
sudo apt update
sudo apt install -y docker.io docker-compose git
```

Docker servisini başlat:

```bash
sudo systemctl enable --now docker
```

Docker durumunu kontrol et:

```bash
sudo systemctl status docker
```

Kullanıcını Docker grubuna ekle:

```bash
sudo usermod -aG docker $USER
```

Sonra oturumu kapatıp tekrar aç veya:

```bash
newgrp docker
```

Docker testi:

```bash
docker run --rm hello-world
```

---

## Projeyi indirme

Bu fork'un GitHub adresini kullan:

```bash
git clone https://github.com/wantedhg/Frappe_hrms
cd Frappe_hrms/docker
```

> `https://github.com/wantedhg/Frappe_hrms` yerine bu deponun kendi GitHub clone adresini yazın.

---

## Kurulum

Docker Compose ile servisleri başlat:

```bash
docker compose up -d
```

Eski Docker Compose sürümü kullanıyorsanız:

```bash
docker-compose up -d
```

Container durumlarını kontrol et:

```bash
docker compose ps
```

Şu servislerin `Up` veya `running` görünmesi gerekir:

- frappe
- mariadb
- redis

Kurulum loglarını takip etmek için:

```bash
docker compose logs -f frappe
```

Kurulum tamamlandığında şu mesajı görmelisiniz:

```text
Thank you for installing Frappe HR!
```

---

## Web arayüzüne giriş

Önce şu adresi deneyin:

```text
http://localhost:8000
```

Çalışmazsa:

```text
http://127.0.0.1:8000
```

Frappe site adı kullanılıyorsa:

```text
http://hrms.localhost:8000
```

Linux'ta `hrms.localhost` çalışmazsa:

```bash
echo "127.0.0.1 hrms.localhost" | sudo tee -a /etc/hosts
```

Sonra tarayıcıda:

```text
http://hrms.localhost:8000
```

### İlk giriş

```text
Kullanıcı adı: Administrator
Parola: admin
```

İlk girişten sonra yönetici parolasını mutlaka değiştirin.

---

## Web servisi dışarıdan erişilemiyorsa

Bazı kurulumlarda Frappe web servisi container içinde yalnızca `127.0.0.1` adresinde çalışabilir.

Bu durumda Procfile dosyalarını düzeltin:

```bash
docker compose exec frappe sh -lc '
for f in \
/home/frappe/frappe-bench/Procfile \
/home/frappe/frappe-bench/frappe-bench/Procfile
do
  if [ -f "$f" ]; then
    sed -i "s|^web:.*|web: bench serve --host 0.0.0.0 --port 8000|" "$f"
  fi
done
'
```

Değişikliği kontrol edin:

```bash
docker compose exec frappe sh -lc \
'grep -R "^web:" /home/frappe/frappe-bench 2>/dev/null'
```

Frappe servisini yeniden başlatın:

```bash
docker compose restart frappe
```

Yaklaşık 10 saniye bekleyin:

```bash
sleep 10
```

Test edin:

```bash
curl -I -H "Host: hrms.localhost" http://127.0.0.1:8000
```

Başarılı sonuç:

```text
HTTP/1.1 200 OK
```

---

## Yerel ağdaki diğer bilgisayarlardan erişim

Sunucu bilgisayarın IP adresini öğren:

```bash
hostname -I
```

Örneğin:

```text
192.168.1.45
```

Aynı ağdaki başka bir bilgisayardan doğrudan deneyin:

```text
http://192.168.1.45:8000
```

Frappe site adı nedeniyle doğrudan IP çalışmazsa, diğer bilgisayarın hosts dosyasına şu satırı ekleyin:

```text
192.168.1.45 hrms.localhost
```

Linux ve macOS:

```text
/etc/hosts
```

Windows:

```text
C:\Windows\System32\drivers\etc\hosts
```

Sonra şu adresi açın:

```text
http://hrms.localhost:8000
```

### Linux firewall

Firewall aktifse yalnızca yerel ağ için 8000 portuna izin verin:

```bash
sudo ufw status
```

Örneğin ağınız `192.168.1.x` ise:

```bash
sudo ufw allow from 192.168.1.0/24 to any port 8000 proto tcp
```

> 8000 portunu doğrudan internete açmayın. İnternet üzerinden erişim için HTTPS, reverse proxy, güçlü parola ve güvenlik yapılandırması gereklidir.

---

## Türkçe dil ayarı

Kullanıcı ayarlarından dili değiştirmeyi deneyin:

```text
Profil > My Settings > Language > Turkish
```

Terminalden Administrator kullanıcısının dilini ayarlamak için:

```bash
docker compose exec frappe bench \
--site hrms.localhost set-value User Administrator language tr
```

Önbelleği temizleyin:

```bash
docker compose exec frappe bench \
--site hrms.localhost clear-cache
```

Frappe container'ını yeniden başlatın:

```bash
docker compose restart frappe
```

Türkçe çeviri eksikse bazı ekranlar İngilizce kalabilir.

---

## Hata ayıklama

### Container durumları

```bash
docker compose ps -a
```

### Frappe logları

```bash
docker compose logs --tail=200 frappe
```

### Tüm servislerin logları

```bash
docker compose logs --tail=200
```

### 8000 portunu kontrol etme

```bash
sudo ss -ltnp | grep ':8000'
```

Beklenen çıktı Docker'a ait olmalıdır:

```text
docker-proxy ... 0.0.0.0:8000
```

### Docker servis durumu

```bash
sudo systemctl status docker
```

### Docker servisini başlatma

```bash
sudo systemctl start docker
```

### Frappe servisini yeniden oluşturma

```bash
docker compose up -d
```

> `docker compose down -v` komutunu dikkatli kullanın. `-v` Docker volume'lerini silebilir ve veritabanı kaybına neden olabilir.

---

## Sık görülen uyarılar

### version is obsolete

```text
the attribute version is obsolete
```

Bu kritik bir hata değildir. `docker-compose.yml` dosyasındaki eski `version:` satırı silinebilir.

### rename_field not found

```text
rename_field: ... not found in table
```

Bu genellikle migration sırasında görülen bilgilendirme mesajıdır. Kurulum sonunda şu mesaj görülüyorsa kurulum tamamlanmıştır:

```text
Thank you for installing Frappe HR!
```

### Development server warning

```text
WARNING: This is a development server
```

Bu geliştirme ortamı uyarısıdır. Test ve yerel kullanım için sorun değildir. Üretim ortamında reverse proxy, HTTPS, yedekleme ve güvenlik yapılandırması kullanılmalıdır.

---

## Durdurma

```bash
docker compose stop
```

## Yeniden başlatma

```bash
docker compose start
```

## Container durumlarını görme

```bash
docker compose ps
```

## Veritabanı yedeği

Silme veya yeniden kurulum işlemlerinden önce mutlaka yedek alın.

```bash
docker compose exec frappe bench \
--site hrms.localhost backup
```

---

## Lisans

Bu proje Frappe HR ve ilgili açık kaynak bileşenlerin lisanslarına tabidir. Ticari kullanım, dağıtım ve değişiklikler için ilgili lisans dosyalarını inceleyin.
