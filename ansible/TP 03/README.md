# TP n°03 : installer un service

## Introduction

Ce TP a pour but d'introduire les variables spéciales (facts et magic variables), le debug, les templates, les lookups, les conditionnelles et les blocks.

## Prérequis

Dans ce TP, c'est à vous de fournir l'inventaire. Si vous avez suivi le TP n°02, l'inventaire se trouve dans le dossier de ce TP. Vous avez alors deux choix :
- Dupliquer l'inventaire en le copiant dans le répertoire du TP actuel, et supprimer les variables désormais inutiles
- Déplacer l'inventaire dans un dossier commun, et conserver les variables existantes

> En fonction des cas d'usage, les deux solutions sont acceptables. Dans le cas de déploiements de taille modérée, on peut privilégier d'avoir un seul inventaire pour l'ensemble des déploiements afin de limiter le risque d'actions concurrentes, même si cela implique d'avoir des variables "inutiles" pour un des déploiements. A contrario, sur des déploiements plus importants, avoir un seul inventaire par déploiement permet de ne pas surcharger les inventaires et de faciliter la lecture.

Dans les deux cas, il faudra bien entendu rajouter les nouvelles variables attendues par le playbook, et adapter le chemin de l'inventaire dans la commande `ansible-playbook` en conséquence.

## Déroulé

### Compléter le playbook

Le but de ce TP est d'installer un Wordpress sur un premier serveur, avec une base de données déportée sur un autre seveur. Le playbook va donc devoir effectuer des actions différentes sur les deux machines.

On va donc retrouver plusieurs parties distinctes dans le playbook (des `plays`), chacune avec son nom et sa cible (via le mot-clé `hosts`) : une première partie (`hosts: all`) adresse l'ensemble des machines pour les actions communes, et les deux autres ne vont s'exécuter respectivement que sur les machines des groupes `backend` et `frontend`.

> L'ordre a une importance ici : on veut que le backend (la base de données) soit opérationnel avant de faire l'ensemble des actions sur le frontend. Cela évitera de multiplier les `plays` qui cibleront alternativement un groupe puis l'autre.

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
