# Samba Active Directory Domain Controller

## Rôle

Le serveur Samba AD DC fournit l’authentification centralisée du domaine,
la gestion des utilisateurs, des groupes, des ordinateurs et des stratégies
d’accès. Il joue également le rôle de serveur DNS interne pour le domaine
Active Directory.

## Fichiers principaux

- `/etc/samba/smb.conf`
- `/etc/krb5.conf`
- `/etc/resolv.conf`

## Configuration principale

```ini
[global]
    workgroup = SALAR
    realm = SAL.LOCAL
    server role = active directory domain controller
    dns forwar
```

## Provisionnement du domaine

Le domaine a été créé avec la commande suivante :

```bash
sudo samba-tool domain provision \
  --server-role=dc \
  --use-rfc2307 \
  --dns-backend=SAMBA_INTERNAL \
  --realm=SALAR.COM \
  --domain=SALAR \
  --adminpass='Mejj_2064'
```

Le paramètre `dns forwarder` permet à Samba de transmettre les requêtes DNS
externes vers le pare-feu ou le résolveur configuré dans le réseau. [176][179]

## Étapes réalisées

1. Installation de Samba et des dépendances nécessaires.
2. Configuration de l’adresse IP statique, du nom d’hôte et de la résolution DNS.
3. Provisionnement du domaine Active Directory avec Samba.
4. Configuration du DNS interne et du redirecteur DNS.
5. Création des unités d’organisation, utilisateurs et groupes.
6. Intégration des postes clients au domaine.
7. Tests d’authentification Kerberos et d’accès aux ressources du domaine.

## Vérifications

Vérification de la configuration Samba :

```bash
testparm
Load smb config files from /etc/samba/smb.conf
Loaded services file OK.
Weak crypto is allowed by GnuTLS (e.g. NTLM as a compatibility fallback)

Server role: ROLE_ACTIVE_DIRECTORY_DC

Press enter to see a dump of your service definitions

# Global parameters
[global]
        dns forwarder = 1.1.1.1 8.8.8.8
        netbios name = MEJJ
        passdb backend = samba_dsdb
        realm = SALAR.COM
        server role = active directory domain controller
        workgroup = SALAR
        rpc_server:tcpip = no
        rpc_daemon:spoolssd = embedded
        rpc_server:spoolss = embedded
        rpc_server:winreg = embedded
        rpc_server:ntsvcs = embedded
        rpc_server:eventlog = embedded
        rpc_server:srvsvc = embedded
        rpc_server:svcctl = embedded
        rpc_server:default = external
        winbindd:use external pipes = true
        idmap_ldb:use rfc2307 = yes
        idmap config * : backend = tdb
        map archive = No
        vfs objects = dfs_samba4 acl_xattr


[sysvol]
        path = /var/lib/samba/sysvol
        read only = No


[netlogon]
        path = /var/lib/samba/sysvol/salar.com/scripts
        read only = No
```

Vérification de l’état du service :

```bash
systemctl status samba-ad-dc
● samba-ad-dc.service - Samba AD Daemon
     Loaded: loaded (/usr/lib/systemd/system/samba-ad-dc.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-10-16 19:39:10 WAT; 53min ago
 Invocation: 65b787ff0761415387ddab7014667aad
       Docs: man:samba(8)
             man:samba(7)
             man:smb.conf(5)
    Process: 835 ExecCondition=/usr/share/samba/is-configured samba (code=exited, status=0/SUCCESS)
   Main PID: 907 (samba)
     Status: "samba: ready to serve connections..."
      Tasks: 63 (limit: 411)
     Memory: 320.5M (peak: 417M)
        CPU: 22.825s
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
