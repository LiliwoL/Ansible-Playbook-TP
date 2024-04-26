# TP - Ansible générer et pousser des clés SSH

<!-- TOC -->
* [TP - Ansible générer et pousser des clés SSH](#tp---ansible-générer-et-pousser-des-clés-ssh)
* [Objectif](#objectif)
* [Rappel du contexte:](#rappel-du-contexte)
* [Déroulé](#déroulé)
* [Mission 1: Créer la clé SSH](#mission-1-créer-la-clé-ssh)
* [Mission 2: Déployer la clé sur les hôtes](#mission-2-déployer-la-clé-sur-les-hôtes)
* [Mission 3: Sécuriser les hôtes en retirant la connexion SSH par mot de passe](#mission-3-sécuriser-les-hôtes-en-retirant-la-connexion-ssh-par-mot-de-passe)
  * [Rappels: Connexion en SSH](#rappels-connexion-en-ssh)
    * [Connexion à un port spécifique (ici le port 22)](#connexion-à-un-port-spécifique-ici-le-port-22)
    * [Connexion en fournissant une clé privée (la clé publique étant déjà sur le serveur qu'on cherche à joindre)](#connexion-en-fournissant-une-clé-privée-la-clé-publique-étant-déjà-sur-le-serveur-quon-cherche-à-joindre)
* [Sources](#sources)
<!-- TOC -->

v0.1

---
<div style="page-break-after: always;"></div>

# Objectif

On souhaite gérer nos hôtes depuis notre contrôleur sans avoir à saisir de mot de passe.
Pour cela, on va utiliser le mécanisme des **clés SSH**.

---
<div style="page-break-after: always;"></div>

# Rappel du contexte:

- Une machine **Contrôleur Ansible**
- Une machine **Hôte Ansible** nommée _Web-Etudiant-1_
- Une machine **Hôte Ansible** nommée _Web-Etudiant-2_
- Une machine **Hôte Ansible** nommée _Web-Etudiant-3_

![8a390e8258f4e01baf525cdfaea5aaf2.png](:/da3e7068f4954ace9a8903e94d0709b6)

---
<div style="page-break-after: always;"></div>

# Déroulé

- Créer une clef SSH sur le contrôleur et la pousser sur les hôtes
- Reboot (après upgrade par exemple) et la reprise des tâches où on en était resté

---
<div style="page-break-after: always;"></div>

# Mission 1: Créer la clé SSH

Deux tâches à réaliser:

- **openssh_keypair** : pour créer une clef SSH avec différents paramètres
- **authorized_key** : pour déployer la clef publique dans le fichier **authorized_keys** de vos utilisateurs

> ⚠️ Créer une clé SSH c’est bien mais en créer une différente sur chaque machine ne sera pas forcément une grande idée. Certes, niveau sécurité vous auriez une infrastructure un peu plus robuste, mais la gestion des clefs risque vite de déborder et devenir insoutenable.

Playbook avec une tâche pour générer la clé sur le **contrôleur**:
```
#########################################################
# Playbook: SSH Keys generation and deployment / SSH configuration
#########################################################
---
- name: Playbook SSH generation and deployment
  hosts: all
  become: true
  
		# ========================================
    # Génération de la clé SSH en local (sur le contrôleur)
    # La clé sera générée dans /tmp/sio et /tmp/sio.pub
    # ========================================
    - name: Generate SSH key
      openssh_keypair:
        path: /tmp/sio
        type: rsa
        size: 4096
        state: present
        force: no
      run_once: yes
      delegate_to: localhost
```
> ⚠️ On pourrait ajouter une **passhrase** pour plus de sécurité

Nous utilisons donc le module **openssh_keypair** en lui demandant de créer un jeu de clés nommé **sio** de type **rsa** et de longueur **4096**.

Et pour **faire cela sur le contrôleur ansible** nous lui ajoutons le paramètre **delegate_to** en mentionnant **localhost** (run local).

Bien sûr ansible souhaite par défaut **faire tourner cette tâche pour toutes les machines du groupe all** tel que mentionné dans **hosts**. C’est pourquoi avec un **run_once** à **yes**, nous allons limiter ce run à **une seule itération** c’est à dire pour la première machine du groupe all.

> ---
> ✍️**Travail à faire**
> - Exécuter ce **playbook** sur le contrôleur, et assurez vous que la clé **SSH** a bien été générée.
> ---

---
<div style="page-break-after: always;"></div>

# Mission 2: Déployer la clé sur les hôtes

Pour le déploiement de la clé publique dans le **authorized_key** de l'**hôte**, là encore ansible fournit une module bien pratique.

> **⚠️** Module **authorized_key** https://docs.ansible.com/ansible/latest/collections/ansible/posix/authorized_key_module.html
  ```
		# ========================================
    # Déploiement de la clé SSH sur les hôtes pour le compte sio
    # ========================================
    - name: Deploy SSH Key
      authorized_key:
        user: sio
        key: "{{ lookup('file', '/tmp/sio.pub') }}"
        state: present
```
> On ajoutera éventuellement un **become: true** pour que notre **user** avec lequel on se connecte à l'hôte puisse **avoir suffisament de droits** pour aller dans les répertoires **home** de tous les utilisateurs.
> **Attention, il aura fallu au préalable installer le paquet sudo sur l'hôte pour faire un become: yes**

Ensuite on va **rechercher le contenu de notre clé publique localement** (contrôleur ansible) avec un **lookup** de type file.

On indique enfin pour cible le user nommé **sio** (sur l'hôte).

> ---
> ✍️**Travail à faire**
> - Exécuter ce **playbook** sur le contrôleur, et assurez vous que la clé **SSH** a bien été envoyée sur l'hôte.
    > **⚠️** Pour le moment, **la connexion par clé SSH n'est pas activée** sur le serveur!
> ---

---
<div style="page-break-after: always;"></div>

# Mission 3: Sécuriser les hôtes en retirant la connexion SSH par mot de passe

On va ensuite **modifier** la configuration du serveur SSH de l'hôte en lui **envoyant** un fichier de configuration qui va:

- Modifier le port d'écoute du serveur SSH
- Interdire la connexion du compte **root**
- Interdire la connexion SSH par mot de passe
- Autoriser la connexion SSH par **clé**

- Redémarrer le serveur SSH

> 📂 Fichiers à retrouver ici: https://github.com/LiliwoL/Ansible-Playbook-TP/tree/main/03.%20TP%20-%20Ansible%20avec%20les%20clés%20SSH

Vous devrez reproduire l'arborescence:
```
playbook-ssh-server
├── files
│   ├── ssh-setup.j2
├── vars
│   └── default.yml
├── playbook-ssh-setup.yml
└── README.md
```

> ---
> ✍️**Travail à faire**
> - Exécuter ce **playbook** sur le contrôleur, et assurez vous que la configuration du serveur est bien en place.
> - Testez une connexion **SSH** avec l'utilisateur **sio** et la clé générée sur le **contrôleur**.
> ---

## Rappels: Connexion en SSH

### Connexion à un port spécifique (ici le port 22)

`ssh user@192.168.1.31 -p 2222`

### Connexion en fournissant une clé privée (la clé publique étant déjà sur le serveur qu'on cherche à joindre)

`ssh user@192.168.1.31 -p 2222 -i /tmp/sio`

> **/tmp/sio** est le fichier de clé privée


---
<div style="page-break-after: always;"></div>

# Sources

https://xavki.blog/ansible-comment-generer-des-clefs-ssh-et-les-pousser-organiser-un-reboot/
https://www.cyberciti.biz/faq/how-to-upload-ssh-public-key-to-as-authorized_key-using-ansible/
https://www.unix-experience.fr/ansible/manage_ssh_key/