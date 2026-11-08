# Installer XAMPP sur Ubuntu (avec icône dans le menu)

Guide pour installer XAMPP sur Ubuntu et ouvrir directement le **tableau de bord (XAMPP Control Panel)** en cliquant sur une icône du menu des applications.

## Sommaire

1. [Télécharger l'installateur](#1-télécharger-linstallateur)
2. [Installer](#2-installer)
3. [Démarrer, vérifier et arrêter](#3-démarrer-vérifier-et-arrêter)
4. [Créer l'icône du menu](#4-créer-licône-du-menu)
5. [Dépannage](#5-dépannage)

---

## 1. Télécharger l'installateur

> **Attention :** n'utilise pas le lien SourceForge `.../files/latest/download`. Avec `wget`, il renvoie l'installateur **Windows** (`.exe`), inutilisable sur Ubuntu.

### Option A : depuis le site officiel (recommandé)

1. Ouvre <https://www.apachefriends.org/fr/download.html>.
2. Clique sur **XAMPP pour Linux** : le fichier `xampp-linux-x64-X.X.X-0-installer.run` arrive dans `~/Téléchargements`.
3. Renomme-le pour simplifier les commandes suivantes :

```bash
cd ~/Téléchargements
mv xampp-linux-*-installer.run xampp-installer.run
ls -lh xampp-installer.run
```

### Option B : en ligne de commande

Les versions Linux sont dans le dossier `XAMPP Linux` de SourceForge : <https://sourceforge.net/projects/xampp/files/XAMPP%20Linux/>. Choisis une version, puis télécharge-la sous le nom `xampp-installer.run` (exemple avec la 8.2.12, à adapter) :

```bash
cd ~/Téléchargements
wget -O xampp-installer.run "https://sourceforge.net/projects/xampp/files/XAMPP%20Linux/8.2.12/xampp-linux-x64-8.2.12-0-installer.run/download"
```

### Vérifier le fichier

```bash
ls -lh xampp-installer.run
file xampp-installer.run
```

Le fichier doit peser environ 150 Mo et `file` doit indiquer un exécutable **ELF** pour Linux. S'il pèse quelques Ko ou si `file` indique du HTML, le téléchargement a échoué : refais-le depuis le site officiel.

## 2. Installer

Depuis le dossier où se trouve le fichier (`~/Téléchargements`) :

```bash
chmod +x xampp-installer.run
sudo ./xampp-installer.run
```

XAMPP s'installe dans `/opt/lampp`.

Si l'installateur échoue sur Ubuntu 24.04 à cause de bibliothèques manquantes :

```bash
sudo apt install -y libncurses5 libnsl2
```

## 3. Démarrer, vérifier et arrêter

```bash
sudo /opt/lampp/lampp start
```
```bash
sudo /opt/lampp/lampp status
```
```bash
sudo /opt/lampp/lampp stop
```

Ouvre ensuite <http://localhost> dans le navigateur pour vérifier que tout fonctionne.

## 4. Créer l'icône du menu

L'installateur ne crée pas toujours de raccourci. On en crée un qui ouvre directement le tableau de bord.

> **Pourquoi un script ?** Le tableau de bord doit tourner en root, mais `pkexec` seul ne lui transmet pas l'accès à l'écran. Résultat : rien ne s'affiche. Le script règle ce problème.

### Étape 1 : installer les dépendances

```bash
sudo apt update
sudo apt install -y x11-xserver-utils libxcb-xinerama0
```

### Étape 2 : créer le script de lancement

```bash
mkdir -p ~/.local/bin
nano ~/.local/bin/xampp-panel.sh
```

Colle ce contenu :

```bash
#!/bin/bash
# Autorise root à afficher une fenêtre sur ta session
xhost +SI:localuser:root >/dev/null 2>&1

# Lance le tableau de bord XAMPP en root, avec l'accès à l'écran
pkexec env \
  DISPLAY="$DISPLAY" \
  XAUTHORITY="${XAUTHORITY:-$HOME/.Xauthority}" \
  QT_QPA_PLATFORM=xcb \
  /opt/lampp/manager-linux-x64.run

# Retire l'autorisation à la fermeture
xhost -SI:localuser:root >/dev/null 2>&1
```

Enregistre avec `Ctrl + O`, puis `Entrée`, puis `Ctrl + X`. Rends-le exécutable :

```bash
chmod +x ~/.local/bin/xampp-panel.sh
```

### Étape 3 : créer le lanceur du menu

Le lanceur permet d'afficher **XAMPP Control Panel** dans le menu des applications Ubuntu et de démarrer son interface en cliquant sur son icône.

#### 3.1 Créer le fichier lanceur

Ouvre le terminal (`Ctrl + Alt + T`) et exécute :

```bash
mkdir -p ~/.local/share/applications
nano ~/.local/share/applications/xampp-panel.desktop
```

#### 3.2 Identifier le nom d'utilisateur

Dans un autre terminal, exécute :

```bash
whoami
```

Cette commande affiche le nom de l'utilisateur actuellement connecté à Ubuntu.

Par exemple, si le terminal affiche :

```text
jean
```

Le dossier personnel de cet utilisateur est :

```text
/home/jean
```

Chaque utilisateur doit utiliser son propre nom dans le chemin du lanceur.

#### 3.3 Configurer le lanceur

Dans le fichier ouvert avec Nano, copie le contenu suivant :

```ini
[Desktop Entry]
Version=1.0
Type=Application
Name=XAMPP Control Panel
Comment=Gérer les services XAMPP
Exec=/home/NOM_UTILISATEUR/.local/bin/xampp-panel.sh
Icon=/opt/lampp/htdocs/favicon.ico
Terminal=false
Categories=Development;
```

**Important : Il faut personnaliser la ligne `Exec=`**

Remplace `NOM_UTILISATEUR` par le nom obtenu avec la commande `whoami`.

Par exemple :
- Si `whoami` affiche `jean`, utilise `/home/jean`.

La ligne doit donc avoir cette forme :

```ini
Exec=/home/jean/.local/bin/xampp-panel.sh
```

Ne conserve pas le texte `NOM_UTILISATEUR` : il doit impérativement être remplacé.

**Explication des paramètres :**

- `Exec=` indique le script qui sera exécuté au clic sur l'icône. Il doit pointer vers le script `xampp-panel.sh` créé à l'étape 2. Ne remplace pas cette ligne par `pkexec /opt/lampp/manager-linux-x64.run`, car cela ne transmet pas correctement les informations nécessaires à l'affichage de l'interface graphique.
- `Icon=` définit l'image affichée dans le menu Ubuntu. Ici, on utilise le favicon de XAMPP.
- `Name=` définit le nom affiché dans le menu des applications.

#### 3.4 Enregistrer le fichier

Dans Nano :

1. Appuie sur `Ctrl + O`.
2. Appuie sur `Entrée` pour confirmer.
3. Appuie sur `Ctrl + X` pour quitter.

### Étape 4 : Rendre le lanceur exécutable et l'activer

```bash
chmod +x ~/.local/share/applications/xampp-panel.desktop
update-desktop-database ~/.local/share/applications
```

### Étape 5 : vérifier

Ouvre le menu des applications Ubuntu (touche `Super`) et cherche **XAMPP Control Panel**. Un clic ouvre directement le tableau de bord, après la demande de mot de passe.

Si l'icône n'apparaît pas tout de suite, déconnecte-toi puis reconnecte-toi.

## 5. Dépannage

### Le tableau de bord ne s'ouvre toujours pas

Lance le script depuis un terminal pour voir l'erreur :

```bash
~/.local/bin/xampp-panel.sh
```

### Erreur `xcb` ou `Qt platform plugin`

```bash
sudo apt install -y libxcb-cursor0 libxcb-xinerama0
```

### Apache ne démarre pas (port 80 déjà utilisé)

```bash
sudo lsof -i :80
sudo systemctl stop apache2
sudo systemctl disable apache2   # pour qu'il ne redémarre pas au boot
```

### MySQL ne démarre pas (conflit avec un MySQL système)

```bash
sudo systemctl stop mysql
```

### L'icône n'est pas jolie ou absente

Remplace la ligne `Icon=` par une icône système, par exemple :

```ini
Icon=applications-development
```

### Désinstaller XAMPP

```bash
sudo /opt/lampp/uninstall
rm ~/.local/share/applications/xampp-panel.desktop
rm ~/.local/bin/xampp-panel.sh
```
