# Installer GitHub Copilot sur Ubuntu (Linux)

Ce guide explique comment installer l’application de bureau **GitHub Copilot** au format AppImage et créer un raccourci dans le menu des applications Ubuntu.

## 1. Pages officielles et téléchargement

- **Présentation de GitHub Copilot (en français)** : [https://gh.io/copilot-app-linux](https://gh.io/copilot-app-linux)
- **Dépôt officiel de l’application GitHub Copilot** : https://github.com/github/app
- **Téléchargements officiels (versions disponibles)** : https://github.com/github/app/releases

La page de présentation est une page d’information : ce n’est pas, à elle seule, le téléchargement direct du fichier Linux. Pour Ubuntu, ouvre le dépôt officiel puis la section **Linux**, ou consulte les versions publiées (Releases).

Choisis le fichier adapté à l’architecture de ton ordinateur :
- `GitHub-Copilot-linux-x64.AppImage` pour Linux x86-64 (la plupart des PC Intel/AMD) ;
- `GitHub-Copilot-linux-arm64.AppImage` pour Linux ARM64.

Le reste de ce guide reprend le nom du fichier x64 utilisé dans l’installation décrite ici. Si tu télécharges la version ARM64, adapte le nom du fichier dans les commandes.

## 2. Placer le fichier AppImage

Dans ce guide, le fichier est placé dans le dossier `Applications` de ton dossier personnel.

Crée le dossier si nécessaire :

```bash
mkdir -p ~/Applications
```

Si le fichier est dans `Téléchargements`, déplace-le :

```bash
mv ~/Téléchargements/GitHub-Copilot-linux-x64.AppImage ~/Applications/
```

S'il est déjà dans `~/Applications`, ne refais pas cette commande.

## 3. Rendre le fichier exécutable et le tester

```bash
chmod +x ~/Applications/GitHub-Copilot-linux-x64.AppImage
```

Lance-le pour vérifier que l’application démarre :

```bash
~/Applications/GitHub-Copilot-linux-x64.AppImage
```

## 4. Extraire l’icône de l’application

Place-toi dans le dossier `Applications` :

```bash
cd ~/Applications
./GitHub-Copilot-linux-x64.AppImage --appimage-extract
```

Cette commande crée un dossier `squashfs-root`. Copie l’icône principale dans le dossier des icônes personnelles :

```bash
mkdir -p ~/.local/share/icons
cp ~/Applications/squashfs-root/"GitHub Copilot.png" ~/.local/share/icons/github-copilot.png
```

Si le fichier `GitHub Copilot.png` n’existe pas dans `squashfs-root`, recherche les icônes disponibles :

```bash
find ~/Applications/squashfs-root \( -iname '*github*.png' -o -iname '*copilot*.png' \)
```

Dans ce cas, adapte le chemin de la commande `cp` avec le fichier trouvé.

## 5. Créer le raccourci dans le menu Ubuntu

Crée le dossier des lanceurs personnels :

```bash
mkdir -p ~/.local/share/applications
```

Ouvre le fichier du raccourci :

```bash
gedit ~/.local/share/applications/github-copilot.desktop
```

Si `gedit` n’est pas installé, utilise l’éditeur de texte disponible sur ton système.

Colle le contenu suivant :

```ini
[Desktop Entry]
Version=1.0
Type=Application
Name=GitHub Copilot
Comment=Assistant de programmation
Exec=/home/NOM_UTILISATEUR/Applications/GitHub-Copilot-linux-x64.AppImage
Icon=/home/NOM_UTILISATEUR/.local/share/icons/github-copilot.png
Terminal=false
Categories=Development;
StartupNotify=true
```

**Important :** remplace les deux occurrences de `NOM_UTILISATEUR` par ton nom d’utilisateur Ubuntu.

Pour le connaître, ouvre un terminal et exécute :

```bash
whoami
```

Par exemple, si la commande affiche `lea`, remplace `NOM_UTILISATEUR` par `lea` dans les deux chemins. Ne laisse pas le texte `NOM_UTILISATEUR` tel quel.

Enregistre le fichier et ferme l’éditeur.

## 6. Activer le raccourci

```bash
chmod +x ~/.local/share/applications/github-copilot.desktop
update-desktop-database ~/.local/share/applications
```

Appuie sur la touche **Super** (touche Windows), recherche **GitHub Copilot**, puis lance l’application.

## 7. Dépannage

- **L’application ne démarre pas :** vérifie que le fichier AppImage est bien présent dans `~/Applications` et qu'il est exécutable.
- **Le raccourci ne fonctionne pas :** vérifie les chemins `Exec=` et `Icon=` ainsi que le nom obtenu avec `whoami`.
- **L’icône n’apparaît pas ou ne se met pas à jour :** vérifie que le fichier PNG existe au chemin indiqué dans `Icon=`. Déconnecte-toi puis reconnecte-toi à ta session Ubuntu si nécessaire.
- **Tu as téléchargé une nouvelle version ou déplacé le fichier :** adapte le chemin `Exec=` dans `github-copilot.desktop`.

## Sources officielles

- [GitHub Copilot – présentation en français](https://github.com/features/copilot?locale=fr-fr)
- [Dépôt officiel de l’application GitHub Copilot](https://github.com/github/app)
- [Versions et téléchargements](https://github.com/github/app/releases)

Les noms de fichiers et les versions peuvent évoluer. Vérifie toujours les fichiers proposés sur le dépôt officiel avant de télécharger.
