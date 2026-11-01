# Installer Flatpak, Flathub et Bazaar sur Ubuntu

Ce guide explique comment activer [Flatpak](https://flatpak.org/) sur Ubuntu, ajouter le dépôt [Flathub](https://flathub.org/) et installer **Bazaar**, une boutique graphique pour gérer les applications Flatpak.

## Prérequis

- Ubuntu 18.10 (Cosmic Cuttlefish) ou version ultérieure
- Un accès `sudo`
- Une connexion Internet

## 1. Installer Flatpak

Ouvrir le Terminal et exécuter :

```bash
sudo apt install flatpak
```

## 2. Ajouter le dépôt Flathub

Flathub est la principale source d'applications Flatpak. Pour l'activer :

```bash
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

Il est aussi possible de télécharger et d'ouvrir directement le [fichier du dépôt Flathub](https://dl.flathub.org/repo/flathub.flatpakrepo).

## 3. Installer Bazaar

Une fois Flathub activé, installer Bazaar depuis Flathub :

```bash
flatpak install flathub io.github.kolunmi.Bazaar
```

Bazaar peut ensuite être lancé depuis le menu des applications du bureau. Il permet d'installer des applications Flatpak de façon graphique.

## 4. Redémarrer

Pour finaliser la configuration, redémarrer le système :

```bash
sudo reboot
```

## Vérification

Après le redémarrage, vérifier que Flathub est bien configuré :

```bash
flatpak remotes
```

La liste doit contenir `flathub`.

## Commandes utiles

| Action | Commande |
| --- | --- |
| Rechercher une application | `flatpak search <nom>` |
| Installer une application | `flatpak install flathub <identifiant>` |
| Lister les applications installées | `flatpak list` |
| Mettre à jour les applications | `flatpak update` |
| Désinstaller une application | `flatpak uninstall <identifiant>` |

## Ressources

- [Guide officiel Flathub pour Ubuntu](https://flathub.org/en/setup/Ubuntu)
- [Documentation Flatpak](https://docs.flatpak.org/)
