# Samba Active Directory Domain Controller

## Rôle

Le serveur Samba pour le partage de fichier fournit des repertoires pour les
utilisateurs du reseau pour l'echange de fichiers.

## Fichiers principaux

- `/etc/samba/smb.conf`
- `/etc/samba/nsswitch.conf`

## Configuration principale

```ini
[global]
    workgroup = SALAR
    realm = SALAR.COM
    security = ADS

    kerberos method = secrets and keytab

    winbind refresh tickets = yes
    winbind use default domain = yes
    winbind enum users = yes
    winbind enum groups = yes

    idmap config * : backend = tdb
    idmap config * : range = 3000-7999

    idmap config SALAR : backend = rid
    idmap config SALAR : range = 10000-999999

    template shell = /bin/bash
    template homedir = /home/%U

    winbind separator = +

    client min protocol = SMB2
    server min protocol = SMB2
[private]
    path = /srv/samba/private
    read only = no
    browseable = yes

    valid users = @"GRP_ADMINS"
    force group = "grp_admins"

    create mask = 0660
    directory mask = 2770


[public]
    path = /srv/samba/public
    read only = no
    browseable = yes

    valid users = @"GRP_ADMINS" @"GRP_STAGIAIRE"

    create mask = 0660
    directory mask = 2770
```

## Creation des repertoires

mkdir -p /srv/samba/public
mkdir -p /srv/samba/private

## Étapes réalisées

1. Installation de Samba, Winbind et des dépendances nécessaires.
2. Configuration de l’adresse IP statique, du nom d’hôte et de la résolution DNS.
3. Jointure au domaine
3. Creation des repertoires
4. Attribution des utilisateurs et groupes proprietaires des repertoires
5. configuration des droits d'utilisateurs
6. Test d'acces aux repertoires

## Vérifications

Vérification de la configuration Samba :

```bash
testparm
Load smb config files from /etc/samba/smb.conf
Loaded services file OK.
Weak crypto is allowed by GnuTLS (e.g. NTLM as a compatibility fallback)

'winbind separator = +' might cause problems with group membership.

SUGGESTION: You may want to use 'sync machine password to keytab' parameter instead of 'kerberos method'.

Server role: ROLE_DOMAIN_MEMBER

Press enter to see a dump of your service definitions

# Global parameters
[global]
        client min protocol = SMB2
        kerberos method = secrets and keytab
        realm = SALAR.COM
        security = ADS
        server min protocol = SMB2
        template homedir = /home/%U
        template shell = /bin/bash
        winbind enum groups = Yes
        winbind enum users = Yes
        winbind refresh tickets = Yes
        winbind separator = +
        winbind use default domain = Yes
        workgroup = SALAR
        idmap config salar : range = 10000-999999
        idmap config salar : backend = rid
        idmap config * : range = 3000-7999
        idmap config * : backend = tdb


[private]
        create mask = 0660
        directory mask = 02770
        force group = "grp_admins"
        path = /srv/samba/private
        read only = No
        valid users = @GRP_ADMINS


[public]
        create mask = 0660
        directory mask = 02770
        path = /srv/samba/public
        read only = No
        valid users = @GRP_ADMINS @GRP_STAGIAIRE
```

Vérification de l’état du service :

```bash
systemctl status smbd
● smbd.service - Samba SMB Daemon
     Loaded: loaded (/usr/lib/systemd/system/smbd.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-10-16 20:48:59 WAT; 48min ago
 Invocation: 3383daf357924ce782861433361cb8b9
       Docs: man:smbd(8)
             man:samba(7)
             man:smb.conf(5)
    Process: 3550 ExecCondition=/usr/share/samba/is-configured smb (code=exited, status=0/SUCCESS)
    Process: 3553 ExecStartPre=/usr/share/samba/update-apparmor-samba-profile (code=exited, status=0/SUCCESS)
   Main PID: 3558 (smbd)
     Status: "smbd: ready to serve connections..."
      Tasks: 4 (limit: 1038)
     Memory: 12.6M (peak: 44.7M)
        CPU: 896ms
     CGroup: /system.slice/smbd.service
             ├─3558 /usr/sbin/smbd --foreground --no-process-group
             ├─3561 "smbd: notifyd" .
             ├─3562 "smbd: cleanupd "
             └─3630 "smbd: client [192.168.1.99]"

oct. 16 20:48:59 file.salar.com systemd[1]: Starting smbd.service - Samba SMB Daemon...
oct. 16 20:48:59 file.salar.com systemd[1]: Started smbd.service - Samba SMB Daemon.
```

Liste des utilisateurs du domaine :

```bash
samba-tool user list
jeje
admin
Guest
Administrator
krbtgt
leopold
John
```

Affichage du niveau fonctionnel du domaine et de la forêt :

```bash
samba-tool domain level showa
Domain and forest function level for domain 'DC=salar,DC=com'
Forest function level: (Windows) 2008 R2
Domain function level: (Windows) 2008 R2
Lowest function level of a DC: (Windows) 2008 R2
```

Obtention d’un ticket Kerberos pour l’administrateur :

```bash
kinit Administrator
Warning: Your password will expire in 12 days on jeu. 29 oct. 2026 18:11:26

klist
Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: Administrator@SALAR.COM

Valid starting       Expires              Service principal
16/10/2026 20:35:00  17/10/2026 06:35:00  krbtgt/SALAR.COM@SALAR.COM
        renew until 17/10/2026 20:34:56
```

## Tests réalisés

- Vérification de la résolution DNS du domaine.
- Obtention d’un ticket Kerberos avec `kinit`.
- Jointure d’un poste client au domaine.
- Connexion d’un utilisateur du domaine sur un poste client.
- Accès aux ressources partagées après authentification.

## Problèmes rencontrés

- Résolution DNS du domaine corrigée en vérifiant le contenu de `/etc/resolv.conf`.
- Le redirecteur DNS a été configuré dans `smb.conf` pour permettre la
  résolution des noms externes.
