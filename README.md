# CRUD-MASTER
# français
Instructions
Les API sont un moyen très courant et pratique de déployer des services de manière modulaire. Dans cet exercice, nous allons créer une infrastructure de microservices simple, dotée d'une passerelle API connectée à deux services. L'un, l'API d'inventaire, récupère les données d'une base de données PostgreSQL, tandis que l'autre, l'API de facturation, traite exclusivement les messages reçus via RabbitMQ, sans interaction directe avec la base de données. La communication entre ces services s'effectuera via HTTP et des systèmes de file d'attente de messages. Chacun de ces services fonctionnera sur des machines virtuelles distinctes, offrant ainsi un environnement séparé pour leurs fonctionnalités.

Aperçu général
Diagramme d'architecture CRUD Master

Nous allons mettre en place une plateforme de streaming de films, où une API ( inventory) disposera d'informations sur les films disponibles et une autre ( billing) traitera les paiements.

Nous allons créer une plateforme de streaming de films. Une API ( inventory) fournira des informations sur les films disponibles, tandis qu'une autre ( billing) gérera le traitement des paiements.

La passerelle API communiquera en HTTP avec le inventoryservice et utilisera RabbitMQ pour billingle service.

Dans cet exercice, vous devrez installer Python3 (avec Flask, SQLAlchemy et d'autres packages), PostgreSQL, RabbitMQ, Postman, VirtualBox (ou un logiciel équivalent tel que VMware) et Vagrant.

Même si cela peut paraître complexe au premier abord, de nombreuses ressources sont disponibles sur le site officiel et sur les blogs communautaires pour configurer ces outils. De plus, les détails de configuration peuvent varier d'une plateforme à l'autre ; n'hésitez donc pas à les tester et à vous assurer que tout est correctement installé avant de poursuivre.

API 1 : Inventaire
Définition de l'API d'inventaire
Cette API sera une API RESTful CRUD (Création, Lecture, Mise à jour, Suppression). Elle utilisera une base de données PostgreSQL. Elle fournira des informations sur les films présents dans l'inventaire et permettra aux utilisateurs d'effectuer des opérations de base.

Une méthode courante consiste à utiliser Flask, un framework web Python populaire. Nous l'associerons à SQLAlchemy, un ORM qui abstraire et simplifie les interactions entre notre API et la base de données.

Voici les points de terminaison avec les requêtes HTTP possibles :

/api/movies: OBTENIR, PUBLIER, SUPPRIMER
/api/movies/:id: OBTENIR, METTRE, SUPPRIMER
Quelques détails sur chacun d'eux :

GET /api/moviesrécupérer tous les films.

GET /api/movies?title=[name]récupérer tous les films avec namedans le titre.

POST /api/moviescréer une nouvelle entrée de produit.

DELETE /api/moviessupprimer tous les films de la base de données.

GET /api/movies/:idrécupérer un seul film par id.

PUT /api/movies/:idmettre à jour un seul film par id.

DELETE /api/movies/:idsupprimer un seul film par id.

L'API devrait fonctionner sur http://localhost:8080/.

Définition de la base de données
Pour la base de données, nous utiliserons PostgreSQL. La base de données s'appellera movies_db.

Le moviestableau contiendra les colonnes suivantes :

id: identifiant unique généré automatiquement.
title: le titre du film.
description: la description du film.
Test de l'API d'inventaire
Pour tester l'exactitude de votre API, utilisez Postman ou un outil similaire. Créez un ou plusieurs tests pour chaque point de terminaison, puis exportez la configuration afin de pouvoir reproduire facilement les tests sur différentes machines.

La configuration sera vérifiée lors de l'audit.

API 2 : Facturation
Définition de l'API de facturation
Cette API reçoit uniquement les messages via RabbitMQ et consomme les messages de la file d'attente billing_queue. Les messages reçus sont des objets JSON « stringifiés », comme dans cet exemple :

{
  "user_id": "3",
  "number_of_items": "5",
  "total_amount": "180"
}
L'API analysera le message et créera une nouvelle entrée dans la billing_dbbase de données. Elle accusera également réception du traitement du message par la file d'attente RabbitMQ. Au démarrage, l'API prendra en charge tous les messages présents dans la file d'attente.

Jetez un œil à pikala bibliothèque Python pour un moyen simple d’interagir avec RabbitMQ.

Définition de la base de données
Pour la base de données, nous utiliserons également PostgreSQL. La base de données s'appellera billing_db.

Le orderstableau contiendra les colonnes suivantes :

id: identifiant unique généré automatiquement.
user_id: l'identifiant de l'utilisateur effectuant la commande.
number_of_items: le nombre d'articles inclus dans la commande.
total_amount: le coût total de la commande.
Test de l'API de facturation
Pour tester cette API voici quelques étapes :

Publiez un message directement dans billing_queueRabbitMQ à l'aide de son interface utilisateur ou de sa CLI.
Lorsque l'API de facturation est en cours d'exécution, les commandes doivent apparaître instantanément dans la orderstable de la billing_dbbase de données.
Lorsque l'API de facturation n'est pas en cours d'exécution, les requêtes adressées à la passerelle API doivent toujours renvoyer une réussite, mais la orderstable de la billing_dbbase de données ne sera pas mise à jour.
Lorsque l'API de facturation est redémarrée, les messages non traités doivent être traités et la orderstable de la billing_dbbase de données doit être mise à jour.
La passerelle API
La passerelle se chargera d'acheminer les requêtes vers le service approprié en utilisant le bon protocole (il peut s'agir de HTTP pour l'API d'inventaire ou de RabbitMQ pour l'API de facturation).

Interface avec l'API d'inventaire
La passerelle achemine toutes les requêtes vers /api/moviesl'API 1, sans avoir à vérifier les informations qui lui sont transmises. Elle renvoie la réponse exacte reçue par l'API 1.

Interface avec l'API de facturation
La passerelle recevra les requêtes POST api/billinget enverra un message via RabbitMQ dans une file d'attente appelée billing_queue. Le contenu du message sera le corps de la requête POST, dont la chaîne JSON.stringifysera . La passerelle devrait pouvoir envoyer des messages à la file d'attente même si l'API 2 n'est pas en cours d'exécution. Une fois l'API 2 démarrée, elle devrait pouvoir traiter ce message et renvoyer un accusé de réception.

Un exemple de requête POST à http://[API_GATEWAY_URL]:[API_GATEWAY_PORT]/api/billing/​​:

{
  "user_id": "3",
  "number_of_items": "5",
  "total_amount": "180"
}
Une fois le traitement réussi, vous pouvez vous attendre à un message de réponse tel que « Message publié » ou un accusé de réception similaire.

N'oubliez pas de configurer Content-Type: application/jsonle corps de la requête.

Documentation de l'API
Une bonne documentation est essentielle à toute API. De par leur conception, les API sont destinées à être utilisées par d'autres. C'est pourquoi de très grands efforts ont été déployés pour créer des méthodes de documentation standardisées et faciles à mettre en œuvre.

Pour vous initier à une documentation de qualité, vous devez créer un fichier de documentation OpenAPI pour la passerelle API. Il existe de nombreuses façons de procéder. Un bon point de départ pourrait être d'utiliser SwaggerHub avec au moins une description pertinente pour chaque point de terminaison. N'hésitez pas à implémenter toute fonctionnalité supplémentaire qui vous semble pertinente.

Vous devez également créer un README.mdfichier à la racine de votre projet avec des instructions détaillées sur la manière de créer et d'exécuter votre infrastructure et sur les choix de conception que vous avez faits pour la structurer.

Machines virtuelles
Aperçu général
Vous utiliserez Vagrant pour configurer trois VM différentes afin de tester les interactions et l'exactitude des réponses entre votre infrastructure API.

Vagrant est un logiciel open source qui vous aide à créer et à gérer des machines virtuelles. Avec Vagrant, vous pouvez créer un environnement de développement identique à votre environnement de production, ce qui simplifie le développement, les tests et le déploiement de vos applications.

Vos VM seront structurées comme suit :

gateway-vm: Cette VM contiendra uniquement le api-gateway.
inventory-vm: Cette VM contiendra l' inventory-appAPI et la base de données movies_db.
billing-vm: Cette VM contiendra l' billing-appAPI, la base de données orders et RabbitMQ.
Vagrant est conçu pour le développement et ne doit pas être utilisé dans des environnements de production.

Variables d'environnement
Pour simplifier le processus de création, il est recommandé de définir les variables essentielles dans un .envfichier. Cette approche facilite la modification ou la mise à jour d'informations critiques telles que les URL, les mots de passe, les noms d'utilisateur, etc.

Pour cet exercice, répertoriez toutes les variables d'environnement requises dans le fichier README.md. Une fois ces variables identifiées, créez un .envfichier contenant les informations d'identification nécessaires.

Ces variables seront utilisées par Vagrant et distribuées sur les différents microservices pour centraliser les informations d'identification.

Votre .envfichier doit contenir toutes les informations d’identification nécessaires et aucun des microservices ne doit avoir d’informations d’identification codées en dur dans le code source.

Pour les besoins de cet exercice, le .envfichier doit être inclus dans votre référentiel. Dans des scénarios réels, il est essentiel d'éviter d'inclure des données sensibles dans les référentiels pour éviter d'éventuelles fuites.

Configuration des VM
Vous disposerez d'un Vagrantfileoutil qui créera et démarrera les trois machines virtuelles. Il importera les variables d'environnement et les transmettra à chaque API.
Vous disposerez d'un scripts/répertoire contenant tous les scripts que vous souhaiterez exécuter pour installer les outils nécessaires sur chaque machine virtuelle. Ces scripts peuvent également être très utiles pour configurer les bases de données.
Votre configuration fonctionnera correctement pour les commandes suivantes (exécutées depuis la racine du projet) :

vagrant up: Démarre toutes les machines virtuelles.
vagrant status: Affiche l'état de toutes les machines virtuelles.
vagrant ssh <vm-name>:Vous permettra d'accéder à la VM via SSH.
Gérez vos applications Python avec PM2
PM2 est un gestionnaire de processus pour les applications Node.js qui simplifie la gestion et la mise à l'échelle de votre application. Il est conçu pour assurer son fonctionnement continu, même en cas de panne imprévue.

PM2 peut être utilisé pour démarrer, arrêter et répertorier les applications Node.js, ainsi que pour surveiller leur utilisation des ressources et la sortie du journal.

De plus, PM2 fournit un certain nombre de fonctionnalités pour gérer plusieurs applications, telles que l'équilibrage de charge et les redémarrages automatiques.

Dans notre situation, nous l'utiliserons principalement pour tester la résilience des messages envoyés à l'API de facturation lorsque l'API n'est pas opérationnelle.

Après avoir accédé à votre VM via SSH, vous pouvez exécuter les commandes suivantes :

sudo pm2 list:Répertoriez toutes les applications en cours d'exécution.
sudo pm2 stop <app_name>: Arrêter une application spécifique.
sudo pm2 start <app_name>:Démarrer une application spécifique.
Organisation du projet
README.md
En tant que bon exercice et outil utile, il vous est demandé de fournir une README.mddescription du projet.

L'idée d'un README.mdest de donner en quelques lignes suffisamment de contexte sur un projet pour comprendre de quoi il s'agit et comment le mener.

Ce fichier doit inclure des instructions pour exécuter et tester le projet, il doit également donner un aperçu bref et clair de la pile utilisée pour le construire.

Structure globale du fichier
Vous pouvez organiser votre structure de fichiers interne comme vous le souhaitez. Ceci dit, voici une méthode courante pour structurer ce type de projets :

.
├── README.md
├── config.yaml
├── .env
├── scripts
│   └── [...]
├── srcs
│   ├── api-gateway
│   │   ├── app
│   │   │   ├── __init__.py
│   │   │   └── ...             // Other python files
│   │   ├── requirements.txt
│   │   └── server.py
│   ├── billing-app
│   │   ├── app
│   │   │   ├── __init__.py
│   │   │   └── ...             // Other python files
│   │   ├── requirements.txt
│   │   └── server.py
│   └── inventory-app
│   │   ├── app
│   │   │   ├── __init__.py
│   │   │   └── ...             // Other python files
│       ├── requirements.txt
│       └── server.py
└── Vagrantfile
Lors des tests et avant de l'automatiser via la création de la machine virtuelle, vous devriez pouvoir démarrer la passerelle API et les deux API en utilisant la commande python server.pydans leurs répertoires respectifs.

Il est recommandé de développer vos API à l'aide d'environnements virtuels Python distincts afin d'isoler les exigences de chaque API. Vous pouvez utiliser venvou tout autre outil équivalent.

Si vous décidez d’utiliser une structure différente pour votre projet, n’oubliez pas que vous devez être en mesure d’expliquer et de justifier votre décision lors de l’audit.

En tant que bonne pratique, il est fortement conseillé d'ajouter venv/à votre .gitignoreafin de ne pas télécharger de fichiers inutiles dans votre dépôt git (ils seront générés automatiquement pendant le processus de construction).

# anglais
Instructions
APIs are a very common and convenient way to deploy services in a modular way. In this exercise we will create a simple microservices infrastructure, having an API Gateway connected to two services. While one service, the inventory API, retrieves data from a PostgreSQL database, the other service, the billing API, exclusively processes messages received through RabbitMQ without direct database interactions. Communication between these services will occur via HTTP and message queuing systems. Each of these services will operate within distinct virtual machines, facilitating a segregated environment for their functionalities.

General overview
CRUD Master architecture diagram

We will set up a movie streaming platform, where one API (inventory) will have information on the movies available and another one (billing) will process the payments.

We'll establish a movie streaming platform. One API (inventory) will provide details about available movies, while another (billing) will handle payment processing.

The API gateway will communicate in HTTP with the inventory service and using RabbitMQ for billing service.

In this exercise you will need to install Python3 (with Flask, SQLAlchemy and other packages), PostgreSQL, RabbitMQ, Postman, VirtualBox (or an equivalent software such as VMware) and Vagrant.

While it may seem overwhelming at first, there are a lot of resources available both on official website and on community blogs about setting up those tools. Also, the specific configuration details may change from platform to platform so don't hesitate to play around with it and be sure everything is installed correctly before to move on.

API 1: Inventory
Definition of the Inventory API
This API will be a CRUD (Create, Read, Update, Delete) RESTful API. It will use a PostgreSQL database. It will provide information about the movies present in the inventory and allow users to do basic operations on it.

A common way to do so is to use Flask which is a popular Python web framework. We will couple it with SQLAlchemy, an ORM which will abstract and simplify the interactions between our API and the database.

Here are the endpoints with the possible HTTP requests:

/api/movies: GET, POST, DELETE
/api/movies/:id: GET, PUT, DELETE
Some details about each one of them:

GET /api/movies retrieve all the movies.

GET /api/movies?title=[name] retrieve all the movies with name in the title.

POST /api/movies create a new product entry.

DELETE /api/movies delete all movies in the database.

GET /api/movies/:id retrieve a single movie by id.

PUT /api/movies/:id update a single movie by id.

DELETE /api/movies/:id delete a single movie by id.

The API should work on http://localhost:8080/.

Defining the Database
For the database we will use PostgreSQL. The database will be called movies_db.

The movies table will contain the following columns:

id: auto-generated unique identifier.
title: the title of the movie.
description: the description of the movie.
Testing the Inventory API
In order to test the correctness of your API you should use Postman or a similar tool. You have to create one or more tests for every endpoint and then export the configuration, so you will be able to reproduce the tests on different machines easily.

The configuration will be checked during the audit.

API 2: Billing
Definition of the billing API
This API will only receive messages through RabbitMQ, specifically it will consume messages on the queue billing_queue. The message it receives are going to be a "stringified" JSON object as in this example:

{
  "user_id": "3",
  "number_of_items": "5",
  "total_amount": "180"
}
It will parse the message and create a new entry in the billing_db database. It will also acknowledge the RabbitMQ queue that the message has been processed. When the API is started it will take and process all messages present in the queue.

Take a look into pika Python library for an easy way to interface with RabbitMQ.

Defining the Database
For the database we will use PostgreSQL here as well. The database will be called billing_db.

The orders table will contain the following columns:

id: auto-generated unique identifier.
user_id: the id of the user making the order.
number_of_items: the number of items included in the order.
total_amount: the total cost of the order.
Testing the Billing API
To test this API here are some steps:

Publish a message directly to the billing_queue in RabbitMQ using its UI or CLI.
When the Billing API is running the orders should appear instantaneously in the orders table in the billing_db database.
When the Billing API is not running the queries to the API Gateway should still return success but the orders table in the billing_db database won't be updated.
When the Billing API is started again the unfulfilled messages should be processed and the orders table in the billing_db database should be updated.
The API Gateway
The Gateway will take care of routing the requests to the appropriate service using the right protocol (it could be HTTP for the Inventory API or RabbitMQ for the Billing API).

Interfacing with Inventory API
The gateway will route all requests to /api/movies at the API 1, without any need to check the information passed through it. It will return the exact response received by the API1.

Interfacing with Billing API
The gateway will receive POST requests from api/billing and send a message using RabbitMQ in a queue called billing_queue. The content of the message will be the POST request body stringified with JSON.stringify. The Gateway should be able to send messages to queue even if the API 2 is not running. When the API2 will be started it should be able to process that message and send an acknowledgement back.

An example of POST request to http://[API_GATEWAY_URL]:[API_GATEWAY_PORT]/api/billing/:

{
  "user_id": "3",
  "number_of_items": "5",
  "total_amount": "180"
}
Upon successful processing, you can expect a response message such as "Message posted" or a similar acknowledgment.

Remember to set up Content-Type: application/json for the body of the request.

Documenting the API
Good documentation is a very critical feature of every API. By design the APIs are meant for others to use, so there have been very good efforts to create standard and easy to implement ways to document it.

As an introduction to the art of great documentation you must create an OpenAPI documentation file for the API Gateway. There are many different ways to do so, a good start could be using SwaggerHub with at least a meaningful description for each endpoint. Feel free to implement any extra feature as you see fit.

You must also create a README.md file at the root of your project with detailed instructions on how to build and run your infrastructure and which design choices you made to structure it.

Virtual Machines
General overview
You will Vagrant to set up three different VMs in order to test the interactions and correctness of responses between your APIs infrastructure.

Vagrant is an open-source software that helps you create and manage virtual machines. With Vagrant, you can create a development environment that is identical to your production environment, which makes it easier to develop, test, and deploy your applications.

Your VMs will be structured as follows:

gateway-vm: This VM will only contain the api-gateway.
inventory-vm: This VM will contain the inventory-app API and the database movies_db.
billing-vm: This VM will contain the billing-app API, database orders and RabbitMQ.
Vagrant is designed for development and should not be used in production environments.

Environment variables
To simplify the building process, it's recommended to define essential variables in a .env file. This approach facilitates the modification or update of critical information such as URLs, passwords, usernames and so on.

For this exercise, consider listing all required environment variables in the README.md file. Once you have these variables identified, create a .env file with the necessary credentials.

These variables will be utilized by Vagrant and distributed across the various microservices to centralize the credentials.

Your .env file should contain all the necessary credentials and none of the microservices should have any credential hard coded in the source code.

For the purpose of this exercise, the .env file must be included in your repository, in real-world scenarios, it's crucial to avoid including sensitive data in repositories to prevent potential leaks.

Configuration of the VMs
You will have a Vagrantfile which will create and start the three VMs. It will import the environment variables and pass them through each API.
You will have a scripts/ directory which will store all the scripts you may want to run in order to install the necessary tools on each VM. Those scripts may also be very useful for setting up the databases.
Your configuration will work properly for the following commands (executed from the root of the project):

vagrant up: Starts all the VMs.
vagrant status: Shows the status for all the VMs.
vagrant ssh <vm-name>: Will let you access the VM through SSH.
Manage Your Python applications with PM2
PM2 is a process manager for Node.js applications that makes it easy to manage and scale your application. It is designed to keep your application running continuously, even in the event of an unexpected failure.

PM2 can be used to start, stop, and list Node.js applications, as well as monitor their resource usage and log output.

Additionally, PM2 provides a number of features for managing multiple applications, such as load balancing and automatic restarts.

In our situation we will use it mainly to test resilience for messages sent to the Billing API when the API is not up and running.

After entering in your VM via SSH you may run the following commands:

sudo pm2 list: List all running applications.
sudo pm2 stop <app_name>: Stop a specific application.
sudo pm2 start <app_name>: Start a specific application.
Project organization
README.md
As a good exercise and a helpful tool it is required for you to deliver a README.md describing the project.

The idea of a README.md is to give in few lines enough context about a project to understand what is it about and how to run it.

This file should include instructions to run and test the project, it should also give a brief and clear overview of the stack used to build it.

Overall file structure
You can organize your internal file structure as you prefer. That said here is a common way to structure this kind of projects that may help you:

.
├── README.md
├── config.yaml
├── .env
├── scripts
│   └── [...]
├── srcs
│   ├── api-gateway
│   │   ├── app
│   │   │   ├── __init__.py
│   │   │   └── ...             // Other python files
│   │   ├── requirements.txt
│   │   └── server.py
│   ├── billing-app
│   │   ├── app
│   │   │   ├── __init__.py
│   │   │   └── ...             // Other python files
│   │   ├── requirements.txt
│   │   └── server.py
│   └── inventory-app
│   │   ├── app
│   │   │   ├── __init__.py
│   │   │   └── ...             // Other python files
│       ├── requirements.txt
│       └── server.py
└── Vagrantfile
When testing and before automating it through the VM build you should be able to start the API Gateway and the two APIs by using the command python server.py inside their respective directories.

As a best practice, you should develop your APIs using separates python virtual environments to isolate the requirements needed for each API. You can use venv or any equivalent tool.

If you decide to use a different structure for your project remember you should be able to explain and justify your decision during the audit.

As a best practice it is strongly advised to add venv/ to your .gitignore in order not to upload useless files into your git repository (they will be auto-generated during the build process).


# crud-master

Pour plus d'informations sur ce projet
[ici](https://github.com/01-edu/public/blob/master/subjects/devops/crud-master/README.md).

## Configuration

Pour pouvoir exécuter cette application, les programmes suivants doivent être installés sur votre machine
programmes suivants installés sur votre machine :

- [Vagrant](https://developer.hashicorp.com/vagrant/docs/installation).
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads).

Pour interagir avec l'application, il est recommandé d'installer les programmes suivants, ou tout autre programme équivalent : [Vagrant]().
suivants, ou tout autre programme équivalent :

- [Postman](https://www.postman.com/downloads/), ou tout autre outil permettant de
  pour tester par programme les points d'extrémité de l'API.
- [DBeaver](https://dbeaver.io/download/), ou tout autre outil permettant d'interagir et de visualiser le contenu d'une base de données SQL.
  d'interagir et de visualiser le contenu d'une base de données SQL.

  Pour lancer l'application, suivez les instructions ci-dessous :

- Créez un fichier `.env` à la racine du dossier du projet comme dans l'exemple
  exemple fourni. Vous pouvez simplement `cp .env.example .env`
- Installez le plugin _vagrant-env_ en exécutant : `vagrant plugin install vagrant-env`
- Exécutez la commande `vagrant up` pour créer toutes les VMs - cela peut prendre un certain temps en fonction des ressources de votre machine locale.
  Cela peut prendre un certain temps en fonction des ressources de votre machine locale.
- Interagissez avec le cluster de VMs en utilisant Postman, `curl` ou tout autre outil de votre choix.
  de votre choix. Il est possible de voir l'adresse IP de la passerelle API et le port
  dans les fichiers [`config.yaml`](./config.yaml) et [`.env`](./.env).

Pour vérifier que tout fonctionne comme prévu :

- Vérifiez que toutes les machines virtuelles fonctionnent

```console
$ vagrant status
État actuel des machines :

BillingVM en cours d'exécution (virtualbox)
InventoryVM en cours d'exécution (virtualbox)
GatewayVM en cours d'exécution (virtualbox)

Cet environnement représente plusieurs machines virtuelles. Les VM sont toutes répertoriées
ci-dessus avec leur état actuel. Pour plus d'informations sur une VM spécifique
VM spécifique, lancez `vagrant status NOM`.
```

(vous devriez voir le même message ou un message similaire)

- Vérifiez que votre API Gateway est capable de recevoir des requêtes HTTP. Par exemple,
  vous devriez pouvoir reproduire un flux de travail similaire (l'adresse IP et le port doivent être
  être ceux définis dans votre configuration) :

``console
$ curl -X POST -H « Content-Type : application/json » \N -d '{« titre » : 1))
    -d '{« title » : « film », “description” : « intrigue merveilleuse"}' \
    192.168.56.30:3000/api/movies
{« message » : « movie movie inserted successfully »}
$ curl -s 192.168.56.30:3000/api/movies | jq
{
  « films » : [
    {
      « description » : « merveilleuse intrigue »,
      « id » : 3,
      « title » : « film »
    }
  ]
}
$ curl -X DELETE 192.168.56.30:3000/api/movies
{« message » : « tous les films ont été supprimés avec succès »}
$ curl 192.168.56.30:3000/api/movies
{« films » :[]}
$
```



Check more information about this project
[here](https://github.com/01-edu/public/blob/master/subjects/devops/crud-master/README.md).

## Setup

In order to be able to run this application you need to have the following
programs installed on your machine:

- [Vagrant](https://developer.hashicorp.com/vagrant/docs/installation).
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads).

To interact with the application, it is recommended to install the following
programs, or any equivalent ones:

- [Postman](https://www.postman.com/downloads/), or any other tool to
  programmatically test API endpoints.
- [DBeaver](https://dbeaver.io/download/), or any other tool to interact and
  visualize the content of a SQL database.

To launch the application, follow the below instructions:

- Create a `.env` file in the root of the project folder as the example
  provided. You can simply `cp .env.example .env`
- Install _vagrant-env_ plug in running: `vagrant plugin install vagrant-env`
- Run the command `vagrant up` to create all the VMs - this might take a while
  depending on the resources of your local machine.
- Interact with the VM cluster using Postman, `curl` or any other tool of your
  choice. It is possible to see the IP address of the API Gateway and the port
  in the [`config.yaml`](./config.yaml) and [`.env`](./.env) files

To check if everything is working as expected:

- Check that all the VMs are running

```console
$ vagrant status
Current machine states:

BillingVM                 running (virtualbox)
InventoryVM               running (virtualbox)
GatewayVM                 running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

(you should see the same message or a similar one)

- Check that your API Gateway is able to receive HTTP requests. For example,
  you should be able to replicate a similar workflow (IP address and port must
  be the ones defined in your configuration):

```console
$ curl -X POST -H "Content-Type: application/json" \
    -d '{"title": "movie", "description": "wonderful plot"}' \
    192.168.56.30:3000/api/movies
{"message":"movie movie inserted successfully"}
$ curl -s 192.168.56.30:3000/api/movies | jq
{
  "movies": [
    {
      "description": "wonderful plot",
      "id": 3,
      "title": "movie"
    }
  ]
}
$ curl -X DELETE 192.168.56.30:3000/api/movies
{"message":"all movies deleted successfully"}
$ curl 192.168.56.30:3000/api/movies
{"movies":[]}
$
```





