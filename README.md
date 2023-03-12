#  Orchestration de conteneurs - CICD

## Description 

Ce projet a pour but d'automatiser le déploiement d'une application.

Le projet utilisera une instance de GitLab afin d'organiser notre code. Cette instance est déployée sur notre serveur avec l'aide de Docker.

Le projet est composé d'un fichier .gitlab-ci.yml qui permet de lancer un script à chaque modification sur notre repository. Mais également d'un fichier docker-compose.yml permettant de déployer GitLab ainsi que notre application sur les différents environnements.

## 1 - Déployer gitlab

Nous souhaitons déployer une instance de GitLab directement sur notre serveur. Pour ce faire, nous allons utiliser le fichier gitlab.docker-compose.yaml qui contient deux parties distinctes. La première partie nous permet de récupérer l'image de GitLab, de déclarer les volumes nécessaires pour sauvegarder les informations en cas de redémarrage du Docker, et d'indiquer les différents ports à utiliser pour notre application. La deuxième partie nous permet d'installer et de lancer les runners de GitLab. Ils ont pour objectif de nous permettre de lancer automatiquement le script de CICD et de gérer les pipelines de déploiement.

Une fois notre installation de GitLab terminée, nous pouvons donc cloner notre repository dans GitLab afin d'automatiser son déploiement.

## 2 - Build 

Une fois notre application clonée et disponible sur GitLab, la première étape est de builder automatiquement notre projet à chaque push sur GitLab. Cela sera réalisable grâce au fichier `.gitlab-ci.yml` que l'on créera à la racine de notre projet. La première étape de notre pipeline sera de builder notre application. Pour ce faire, nous utiliserons les jobs "build_API" et "build_console". Ces jobs nous permettront de lancer les commandes "dotnet" pour builder automatiquement nos applications.

Dans l'étape de build, nous créerons également une image Docker de notre application, uniquement si le push est effectué sur la branche "master".

## 3 - Test

La deuxième étape de notre pipeline consiste à tester notre application. Pour ce faire, nous allons utiliser le job "test". Ce job nous permettra de lancer les commandes dotnet pour tester automatiquement nos applications. Nous utiliserons également Coverlet pour générer un rapport de couverture de code.

## 4 - Generate

Ce job va nous permettre d'exécuter une fonction de notre projet. Nous allons récupérer le contenu de la réponse et le stocker grâce à l'artefact.

## 5 - Publish

La prochaine étape est de publier notre application sur DockerHub. Pour ce faire, nous allons utiliser le job publish qui, grâce aux variables d'environnement créées dans GitLab, va nous permettre de pousser notre image sur DockerHub.

## 6 - Tag des images

Au cours du pipeline, nous avons créé une image Docker de notre application. Nous allons maintenant ajouter différents tags à notre image afin de l'identifier plus facilement, que ce soit sur Docker Hub ou sur notre serveur. Pour ce faire, nous allons utiliser la commande `docker tag <nom d'origine> <nom avec le nouveau tag>` pour les images locales et `docker push <nom avec le nouveau tag>` pour les images sur Docker Hub.

## 7 - Badge

Au cours de la pipeline, nous allons générer deux badges sur notre dépôt GitLab. Un badge pour le build et un badge pour les tests. Ces badges nous permettront de visualiser l'état de notre pipeline directement sur notre dépôt GitLab, ainsi que d'afficher le score de couverture de code de notre application à l'aide de Coverlet. Pour plus d'informations sur les badges GitLab, vous pouvez consulter la documentation suivante : https://docs.gitlab.com/ee/user/project/badges.html.

## 8 - Clean

Après avoir publié nos images sur DockerHub, nous allons supprimer les images de notre serveur afin de ne pas surcharger notre disque dur. Pour ce faire, nous allons utiliser le job clean. Ce job va nous permettre de supprimer les images de notre serveur. Cette étape est réalisée dans tous les cas, que les étapes précédentes soient un succès ou non.

## 9 - Security

Cette étape a pour but de lancer une analyse de sécurité de notre application avec Syft. Nous allons récupérer le résultat de l'analyse et le stocker dans un fichier JSON grâce à l'artefact.


## 10 - Deploy

Pour finir notre pipeline, nous allons déployer notre application sur les différents environnements. Pour ce faire, nous allons utiliser le job deploy. Ce job va nous permettre de lancer les commandes docker-compose permettant de déployer notre application sur les différents environnements. Nous allons déployer l'application sur un environnement de staging et un environnement de production. Le job permettant le déploiement sur l'environnement de staging est lancé automatiquement, tandis que le déploiement sur l'environnement de production est lancé manuellement. Pour ce faire, nous allons utiliser le bouton "play" dans notre pipeline sur GitLab.

Il faut au préalable créer les dossiers `stage` et `production` à la racine de notre repository, ainsi que des fichiers `docker-compose.yml` dans ces dossiers. Ces fichiers ont pour but de récupérer l'image téléchargée depuis Docker Hub et de lancer notre application sur les ports 8080 pour l'environnement de staging et 8081 pour l'environnement de production.