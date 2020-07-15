# 2FA
2FA zum Testen für bwHPC

Bei diesem Ansatz wird nur ein SSH-Zugang des Nutzers benötigt und der QR-Code für Auth-Apps wird angezeigt. Dieser wird nur beim ersten Login angezeigt.

Zukünftig könnte man auch Yubikeys etc. unterstützen. Derzeit funktioniert das nur nur über die Yubikey-Mobil-App, die vergleichbar mit den Auth-Apps ist.

## Mobile-Apps

Mobile Authenticator-Apps für Andriod

* [FreeOTP Authenticator](market://details?id=org.fedorahosted.freeotp)
* [Aegis Authenticator ](market://details?id=com.beemdevelopment.aegis)
* [Google Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2)
* [Yubico Authenticator](market://details?id=com.yubico.yubioath)

und IOS

* [FreeOTP Authenticator](https://apps.apple.com/us/app/freeotp-authenticator/id872559395)
* [Google Authenticator](https://apps.apple.com/us/app/google-authenticator/id388497605)
* [Yubico Authenticator](https://apps.apple.com/us/app/yubico-authenticator/id1476679808)

## Grundkonzept

Es gibt drei Knotenarten:
* Den Knoten auf dem der "Zweite Faktor" erzeugt wird, im folgenden [QR-Generator-Knoten](./README.qr-node.md)
* [Managementknoten](README.manager-node.md), die diese entgegen nehmen.
* Als letztes, die [Loginknoten](README.login-node.md) auf denen die Zwei-Faktor-Authentifizierung erfolgen soll.
Prinzipiell ist es möglich, dass die selben Knoten die funktionalität von Managementknoten und Loginknoten übernehmen.

Wie die einzelnen Knoten installiert werden findet sich in den drei Dateien:
[README.login-node.md](README.login-node.md) [README.manager-node.md](README.manager-node.md) und [README.qr-node.md](./README.qr-node.md)

Da auf dem Managementknoten Credentials erzeugt werden, die auf dem QR-Knoten benötigt werden, sollte mit der Installtion von diesen begonnen werden.

## Nutzungsweise
Nutzer können sich einmalig auf dem QR-Generator-Knoten einen QR-Code holen, den sie mit einer passnden App abscannen.

Mit dieser App können sie dann Einmalpasswörter erzeugen mit denen sie sich auf den Loginknoten anmelden können.

Eine erneute Erzugung des QR-Codes ist nur möglich, wenn der User administrativ per oathdel wieder erneut freigeschalten wurde.

## Wichtge Daten für Admins
#/etc/secret:
Hier liegt das shared secret mit dem Management und QR-Generator-Knoten Credentials austauschen. Es muss entsprechend auf allen Knoten dieser typen das selbe sein.

#/root/.gnupg/ /etc/public_key.gpg:
Der Key mit dem die seeds verschlüsselt werden. Sie müssen per gpg --key-gen einmalig erzeugt werden.

Der öffentliche Schlüssel muss auf jedem QR-Generator-Knoten importiert werden. Der Private auf jedem Management-Knoten.

#/home/2fa/users.oath /home/2fa/users.lock
Hier liegt die Liste der User und deren Seeds.

Sie muss zwischen allen QR-Generator-Knoten und Management-Knoten jederzeit synchron gehalten werden. Passiert dies nicht, können die gleichen "Einmalpasswörter" mehrfach verwendet werden.
Es empfiehlt sich stark ein geshartes Dateisystem. Prinzipiell ist aber auch jeder andere Synchronisierungsmechanismus möglich.
