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

### Option A : la dernière version, automatiquement (recommandé)

SourceForge propose un lien « dernière version » qui ne contient aucun numéro :

```bash
wget --content-disposition "https://sourceforge.net/projects/xampp/files/latest/download"
ls -lh xampp-*.run
```

`--content-disposition` garde le vrai nom du fichier (avec sa version). Vérifie que le fichier téléchargé contient bien `linux` dans son nom. Si ce n'est pas le cas (par exemple un installateur Windows), utilise l'option B.

### Option B : script qui trouve la dernière version Linux

Ce script lit la liste des fichiers XAMPP Linux sur SourceForge et télécharge le plus récent.

```bash
#!/bin/bash
set -e

RSS="https://sourceforge.net/projects/xampp/rss?path=/XAMPP%20Linux"

URL=$(curl -sL "$RSS" \
  | grep -oP 'https://sourceforge\.net/projects/xampp/files/XAMPP%20Linux/[^<"]*xampp-linux-x64-[^<"]*-installer\.run' \
  | sort -uV | tail -n1)

if [ -z "$URL" ]; then
  echo "Aucune version trouvée. Vérifie https://www.apachefriends.org/download.html"
  exit 1
fi

echo "Téléchargement : $URL"
wget "$URL" -O xampp-installer.run
chmod +x xampp-installer.run
echo "Prêt : sudo ./xampp-installer.run"
```

Enregistre-le sous `get-xampp.sh`, puis :

```bash
chmod +x get-xampp.sh
./get-xampp.sh
```

### Option C : version précise (manuelle)

Si tu veux une version PHP bien précise, remplace le numéro dans l'URL :

```bash
wget "https://sourceforge.net/projects/xampp/files/XAMPP%20Linux/8.2.12/xampp-linux-x64-8.2.12-0-installer.run/download" -O xampp-installer.run
```

> Le numéro de version change à chaque mise à jour. En cas d'erreur 404, récupère le lien actuel sur <https://www.apachefriends.org/download.html>.

### Mettre à jour une installation existante

Avant d'installer une nouvelle version par-dessus l'ancienne, sauvegarde tes sites et tes bases de données :

```bash
sudo /opt/lampp/lampp stop
sudo cp -r /opt/lampp/htdocs ~/backup-htdocs
sudo cp -r /opt/lampp/var/mysql ~/backup-mysql
```

Lance ensuite le nouvel installateur (étape 2). Si besoin, remets tes fichiers depuis les sauvegardes.

## 2. Installer

```bash
chmod +x xampp-installer.run
sudo ./xampp-installer.run
```

XAMPP s'installe dans `/opt/lampp`.

## 3. Démarrer, vérifier et arrêter

```bash
sudo /opt/lampp/lampp start
sudo /opt/lampp/lampp status
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

**Important : personnaliser la ligne `Exec=`**

Remplace `NOM_UTILISATEUR` par le nom obtenu avec la commande `whoami`.

Par exemple :
- Si `whoami` affiche `jean`, utilise `/home/jean`.
- Si `whoami` affiche `pierre`, utilise `/home/pierre`.
- Si `whoami` affiche `alice`, utilise `/home/alice`.

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

#### 3.5 Rendre le lanceur exécutable

Exécute dans le terminal :

```bash
chmod +x ~/.local/share/applications/xampp-panel.desktop
```

Le lanceur est maintenant créé et prêt à être activé à l'étape suivante.

### Étape 4 : rendre le lanceur exécutable et l'activer

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
