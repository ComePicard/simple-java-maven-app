# simple-java-maven-app

This repository is for the
[Build a Java app with Maven](https://jenkins.io/doc/tutorials/build-a-java-app-with-maven/)
tutorial in the [Jenkins User Documentation](https://jenkins.io/doc/).

The repository contains a simple Java application which outputs the string
"Hello world!" and is accompanied by a couple of unit tests to check that the
main application works as expected. The results of these tests are saved to a
JUnit XML report.

The `jenkins` directory contains an example of the `Jenkinsfile` (i.e. Pipeline)
you'll be creating yourself during the tutorial and the `jenkins/scripts` subdirectory
contains a shell script with commands that are executed when Jenkins processes
the "Deliver" stage of your Pipeline.

# Configuration

## Setup de Azure

Se connecter à Microsoft Azure depuis ce [lien](https://portal.azure.com/?Microsoft_Azure_Education_correlationId=281e36fb-79f3-49fc-95fb-6ab2d3327604#home).
Créez si ce n'est pas déjà fais un *Groupe de ressources*.
Créez dans ce *Groupe de ressource* une machine virtuel **Ubuntu 20.04.2 LTS**.
Remplissez les champs et mettez le **type d'authentification** sur **mot de passe**.
Dans les détails de votre machine virtuelle, rendez-vous à présent dans l'onglet *Mise en réseau*, puis sélectionnez *Ajouter une règle de port d'entrée*.
Vérifiez que le champ **Plages de ports de destination** est bien mis à **8080**, si n'est pas le cas faites-le.

## Lancement de la machine virtuelle

Installez [Remote Desktop Manager](https://devolutions.net/remote-desktop-manager/fr/home/download) en édition gratuite.
Exécutez le fichier .exe téléchargé et procédez à l'installation.
Une fois le téléchargement terminé, lancez l'application.
Rendez-vous sur l'onglet **Dashboard** (probablement **tableau de bord** en français).
Cliquez sur **New Entry**, recherchez *ssh shell* et sélectionnez le.
Entrez le nom souhaité de votre ssh shell.
Renseignez dans *host* l'adresse IP publique de votre machine virtuelle (trouvable dans les détails de votre machine virtuelle Azure), dans *name* le nom de votre machine virtuelle et dans *password* son mot de passe.
Validez, puis cliquez sur Open Session pour la démarrer.
Il se peut qu'elle ne fonctionne pas du premier coup, n'hésitez pas si cela ne fonctionne pas à la relancer directmement.
 

## Installation de Docker

```docker
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

```docker
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

```docker
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```
sudo apt-get update
```

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

## Installation de Jenkins

```
sudo docker network create jenkins-net

docker run \
	--name jenkins-docker \
	--rm \
	--detach \
	--privileged \
	--network jenkins-net \
	--network-alias docker \
	--env DOCKER_TLS_CERTDIR=/certs \
	--volume jenkins-data:/var/jenkins_home \
	--publish 3000:3000 --publish 2376:2376 \
	docker:bind\
	--storage-driver overlay2
```

Créez un fichier **Dockerfile** :

```
nano Dockerfile
```
Insérez y le code suivant :

```
FROM jenkins/jenkins:2.346.2-jdk11
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \ 
 https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins blueocean:1.25.5 docker-workflow:1.28
```

Créez le conteneur Docker :

```
sudo docker build -t jenkins-training:v1
```

Exécutez le conteneur:
```
sudo docker run \
    --name jenkins-training3 \
    --detach \
    --network jenkins-net \
    --env DOCKER_HOST=tcp://docker:2376 \
    --env DOCKER_CERT_PATH=/certs/client \
    --env DOCKER_TLS_VERIFY=1 \
    --publish 8080:8080 --publish 50000:50000 \
    --volume jenkins-data:/var/jenkins_home \
    --volume jenkins-docker-certs:/certs/client:ro \
    --volume "$HOME":/home \
    --restart=on-failure \
    --env JAVA_OPTS="-Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true" \
    jenkins-training:v1
```

## Import du projet github

Créez un répertoire *github* :
```
mkdir github
```

Connectez-vous en tant que root :
```
sudo su -
```

...et créez une clé ssh :
```
ssh-keygen -t rsa -b 4096
```

Confirmez en appuyant sur *Entrée* jusqu'à reprendre la main, puis récupérez la clé public que vous venez de créer :
```
cat .ssh/id_rsa.pub
```

Pensez à sortir du mode root en faisant **ctrl+D** !

Copiez la chaîne de caractère affichée puis rendez-vous sur Github.
Dans votre profil rendez-vous dans les paramètres : Settings → SSH and GPG keys.
Cliquez sur *New SSH key*, nommez votre clé, puis copiez le texte obtenu ultérieurement dans le champ *key*.

Rendez-vous à présent sur le repository git voulu (à but de test vous-pouvez forker ce [repository](https://github.com/DobbyBop/simple-java-maven-app)).
Puis cloner le repository en SSH dans votre machine virtuelle :
```
cd github
git clone <lien SSH copié>
```

## Configuaration du Jenkinsfile

Rendez-vous dans le repository copié et créez/modifiez le fichier Jenkinsfile :
```
cd <repository>

# Renvoie une erreur si un dossier jenkins existe déjà
mkdir jenkins

cd jenkins

# Créez ou modifiez le fichier Jenkinsfile
sudo nano Jenkinsfile
```

Voici quelques lien utiles pour remplir correctement le Jenkinsfile : 
[Exemple pour une app Java avec Maven](https://www.jenkins.io/doc/tutorials/build-a-java-app-with-maven/)
[Documentation Pipeline](https://www.jenkins.io/doc/book/pipeline/)

Une fois que cela est fait, ouvrez une page internet, et tapez l'url suivant :
```
http://<ip_machine_virtuelle>:8080
```

Sélectionnez l'installation par défaut, entrez *root* partout où des identifiants vous sont demandés.

Pour afficher le mot de passe demandé :
```
sudo cat <path>
```

Une fois arrivé sur le dashboard, sélectionnez *Nouveau item*, donnez lui un ptit nom, sélectionnez Pipeline puis validez.

Les tests devraient se lancer et fonctionner !

