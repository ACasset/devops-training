# TP n°02 : créer un utilisateur

## Introduction

Ce TP a pour but d'introduire l'utilisation des variables, des fichiers et d'Ansible Vault.

## Prérequis

Pour ce TP, vous allez avoir besoin :
- d'un contrôleur avec Ansible
- d'un hôte distant sur lequel la clé SSH du contrôleur a été copiée
- d'un inventaire configuré pour accéder à l'hôte distant

> Si vous avez déroulé le TP n°01, vous devriez déjà avoir ces éléments à disposition.

## Déroulé

### Compléter l'inventaire

Une des grandes forces d'Ansible est de pouvoir adresser plusieurs machines en parallèle, tout en utilisant des valeurs spécifiques à chacune. Comme dans le TP n°01, un inventaire est fourni dans le dossier, mais celui-ci est structuré différemment : lisez les quelques commentaires pour mieux comprendre.

> N'hésitez pas à changer la méthode de déclaration des hôtes si vous le voulez. Comme évoqué dans le TP n°1, il n'y a pas de mauvaise méthode.

Cet inventaire comporte quelques trous. Vous savez en principe déjà comment compléter les variables `ansible_user` à la ligne 8, et les FQDN/IPs aux lignes 15 et 22.

Les variables `nom_utilisateur` aux lignes 17 et 24 sont pré-remplies, mais vous pouvez les modifier à votre convenance. Gardez juste à l'esprit que l'on va créer des utilisateurs Unix avec, donc évitez les caractères spéciaux.

Enfin, les variables `mot_de_passe_utilisateur` aux lignes 18 et 25 sont un peu plus délicates : il nous faut absolument éviter de mettre en clair des éléments sensibles tels qu'un mot de passe dans un fichier YAML, car celui-ci va être typiquement versionné et publiquement accessible sur un dépôt de code.

Heureusement, Ansible propose un outil adapté à ce cas de figure : `ansible-vault`. Pour s'en servir, on va pouvoir utiliser la commande suivante :
```bash
ansible-vault encrypt_string
New Vault password: # renseignez le mot de passe de déchiffrement du secret, puis appuyez sur entrée
Confirm New Vault password: # confirmez le mot de passe de déchiffrement du secret, puis appuyez sur entrée
Reading plaintext input from stdin. (ctrl-d to end input)
# renseignez la valeur que vous voulez chiffrer, puis appuyez sur Ctrl et D en même temps
```

`ansible-vault` va chiffrer la valeur avec le mot de passe, et vous restituer un bloc de texte de la forme suivante :
```yaml
!vault |
$ANSIBLE_VAULT;1.1;AES256
38633264363430373862326661383536643532313338303635353565663934393261653363373365
6438393738316466386562383963663866373531343730630a393034373363316639653839663036
37663133363234626235613733623765373662363534313864333761343435363431633334626137
3364633065383265370a333832373966383363353733653738326561663766643330373164653862
3066
```

Ce bloc est à insérer dans l'inventaire, tout en respectant l'indentation du YAML. Voici un exemple :
```yaml
groupe:
  hosts:
    hote:
      variable: !vault |
        $ANSIBLE_VAULT;1.1;AES256
        38633264363430373862326661383536643532313338303635353565663934393261653363373365
        6438393738316466386562383963663866373531343730630a393034373363316639653839663036
        37663133363234626235613733623765373662363534313864333761343435363431633334626137
        3364633065383265370a333832373966383363353733653738326561663766643330373164653862
        3066
```

> Le caractère `|` sert à indiquer que la valeur de la variable s'étale sur plusieurs lignes. Vous pouvez en apprendre plus sur le site [YAML multiline](https://yaml-multiline.info/)

`ansible-vault` possède d'autres capacités que `encrypt_string`. Vous pouvez en apprendre plus sur [le site officiel](https://docs.ansible.com/ansible/latest/vault_guide/index.html).

> Attention, si vous vous servez d'Ansible Vault pour chiffrer un fichier (avec la commande `ansible-vault encrypt`), l'ensemble du fichier est contenu dans le bloc de texte restitué, y compris les retours à la ligne en fin de fichier. Cela peut poser des problèmes dans certains cas. Si vous rencontrez des difficultés avec des valeurs chiffrées, essayez de les re-chiffrer avec `encrypt_string`.

Vous allez maintenant pouvoir renseigner la valeur de la variable `mot_de_passe_utilisateur` avec la valeur chiffrée par `ansible-vault`.

Ce TP également utilise un nouveau mot clé `become` dans son playbook. Ce mot clé sert à effectuer une escalade de privilèges temporaire pour effectuer des actions avec les droits administrateur sur les hôtes. En fonction de votre configuration, il va peut-être falloir compléter votre inventaire. Connectez-vous à votre hôte en SSH et lancez la commande `sudo echo` :
- Si une ligne vide apparaît, tout va bien.
- En revanche, si le système vous demande un mot de passe, vous avez deux solutions :
  - Si vous êtes à l'aise avec Unix (d'un point de vue technique et d'un point de vue sécurité), vous pouvez rajouter l'instruction `NOPASSWD` dans le fichier des `sudoers`.
  - Sinon, vous pouvez modifier votre inventaire pour rajouter la variable `ansible_become_password` au même niveau que la variable `ansible_user`. Ne renseignez pas le mot de passe tout de suite, nous allons voir comment faire proprement un peu plus loin.

> Si vous avez dû rajouter la variable `ansible_become_password` dans votre inventaire, vous pouvez utiliser à nouveau `ansible-vault` pour chiffrer le mot de passe pour l'escalade des privilèges et compléter votre inventaire. Utilisez la même clé de chiffrement que les autres secrets.

### Lancer l'exécution

Comme à chaque fois que l'on veut exécuter un playbook, on lance la commande `ansible-playbook`, mais comme certaines variables sont chiffrées, il faut rajouter un argument pour spécifier la clé de déchiffrement :
```bash
ansible-playbook --inventory ../inventory.yml --ask-vault-pass playbook.yml
Vault password: # renseignez le mot de passe de déchiffrement du secret, puis appuyez sur entrée
```

Une fois l'exécution terminée, connectez-vous sur un des deux hôtes avec l'utilisateur nouvellement créé (vérifiez dans l'inventaire quel utilisateur est associé à quel hôte).
Si vous faites un `ls` pour constater la présence du fichier `fichier_statique`, vous devriez vous rendre compte que le fichier appartient à l'utilisateur et au groupe `root`. C'est l'escalade de privilège qui provoque cela, et c'est un mécanisme qu'il faudra garder à l'esprit lorsqu'on utilise le `become`.

### Compléter le playbook

Pour corriger cela, il va falloir retourner consulter la documentation du module `ansible.builtin.copy`, et ajouter à minima le paramètre `owner` dans le playbook, et éventuellement `group` et `mode`. Utilisez à nouveau la variable `nom_utilisateur` pour que la valeur soit adaptée à l'hôte sur lequel le playbook s'exécutera.

> Faites attention au format du paramètre `mode`, celui-ci est au format octal et est donc un peu particulier.

### Relancer l'exécution

Une fois ces paramètres rajoutés, vous pouvez relancer l'exécution du playbook. Si vous avez bien rajouté les paramètres, vous pourrez constater que la copie du fichier est bien en état `changed`, et aller ensuite vérifier que la modification est correcte dans le système de fichiers des deux hôtes.
