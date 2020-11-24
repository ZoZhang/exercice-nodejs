### Exercice NodeJs

L'objectif est de pratiquer les usages basiques sur NodeJs afin de mettre les niveaux fondamentaux.


#### Installation

Vous pouvez télécharger / installer le package avec le lien suivant

[https://nodejs.org/en/download/](https://nodejs.org/en/download/)


#### Exercice - Simple Projet NodeJs


**1. Initialisation d'un projet** 

Vous créez un dossier ***"tp-nodejs"***, entrez dedans et taper la commande `npm init` pour initialise un fichier d'une description du projet `package.json`


```
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help init` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (tp-nodejs)
version: (1.0.0)
description:
git repository:
keywords:
author:
license: (ISC)
About to write to ~/tp-nodejs/package.json:

{
  "name": "tp-nodejs",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "description": ""
}

Is this OK? (yes)
```

**2. Création d'une structure du projet** 

Vous devez créer les dossiers et le fichier comme l'arbre suivant

```
├─models/  
├─public/
├─routes/
├─views/
├─index.js
└─package.json
```

Normalement, nous mettons les fichiers comme ci-dessous:

*  `models` 	- les fichiers pour l'exploitation de la base de données <br/>
*  `public` 	- les fichiers statiques comme css、images etc. <br/>
*  `routes`  -  les routes du projet <br/>
*  `views`   - les fichiers templates  du projet <br/>
*  `index.js` - le fichier pricipale d'entre du projet 

**3. Installation les dépendances du projet** 

Installez le framework express afin de lancer un projet *http* rapide<br/>
 `npm install express --save`

**4. Lance un projet http rapide**

Tapez ces codes ci-dessous dans le fichier `index.js`

````
var express = require('express');
var app = express();

// mette `public` à la répertoire statiques  
app.use('/public', express.static('public'));

app.get('/', function(res, rep) {
    rep.send('Hello, word!');
});

var server = app.listen(3000, function() {

console.log(server.address());

    var port = server.address().port
    
    console.log("Node.JS server démarré, consulté sur un navigateur： http://localhost:%s", port)

})
````

Et puis, tapez ce commande dans le terminale  `node index.js`

Lorsque le serveur `http` démarre, vous pouvez consulter le site dans un navigateur sur l'adresse : [http://localhost:3000](http://localhost:3000)

Jusqu'à cet étape, vous avez bien réussir à créer une page statique sur NodeJs.

**5. Utilise les fichiers templates**

Maintenant, nous allons faire un projet en utilisant les fichiers templates.

Créez les deux fichiers dans le dossier `views` 

```
├── homePage.ejs
└── websocketPage.ejs

```

Mettez le code suivant dans les deux fichiers

**homePage.ejs**

```
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title>Home</title>
  </head>
  <body>
     <h1>Je suis sur Home Page</h1>
  </body>
</html>
```

**websocketPage.ejs**

```
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title>Websocket Page</title>
  </head>
  <body>
     <h1>Je suis sur websocket Page</h1>
  </body>
</html>
```

Ensuite, installez le module `ejs` en utilisant `npm install ejs --save`

et puis, ajoutez les codes suivants sur un bon endroit dans le fichier `index.js` et re-lance ce projet.

```
app.set("view engine", "ejs");
app.set("views", "./views");

app.get('/', function(res, rep) {
	rep.render("homePage");
});

app.get('/websocket', function(res, rep) {
	rep.render("websocketPage");
});
```

Finalement, vous pouvez voir deux pages différents sur les chemins du url: `/` et `/websocket`.

Si vous finissez cet exercice, vous pouvez continuer sur la partie suivante `websocket` 


**6. Lance un service websocket**

Tout d'abord, nous devons installer le module `websocket` par la commande suivante:
`npm install websocket --save`

Créez un fichier `websocket_server.js` et mettez le code suivant

```
// Node.js WebSocket server script
const http = require('http');
const WebSocketServer = require('websocket').server;

const port = 8899;
const server = http.createServer();

server.listen(port);

const wsServer = new WebSocketServer({
    httpServer: server
});


wsServer.on('request', function(request) {

    const connection = request.accept(null, request.origin);

    connection.sendUTF('Bienvenu mec !');

    connection.on('message', function(message) {

 		if (message.type === 'utf8') {

            console.log('Message reçu: ' + message.utf8Data);
            connection.sendUTF(message.utf8Data);

        } else  if (message.type === 'binary') {
            console.log('Message binaire reçu de ' + message.binaryData.length + ' Octets');
            connection.sendBytes(message.binaryData);
        }

    });

    connection.on('close', function(reasonCode, description) {
     	console.log((new Date()) + ' Peer ' + connection.remoteAddress + ' déconnecté.');
    });
});

console.log("websocket server démarré, vous pouvez se connecter sur l'adresse ws://127.0.0.1:%s", port)
```

Et après, lancez ce service en utilisant la commande `node websocket_server.js &` en dernier fond

**7. Test un service websocket**

Mettez le code ci-dessous dans le template `websocketPage .ejs`

```
<input type="text" id="input" placeholder="Message" />
<hr />
<pre id="output"></pre>

<script>
  var host   = 'ws://127.0.0.1:8899';
  var socket = null;
  var input  = document.getElementById('input');
  var output = document.getElementById('output');
  var print  = function (message) {
      var samp       = document.createElement('samp');
      samp.innerHTML = message + '\n';
      output.appendChild(samp);

      return;
  };

  input.addEventListener('keyup', function (evt) {
      if (13 === evt.keyCode) {
          var msg = input.value;

          if (!msg) {
              return;
          }

          try {
              socket.send(msg);
              input.value = '';
              input.focus();
          } catch (e) {
              console.log(e);
          }

          return;
      }
  });

  try {
      socket = new WebSocket(host);
      socket.onopen = function () {
          print('connexion est ouverte');
          input.focus();

          return;
      };
      socket.onmessage = function (msg) {
          print(msg.data);

          return;
      };
      socket.onclose = function () {
          print('connexion est fermée');

          return;
      };
  } catch (e) {
      console.log(e);
  }
</script>
```

Lancez le service `http` et ouvrez la page [http://localhost:3000/websocket](http://localhost:3000/websocket)

### **Qu'est ce que vous observez lorsque vous êtes sur la page websocket ?**

### **Essayez d'envoyer un message sur cette page et observez sur le terminal où vous lancez le service websocket.**

