## STARTING SETUP
creo la network in modo che i container possano comunicare tra loro senza esporre le porte

- docker network create goals-net

## BUILD IMAGES 

- docker build -t  goals-node .
- docker build -t  goals-react .

## RUN CONTAINERS
Starto i container utilizzando la network creata in precedenza 

- docker run --name mongodb --rm -d --network goals-net mongo
- docker run --name goals-backend --rm -d --network goals-net -p 8080:8080 goals-node
- docker run --name goals-frontend --rm -d -it -p 3000:3000 goals-react

Devo utilizzare -it quando starto il frontend, configurazione specifica di un'applicazione react.
A differenza di un'applicazione node, che runna in un server, le applicazioni react runnano in un browser (è solo il development server che runna in un server).
Infatti non uso il comando --network, è inutile ai fini dell'app react.

Per il frontend pubblico la porta perchè io voglio interagire con l'applicazione dal mio host.

Per il backend pubblico la porta perchè l'applicazione react runna nel browser NON nel container. Questo vuol dire che non c'è 
un docker engine in grado di risolvere l'indirizzo e quindi il frontend deve puntare a un vero URL; di conseguenza il backend deve esporre
la porta per permettere la comunicazione in ingresso dall'applicazione react.

## DATA PERSISTENCE MONGODB

<img src="https://miro.medium.com/v2/resize:fit:828/format:webp/1*f-nRNKRsQWT7_uLX9PuMEA.png">

Se stoppo il container mongodb e lo rimuovo, ovviamente i dati vengono cancellati

Risolvo con i volumes, cioè copiando i dati . Per fare questo devo sapere dove mongo li salva.
Da documentazione https://hub.docker.com/_/mongo :

1. Create a data directory on a suitable volume on your host system, e.g. /my/own/datadir.

2. Start your mongo container like this: 
   $ docker run --name some-mongo -v /my/own/datadir:/data/db -d mongo

Quindi per startare il container mongo:

- docker run --name mongodb --rm -d --network goals-net **-v data:/data/db** mongo

Docker crea il volume "data" (non sappiamo dove ma non ci interessa, altrimenti usa bind mount) e copia tutto quello presente in /data/db.
Se il volume esiste già, docker caricherà in /data/db tutto quello già presente.

## MONGODB SECURITY

Da documentazione https://hub.docker.com/_/mongo :

"...
MONGO_INITDB_ROOT_USERNAME, MONGO_INITDB_ROOT_PASSWORD
These variables, used in conjunction, create a new user and set that user's password. ""

Quindi per startare il container mongo con auth e data persistence:

- docker run \
    --name mongodb \
    --rm -d --network goals-net \
    -v data:/data/db \
    -e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
    -e MONGO_INITDB_ROOT_PASSWORD=mongosecret \
    mongo

## BACKEND DATA LOG PERSISTENCY

Invece che un named volume uso un bind mount in modo da essere in grado di recuperare i dati anche sul mio host
Per il live update del source code sul container, sempre un bind mount. 
NB: il bind mount richiede il path completo (ovviamente cambiare il path)

Quindi per startare il container backend con live code reload e data persistence:

- docker run \
 --name goals-backend --rm -d --network goals-net -p 8080:8080 \
  **-v "C:\Users\lorenzo.grandi\Downloads\multi-01-starting-setup\multi-01-starting-setup\backend:/app"** \
  **-v "C:\Users\lorenzo.grandi\Downloads\multi-01-starting-setup\multi-01-starting-setup\backend:/app/logs"** \
  **-v /app/node_modules** \
 goals-node

Con l'istruzione -v /app/node_modules dico a docker di salvare in un volume anonimo il node_modules, in modo da non sovrascriverlo nel container.

## DYNAMIC ENVIRONMENT VARIABLES ASSIGNMENT

Ho definito nel Dockerfile come variabili d'ambiente le credenziali per accedere a mongodb con dei valori di default.
Definisco in fase di start del container quelli effettivi 

- docker run \
  --name goals-backend --rm -d --network goals-net -p 8080:8080 \
  -v "C:\Users\lorenzo.grandi\Downloads\multi-01-starting-setup\multi-01-starting-setup\backend:/app" \
  -v "C:\Users\lorenzo.grandi\Downloads\multi-01-starting-setup\multi-01-starting-setup\backend:/app/logs" \
  -v /app/node_modules \
  **-e MONGODB_USERNAME=mongoadmin** \
  **-e MONGODB_PASSWORD=mongosecret** \
  goals-node

## REACT APPLICATION LIVE CODE RELOADING

Dato che nella mia applicazione di frontend è solo la cartella src che contiene codice utile per il live code reloading
è solo su quella che farò il bind mount (non ho bisogno di nodemon perchè il live code reloading è nativo in react)

 - docker run 
   --name goals-frontend --rm -d -it -p 3000:3000 \
   **-v "C:\Users\lorenzo.grandi\Downloads\multi-01-starting-setup\multi-01-starting-setup\frontend\src:/app/src"** \
   **-v /app/node_modules** \
   goals-react

NB: se si usa Docker for Windows e WSL2 il live code reloading non funziona, assicurarsi di usare la variabile d'ambiente CHOKIDAR_USEPOLLING=true:

- docker run
  --name goals-frontend --rm -d -it -p 3000:3000 \
  -v "C:\Users\lorenzo.grandi\Downloads\multi-01-starting-setup\multi-01-starting-setup\frontend\src:/app/src" \
  -v /app/node_modules \
  **-e CHOKIDAR_USEPOLLING=true** \
  goals-react