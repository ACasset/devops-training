# TP 05 : Synthèse

## Introduction

Ce TP a pour but de réutiliser l'ensemble des connaissances acquises au cours des TPs 01 à 04.

## Prérequis

Ce TP part du principe que vous venez de dérouler les TPs 01 à 04.

Vous devriez donc déjà avoir à disposition :
- un contrôleur avec Ansible
- deux hôtes distants sur lesquels la clé SSH du contrôleur a été copiée
- un inventaire configuré pour accéder aux hôtes distants, et structuré avec un groupe `frontend` et un groupe `backend`

## Déroulé

Ce TP est un TP de synthèse où rien n'est fourni.

L'objectif est d'écrire :
- un rôle pour installer une base de données MySQL
- un rôle pour installer un serveur web nginx
- un rôle pour installer PHP
- un rôle pour installer et configurer une instance de WordPress
- un playbook pour orchestrer ces quatre rôles

Voyez cela comme une mise en situation : un client vous demande d'automatiser l'installation d'un WordPress avec une base de données déportée. À vous donc d'écrire les rôles et le playbook susmentionnés.

Un ingénieur Dev(Sec)Ops doit souvent composer avec de multiples logiciels, dont certains lui seront imposés et initialement inconnus.

### Suggestions

Avant de vous lancer dans l'écriture de vos rôles, considérez les points suivants :
- Utilisez les modules `community.mysql` pour interagir avec votre instance de MySQL
- Lors de la configuration de WordPress, il faut configurer des `salts` dans le fichier `config.php`

### Erreurs courantes

Une erreur courante que vous êtes susceptibles de rencontrer est `A MySQL module is required:`. Celle-ci est liée à une librairie Python manquante lorsqu'Ansible tente d'utiliser les modules MySQL. Vous pouvez retrouver la version minimale dans la partie `requirements` de la [documentation officielle](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_info_module.html#requirements). Pour l'installer, une solution simple est d'ajouter le package `python3-pymysql`.
