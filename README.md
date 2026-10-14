# Linux Enterprise Services Lab

Mise en place d’une infrastructure Linux d’entreprise incluant un contrôleur 
de domaine Samba AD, un serveur DHCP Kea, un partage de fichiers, une 
messagerie complète, un reverse proxy Nginx, une supervision Zabbix et une 
détection d’intrusion Suricata intégrée à OPNsense.

## Objectifs

- Mettre en place une authentification centralisée avec Samba Active Directory
- Fournir les adresses IP avec Kea DHCP
- Offrir un partage de fichiers sécurisé avec Samba
- Déployer une messagerie interne avec Postfix, Dovecot et Roundcube
- Exposer les services web via Nginx
- Superviser les serveurs et services avec Zabbix
- Centraliser les journaux via Syslog
- Détecter les activités suspectes avec Suricata sur OPNsense

## Architecture

![Architecture](screenshots/infrastructure-topology.png)

## Services déployés

| Service | Rôle |
|---|---|
| OPNsense | Pare-feu, routage, règles de sécurité |
| Suricata | Détection d’intrusion (IDS/IPS) |
| Kea DHCP | Attribution automatique des adresses IP |
| Samba AD DC | Contrôleur de domaine et authentification |
| Samba File Server | Partage de fichiers et permissions |
| Postfix | Serveur SMTP |
| Dovecot | Serveur IMAP/POP3 |
| Roundcube | Webmail |
| Nginx | Reverse proxy / serveur web |
| Zabbix | Supervision des hôtes et services |
| Syslog | Centralisation des journaux |

## Compétences démontrées

- Administration Linux
- Services réseau et systèmes
- Active Directory / Samba AD
- Sécurité réseau et détection d’intrusion
- Supervision et centralisation des logs
- Documentation d’infrastructure
