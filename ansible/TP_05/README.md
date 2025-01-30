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

L'objectif est, dans un premier temps, d'écrire 4 rôles :
- un rôle pour installer une base de données MySQL
- un rôle pour installer un serveur web nginx
- un rôle pour installer PHP
- un rôle pour installer et configurer une instance de WordPress

Ensuite, à vous d'écrire un playbook pour séquencer ces trois rôles et installer une instance de WordPress avec une base de données déportée.

### Écrire les rôles et le playbook

Voyez cela comme une mise en situation : un client vous demande d'automatiser l'installation d'un WordPress avec une base de données déportée. À vous donc d'écrire les rôles et le playbook susmentionnés.

Un ingénieur Dev(Sec)Ops doit souvent composer avec de multiples logiciels, dont certains lui seront imposés et initialement inconnus.

> <TODO: utiliser les modules community.mysql>

<TODO: call à l'API WordPress pour les salts>

### Compléter le template

Enfin, parmi les tasks déjà présentes dans le playbook, vous verrez vers la fin un appel au module `ansible.builtin.template`. Celui-ci va lire le fichier qu'on lui indique (`wp-config.php.j2` dans notre cas), et remplacer des valeurs dedans avant de le copier sur la machine cible.

Contrairement à Ansible qui utilise l'extension `.yml` et le format YAML, les templates utilisent l'extension `.j2` et le format Jinja2. Vous l'avez en réalité déjà rencontré dans le TP précédent lors de l'utilisation des variables. N'hésitez pas à vous référer à la [documentation officielle](https://jinja.palletsprojects.com/en/latest/templates/) pour en apprendre plus sur les capacités de l'outil.

Quelques unes des valeurs sont déjà renseignées, notamment toute une partie concernant les salts, mais vous allez pouvoir référencer les variables `utilisateur_bdd`, `mot_de_passe_bdd` et `nom_bdd` aux endroits adéquats. Notez l'utilisation de la variable `groups[backend][0]` qui permet de référencer un élément de l'inventaire, dans notre cas le nom de la première (et seule) machine du groupe `backend`. Cela permet au fichier de configuration de pointer dynamiquement sur la bonne machine en modifiant simplement l'inventaire. C'est ce qu'on appelle une `magic variable`. Il en existe un certain nombre que vous pourrez retrouver dans la [documentation officielle](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#magic-variables).

> Contrairement à l'utilisation de variables dans le YAML, pas besoin de rajouter de guillemets dans un contexte Jinja2 pour que les caractères `{{ }}` soient correctement interprétés.

### Jouer le playbook

Une fois tout cela configuré, vous allez pouvoir jouer le playbook. Ce TP représente une grande marche par rapport aux précédents, et il est tout à fait normal qu'il ne fonctionne pas du premier coup.

> Une erreur courante que vous êtes susceptibles de rencontrer est `A MySQL module is required:`. Celle-ci est liée à une librairie Python manquante lorsqu'Ansible tente d'utiliser les modules MySQL. Vous pouvez retrouver la version minimale dans la partie `requirements` de la [documentation officielle](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_info_module.html#requirements). Pour l'installer, une solution simple est d'ajouter le package `python3-pymysql`

Même dans le cas où votre playbook se termine sans erreur, cela ne veut pas dire que l'exécution va bien se passer. Selon la façon dont vous avez déclaré vos hôtes, vous êtes notamment susceptibles de rencontrer une erreur concernant la base de données, dont le nom n'est pas résolvable. TODO: finir l'explication (= référence à `groups.` dépendante de la façon dont l'inventaire est déclaré ===> hostvars[groups['backend'][0]]['ansible_host'] )

### Rendre le playbook idempotent

Si on exécute le playbook à nouveau, on verra plusieurs tasks, dont `Copie du fichier de configuration`, sont en état `changed`. Cela s'explique par l'appel à l'API WordPress, qui génère de nouveaux salts à chaque fois, et le fichier `wp-config.php` est donc remplacé. Dans le cas de WordPress, ce n'est pas critique, car cela invalide juste les sessions (ce qui peut déjà être pénible), mais cela peut le devenir dans d'autres cas (par exemple avec la clé de chiffrement d'une base de données, dont le changement dans la configuration empêcherait tout accès).

On se retrouve donc dans un cas de figure où les actions effectuées ne permettent pas une idempotence native avec les modules Ansible. Pour atteindre cette idempotence, on va donc devoir effectuer des vérifications supplémentaires, et conditionner l'exécution des tasks au résultat de ces vérifications.

Dans notre cas, il va falloir rechercher les signes que le playbook a déjà été joué. On peut envisager plusieurs approches :
- La présence des fichiers de WordPress dans le système de fichiers, mais est-on sûr que le playbook s'est déroulé jusqu'au bout et que WordPress est fonctionnel ?
- L'ajout d'une task supplémentaire à la fin du playbook, pour créer un fichier indiquant que 

Une fois cette vérification mise en place, on va pouvoir utiliser le mot-clé `when` sur les tasks concernées comme suit :
```yaml

```

On va pouvoir inclure toutes les tâches dans un `block` comme suit, afin de n'avoir la condition qu'une seule fois :
```yaml

```

## Conclusion


