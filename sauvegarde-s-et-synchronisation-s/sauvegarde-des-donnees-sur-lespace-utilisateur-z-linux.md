# Sauvegarde des données sur l'espace utilisateur Z: \(Linux\)

## Installation

### Pré-Requis

* [Introduction](../) 
* [Espace de travail](../environnement-de-travail/comment-fonctionnent-les-espaces-x-et-z.md) dans le datacenter de l'université

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

Nous devons d'abord créer un point de montage qui servira à relier un dossier de votre ordinateur avec votre dossier personnel sur le datacenter.

```bash
sudo mkdir -p /media/dupretyl/datacenter
```

La configuration du point de montage sous linux se fait en modifiant le fichier `/etc/fstab`en mode super-utilisateur `sudo` : 

```bash
sudo gedit /etc/fstab &
```

Vous pouvez ajouter la ligne suivante à ce fichier, en faisant attention de modifier les éléments suivants par vos propres identifiants : 

```text
//ur.univ-rouen.fr/urdatas/ /media/dupretyl/datacenter cifs user,noauto,user=dupretyl,password=42spinaxis,uid=1000 0 0
```

### Initialisation de borg

```bash
borg init --encryption=[TYPECHIFFREMENT] [DESTINATION]
```

La commande borg init \(voir [documentation](https://borgbackup.readthedocs.io/en/stable/usage/init.html)\) doit prendre un type de chiffrement et une destination \(**repository**\) pour initialiser le répertoire de dépot des différentes sauvegardes. 

Pour la configuration du **\[TYPECHIFFREMENT\]**, plusieurs options sont disponibles : 

* aucune chiffrement : `--encryption=none`
* sans chiffrement mais avec authentification : `--encryption=authenticated-blake2`
* avec mot de passe et clef : `--encryption=keyfile-blake2`

{% hint style="warning" %}
Le mode sans chiffrement est très fortement déconseillé, surtout en cas de stockage de données sensibles !
{% endhint %}

La **\[DESTINATION\]** correspond à notre point de montage dans `/media/reyman/datacenter/personnels/dupretyl/BACKUP`

Ce qui donne au final la commande suivante : 

```bash
borg init --encryption=authenticated-blake2 /media/reyman/datacenter/personnels/dupretyl/BACKUP
```

