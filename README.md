# 2FA

2FA zum Testen für bwHPC

Nutzer können sich einmalig auf dem QR-Generator-Knoten einen QR-Code erzeugen lassen, den sie mit einer geeigneten App abscannen. Diese erzeugt nun Einmalpasswörter, die als zweiter Faktor beim Login auf dem eigentlichen System abgefragt werden. Zukünftig könnte man auch Yubikeys etc. unterstützen, doch funktioniert derzeit lediglich die Yubikey-Mobil-App.

Eine erneute Erzeugung des QR-Codes ist nur möglich, wenn der Nutzer administrativ auf dem QR-Generator-Knoten wieder freigeschalten wurde.

Dies ist ein Fork der von [nemo-cluster/2fa](https://github.com/nemo-cluster/
2fa) die wesentlichen Änderungen sind:
* Es wird sichere auf RSA basierende Verschlüsslung aus gpg statt des textbook 
RSA aus openssl verwendet.
* Es wird sichergestellt, dass die Einmalpasswörter wirklich nur einmalig 
verwendet werden können.
* Um vorher genanntes zu erreichen wurde der Updatemechanismus angepasst.
* Mehrere der Management-Knoten (HA-Setups) werden supportet.

## Installation

Es gibt drei Knotentypen, wobei Management- und Loginknoten identisch sein können. Im einfachsten Fall nutzen alle drei Knotentypen die selbe Nutzerauthentifizierung (z.B. LDAP). Da auf dem Managementknoten Credentials erzeugt werden, die der QR-Generator-Knoten ebenfalls benötigt, sollte mit dessen Installation begonnen werden:

* [QR-Generator-Knoten](./README.qr-node.md), dient ausschliesslich zum Erzeugen des zweiten Faktors, dem QR-Code. 
* [Managementknoten](README.manager-node.md) nehmen diesen QR-Code entgegen.
* [Loginknoten](README.login-node.md), auf denen die Zwei-Faktor-Authentifizierung erfolgen soll.

## Geeignete Mobile-Apps

### Mobile Authenticator-Apps für Android

* [FreeOTP Authenticator](market://details?id=org.fedorahosted.freeotp) ([PlayStore](https://play.google.com/store/apps/details?id=org.fedorahosted.freeotp) [Webseite](https://freeotp.github.io/)
* [Aegis Authenticator ](market://details?id=com.beemdevelopment.aegis)
* [Google Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2)
* [Yubico Authenticator](market://details?id=com.yubico.yubioath)
* [Microsoft Authentificator](market://details?id=com.azure.authenticator) [Webseite](https://www.microsoft.com/en-us/account/authenticator)

### IOS

* [FreeOTP Authenticator](https://apps.apple.com/us/app/freeotp-authenticator/id872559395)
* [Google Authenticator](https://apps.apple.com/us/app/google-authenticator/id388497605)
* [Yubico Authenticator](https://apps.apple.com/us/app/yubico-authenticator/id1476679808)

### Amazon Fire Tablets/Sticks:
[Mufri Authenticator](https://www.amazon.com/Mufri-Authenticator-Plus/dp/B00F48EBSU/)

### Samsung Geräte
[Microsoft Authentificator](http://apps.samsung.com/appquery/appDetail.as?appId=com.azure.authenticator)

## Infos für Admins
* Soll ermöglicht werden, dass ein User sich neue Credentials erstellen kann, muss oathdel auf dem QR-Geneartor-Knoten aufgerufen werden. Diese überschreiben die alten sobald der User dies auch wirklich macht.
* Soll ein 2nd Faktor deaktiviert werden muss auf dem Management deactivateuser $user aufgerufen werden.
Diese funktionen sind also komplementär.

## Wichtige Dateien und Verzeichnisse

* `/etc/2fa/secret`: Hier liegt das shared secret, mit dem Management und QR-Generator-Knoten Credentials austauschen. Es muss entsprechend auf diesen Knotentypen identisch sein.

* `/<Pfad>/2fa/users.oath /<Pfad>/2fa/users.lock`: Hier liegt die Liste der User und deren Seeds, welche zwischen QR-Generator- und Management-Knoten synchron sein muss. Es empfiehlt sich daher ein geshartes Dateisystem (NFS, Lustre, etc.), prinzipiell sind aber auch andere Synchronisierungsmechanismus möglich.

* `/root/.gnupg/ /etc/public_key.gpg`: Der Key mit dem die Seeds verschlüsselt werden. Der öffentliche Schlüssel muss auf jedem QR-Generator-Knoten, der private Schlüssel auf jedem Management-Knoten importiert werden.

