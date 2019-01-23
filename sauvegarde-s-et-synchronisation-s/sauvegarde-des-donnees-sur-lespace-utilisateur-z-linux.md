# Sauvegarde des données sur l'espace utilisateur Z: \(Linux\)

## Pré-Requis

* [Introduction](../) 
* [Espace de travail](../environnement-de-travail/comment-fonctionnent-les-espaces-x-et-z.md) dans le datacenter de l'université

## Installation

### Borg backup 💾 

Nous allons utiliser l'outil python **Borg**, fork d'**Attic** pour la sauvegarde de vos données. Il s'agit d'un outil de sauvegarde performant, régulièrement mis à jour, et très bien documenté. Pour l'installer sous Linux, vous pouvez soit :

*  l'installer via le gestionnaire de paquet de votre système : **apt-get, yum, etc.** Le problème de cette méthode c'est que les paquets ne sont pas toujours les plus à jours, surtout si vous êtes sous une ancienne version de la distribution.

```bash
# Avec Ubuntu
apt-get install borgbackup
```

* l'installer via le gestionnaire de paquet python **pip** en spécifiant le dernier [numéro de version](https://github.com/borgbackup/borg/release).

```bash
pip install "borgbackup==1.1.8"
```

### Configuration du point de montage sous Linux

Nous devons d'abord créer un point de montage qui servira à relier un dossier de votre ordinateur avec votre dossier personnel sur le datacenter. Le nom de l'utilisateur sur l'ordinateur est _tylor_, vous pouvez connaître le votre en tapant la commande `whoami` dans votre terminal.

```bash
sudo mkdir -p /media/tylor/datacenter
```

La configuration du point de montage sous linux se fait en modifiant le fichier `/etc/fstab`en mode super-utilisateur `sudo` : 

```bash
sudo gedit /etc/fstab &
```

Vous pouvez ajouter la ligne suivante à ce fichier, en faisant attention de modifier les éléments suivants par vos propres identifiants : 

```text
//ur.univ-rouen.fr/urdatas/ /media/tylor/datacenter cifs user,noauto,user=dupretyl,password=42spinaxis,uid=1000 0 0
```

## Script d'automatisation des sauvegardes

### Pré-requis pour le chiffrement du dépôt des sauvegardes

Nous avons choisi de chiffrer nos données avec un mot de passe. Comme nous voulons une sauvegarde qui puisse s’exécuter en tache de fond, nous ne voulons pas avoir à saisir ce mot de passe à chaque opération... Sur ce point, [la documentation de **Borg**](https://borgbackup.readthedocs.io/en/stable/faq.html#how-can-i-specify-the-encryption-passphrase-programmatically) recommande d'utiliser un fichier \(_borg-passphrase_\) contenant le mot de passe \(_42spinAxis_\), configuré avec des permissions de lecture/écriture restrictives : 

```bash
echo "42spinAxis" >> ~/.borg-passphrase
chmod 400 ~/.borg-passphrase
```

### Initialisation du dépôt des sauvegardes

```bash
borg init --encryption=[TYPECHIFFREMENT] [DESTINATION]
```

La commande borg init \(voir [documentation](https://borgbackup.readthedocs.io/en/stable/usage/init.html)\) doit prendre un type de chiffrement et une destination \(**repository**\) pour initialiser le répertoire de dépot des différentes sauvegardes. 

Pour la configuration du **\[TYPECHIFFREMENT\]**, plusieurs options sont disponibles : 

1. aucune chiffrement : `--encryption=none`
2. sans chiffrement mais avec authentification : `--encryption=authenticated-blake2`
3. avec mot de passe et clef : `--encryption=keyfile-blake2`

{% hint style="warning" %}
Le mode sans chiffrement est très fortement déconseillé, surtout en cas de stockage de données sensibles !
{% endhint %}

La **\[DESTINATION\]** correspond à notre point de montage dans `/media/tylor/datacenter/personnels/dupretyl/BACKUP`

Ce qui donne, avec le mode chiffrement choisi \(2.\), la commande suivante, que l'on ajoute à la suite de notre script de backup : 

```bash
borg init --encryption=authenticated-blake2 /media/tylor/datacenter/personnels/dupretyl/BACKUP
```

Borg vous demande votre _passphrase_, nous mettons dans notre cas : _42spinAxis_

{% hint style="danger" %}
Ne perdez surtout pas votre **mot de passe**, ou votre clef car vous ne pourriez plus relire vos sauvegardes !!
{% endhint %}

### Répertoire à sauvegarde

pattern /// 

### Code bash pour le script 

```bash
cd 
touch backup.sh
chmod +x backup.sh
```

Ouvrir le fichier backup.sh avec votre éditeur de texte préféré.

```bash
#!/bin/sh

# Initialisation de la variable REPOSITORY  
export REPOSITORY=/media/tylor/datacenter/personnels/dupretyl/BACKUP

# Récupération du mot de passe 
export BORG_PASSCOMMAND="cat ~/.borg-passphrase"


```

## 



