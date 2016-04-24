# Playbook Citoyens

Ce projet contient un playbook et des rôles ansible pour les applications de Regards Citoyens.

## Prérequis

Les playbooks supposent qu'il existe sur les machines cible un utilisateur 'rcdeploy' qui peut exécuter des commandes en `sudo` sans mot de passe.

> Il est conseillé de n'autoriser la connexion de cet utilisateur qu'avec une paire de clés SSH (par exemple en supprimant son mot de passe).

Les applications sont hébergées dans le répertoire `/srv`, chacune avec un utilisateur dédié, sans mot de passe.

La version 2.0 d'ansible (ou plus récente) doit être utilisée.

### Vérification

Pour vérifier le fonctionnement d'ansible et la connexion aux cibles, exécuter depuis la racine de ce repository :

    ansible all --private-key=CLEPRIVEE -m ping

Ansible doit répondre avec un "ping: pong" pour chaque cible.

### Exécution

    ansible-playbook --private-key=CLEPRIVEE site.yml

## Groupes de serveurs

Les groupes de serveurs suivants sont définis :

* `all_servers` : contient toutes les machines ; y seront exécutés les rôles `apache`, `common` et `munin-node` ;
* `git_server`: contient le serveur hébergeant gogs ; le rôle `gogs` y est exécuté ;
* `lfdl_server`: contient le serveur hébergeant La Fabrique de la Loi ; le rôle `lafabrique` y est exécuté ;
* `pad_server`: contient le serveur hébergeant le pad ; le rôle `pad` y est exécuté ;

*Note :* les groupes nommés `*_server` (au singulier) sont destinés à ne contenir qu'une machine, mais fonctionnent aussi bien avec plusieurs serveurs.

---

## Rôles applicatifs

Ces rôles permettent l'installation des applications créées par Regards Citoyens.

### lafabrique

*Dépend de : common, apache*

Ce rôle installe la fabrique de la loi.

*Variables :*

* `lafabrique_home` (`/srv/lafabrique`) : homedir pour l'user lafabrique
* `lafabrique_domain` (`www.lafabriquedelaloi.fr`) : domaine
* `lafabrique_api_repo` (`git://github.com/regardscitoyens/the-law-factory-parser.git`) : repo git pour l'api
* `lafabrique_api_branch` (`master`) : branche git pour l'api
* `lafabrique_www_repo` (`git://github.com/regardscitoyens/the-law-factory.git`) : repo git pour le frontend
* `lafabrique_www_branch` (`css-refactor`) : branche git pour le frontend
* `lafabrique_ssl_cert` (non défini) : chemin *distant* vers le certificat SSL à utiliser ; s'il est indéfini, SSL ne sera pas utilisé sur le vhost
* `lafabrique_ssl_key` (non défini): chemin *distant* vers la clé privée serveur pour le certificat SSL

---

## Rôles utilitaires

Ces rôles permettent l'installation de services tiers utilisés par Regards Citoyens ou par les applications.

*Variables communes :*

* `contact_email` (`contact@regardscitoyens.org`) : e-mail de contact utilisé dans plusieurs applications

### apache

Installe apache, active quelques modules utiles et désactive le site par défaut.

*Variables :*

* `apache_default_redirect` (`https://www.regardscitoyens.org/`) : redirection utilisée pour tout vhost inconnu (et le site par défaut)

### common

Installe des paquets utiles et crée un groupe commun 'rcapps'.

### gogs

*Dépend de : go, mariadb*

Installe gogs et configure un reverse-proxy apache pour y accéder avec un vhost.

**Attention** : après la première exécution de ce rôle sur une cible, accéder immédiatement à l'interface Web de gogs pour définir le compte utilisateur et ainsi désactiver l'interface d'installation.

*Variables :*

* `gogs_title` (`Git Regards Citoyens`) : titre de l'appli
* `gogs_home` (`/srv/gogs`) : homedir pour l'user gogs
* `gogs_port` (`3000`) : port d'écoute local (interface loopback uniquement)
* `gogs_domain` (`git.regardscitoyens.org`) : domaine
* `gogs_ssl_cert` (non défini) : chemin *distant* vers le certificat SSL à utiliser ; s'il est indéfini, SSL ne sera pas activé sur le vhost
* `gogs_ssl_key` (non défini) : chemin *distant* vers la clé privée serveur pour le certificat SSL
* `gogs_db_name` (`gogs`) : nom de la BDD mysql
* `gogs_db_user` (`gogs`) : nom de l'user mysql
* `gogs_db_pass` (`gogs`) : mot de passe mysql

### munin-master

Installe munin et le configure pour récupérer les informations de tous les serveurs ayant le rôle munin-node ; met en place un vhost apache pour accéder aux logs.  Si SSL est activé, l'accès aux stats est protégé par authentification HTTP Basic.

*Variables :*

* `munin_ssl_cert` (non défini) : chemin *distant* vers le certificat SSL à utiliser ; s'il est indéfini, SSL ne sera pas activé sur le vhost
* `munin_ssl_key` (non défini) : chemin *distant* vers la clé privée serveur pour le certificat SSL
* `munin_domain` (`munin.regardscitoyens.org`) : domaine pour le vhost munin
* `munin_htpassword` (non défini) : doit être défini lors de la première exécution du rôle avec SSL actif ; définit le mot de passe de l'user rcmunin pour l'authentification HTTP.

### munin-node

Installe munin-node et autorise les serveurs ayant le rôle munin-master à y accéder.

### pad

*Dépend de : mariadb*

Installe etherpad-lite et configure un reverse-proxy apache pour y accéder avec un vhost ; permet aussi d'importer les données depuis un etherpad (scala).

*Variables :*

* `etherpad_title` (`Pad Regards Citoyens`) : titre du pad
* `etherpad_home` (`/srv/etherpad`) : homedir pour l'user etherpad
* `etherpad_port` (`9001`) : port d'écoute local (interface loopback uniquement)
* `etherpad_ssl_cert` (non défini) : chemin *distant* vers le certificat SSL à utiliser ; s'il est indéfini, SSL ne sera pas activé sur le vhost
* `etherpad_ssl_key` (non défini) : chemin *distant* vers la clé privée serveur pour le certificat SSL
* `etherpad_domain` (`pad.regardscitoyens.org`) : domaine du pad
* `etherpad_db_name` (`etherpad`) : nom de la BDD mysql
* `etherpad_db_user` (`etherpad`) : nom de l'user mysql
* `etherpad_db_pass` (`etherpad`) : mot de passe mysql
* `etherpad_import` (non défini) : chemin vers un fichier de données à importer ; il doit s'agit d'un dump SQL compressé en XZ. **Attention, toute donnée existante sera écrasée de manière irréversible si cette variable est définie.**

---

## Rôles techniques

Ces rôles sont destinés à être utilisés comme dépendance d'un autre rôle.

### go

Installe go depuis la distribution binaire officielle.

*Variables :*

* `go_tgz` (`go1.6.linux-amd64.tar.gz`) : nom de l'archive tgz à télécharger
* `go_url` (`https://storage.googleapis.com/golang/{{ go_tgz }}`) : URL de l'archive tgz à télécharger
* `go_home` (`/root`) : chemin où télécharger (temporairement) l'archive
* `go_version_target` (`go version go1.6 linux/amd64`) : version attendue de go ; si `go version` renvoie autre chose, l'archive téléchargée sera installée.

### mariadb

Installe mariadb.
