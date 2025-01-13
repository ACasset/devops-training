# TP n°03 : installer un service

## Introduction

Ce TP a pour but d'introduire les magic variables, le debug, les templates, les lookups, les conditionnelles et les blocks.

## Prérequis

Dans ce TP, c'est à vous de fournir l'inventaire. Si vous avez suivi le TP n°02, l'inventaire se trouve dans le dossier de ce TP. Vous avez alors deux choix :
- Dupliquer l'inventaire en le copiant dans le répertoire du TP actuel, et supprimer les variables désormais inutiles
- Déplacer l'inventaire dans un dossier commun, et conserver les variables existantes

> En fonction des cas d'usage, les deux solutions sont acceptables. Dans le cas de déploiements de taille modérée, on peut privilégier d'avoir un seul inventaire pour l'ensemble des déploiements afin de limiter le risque d'actions concurrentes, même si cela implique d'avoir des variables "inutiles" pour un des déploiements. A contrario, sur des déploiements plus importants, avoir un seul inventaire par déploiement permet de ne pas surcharger les inventaires et de faciliter la lecture.

Dans les deux cas, il faudra bien entendu rajouter les nouvelles variables attendues par le playbook, et adapter le chemin de l'inventaire dans la commande `ansible-playbook` en conséquence.

## Déroulé

### Compléter le playbook

Le but de ce TP est d'installer un Wordpress sur un premier serveur, avec une base de données déportée sur un autre seveur. Le playbook va donc devoir effectuer des actions différentes sur les deux machines.

On va donc retrouver plusieurs parties distinctes dans le playbook (des `plays`), chacune avec son nom et sa cible (via le mot-clé `hosts`) : une première partie (`hosts: backend`) adresse les machines du groupe du même nom pour y installer la base de données, et une seconde partie (`hosts: frontend`) va procéder à l'installation du serveur web, de PHP et de WordPress. Si on voulait effectuer des actions communes à tous les hosts (comme la sécurisation des machines par exemple), on pourrait rajouter une partie en conséquence (`hosts: all`) tout en haut du playbook.

> L'ordre a une importance ici : on veut que le backend (la base de données) soit opérationnel avant de faire l'ensemble des actions sur le frontend. Cela évitera de multiplier les `plays` qui cibleraient alternativement un groupe puis l'autre.

Il va donc falloir faire plusieurs choix pour ce TP :
- Pour le backend : la base de données à installer (MySQL ou MariaDB)
- Pour le frontend :
  - Le serveur web à installer (Nginx ou Apache)
  - la version de PHP à installer (parmi les [versions actuellement supportées](https://www.php.net/supported-versions.php))

Un ingénieur Dev(Sec)Ops doit souvent composer avec de multiples logiciels, dont certains lui seront imposés et inconnus au début. Dans notre cas, choisissez les logiciels avec lesquels vous êtes le plus à l'aise pour vous faciliter la tâche.

Des tasks sont déjà présentes dans le playbook pour ces installations, mais il va vous falloir les personnaliser avec le nom des paquets, et éventuellement rajouter d'autres tasks pour procéder à des étapes de configuration. En fonction de vos choix, vous pourriez avoir besoin de la collection [`community.mysql`](https://galaxy.ansible.com/ui/repo/published/community/mysql/docs/).

### Compléter l'inventaire

Vous l'avez probablement déjà vu dans le playbook, plusieurs variables sont référencées, et il va falloir les rajouter dans l'inventaire : `utilisateur_bdd`, `mot_de_passe_bdd` et `nom_bdd` pour le groupe `all` (car le backend va en avoir besoin pour configurer la base de données, et le frontend va en avoir besoin pour y accéder), et `nom_dossier_wordpress` pour le groupe `frontend`. La valeur de cette dernière n'a pas beaucoup d'importance, elle est surtout là pour permettre de la modularité.

### Compléter le template

Enfin, parmi les tasks déjà présentes dans le playbook, vous verrez vers la fin un appel au module `ansible.builtin.template`. Celui-ci va lire le fichier qu'on lui indique (`wp-config.php.j2` dans notre cas), et remplacer des valeurs dedans avant de le copier sur la machine cible.

Contrairement à Ansible qui utilise l'extension `.yml` et le format YAML, les templates utilisent l'extension `.j2` et le format Jinja2. Vous l'avez en réalité déjà rencontré dans le TP précédent lors de l'utilisation des variables. N'hésitez pas à vous référer à la [documentation officielle](https://jinja.palletsprojects.com/en/latest/templates/) pour en apprendre plus sur les capacités de l'outil.

Quelques unes des valeurs sont déjà renseignées, notamment toute une partie concernant les salts, mais vous allez pouvoir référencer les variables `utilisateur_bdd`, `mot_de_passe_bdd` et `nom_bdd` aux endroits adéquats. Notez l'utilisation de la variable `groups[backend][0]` qui permet de référencer un élément de l'inventaire, dans notre cas le nom de la première (et seule) machine du groupe `backend`. Cela permet au fichier de configuration de pointer dynamiquement sur la bonne machine en modifiant simplement l'inventaire. C'est ce qu'on appelle une `magic variable`. Il en existe un certain nombre que vous pourrez retrouver dans la [documentation officielle](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#magic-variables).

> Contrairement à l'utilisation de variables dans le YAML, pas besoin de rajouter de guillemets dans un contexte Jinja2 pour que les caractères `{{` et `}}` soient correctement interprétés.

### Lancer le playbook

Une fois tout cela configuré, vous allez pouvoir lancer le playbook. Ce TP représente une grande marche par rapport aux précédents, et il est tout à fait normal qu'il ne fonctionne pas du premier coup.

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
