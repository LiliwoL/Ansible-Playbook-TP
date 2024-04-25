# TP - Ansible pour la provision de serveurs Web

<!-- TOC -->
* [TP - Ansible pour la provision de serveurs Web](#tp---ansible-pour-la-provision-de-serveurs-web)
* [Contexte](#contexte)
* [Mission 1: Préparation des machines](#mission-1-préparation-des-machines)
* [Mission 2: Rédaction du fichier des hôtes sur le contrôleur](#mission-2-rédaction-du-fichier-des-hôtes-sur-le-contrôleur)
* [Mission 3: Rédaction du playbook](#mission-3-rédaction-du-playbook)
  * [Contenu du playbook](#contenu-du-playbook)
    * [playbook.yml](#playbookyml)
    * [Fichier de variables /vars/default.yml](#fichier-de-variables-varsdefaultyml)
    * [Fichiers de template](#fichiers-de-template)
* [Lancer la commande](#lancer-la-commande)
  * [Vérification](#vérification)
<!-- TOC -->
v0.1

---
<div style="page-break-after: always;"></div>

# Contexte

On veut mettre à disposition de chaque étudiant de BTS SISR2 une machine hébergeant un serveur Web avec Wordpress installé.

Cette machine hébergera le **portfolio** de l'étudiant.

---
<div style="page-break-after: always;"></div>

# Mission 1: Préparation des machines

Vous devez créer (à minima) ces machines basées sur Debian 12.5:

- Une machine **Contrôleur Ansible**
- Une machine **Hôte Ansible** nommée *Web-Etudiant-1*
- Une machine **Hôte Ansible** nommée *Web-Etudiant-2*
- Une machine **Hôte Ansible** nommée *Web-Etudiant-3*

![8a390e8258f4e01baf525cdfaea5aaf2.png](:/da3e7068f4954ace9a8903e94d0709b6)

Vous devez vous baser sur le fichier **ova** situé dans *\\172.16.0.6\doc\MachinesVirtuelles\Linux\Debian-12.5-base.ova*.

Cette machine virtuelle dispose déjà de **ssh** installé et du compte **root:azertysio**.

> ---
> ✍️**Travail à faire**
> - Définissez un adressage IP pour ce TP.
> - Réaliser un schéma réseau reprenant les différentes machines et affichant toutes les informations nécessaires (Noms des machines, Adresses IP, Réseau utilisé).
> - Créez les machines virtuelles, configurez le nom et l'adresse IP.
> - Vérifiez la connexion entre les machines
> ---
---
<div style="page-break-after: always;"></div>

# Mission 2: Rédaction du fichier des hôtes sur le contrôleur

> ---
> ✍️**Travail à faire**
> - A partir de votre adressage IP et de vos noms de machines, rédiger le fichier **hosts**.
> - Vérifiez la connexion du **contrôleur** avec ses **hôtes**.
> ---
---
<div style="page-break-after: always;"></div>

# Mission 3: Rédaction du playbook

L’exécution de ce **playbook** exécutera les actions suivantes sur vos hôtes Ansible :

1. Mise à jour du système
2. Installation des packages LAMP et des extensions PHP requises.
3. Création et validation d’un nouveau VirtualHost Apache pour le site WordPress.
4. Activation du module de réécriture Apache (mod_rewrite).
5. Désactivation du site web Apache par défaut.
6. Définit le mot de passe pour l’utilisateur root de MySQL.
7. Suppression des comptes MySQL anonymes et de la base de données test.
8. Création d’une nouvelle base de données MySQL et d’un utilisateur pour le site WordPress.
9. Télécharger et décompacter WordPress.
10. Définir la propriété et les autorisations correctes du répertoire.
11. Définir le fichier wp-config.php en utilisant le modèle fourni.

---
> **⚠️** Pour continuer, vous devez récupérer les fichiers du **playbook** sur https://github.com/LiliwoL/Ansible-Playbook-TP

Sur le contrôleur Ansible, vous allez devoir créer l'arboresence suivante (en utilisant les fichiers du dépôt Git):
```
playbook-wordpress
├── files
│   ├── apache.conf.j2
│   └── wp-config.php.j2
├── vars
│   └── default.yml
├── playbook.yml
└── README.md
└── .gitignore
```
---
## Contenu du playbook

### playbook.yml

> 📂 https://github.com/LiliwoL/Ansible-Playbook-TP/blob/main/playbook.yml

Ce fichier contient toutes les **actions** qui seront à exécuter sur l'**hôte**.
On y retrouve étape par étape les points de la liste ci-dessus.

D'abord, on définit le groupe de serveurs qui recevront ce playbook (ici c'est **all**).
On demande à ce que le script soit exécuté en **sudo** avec la directive **become: true**.

Ensuite, on **charge le fichier de variables** qui contient les options de configuration.

Notez les modules ansibles utilisés.
- shell
- mysql_user
- mysql_db
- unarchive
- file
- template

> Pour les modules **mysql (ou mariadb)**
> [mysql_db](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_db_module.html#ansible-collections-community-mysql-mysql-db-module) – Gestion des bases de données
> [mysql_info](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_info_module.html#ansible-collections-community-mysql-mysql-info-module)  – Récupère les informations des serveurs
> [mysql_query](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_query_module.html#ansible-collections-community-mysql-mysql-query-module) – Utilisé pour exécuter des requêtes
> [mysql_replication](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_replication_module.html#ansible-collections-community-mysql-mysql-replication-module) – Pour gérer la réplication
> [mysql_role](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_role_module.html#ansible-collections-community-mysql-mysql-role-module) – Pour gérer les rôles MySQL
> [mysql_user](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_user_module.html#ansible-collections-community-mysql-mysql-user-module) – Pour ajouter ou supprimer des utilisateurs à une base de données
> [mysql_variables](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_variables_module.html#ansible-collections-community-mysql-mysql-variables-module) – Gérer les variables MySQL

---
### Fichier de variables /vars/default.yml

> 📂https://github.com/LiliwoL/Ansible-Playbook-TP/blob/main/vars/default.yml

Ce fichier contient les valeurs qui seront utilisées dans le **playbook**, comme les identifiants de la base de données, le domaine d'Apache, etc.

---
### Fichiers de template

> 📂https://github.com/LiliwoL/Ansible-Playbook-TP/blob/main/files/apache.conf.j2
> 📂https://github.com/LiliwoL/Ansible-Playbook-TP/blob/main/files/wp-config.php.j2

Ces fichiers sont des **templates** contenant la configuration pour les **hôtes virtuels** Apache2 et Wordpress.

Ces **templates** utilisent des valeurs définies dans le fichier *vars/default.yml*.

Le fichier *apache.conf.j2* est un template au format **Jinja 2**. Il configure un **hôte virtuel Apache**.

Le fichier *wp-config.php.j2* est utilisé pour définir la configuration globale de wWordpress.

---
<div style="page-break-after: always;"></div>

# Lancer la commande

Sur le contrôleur:

`ansible-playbook playbook.yml -u root --ask-pass`

> --ask-pass est nécessaire car les clés SSH ne sont pas encore gérées

---
## Vérification

Si tout s'est bien déroulé, dans votre navigateur, tapez l'adresse d'un hôte:

`http://server_host_or_IP`

Vous devriez voir ceci: *(preuve que Wordpress s'est bien installé)*

![2627894c34c77f9fcf5d2a12c9db768e.png](:/41e32fec213146d399cca114546a6be8)

Choisisssez une langue, Wordpress va vous demander un **utilisateur** et un **mot de passe** pour l'interface d'admnistration.

![55ee8bd7ab4926911a37b3ff866c14ac.png](:/852853f255484d55bcf33085910d78df)

![4607872ac54b04d5a2dd0cae8561b148.png](:/1f727eea6fc3427594842aba6cedc894)

Voilà! Votre Wordpress s'est bien installé!