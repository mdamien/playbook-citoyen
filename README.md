# Playbooks Citoyens

Ce projet contient des playbooks ansible pour les applications de Regards Citoyens.

## Prérequis

Les playbooks supposent qu'il existe sur les machines cible un utilisateur 'rcdeploy' qui peut exécuter des commandes en `sudo` sans mot de passe.

> Il est conseillé de n'autoriser la connexion de cet utilisateur qu'avec une paire de clés SSH (par exemple en supprimant son mot de passe).

Les applications sont hébergées dans le répertoire `/srv`, chacune avec un utilisateur dédié, sans mot de passe.

La version 2.0 d'ansible (ou plus récente) doit être utilisée.

## Vérification

Pour vérifier le fonctionnement d'ansible et la connexion aux cibles, exécuter depuis la racine de ce repository :

    ansible all --private-key=CLEPRIVEE -m ping

Ansible doit répondre avec un "ping: pong" pour chaque cible.

## Exécution

    ansible-playbook --private-key=CLEPRIVEE site.yml

## Rôles applicatifs

*Variables communes :*

* `contact_email` (`contact@regardscitoyens.org`) : e-mail de contact utilisé dans plusieurs applications

### pad

*Dépend de : common, apache, mariadb*

Installe etherpad-lite et configure un reverse-proxy apache pour y accéder avec un vhost ; permet aussi d'importer les données depuis un etherpad (scala).

*Variables :*

* `etherpad_home` (`/srv/etherpad`) : homedir pour l'user etherpad
* `etherpad_port` (`9001`) : port d'écoute local (interface loopback uniquement)
* `etherpad_title` (`Pad Regards Citoyens`) : titre du pad
* `etherpad_domain` (`pad.regardscitoyens.org`) : domaine du pad
* `etherpad_db_name` (`etherpad`) : nom de la BDD mysql
* `etherpad_db_user` (`etherpad`) : nom de l'user mysql
* `etherpad_db_pass` (`etherpad`) : mot de passe mysql
* `etherpad_import` (non défini) : chemin vers un fichier de données à importer ; il doit s'agit d'un dump SQL compressé en XZ. **Attention, toute donnée existante sera écrasée de manière irréversible si cette variable est définie.**

## Rôles communs/utilitaires

Ces rôles ne sont pas destinés à être utilisés directement ; ils sont utilisés comme dépendances de rôles applicatifs.

### apache

Installe apache, active quelques modules utiles et désactive le site par défaut.

*Variables :*

* `apache_default_redirect` (`https://www.regardscitoyens.org/`) : redirection utilisée pour tout vhost inconnu (et le site par défaut)

### common

Installe des paquets utiles et crée un groupe commun 'rcapps'.

### mariadb

Installe mariadb.
