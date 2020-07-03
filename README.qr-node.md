### QR-Code-Generator-Knoten

## Vorraussetzungen

Der QR-Code-Generator benötigt Software zu Erzeugen des "Zweiten Faktors".

Installation von Paketabhängigkeiten bei CentOS7
```bash
# EPEL Repo
yum install -y epel-release
# oathtool, QR-Code-Generator und inotify-tools
yum install -y oathtool qrencode inotify-tools pinentry socat nmap-ncat
```

Wenn Nutzer-Homes automatisch angelegt werden sollen, Oddjob installieren
```bash
yum install -y oddjob-mkhomedir
authconfig --enablemkhomedir --update
```

Die passenden Scripte
```bash
curl "https://codeload.github.com/nemo-cluster/2fa/zip/master" > 2fa.zip
unzip 2fa.zip
cd 2fa-master/
```

Weitere sinnvolle Softwarepakete:
* SSSD für die Nutzerauthentifikation
* Firewalld für zusätzliche Firewalls gegen Angreifer
* OpenSSL, sollte bereits installiert sein
* GPG muss schon installiert sein.

## Skripte kopieren


OATH-Skripte kopieren
```bash
cp oathgen oathupdate /usr/local/bin/
cp oathupdate.service /etc/systemd/system/
chmod 755 oathgen oathinotify
```

copy credentials
```bash
scp management:/etc/public_key.gpg /etc/public_key.gpg
install -m 600 /dev/null /etc/secret
ssh management cat /etc/secret > /etc/secret
```

Systemd-Service aktivieren
```bash
systemctl daemon-reload
systemctl enable oathupdate.service
systemctl start oathupdate.service
```

Damit die Nutzer*innen nur ein Skript ausführen können, wird nur das in des SSHD-Konfiguration gestattet.

SSHD-Konfiguration `/etc/ssh/sshd_config` anpassen
```bash
# alle außer root führen automatisch Kommando aus
Match User *,!root
        ForceCommand /usr/local/bin/oathgen
```

 SSH-Dienst neu laden
 ```bash
 systemctl reload sshd.service
```

