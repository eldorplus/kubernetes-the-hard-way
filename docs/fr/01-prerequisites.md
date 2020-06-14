# Les prérequis

## Google Cloud Platform

Ce tutoriel s'appuie sur la [Plateforme Google Cloud](https://cloud.google.com/) pour rationaliser le dimensionnement de l'infrastructure de calcul nécessaire au démarrage d'un cluster Kubernetes à partir de zero. [S'inscrire](https://cloud.google.com/free/) pour 300$ de crédits gratuits.

[Coût estimé](https://cloud.google.com/products/calculator/#id=55663256-c384-449c-9306-e39893e23afb) pour l'exécution de ce tutoriel : 0,23$ par heure (5,46$ par jour).

> Les ressources de compute nécessaires pour ce tutoriel dépassent le niveau gratuit de la Google Cloud Platform.

## Google Cloud Platform SDK

### Installer le Google Cloud SDK

Suivez la [documentation](https://cloud.google.com/sdk/) de Google Cloud SDK pour installer et configurer l'utilitaire en ligne de commande `gcloud`.

Vérifiez que la version du Google Cloud SDK est la 262.0.0 ou supérieure :

```sh
gcloud version
```

### Définir une région et une zone compute par défaut

Ce tutoriel suppose qu'une région et une zone de compute par défaut ont été configurées.

Si vous utilisez l'outil en ligne de commande `gcloud` pour la première fois, `init` est le moyen le plus simple de le faire :

```sh
gcloud init
```

Assurez-vous ensuite d'autoriser gcloud à accéder à la plateforme Cloud avec vos identifiants d'utilisateur Google :

```sh
gcloud auth login
```

Définissez ensuite une région de compute et une zone de calcul par défaut :

```sh
gcloud config set compute/region us-west1
```

Définissez une zone de compute par défaut :

```sh
gcloud config set compute/zone us-west1-c
```

> Utilisez la commande `gcloud compute zones list` pour afficher des régions et des zones supplémentaires.

## Exécuter des commandes en parallèle avec le tmux

[Tmux](https://github.com/tmux/tmux/wiki) peut-être utilisé pour exécuter des commandes sur plusieurs instances de compute en même temps. Les laboratoires de ce tutoriel peuvent avoir besoin d'exécuter les mêmes commandes sur plusieurs instances de compute. Dans ce cas, envisagez d'utiliser tmux et de diviser une fenêtre en plusieurs volets avec des volets de synchronisation activés pour accélérer le processus d'approvisionnement.

> L'utilisation de tmux est facultative et n'est pas obligatoire pour suivre ce tutoriel.

![Tmux capture d'écran](images/tmux-screenshot.png)

> Activez la synchronisation des volets en appuyant sur `ctrl+b` suivi de `shift+:`. Ensuite, tapez `set synchronize-panes on` à l'invite. Pour désactiver la synchronisation : `set synchronize-panes off`.

Suivant : [Installation des outils du client](02-client-tools.md)
