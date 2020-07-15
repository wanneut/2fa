# Zwei-Faktor-Knoten

## Voraussetzungen

Damit der "Zweite Faktor" geprüfte werden kann werden folgendes Paket benötigt.
```bash
# EPEL Repo
yum install -y epel-release
# pam_oath
yum install -y pam_oath pam_ssh_user_auth
```

## PAM konfigurieren

PAM konfigurieren für SSH passiert über die `/etc/pam.d/sshd`. Je nachdem ob die folgende Zeile am Anfang oder Ende steht wird das TOTP-Token vor oder nach der Passworteingabe abgefragt.
```bash
auth	  required pam_oath.so usersfile=/home/2fa/users.oath window=30 digits=6
```


Soll in dem  Fall, dass zuvor schon eine Public-Key-Authentification erfolgte nicht mehr erneut das passwort abgefragt werden, wird pam_ssh_user_auth.so benötigt.


Eine passende: /etc/pam.d/sshd
```bash
auth       requisite    pam_oath.so usersfile=/home/2fa/users.oath window=30 digits=6
auth       [success=1 ignore=ignore default=die]        pam_ssh_user_auth.so
auth       substack     password-auth
auth       include      postlogin
```
### OpenSSH Konfigurieren

PasswordAuthentication funktioniert nicht zusammen mit TOTP deswegen sollte diese aus, und ChallengeResponseAuthentication (die ebenfalls das Passwort abfragen kann) eingeschaltet werden.
Sollen Zusätzlich SSH-Schlüssel mit TOTP abgesichert werden, müssen diese nacheinander abgefragt werden.

Soll beides möglich sein (nur CentOS 8)
```bash
ExposeAuthInfo yes
ChallengeResponseAuthentication yes
PasswordAuthentication no
PubkeyAuthentication yes
UsePAM yes
AuthenticationMethods publickey,keyboard-interactive:pam keyboard-interactive:pam
```

Der in RHEL 7 enthaltene OpenSSH kennt die Option ExposeAuthInfo noch nicht.
Red Hat hat dafür einen Patch mit ähnlicher Funktionalität eingespielt.

Die passende Config sieht dann so aus:
```bash
ExposeAuthenticationMethods pam-and-env
ChallengeResponseAuthentication yes
PasswordAuthentication no
PubkeyAuthentication yes
UsePAM yes
AuthenticationMethods publickey,keyboard-interactive:pam keyboard-interactive:pam
```

Optional:
Soll für gewisse Gruppen kein TOTP verlangt werden, weil sie Keys mit Passwörtern nutzen:
```bash
Match Group nootp
  AuthenticationMethods publickey
```
