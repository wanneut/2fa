# QR-Code-Generator-Knoten

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

### EPEL Repo: oathtool

```bash
yum install -y epel-release
yum install -y oathtool qrencode pinentry socat nmap-ncat vim gpgme
```

### Optional: Automatisches Erstellen der home-Verzeichnisse

```bash
yum install -y oddjob-mkhomedir
authconfig --enablemkhomedir --update
```

## Installation

### CentOS 7 

Hier liegt noch kein base32 bei. Den passenden Pyhton-Einzeiler in den $PATH kopieren:

```bash
cp base32 /usr/bin/
```

### OATH-Skripte kopieren

```bash
cp oathgen oathupdate /usr/local/bin/
cp oathupdate.service /etc/systemd/system/
chmod 755 oathgen
```

Kopieren der Credentials vom Management Knoten

```bash
mkdir /etc/2fa
scp <management-node>:/etc/2fa/public_key.gpg /etc/2fa/public_key.gpg
install -m 600 /dev/null /etc/2fa/secret
ssh <management-node> cat /etc/2fa/secret > /etc/2fa/secret
```

## Konfiguration

### Systemdienst aktivieren

```bash
systemctl daemon-reload
systemctl enable oathupdate.service
systemctl start oathupdate.service
```
               
Damit die Nutzer*innen nur ein Skript ausführen können, wird nur das in der SSHD-Konfiguration gestattet.

### SSHD-Konfiguration `/etc/ssh/sshd_config` anpassen

```bash
# alle außer root führen automatisch Kommando aus
Match User *,!root
        ForceCommand /usr/local/bin/oathgen
```

### SSH-Server neu laden

 ```bash
 systemctl reload sshd.service
```




