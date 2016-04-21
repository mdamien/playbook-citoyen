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

## Rôles applicatifs

Ces rôles permettent l'installation des applications créées par Regards Citoyens.

### lafabrique

*Dépend de : common, apache*

Ce rôle installe la fabrique de la loi.

*Variables :*

---

* `lafabrique_home` (`/srv/lafabrique`) : homedir pour l'user lafabrique
* `lafabrique_domain` (`www.lafabriquedelaloi.fr`) : domaine
* `lafabrique_api_repo` (`git://github.com/regardscitoyens/the-law-factory-parser.git`) : repo git pour l'api
* `lafabrique_api_branch` (`master`) : branche git pour l'api
* `lafabrique_www_repo` (`git://github.com/regardscitoyens/the-law-factory.git`) : repo git pour le frontend
* `lafabrique_www_branch` (`css-refactor`) : branche git pour le frontend
* `lafabrique_ssl_cert` (non défini) : chemin *distant* vers le certificat SSL à utiliser ; s'il est indéfini, SSL ne sera pas utilisé sur le vhost
* `lafabrique_ssl_key` (non défini): chemin *distant* vers la clé privée serveur pour le certificat SSL

## Rôles utilitaires

Ces rôles permettent l'installation de services tiers utilisés par Regards Citoyens ou par les applications.

*Variables communes :*

* `contact_email` (`contact@regardscitoyens.org`) : e-mail de contact utilisé dans plusieurs applications

### gogs

*Dépend de : common, apache, go, mariadb*

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

### pad

*Dépend de : common, apache, mariadb*

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

## Rôles transverses

Ces rôles ne sont pas destinés à être utilisés directement ; ils sont utilisés comme dépendances de rôles applicatifs.

### apache

Installe apache, active quelques modules utiles et désactive le site par défaut.

*Variables :*

* `apache_default_redirect` (`https://www.regardscitoyens.org/`) : redirection utilisée pour tout vhost inconnu (et le site par défaut)

### common

Installe des paquets utiles et crée un groupe commun 'rcapps'.

### go

Installe go depuis la distribution binaire officielle.

*Variables :*

* `go_tgz` (`go1.6.linux-amd64.tar.gz`) : nom de l'archive tgz à télécharger
* `go_url` (`https://storage.googleapis.com/golang/{{ go_tgz }}`) : URL de l'archive tgz à télécharger
* `go_home` (`/root`) : chemin où télécharger (temporairement) l'archive
* `go_version_target` (`go version go1.6 linux/amd64`) : version attendue de go ; si `go version` renvoie autre chose, l'archive téléchargée sera installée.

### mariadb

Installe mariadb.
