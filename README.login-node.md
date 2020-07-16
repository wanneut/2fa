# Zwei-Faktor-Knoten

## Voraussetzungen

Es wird vorausgesetzt, daß alle Knotentypen die selbe Nutzerauthentifizierung nutzen (z.B. LDAP).

## Abhängigkeiten

* EPEL Repo:

```bash
yum install -y epel-release
yum install -y pam_oath pam_ssh_user_auth
```

## Konfiguration

### PAM

Die Konfiguration erfolgt in  `/etc/pam.d/sshd`. Je nachdem, ob die folgende Zeile am Anfang oder Ende steht wird das TOTP-Token vor oder nach der Passworteingabe abgefragt.

```bash
auth	  required pam_oath.so usersfile=/<Pfad>/2fa/users.oath window=30 digits=6
```

Soll nach erfolgter Authentifizierung durch einen Public-Key nicht auch noch das Passwort abgefragt werden, wird zusätzlich `pam_ssh_user_auth.so` benötigt, z.B.

```bash
auth       requisite    pam_oath.so usersfile=/<Pfad>/2fa/users.oath window=30 digits=6
auth       [success=1 ignore=ignore default=die]        pam_ssh_user_auth.so
auth       substack     password-auth
auth       include      postlogin
```
### OpenSSH

`PasswordAuthentication` funktioniert nicht zusammen mit TOTP, daher sollte stattdessen `ChallengeResponseAuthentication` (die ebenfalls das Passwort abfragen kann) eingeschaltet werden. Sollen Zusätzlich SSH-Schlüssel mit TOTP abgesichert werden, müssen diese nacheinander abgefragt werden. Soll beides möglich sein (nur CentOS 8):

```bash
ExposeAuthInfo yes
ChallengeResponseAuthentication yes
PasswordAuthentication no
PubkeyAuthentication yes
UsePAM yes
AuthenticationMethods publickey,keyboard-interactive:pam keyboard-interactive:pam
```

Der in RHEL 7 enthaltene OpenSSH kennt die Option `ExposeAuthInfo` noch nicht. Es gibt aber einen Patch mit ähnlicher Funktionalität von RedHat. Die passende Konfiguration dazu ist:

```bash
ExposeAuthenticationMethods pam-and-env
ChallengeResponseAuthentication yes
PasswordAuthentication no
PubkeyAuthentication yes
UsePAM yes
AuthenticationMethods publickey,keyboard-interactive:pam keyboard-interactive:pam
```

Optional: Soll für gewisse Gruppen kein TOTP verlangt werden, weil sie Keys mit Passwörtern nutzen:

```bash
Match Group nootp
  AuthenticationMethods publickey
```


