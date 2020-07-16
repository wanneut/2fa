# Managment Knoten

Knoten, der die Credentials entgegen nimmt.

## Voraussetzungen

Es wird vorausgesetzt, daß alle Knotentypen die selbe Nutzerauthentifizierung nutzen (z.B. LDAP).

## Abhängigkeiten

### OpenSSL >=1.1.1 

Installation unter RHEL/CentOS 7 (ab Version 8 enthalten):

```bash
wget https://www.openssl.org/source/openssl-1.1.1g.tar.gz
tar xvfz openssl-1.1.1g.tar.gz && cd openssl-1.1.1g
./config --prefix=/opt/openssl --openssldir=/opt/openssl/etc
make
make install
```

Hinweis: "make test" ist fehlerhaft. Prinzipiell liegen auch auf MD5 basierende oathupdate und addline-Scripte bei, die mit dem auf CentOS 7 verfügbaren OpenSSL auskommen. Es wird nicht empfohlen diese Varianten zu nutzen.

### EPEL Repo: `socat`

```bash
yum install -y epel-release
yum install -y socat
```

### Firewall

 Port 165/tcp in der Firewall freigeben.

## Installation

### OATH-Skripte

Eventuell müssen PATH und LD_LIBRARY_PATH für OpenSSL in addline angepasst werden (siehe oben).

```bash
cp addline oathdeact /usr/local/bin/
chmod +x /usr/local/bin/addline /usr/local/bin/oathdeact
cp addline.service /etc/systemd/system/addline.service
systemctl daemon-reload
systemctl enable addline
systemctl start addline
```

### Erstellen der Schlüssel zum Verschlüsseln der "Zweiten Faktoren"

"Pfad" sollte im einfachsten Fall einem verteilten Dateisystem (NFS, Lustre, etc.) entsprechen.

```bash
mkdir /etc/2fa
gpg2 --gen-key                                              # Schlüssel für die Mail-Adresse "key@subission" erzeugen
gpg --export key@subission > /etc/2fa/public_key.gpg     # Für Management- und QR-Generator-Knoten
gpg -a --export-secret-key    > /etc/2fa/private_key.gpg    # Für Backup etc.
install -m 600 /dev/null /etc/2fa/secret
head -c 15 /dev/urandom       > /etc/2fa/secret
mkdir /<Pfad>/2fa
install -m 600 /dev/null /<Pfad>/2fa/users.oath
echo "# Option        User    Prefix  Seed" > /<Pfad>/2fa/users.oath
```



