# Modul 2
## Praktikum Komunikasi Data & Jaringan Komputer

## Anggota Kelompok 
| Nama    | NRP  |
|---------|------|
| Imam Mahmud Dalil Fauzan  | 5027241100  |
| Zaenal Mustofa | 5027241018  |

## Write Up
### 1. ![](img/1.png)
### 4. Terminal Tirion:
```Bash
nano /etc/bind/named.conf.options
# isi dengan ini:
options {
    directory "/var/cache/bind";
    listen-on { any; };
    listen-on-v6 { any; };
    allow-query { any; };

    forwarders { 192.168.122.1; }; // forward outward as required
    recursion yes;                 // let clients use ns1 for recursive queries
    dnssec-validation no;          // simplify lab
};


nano /etc/bind/named.conf.local
# isi dengan ini:
zone "K62.com" {
    type master;
    file "/etc/bind/db.K62.com";
    notify yes;                        // send NOTIFY
    also-notify { 192.242.3.21; };     // Valmar (ns2)
    allow-transfer { 192.242.3.21; };  // only Valmar may AXFR/IXFR
};


nano /etc/bind/db.K62.com
# isi dengan ini:
$TTL 3600
@   IN  SOA ns1.K62.com. admin.K62.com. (
        2025101301 ; serial  (YYYYMMDDnn)  <<-- increase on every edit
        3600       ; refresh
        900        ; retry
        604800     ; expire
        300 )      ; minimum

; authoritative nameservers
@       IN  NS  ns1.K62.com.
@       IN  NS  ns2.K62.com.

; glue for nameservers
ns1     IN  A   192.242.3.20    ; Tirion
ns2     IN  A   192.242.3.21    ; Valmar

; apex (front door)
@       IN  A   192.242.3.2    ; Sirion
```

Validasi dan Reload pada Tirion.
```Bash
named-checkconf
named-checkzone K62.com /etc/bind/db.K62.com
systemctl restart bind9 2>/dev/null || service bind9 restart
```

Terminal Valmar:
```Bash
nano /etc/bind/named.conf.options
# isi dengan ini:
options {
    directory "/var/cache/bind";
    listen-on { any; };
    listen-on-v6 { any; };
    allow-query { any; };

    forwarders { 192.168.122.1; };
    recursion yes;
    dnssec-validation no;
};

nano /etc/bind/named.conf.local
# isi dengan ini:
zone "K62.com" {
    type slave;
    masters { 192.242.3.20; };            // pull from Tirion
    file "/var/cache/bind/db.K62.com"; // local copy will be written here
    allow-notify { 192.242.3.20; };
};
```
Run named di Valmar:
```Bash
pkill named 2>/dev/null || true
mkdir -p /var/cache/bind && chown -R bind:bind /var/cache/bind
named -4 -u bind -c /etc/bind/named.conf
```
Cek pada client lain:
```Bash
cat >/etc/resolv.conf <<DNS
nameserver 192.242.3.20   # ns1.K62.com (Tirion)
nameserver 192.242.3.21   # ns2.K62.com (Valmar)
nameserver 192.168.122.1 # fallback
DNS
```
Test pada client lain:
```Bash
dig K62.com +short
dig ns1.K62.com +short
dig ns2.K62.com +short

ping -c3 K62.com
```

### 5. Membuat Hostname pada tiap Client

Terminal Eonwe:
```Bash
echo "eonwe" > /etc/hostname
hostname -F /etc/hostname
cat >/etc/hosts <<'EOF'
127.0.0.1   localhost
127.0.1.1   eonwe.K62.com eonwe
192.242.3.1 eonwe.K62.com eonwe
EOF
``` 
Terminal Earendil:
```Bash
echo "earendil" > /etc/hostname
hostname -F /etc/hostname
cat >/etc/hosts <<'EOF'
127.0.0.1   localhost
127.0.1.1   earendil.K62.com earendil
192.242.1.2 earendil.K62.com earendil
EOF
```
Terminal Elwing:
```Bash
echo "elwing" > /etc/hostname
hostname -F /etc/hostname
cat >/etc/hosts <<'EOF'
127.0.0.1   localhost
127.0.1.1   elwing.K62.com elwing
192.242.1.3 elwing.K62.com elwing
EOF
```
Terminal Cirdan:
```bash
echo "cirdan" > /etc/hostname
hostname -F /etc/hostname
cat >/etc/hosts <<'EOF'
127.0.0.1   localhost
127.0.1.1   cirdan.K62.com cirdan
192.242.2.2 cirdan.K62.com cirdan
EOF
```
Terminal Elrond:
```bash
echo "elrond" > /etc/hostname
hostname -F /etc/hostname
cat >/etc/hosts <<'EOF'
127.0.0.1   localhost
127.0.1.1   elrond.K62.com elrond
192.242.2.3 elrond.K62.com elrond
EOF
```
Terminal Maglor:
```bash
echo "maglor" > /etc/hostname
hostname -F /etc/hostname
cat >/etc/hosts <<'EOF'
127.0.0.1   localhost
127.0.1.1   maglor.K62.com maglor
192.242.2.4 maglor.K62.com maglor
EOF
```
Terminal Sirion:
```bash
echo "sirion" > /etc/hostname
hostname -F /etc/hostname
cat >/etc/hosts <<'EOF'
127.0.0.1   localhost
127.0.1.1   sirion.K62.com sirion
192.242.3.1 sirion.K62.com sirion
EOF
```
Terminal Tirion:
```bash
echo "tirion" > /etc/hostname
hostname -F /etc/hostname
cat >/etc/hosts <<'EOF'
127.0.0.1   localhost
127.0.1.1   ns1.K62.com tirion
192.242.3.20 ns1.K62.com tirion
EOF
```
Terminal Valmar:
```bash
echo "valmar" > /etc/hostname
hostname -F /etc/hostname
cat >/etc/hosts <<'EOF'
127.0.0.1   localhost
127.0.1.1   ns2.K62.com valmar
192.242.3.21 ns2.K62.com valmar
EOF
```
Terminal Lindon:
```bash
echo "lindon" > /etc/hostname
hostname -F /etc/hostname
cat >/etc/hosts <<'EOF'
127.0.0.1   localhost
127.0.1.1   lindon.K62.com lindon
192.242.3.22 lindon.K62.com lindon
EOF
```
Terminal Vingilot:
```bash
echo "vingilot" > /etc/hostname
hostname -F /etc/hostname
cat >/etc/hosts <<'EOF'
127.0.0.1   localhost
127.0.1.1   vingilot.K62.com vingilot
192.242.3.23 vingilot.K62.com vingilot
EOF
```
Backup File Zona dan Tambahkan record Tirion (di Terminal Tirion):
```bash
cp -a /etc/bind/db.K62.com /etc/bind/db.K62.com.bak
cat <<'EOF' >> /etc/bind/db.K62.com

; === Hostnames (A) sesuai glosarium ===
eonwe    IN A 192.242.3.1
earendil IN A 192.242.1.2  
elwing   IN A 192.242.1.3
cirdan   IN A 192.242.2.2
elrond   IN A 192.242.2.3
maglor   IN A 192.242.2.4
sirion   IN A 192.242.3.2
lindon   IN A 192.242.3.22
vingilot IN A 192.242.3.23

; === Pengecualian untuk NS (pakai ns1/ns2, bukan tirion/valmar) ===
ns1      IN A 192.242.3.20   ; Tirion
ns2      IN A 192.242.3.21   ; Valmar

EOF
```
Cek:
```bash
dig +norecurse @192.242.3.2 K62.com SOA +noall +answer +authority
dig +norecurse @192.242.3.3 K62.com SOA +noall +answer +authority

for h in eonwe earendil elwing cirdan elrond maglor sirion lindon vingilot ns1 ns2; do
  printf "%-8s -> %s\n" "$h" "$(dig +short ${h}.K62.com)"
done
```

11. Di Terminal Sirion:
```bash
apt install nginx -y
nano /etc/nginx/sites-available/reverse_proxy
# masukkan ini:
server {
    listen 80;
    server_name www.K62.com sirion.K62.com;
    # Lokasi untuk /static
    location /static/ {
        proxy_pass http://192.242.3.22/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location /app/ {
        proxy_pass http://192.242.3.23/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
```bash
ln -s /etc/nginx/sites-available/reverse_proxy /etc/nginx/sites-enabled/
rm /etc/nginx/sites-enabled/default

nginx -t
service nginx restart
```
Test di terminal Lain (Elrond):
```bash
curl http://www.K62.com/static/annals/
curl http://www.K62.com/app/
```

12. Buka di Terminal Sirion:
```bash
apt install apache2-utils -y
htpasswd -c /etc/nginx/.htpasswd admin

nano /etc/nginx/sites-available/reverse_proxy
# masukkan ini:
server {
    listen 80;
    server_name www.K62.com sirion.K62.com;
    location /admin {
        auth_basic "Restricted Content";
        auth_basic_user_file /etc/nginx/.htpasswd;
        proxy_pass http://192.242.3.23/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location /static/ {
        proxy_pass http://192.242.3.22/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location /app/ {
        proxy_pass http://192.242.3.23/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
```bash
nginx -t
service nginx restart
```
Buka Terminal Elrond:
```bash
curl -I http://www.K62.com/admin
curl -I --user admin:adpiajd http://www.K62.com/admin
curl -I --user admin:admin http://www.K62.com/admin
```

13. Buka Terminal Sirion:
```bash
nano /etc/nginx/sites-available/reverse_proxy
# masukkan ini:
server {
    listen 80 default_server;
    server_name sirion.K62.com;
    return 301 http://www.K62.com$request_uri;
}

server {
    listen 80;
    server_name www.K62.com;
    location /admin {
        auth_basic "Restricted Content";
        auth_basic_user_file /etc/nginx/.htpasswd;
        proxy_pass http://192.242.3.23/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /static/ {
        proxy_pass http://192.242.3.22/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /app/ {
        proxy_pass http://192.242.3.23/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
```bash
nginx -t
service nginx restart
```
Buka Terminal Elrond:
```bash
curl -I http://192.242.3.2/app/
curl -I http://sirion.K62.com/static/
curl -I http://www.K62.com/app/
```

14. Buka Terminal Vingilot
```bash
nano /etc/apache2/apache2.conf
# masukkan ini dibawah logformat lainnya:
LogFormat "%{X-Forwarded-For}i %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" proxy

# ubah ini:
CustomLog ${APACHE_LOG_DIR}/app_K62_access.log combined
# menjadi:
CustomLog ${APACHE_LOG_DIR}/app_K62_access.log proxy
```
```bash
service apache2 restart
```
Buka Terminal Elrond:
```bash
curl http://www.K62.com/app/
```
Buka Terminal Vingilot:
```bash
tail -n 5 /var/log/apache2/app_K62_access.log
```

15. Buka Terminal Elrond:
```bash
apt install apache2-utils -y
ab -n 500 -c 10 http://www.K62.com/app/
ab -n 500 -c 10 http://www.K62.com/static/
```

16. Buka Terminal Elrond:
```bash
dig static.K62.com +short
```
Buka Terminal Tirion:
```bash
nano /etc/bind/jarkom/db.K62.com
# masukkan ini:
$TTL    604800
@       IN      SOA     ns1.K62.com. root.K62.com. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.K62.com.
@       IN      NS      ns2.K62.com.

K62.com.        IN      A       192.242.3.1
ns1             IN      A       192.242.3.20
ns2             IN      A       192.242.3.21
sirion          IN      A       192.242.3.1
lindon      30  IN      A       192.242.3.22
vingilot        IN      A       192.242.3.23

www             IN      CNAME   sirion.K62.com.
static      30  IN      CNAME   lindon.K62.com.
app             IN      CNAME   vingilot.K62.com.
```
```bash
service named restart
```
Buka Terminal Valmar:
```bash
dig K62.com @localhost SOA
```
Buka Terminal Elrond:
```bash
dig static.K62.com +short
# tunggu 40 detik - 1 menit
dig static.K62.com +short
```