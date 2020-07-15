# Managment Knoten
Knoten, der die Credentials entgegen nimmt

## Requirements

WICHTIG: Es wird ein OpenSSL >=1.1.1 benötigt. Dies ist erst in CentOS 8 enthalten.
Es kann einfach mit ./config && make unter CentOS 7 gebaut werden.
Eventuell müssen PATH und LD_LIBRARY_PATH in addline entsprechend angepasst werden.

Des weiteren wird socat zum übertragen benötigt.

```bash
# EPEL Repo
yum install -y epel-release
# pdsh
yum install -y socat
```


## Scripte

OATH-Skript kopieren
```bash
cp oathcron addline /usr/local/bin/
chmod 755 oathcron
chmod +x /usr/local/bin/addline
cp addline.service /etc/systemd/system/addline.service
systemctl daemon-reload
systemctl enable addline
systemctl start addline
```

Erstellen der Schlüssel zum verschlüsseln der "Zweiten Faktoren"
```bash
gpg2 --gen-key # Schlüssel für die Adresse key@subission.binac erzeugen
gpg --export key@subission.binac > /etc/public_key.gpg
gpg -a --export-secret-key # Für backupzwecke und falls mehr als ein Managment Knoten existiert.
install -m 600 /dev/null /etc/secret
head -c 15 /dev/urandom > /etc/secret
install -m 600 /dev/null /home/2fa/users.oath
echo "# Option        User    Prefix  Seed" > /home/2fa/users.oath
```

## Weiteres
Der Port 165 muss vom QR-Generator-Knoten aus erreichbar sein. 
/etc/secret und /etc/public_key.gpg werden später auf dem QR-Generator-Knoten benötigt.
