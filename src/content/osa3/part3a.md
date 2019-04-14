---
mainImage: ../../images/part-3.svg
part: 3
letter: a
---

<div class="content">

<!-- Siirrämme tässä osassa fokuksen backendiin, eli palvelimella olevaan toiminnallisuuteen. -->
In this part our focus shifts towards the backend, that is towards implementing functionality on the server side of the stack.

<!-- Backendin toteutusympäristönä käytämme [Node.js](https://nodejs.org/en/):ää, joka on melkein missä vaan, erityisesti palvelimilla ja omalla koneellasikin toimiva, Googlen [chrome V8](https://developers.google.com/v8/) -Javascriptmoottoriin perustuva Javascriptin suoritusympäristö. -->
We will be building our backend on top of [NodeJS](https://nodejs.org/en/), which is a JavaScript runtime based on Google's [Chrome V8](https://developers.google.com/v8/) JavaScript engine.

<!-- Kurssimateriaalia tehtäessä on ollut käytössä Node.js:n versio <i>v8.10.0</i>. Huolehdi että omasi on vähintään yhtä tuore (ks. komentoriviltä _node -v_). -->
This course material was written with the version <i>v8.10.0</i> of Node.js. Please make sure that your version of Node is at least as new the version used in the material (you can check the version by running _node -v_ in the command line).

<!-- Kuten [osassa 1](/osa1/javascriptia) todettiin, selaimet eivät vielä osaa uusimpia Javascriptin ominaisuuksia ja siksi selainpuolen koodi täytyy kääntää eli <i>transpiloida</i> esim [babel](https://babeljs.io/):illa. Backendissa tilanne on kuitenkin toinen, uusin Node hallitsee riittävissä määrin myös Javascriptin uusia versioita, joten suoritamme Nodella suoraan kirjoittamaamme koodia ilman transpilointivaihetta. -->
As mentioned in [part 1](/osa1/javascriptia), browsers don't yet support the newest features of JavaScript and that is why the code running in the browser must be <i>transpiled</i> with e.g. [babel](https://babeljs.io/). The situation with JavaScript running in the backend is different. The newest version of Node supports a large majority of the latest features of JavaScript, so we can use the latest features without having to transpile our code.

<!-- Tavoitteenamme on tehdä [osan 2](/osa2) muistiinpanosovellukseen sopiva backend. Aloitetaan kuitenkin ensin perusteiden läpikäyminen toteuttamalla perinteinen "hello world"-sovellus. -->
Our goal is to implement a backend that will work with the notes application from [part 2](/osa2). However, let's start with the basics by implementing a classic "hello world" application.

<!-- **Huomaa**, että kaikki tässä osassa ja sen tehtävissä luotavat sovellukset eivät ole Reactia, eli emme käytä <i>create-react-app</i>-sovellusta tämän osan sovellusten rungon alustamiseen. -->
**Notice** that the applications and exercises in this part are not all React applications, and we will not use the <i>create-react-app</i> utility for initializing the project for this application.

<!-- Osassa 2 oli jo puhe [npm](/osa2#npm):stä, eli Javascript-projektien hallintaan liittyvästä, alunperin Node-ekosysteemistä kotoisin olevasta työkalusta.  -->
We had already mentioned [npm](/osa2#npm) back in part 2, which is tool used for managing JavaScript projects. In fact, npm originates from the Node ecosystem.

<!-- Mennään sopivaan hakemistoon ja luodaan projektimme runko komennolla _npm init_. Vastaillaan kysymyksiin sopivasti ja tuloksena on hakemiston juureen sijoitettu projektin tietoja kuvaava tiedosto <i>package.json</i> -->
Let's navigate to an appropriate directory, and create a new template for our application with the _npm init_ command. We will answer the questions presented by the utility, and the result will be an automatically generated <i>package.json</i> file at the root of the project, that contains information about the project.

```json
{
  "name": "notebackend",
  "version": "0.0.1",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Matti Luukkainen",
  "license": "MIT"
}
```

<!-- Tiedosto määrittelee mm. että ohjelmamme käynnistyspiste on tiedosto <i>index.js</i>. -->
The file defines e.g. the that the entry point of the application is the <i>index.js</i> file.

<!-- Tehdään kenttään <i>scripts</i> pieni lisäys: -->
Let's make a small change to the <i>scripts</i> object:

```bash
{
  // ...
  "scripts": {
    "start": "node index.js", // highlight-line
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  // ...
}
```

<!-- Luodaan sitten sovelluksen ensimmäinen versio, eli projektin juureen sijoitettava tiedosto <i>index.js</i> ja sille seuraava sisältö: -->
Next, let's create the first version of our application by adding an <i>index.js</i> file to the root of the project with the following code:

```js
console.log('hello world')
```

<!-- Voimme suorittaa ohjelman joko "suoraan" nodella, komentorivillä -->
We can run the program directly with Node from the command line:

```bash
node index.js
```

<!-- tai [npm scriptinä](https://docs.npmjs.com/misc/scripts) -->
Or we can run it as an [npm script](https://docs.npmjs.com/misc/scripts):

```bash
npm start
```

<!-- npm-skripti <i>start</i> toimii koska määrittelimme sen tiedostoon <i> package.json</i>  -->
The <i>start</i> npm script works because we defined it in the <i>package.json</i> file:

```bash
{
  // ...
  "scripts": {
    "start": "node index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  // ...
}
```

<!-- Vaikka esim. projektin suorittaminen onnistuukin suoraan käyttämällä komentoa _node index.js_, on npm-projekteille suoritettavat operaatiot yleensä tapana määritellä nimenomaan npm-skripteinä. -->
Even though the execution of the project works when it is started by calling _node index.js_ from the command line, it's customary for npm projects to execute such tasks as npm scripts.

<!-- Oletusarvoinen <i>package.json</i> määrittelee valmiiksi myös toisen yleisesti käytetyn npm-scriptin eli _npm test_. Koska projektissamme ei ole vielä testikirjastoa, ei _npm test_ kuitenkaan tee vielä muuta kuin suorittaa komennon -->
By default the <i>package.json</i> file also defines another commonly used npm script called <i>npm test</i>. Since our project does not yet have a testing library, the _npm test_ command simply executes the following command:

```bash
echo "Error: no test specified" && exit 1
```

<!-- ### Yksinkertainen web-palvelin -->
### Simple web server

<!-- Muutetaan sovellus web-palvelimeksi: -->
Let's change the application into a web server:

```js
const http = require('http')

const app = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' })
  res.end('Hello World')
});

const port = 3001
app.listen(port)
console.log(`Server running on port ${port}`)
```

<!-- Kun sovellus käynnistuu, konsoliin tulostuu -->
Once the application is running, we see the following message get printed to the console:

```bash
Server running on port 3001
```

<!-- Voimme avata selaimella osoitteessa <http://localhost:3001> olevan vaatimattoman sovelluksemme: -->
We can open our humble application in the browser by visiting the address <http://localhost:3001>:

![](../images/3/1.png)

<!-- Palvelin toimii itseasiassa täsmälleen samalla tavalla riippumatta urlin loppuosasta, eli myös sivun <http://localhost:3001/foo/bar> sisältö on sama. -->
In fact, the server works the same way regardless of the latter part of the URL. Also the address <http://localhost:3001/foo/bar> will display the same content.

<!-- **HUOM** jos koneesi portti 3001 on jo jonkun sovelluksen käytössä, aiheuttaa käynnistäminen virheen: -->
**NB** if the port 3001 is already in use by some other application, then starting the server will result in the following error message:

```bash
➜  hello npm start

> hello@1.0.0 start /Users/mluukkai/opetus/_2019fullstack-koodit/osa3/hello
> node index.js

Server running on port 3001
events.js:167
      throw er; // Unhandled 'error' event
      ^

Error: listen EADDRINUSE :::3001
    at Server.setupListenHandle [as _listen2] (net.js:1330:14)
    at listenInCluster (net.js:1378:12)
```

<!-- Sammuta portissa 3001 oleva sovellus (edellisessä osassa json-server käynnistettiin porttiin 3001) tai määrittele sovellukselle jokin toinen portti. -->
You have two options. Either shutdown the application using the port 3001 (the json-server in the last part of the material was using the port 3001), or use a different port for this application.

<!-- Tarkastellaan koodia hiukan. Ensimmäinen rivi -->
Let's take a closer look at first row of the code:

```js
const http = require('http')
```

<!-- ottaa käyttöön Noden sisäänrakennetun [web-palvelimen](https://nodejs.org/docs/latest-v8.x/api/http.html) määrittelevän moduulin. Kyse on käytännössä samasta asiasta, mihin olemme selainpuolen koodissa tottuneet hieman syntaksiltaan erilaisessa muodossa: -->

In the first row, the application imports Node's built-in [web server](https://nodejs.org/docs/latest-v8.x/api/http.html) module. This is practically what we have already been doing in our browser-side code, but with a slightly different syntax:

```js
import http from 'http'
```

<!-- Selaimen puolella käytetään (nykyään) ES6:n moduuleita, eli moduulit määritellään [exportilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) ja otetaan käyttöön [importilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import). -->
These days, code that runs in the browser uses ES6 modules. Modules are defined with an [export](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) and taken into use with an [import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import).

<!-- Node.js kuitenkin käyttää ns. [CommonJS](https://en.wikipedia.org/wiki/CommonJS)-moduuleja. Syy tälle on siinä, että Node-ekosysteemillä oli tarve moduuleihin jo kauan ennen kuin Javascript tuki kielen tasolla moduuleja. Node ei toistaiseksi tue ES-moduuleja, mutta tuki on todennäköisesti jossain vaiheessa [tulossa](https://nodejs.org/api/esm.html). -->
However, Node.js uses so-called [CommonJS](https://en.wikipedia.org/wiki/CommonJS) modules. The reason for this is that the Node ecosystem had a need for modules long before JavaScript supported them in the the language specification. At the time of writing Node does not support ES6 modules, but support for them [will be coming](https://nodejs.org/api/esm.html) somewhere down the road.

<!-- CommonJS-moduulit toimivat kohtuullisessa määrin samaan tapaan kuin ES6-moduulit, ainakin tämän kurssin tarpeiden puitteissa. -->
CommonJS modules function almost exactly like ES6 modules, at least as far as our needs in this course are concerned.

<!-- Koodi jatkuu seuraavasti: -->
The next chunk in our code looks like this:

```js
const app = http.createServer((request, response) => {
  response.writeHead(200, { 'Content-Type': 'text/plain' })
  response.end('Hello World')
})
```

<!-- koodi luo [http](https://nodejs.org/docs/latest-v8.x/api/http.html)-palvelimen metodilla _createServer_ web-palvelimen, jolle se rekisteröi <i>tapahtumankäsittelijän</i>, joka suoritetaan <i>jokaisen</i> osoitteen http:/localhost:3001 alle tulevan HTTP-pyynnön yhteydessä. -->
The code uses the _createServer_ method of the [http](https://nodejs.org/docs/latest-v8.x/api/http.html) module to create a new web server. An <i>event handler</i> is registered to the server, that is called <i>every time</i>  an HTTP request is made to the server's address http:/localhost:3001.

<!-- Pyyntöön vastataan statuskoodilla 200, asettamalla <i>Content-Type</i>-headerille arvo <i>text/plain</i> ja asettamalla palautettavan sivun sisällöksi merkkijono <i>Hello World</i>. -->
The request is responded to with the status code 200, with the <i>Content-Type</i> header set to <i>text/plain</i>, and the content of the site to be returned set to <i>Hello World</i>.

<!-- Viimeiset rivit sitovat muuttujaan _app_ sijoitetun http-palvelimen kuuntelemaan porttiin 3001 tulevia HTTP-pyyntöjä: -->
The last rows bind the http server assigned to the _app_ variable, to listen to HTTP requests sent to the port 3001:

```js
const PORT = 3001
app.listen(PORT)
console.log(`Server running on port ${PORT}`)
```

<!-- Koska tällä kurssilla palvelimen rooli on pääasiassa tarjota frontille JSON-muotoista "raakadataa", muutetaan heti palvelinta siten, että se palauttaa kovakoodatun listallisen JSON-muotoisia muistiinpanoja: -->
The primary purpose of the backend server in this course is to offer raw data in the JSON format to the frontend. For this reason, let's immediately change our server to return a hardcoded list of notes in the JSON format:

```js
const http = require('http')

// highlight-start
let notes = [
  {
    id: 1,
    content: 'HTML on helppoa',
    date: '2017-12-10T17:30:31.098Z',
    important: true,
  },
  {
    id: 2,
    content: 'Selain pystyy suorittamaan vain javascriptiä',
    date: '2017-12-10T18:39:34.091Z',
    important: false,
  },
  {
    id: 3,
    content: 'HTTP-protokollan tärkeimmät metodit ovat GET ja POST',
    date: '2017-12-10T19:20:14.298Z',
    important: true,
  },
]

const app = http.createServer((request, response) => {
  response.writeHead(200, { 'Content-Type': 'application/json' })
  response.end(JSON.stringify(notes))
})
// highlight-end

const port = 3001
app.listen(port)
console.log(`Server running on port ${port}`)
```

<!-- Käynnistetään palvelin uudelleen (palvelin sammutetaan painamalla _ctrl_ ja _c_ yhtä aikaa konsolissa) ja refreshataan selain. -->
Let's restart the server (you can shut the server down by holding _ctrl_ _c_ down at the same time in the console) and let's refresh the browser.

<!-- Headerin <i>Content-Type</i> arvolla <i>application/json</i> kerrotaan, että kyse on JSON-muotoisesta datasta. Muuttujassa _notes_ oleva taulukko muutetaan jsoniksi metodilla <em>JSON.stringify(notes)</em>. -->
The <i>application/json</i> value in <i>Content-Type</i> header informs the receiver that the data is in the JSON format. The _notes_ array gets transformed into JSON with the <em>JSON.stringify(notes)</em> method.

<!-- Kun avaamme selaimen, on tulostusasu sama kuin [osassa 2](/osa2#datan-haku-palvelimelta) käytetyn [json-serverin](https://github.com/typicode/json-server) tarjoamalla muistiinpanojen listalla: -->
When we open the browser, the displayed format is exactly the same as in [part 2](/osa2#datan-haku-palvelimelta) where we used [json-server](https://github.com/typicode/json-server) to serve the list of notes:

![](../images/3/2.png)

<!-- Voimme jo melkein ruveta käyttämään uutta backendiämme osan 2 muistiinpano-frontendin kanssa. Mutta vain _melkein_, sillä kun käynnistämme frontendin, tulee konsoliin virheilmoitus -->
We're nearly ready to start using our new backend with the notes application frontend from part 2. Nearly, but not quite. When we fire up the frontend we see the following error message in the console:

![](../assets/3/3.png)

<!-- Syy virheelle selviää pian, parantelemme kuitenkin ensin koodia muilta osin. -->
We will look into this error soon, but first we will improve our code in other ways.

### Express

<!-- Palvelimen koodin tekeminen suoraan Noden sisäänrakennetun web-palvelimen [http](https://nodejs.org/docs/latest-v8.x/api/http.html):n päälle on mahdollista, mutta työlästä, erityisesti jos sovellus kasvaa hieman isommaksi. -->
Implementing our server code directly with Node's built-in [http](https://nodejs.org/docs/latest-v8.x/api/http.html) web server is possible but cumbersome, especially once the application grows in size.

<!-- Nodella tapahtuvaa web-sovellusten ohjelmointia helpottamaan onkin kehitelty useita _http_:tä miellyttävämmän ohjelmointirajapinnan tarjoamia kirjastoja. Näistä ylivoimaisesti suosituin on [express](http://expressjs.com). -->
Many libraries have been developed to ease server side development with Node, by offering a more pleasing interface to work with than the built-in http module. By far the most popular library intended for this purpose is [express](http://expressjs.com).

<!-- Otetaan express käyttöön määrittelemällä se projektimme riippuvuudeksi komennolla -->
Let's take express into use by defining it as a project dependency with the command:

```bash
npm install express --save
```

<!-- Riippuvuus tulee nyt määritellyksi tiedostoon <i>package.json</i>: -->
The dependency is also added to our <i>package.json</i> file:

```json
{
  // ...
  "dependencies": {
    "express": "^4.16.4"
  }
}

```

<!-- Riippuvuuden koodi asentuu kaikkien projektin riippuvuuksien tapaan projektin juuressa olevaan hakemistoon <i>node\_modules</i>. Hakemistosta löytyy expressin lisäksi suuri määrä muutakin tavaraa -->
The source code for the dependency is installed to the <i>node\_modules</i> directory located in the root of the project. In addition to express, you can find a great amount of other dependencies in the directory:

![](../images/3/4.png)

<!-- Kyseessä ovat expressin riippuvuudet ja niiden riippuvuudet ym... eli projektimme [transitiiviset riippuvuudet](https://lexi-lambda.github.io/blog/2016/08/24/understanding-the-npm-dependency-model/). -->
These are in fact the dependencies of the express library, and the dependencies of all of its dependencies, and so forth. These are called the [transitive dependencies](https://lexi-lambda.github.io/blog/2016/08/24/understanding-the-npm-dependency-model/) of our project.

<!-- Projektiin asentui expressin versio 4.16.4. Mitä tarkoittaa </i>package.json:issa</i> versiomerkinnän edessä oleva väkänen, eli miksi muoto on -->
The version 4.16.4. of express was installed to our project. What does the caret at the end of the version number in <i>package.json</i> mean?

```json
"express": "^4.16.4"
```

<!-- npm:n yhteydessä käytetään ns. [semanttista versiointia](https://docs.npmjs.com/getting-started/semantic-versioning). -->
The versioning model used in npm is called [semantic versioning](https://docs.npmjs.com/getting-started/semantic-versioning).

<!-- Merkintä <i>^4.16.4</i> tarkoittaa, että jos/kun projektin riippuvuudet päivitetään, asennetaan expressistä versio, joka on vähintään <i>4.16.4</i>, mutta asennetuksi voi tulla versio, jonka <i>patch</i> eli viimeinen numero tai <i>minor</i> eli keskimmäinen numero voi olla suurempi. Pääversio eli <i>major</i> täytyy kuitenkin olla edelleen sama. -->
The caret at the end of <i>^4.16.4</i> means, that if and when the dependencies of a project are updated, the version of express that is installed will be at least <i>4.16.4</i>. However, the installed version of express can also be one that has a larger <i>patch</i> number (the last number), or a larger <i>minor</i> number (the middle number). The major version of the library indicated by the first <i>major</i> number must be the same.

<!-- Voimme päivittää projektin riippuvuudet komennolla -->
We can update the dependencies of the project with the command:

```bash
npm update
```

<!-- Vastaavasti jos aloitamme projektin koodaamisen toisella koneella, saamme haettua ajantasaiset, <i>package.json</i>:in määrittelyn kanssa yhteensopivat riippuvuudet komennolla -->
Likewise, if we start working on the project on another computer, we can install all up-to-date dependencies of the project defined in <i>package.json</i> with the command:

```bash
npm install
```

<!-- Jos riippuvuuden <i>major</i>-versionumero ei muutu, uudempien versioiden pitäisi olla [taaksepäin yhteensopivia](https://en.wikipedia.org/wiki/Backward_compatibility), eli jos ohjelmamme käyttäisi tulevaisuudessa esim. expressin versiota 4.99.175, tässä osassa tehtävän koodin pitäisi edelleen toimia ilman muutoksia. Sen sijaan tulevaisuudessa joskus julkaistava express 5.0.0. [voi sisältää](https://expressjs.com/en/guide/migrating-5.html) sellaisia muutoksia, että koodimme ei enää toimisi. -->
If the <i>major</i> number of a dependency does not change, then the newer versions should be [backwards compatible](https://en.wikipedia.org/wiki/Backward_compatibility). This means that if our application happened to use version 4.99.175 of express in the future, then all the code implemented in this part would still have to work without making changes to the code. In contrast, the future 5.0.0. version of express [may contain](https://expressjs.com/en/guide/migrating-5.html) changes, that would cause our application to no longer work.

### Web ja express

<!-- Palataan taas sovelluksen ääreen ja muutetaan se muotoon: -->
Let's get back to our application and make the following changes:

```js
const express = require('express')
const app = express()

let notes = [
  ...
]

app.get('/', (req, res) => {
  res.send('<h1>Hello World!</h1>')
})

app.get('/notes', (req, res) => {
  res.json(notes)
})

const PORT = 3001
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`)
})
```

<!-- Jotta sovelluksen uusi versio saadaan käyttöön, on sovellus uudelleenkäynnistettävä. -->
In order to get the new version of our application into use we have to restart the application.

<!-- Sovellus ei muutu paljoa. Heti alussa otetaan käyttöön _express_, joka on tällä kertaa <i>funktio</i>, jota kutsumalla luodaan muuttujaan _app_ sijoitettava express-sovellusta vastaava olio: -->
The application did not change a whole lot. Right at the beginning of our code we're importing _express_, which this time is a <i>function</i> that is used to create an express application stored in the _app_ variable:

```js
const express = require('express')
const app = express()
```

<!-- Seuraavaksi määritellään sovellukselle kaksi <i>routea</i>. Näistä ensimmäinen määrittelee tapahtumankäsittelijän, joka hoitaa sovelluksen juureen eli polkuun <i>/</i> tulevia HTTP GET -pyyntöjä: -->
Next, we define two <i>routes</i> to the application. The first one defines an event handler, that is used to handle HTTP GET requests made to the application's <i>/</i> root:

```js
app.get('/', (request, response) => {
  response.send('<h1>Hello World!</h1>')
})
```

<!-- Tapahtumankäsittelijäfunktiolla on kaksi parametria. Näistä ensimmäinen eli [request](http://expressjs.com/en/4x/api.html#req) sisältää kaikki HTTP-pyynnön tiedot ja toisen parametrin [response](http://expressjs.com/en/4x/api.html#res):n avulla määritellään, miten pyyntöön vastataan. -->
The event handler function accepts two parameters. The first [request](http://expressjs.com/en/4x/api.html#req) parameter contains all of the information of the HTTP request, and the second [response](http://expressjs.com/en/4x/api.html#res) parameter is used to define how the request is responded to.

<!-- Koodissa pyyntöön vastataan käyttäen _response_-olion metodia [send](http://expressjs.com/en/4x/api.html#res.send), jonka kutsumisen seurauksena palvelin vastaa HTTP-pyyntöön lähettämällä selaimelle vastaukseksi _send_:in parametrina olevan merkkijonon <code>\<h1>Hello World!\</h1></code>. Koska parametri on merkkijono, asettaa express vastauksessa <i>content-type</i>-headerin arvoksi <i>text/html</i>, statuskoodiksi tulee oletusarvoisesti 200.  -->
In our code, the request is answered by using the [send](http://expressjs.com/en/4x/api.html#res.send) method of the _response_ object. Calling the method makes the server respond to the HTTP request by sending a response containing the string <code>\<h1>Hello World!\</h1></code>, that was passed to the _send_ method. Since the parameter is a string, express automatically sets the value of the <i>Content-Type</i> header to be <i>text/html</i>. The status code of the response defaults to 200.

<!-- Asian voi varmistaa konsolin välilehdeltä <i>Network</i> -->
We can verify this from the <i>Network</i> tab in developer tools:

![](../images/3/5.png)

<!-- Routeista toinen määrittelee tapahtumankäsittelijän, joka hoitaa sovelluksen polkuun <i>notes</i> tulevia HTTP GET -pyyntöjä: -->
The second route defines an event handler, that handles HTTP GET requests made to the <i>notes</i> path of the application:

```js
app.get('/notes', (request, response) => {
  response.json(notes)
});
```

<!-- Pyyntöön vastataan _response_-olion metodilla [json](http://expressjs.com/en/4x/api.html#res.json), joka lähettää HTTP-pyynnön vastaukseksi parametrina olevaa Javascript-olioa eli taulukkoa _notes_ vastaavan JSON-muotoisen merkkijonon. Express asettaa headerin <i>Content-type</i> arvoksi <i>application/json</i>. -->
The request is responded to with the [json](http://expressjs.com/en/4x/api.html#res.json) method of the _response_ object. Calling the method will send the __notes__ array that was passed to it as a JSON formatted string. Express automatically sets the <i>Content-Type</i> header with the appropriate value of <i>application/json</i>.

![](../images/3/6.png)

<!-- Pieni huomio JSON-muodossa palautettavasta datasta. -->
Quick observation about data send in the JSON format.

<!-- Aiemmassa, pelkkää Nodea käyttämässä versiossa, jouduimme muuttamaan palautettavan datan json-muotoon metodilla _JSON.stringify_: -->
In the earlier version where we were only using Node, we had to transform the data into the JSON format with the _JSON.stringify__ method:

```js
response.end(JSON.stringify(notes))
```

<!-- Expressiä käytettäessä tämä ei ole tarpeen, sillä muunnos tapahtuu automaattisesti. -->
With express this is no longer required, because this transformation happens automatically.

<!-- Kannattaa huomata, että [JSON](https://en.wikipedia.org/wiki/JSON) on merkkijono, eikä Javascript-olio kuten muuttuja _notes_. -->
It's worth noting, that [JSON](https://en.wikipedia.org/wiki/JSON) is a string, and not a JavaScript object like the value assigned to _notes_.

<!-- Seuraava interaktiivisessa [node-repl](https://nodejs.org/docs/latest-v8.x/api/repl.html):issä suoritettu kokeilu havainnollistaa asiaa: -->
The experiment shown below illustrates this point:

![](../assets/3/5.png)

<!-- Saat käynnistettyä interaktiivisen node-repl:in kirjoittamalla komentoriville _node_. Esim. joidenkin komentojen toimivuutta on koodatessa kätevä tarkastaa konsolissa, suosittelen! -->
The experiment above was done in the interactive [node-repl](https://nodejs.org/docs/latest-v8.x/api/repl.html). You can start the interactive node-repl by typing in _node_ in the command line. The repl is particularly useful for testing how commands work while you're writing application code. I highly recommend this!

### nodemon

<!-- Jos muutamme sovelluksen koodia, joudumme uudelleenkäynnistämään sovelluksen (eli ensin sammuttamaan konsolista _ctrl_ ja _c_ ja sitten käynnistämään uudelleen), jotta muutokset tulisivat voimaan. Verrattuna Reactin mukavaan workflowhun, missä selain päivittyi automaattisesti koodin muuttuessa tuntuu uudelleenkäynnistely kömpelöltä. -->
If we make changes to the application's code we have to restart the application in order to see the changes. We restart the application by first shutting it down by typing _ctrl_ _c_ and then restarting the application. Compared to the convenient workflow in React where the browser automatically reloaded after changes were made, this feels slightly cumbersome.

<!-- Ongelmaan ratkaisu on [nodemon](https://github.com/remy/nodemon): -->
The solution to this problem is [nodemon](https://github.com/remy/nodemon): 

> <i>nodemon will watch the files in the directory in which nodemon was started, and if any files change, nodemon will automatically restart your node application.</i>

<!-- Asennetaan nodemon määrittelemällä se <i>kehitysaikaiseksi riippuvuudeksi</i> (development dependency) komennolla: -->
Let's install nodemon by defining it as a <i>development dependency</i> with the command:

```bash
npm install --save-dev nodemon
```

<!-- Tiedoston <i>package.json</i> sisältö muuttuu seuraavasti: -->
The contents of <i>package.json</i> have also changed:

```json
{
  //...
  "dependencies": {
    "express": "^4.16.4"
  },
  "devDependencies": {
    "nodemon": "^1.18.9"
  }
}
```

<!-- Jos nodemon-riippuvuus kuitenkin meni sovelluksessasi normaaliin "dependencies"-ryhmään, päivitä <i>package.json</i> manuaalisesti vastaamaan yllä näkyvää (versiot kuitenkin säilyttäen). -->
If you accidentally used the wrong command and the nodemon dependency was added under "dependencies" instead of "devDependencies", then manually change the contents of <i>package.json</i> to match what is shown above.

<!-- Kehitysaikaisilla riippuvuuksilla tarkoitetaan työkaluja, joita tarvitaan ainoastaan sovellusta kehitettäessä, esim. testaukseen tai sovelluksen automaattiseen uudelleenkäynnistykseen kuten <i>nodemon</i>. -->
By development dependencies we are referring to tools that are needed only during the development of the application, e.g. for testing or automatically restarting the application like <i>nodemon</i>.

<!-- Kun sovellusta suoritetaan tuotantomoodissa, eli samoin kun sitä tullaan suorittamaan tuotantopalvelimella (esim. Herokussa, mihin tulemme kohta siirtämään sovelluksemme), ei kehitysaikaisia riippuvuuksia tarvita. -->
These development dependencies are not needed when the application is run in production mode on the production server (e.g. Heroku).

<!-- Voimme käynnistää ohjelman <i>nodemonilla</i> seuraavasti: -->
We can start our application with <i>nodemon</i> like this:

```bash
node_modules/.bin/nodemon index.js
```

<!-- Sovelluksen koodin muutokset aiheuttavat nyt automaattisen palvelimen uudelleenkäynnistymisen. Kannattaa huomata, että vaikka palvelin uudelleenkäynnistyy automaattisesti, selain täytyy kuitenkin refreshata, sillä toisin kuin Reactin yhteydessä, meillä ei nyt ole eikä tässä skenaariossa (missä palautamme JSON-muotoista dataa) edes voisikaan olla selainta päivittävää [hot reload](https://gaearon.github.io/react-hot-loader/getstarted/) -toiminnallisuutta. -->
Changes to the application code now causes the server to restart automatically. It's worth noting, that even though the backend server restarts automatically, the browser still has to be manually refreshed. This is because unlike when working in React, we could not even have the [hot reload](https://gaearon.github.io/react-hot-loader/getstarted/) functionality needed to automatically reload the browser.

<!-- Komento on ikävä, joten määritellään sitä varten <i>npm-skripti</i> tiedostoon <i>package.json</i>: -->
The command is long and quite unpleasant, so let's define a dedicated <i>npm script</i> for it in the <i>package.json</i> file:

```bash
{
  // ..
  "scripts": {
    "start": "node index.js",
    "watch": "nodemon index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  // ..
}
```

<!-- Skriptissä ei ole tarvetta käyttää nodemonin polusta sen täydellistä muotoa <i>node\_modules/.bin/nodemon</i> sillä _npm_ osaa etsiä automaattisesti suoritettavaa tiedostoa kyseisestä hakemistosta. -->
In the script there is no need to specify the <i>node\_modules/.bin/nodemon</i> path to nodemon, because _npm_ automatically knows to search for the file from that directory. 

<!-- Voimme nyt käynnistää palvelimen sovelluskehitysmoodissa komennolla -->
We can now start the server in the development mode with the command:

```bash
npm run watch
```

<!-- Toisin kuin skriptejä <i>start</i> tai <i>test</i> suoritettaessa, joudumme sanomaan myös <i>run</i>. -->
Unlike with the <i>start</i> and <i>test</i> scripts, we also have to add <i>run</i> to the command.


### REST

<!-- Laajennetaan sovellusta siten, että se toteuttaa samanlaisen RESTful-periaatteeseen nojaavan HTTP-rajapinnan kuin [json-server](https://github.com/typicode/json-server#routes). -->
Let's expand our application so that it provides the RESTful HTTP API as [json-server](https://github.com/typicode/json-server#routes).

<!-- Representational State Transfer eli REST on Roy Fieldingin vuonna 2000 ilmestyneessä [väitöskirjassa](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) määritelty skaalautuvien web-sovellusten rakentamiseksi tarkoitettu arkkitehtuurityyli. -->
Representation State Transfer, aka. REST was introduced in 2000 in Roy Fielding's [dissertation](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm). REST is an architectural style meant for building scalable web applications.

<!-- Emme nyt rupea määrittelemään REST:iä Fieldingiläisittäin tai rupea väittämään mitä REST on tai mitä se ei ole vaan otamme hieman [kapeamman näkökulman](https://en.wikipedia.org/wiki/Representational_state_transfer#Applied_to_Web_services) miten REST tai RESTful API:t yleensä tulkitaan Web-sovelluksissa. Alkuperäinen REST-periaate ei edes sinänsä rajoitu Web-sovelluksiin. -->
We are not going to dig into Fielding's definition of REST or spend time pondering about what is and isn't RESTful. Instead, we take a more [narrow view](https://en.wikipedia.org/wiki/Representational_state_transfer#Applied_to_Web_services) by only concerning ourselves with how RESTful API's are typically understood in web applications. The original definition of REST is in fact not even limited to web applications.

<!-- Mainitsimme jo [edellisessä osassa](/osa2/#rest-apin-käyttö), että yksittäisiä asioita, meidän tapauksessamme muistiinpanoja kutsutaan RESTful-ajattelussa <i>resursseiksi</i>. Jokaisella resurssilla on URL eli sen yksilöivä osoite. -->
We mentioned in the [previous part](/osa2/#rest-apin-käyttö) that singular things, like notes in the case of our application, are called <i>resources</i> in RESTful thinking. Every resource has an associated URL which is the resource's unique address.

<!-- Erittäin yleinen konventio on muodostaa resurssien yksilöivät URLit liittäen resurssityypin nimi ja resurssin yksilöivä tunniste. -->
One convention is to create the unique address for resources by combining the name of the resource type with the resource's unique identifier.

<!-- Oletetaan että palvelumme juuriosoite on <i>www.example.com/api</i> -->
Let's assume that the root URL of our service is <i>www.example.com/api</i>.

<!-- Jos nimitämme muistiinpanoja <i>note</i>-resursseiksi, yksilöidään yksittäinen muistiinpano, jonka tunniste on 10 URLilla <i>www.example.com/api/notes/10</i>. -->
If we define the resource type of notes to be <i>note</i>, then the address of a note resource with the identifier 10, has the unique address <i>www.example.com/api/notes/10</i>.

<!-- Kaikkia muistiinpanoja edustavan kokoelmaresurssin URL taas on <i>www.example.com/api/notes</i> -->
The URL for the entire collection of all note resources is <i>www.example.com/api/notes</i>.

<!-- Resursseille voi suorittaa erilaisia operaatiota. Suoritettavan operaation määrittelee HTTP-operaation tyyppi, jota kutsutaan usein myös <i>verbiksi</i>: -->
We can execute different operations on resources. The operation to be executed is defined by the HTTP <i>verb</i>.

| URL                   | verb               | functionality                                                  |
| --------------------- | ------------------- | ---------------------------------------------------------------- |
| notes/10 &nbsp;&nbsp; | GET                 | fetches a single resource                                      |
| notes                 | GET                 | fetches all resources in the collection                                 |
| notes                 | POST                | creates a new resource based on the request data              |
| notes/10              | DELETE &nbsp;&nbsp; | removes the identified resource                                     |
| notes/10              | PUT                 | replaces the entire identified resource with the request data
| notes/10              | PATCH               | replaces a part of the identified resource with the request data
|                       |                     |                                                                  |

<!-- Näin määrittyy suurin piirtein asia, mitä REST kutsuu nimellä [uniform interface](https://en.wikipedia.org/wiki/Representational_state_transfer#Architectural_constraints), eli jossain määrin yhtenäinen tapa määritellä rajapintoja, jotka mahdollistavat (tietyin tarkennuksin) järjestelmien yhteiskäytön. -->
This is how we roughly define what REST calls a [uniform interface](https://en.wikipedia.org/wiki/Representational_state_transfer#Architectural_constraints), which means a consistent way of defining interfaces, that make it possible for systems to co-operate.

Tämänkaltaista tapaa tulkita REST:iä on nimitetty kolmiportaisella asteikolla [kypsyystason 2](https://martinfowler.com/articles/richardsonMaturityModel.html) REST:iksi. REST:in kehittäjän Roy Fieldingin mukaan tällöin kyseessä ei vielä ole ollenkaan asia, jota tulisi kutsua [REST-apiksi](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven). Maailman "REST"-apeista valtaosa ei täytäkään puhdasverisen Fieldingiläisen REST-apin määritelmää.

Joissain yhteyksissä (ks. esim [Richardsom, Ruby: RESTful Web Services](http://shop.oreilly.com/product/9780596529260.do)) edellä esitellyn kaltaista suoraviivaisehkoa resurssien [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)-tyylisen manipuloinnin mahdollistavaa API:a nimitetään REST:in sijaan [resurssipohjaiseksi](https://en.wikipedia.org/wiki/Resource-oriented_architecture) arkkitehtuurityyliksi. Emme nyt kuitenkaan takerru liian tarkasti määritelmällisiin asioihin vaan jatkamme sovelluksen parissa.

### Yksittäisen resurssin haku

Laajennetaan nyt sovellusta siten, että se tarjoaa muistiinpanojen operointiin REST-rajapinnan. Tehdään ensin [route](http://expressjs.com/en/guide/routing.html) yksittäisen resurssin katsomista varten.

Yksittäisen muistiinpanon identifioi URL, joka on muotoa <i>notes/10</i>, missä lopussa oleva numero vastaa resurssin muistiinpanon id:tä.

Voimme määritellä expressin routejen poluille [parametreja](http://expressjs.com/en/guide/routing.html) käyttämällä kaksoispistesyntaksia:

```js
app.get('/notes/:id', (request, response) => {
  const id = request.params.id
  const note = notes.find(note => note.id === id)
  response.json(note)
})
```

Nyt <code>app.get('/notes/:id', ...)</code> käsittelee kaikki HTTP GET -pyynnöt, jotka ovat muotoa <i>note/JOTAIN</i>, missä <i>JOTAIN</i> on mielivaltainen merkkijono.

Polun parametrin <i>id</i> arvoon päästään käsiksi pyynnön tiedot kertovan olion [request](http://expressjs.com/en/api.html#req) kautta:

```js
const id = request.params.id
```

Jo tutuksi tulleella taulukon _find_-metodilla haetaan taulukosta parametria vastaava muistiinpano ja palautetaan se pyynnön tekijälle.

Kun sovellusta testataan menemällä selaimella osoitteeseen <http://localhost:3001/notes/1>, havaitaan että se ei toimi, selain näyttää tyhjältä. Tämä on tietenkin softadevaajan arkipäivää, ja on ruvettava debuggaamaan.

Vanha hyvä keino on alkaa lisäillä koodiin _console.log_-komentoja:

```js
app.get('/notes/:id', (request, response) => {
  const id = request.params.id
  console.log(id)
  const note = notes.find(note => note.id === id)
  console.log(note)
  response.json(note)
})
```

Kun selaimella mennään jälleen osoitteeseen <http://localhost:3001/notes/1> konsoliin, eli siihen terminaaliin, mihin sovellus on käynnistetty tulostuu

![](../images/3/8.png)

eli halutun muistiinpanon id välittyy sovellukseen aivan oikein, mutta _find_ komento ei löydä mitään.

Päätetään tulostella konsoliin myös _find_-komennon sisällä olevasta vertailijafunktiosta, joka onnistuu helposti kun tiiviissä muodossa oleva funktio <em>note => note.id === id</em> kirjoitetaan eksplisiittisen returnin sisältävässä muodossa:

```js
app.get('/notes/:id', (request, response) => {
  const id = request.params.id
  const note = notes.find(note => {
    console.log(note.id, typeof note.id, id, typeof id, note.id === id)
    return note.id === id
  })
  console.log(note)
  response.json(note)
})
```

Vierailtaessa jälleen yksittäisen muistiinpanon sivulla jokaisesta vertailufunktion kutsusta tulostetaan nyt monta asiaa. Konsolin tulostus on seuraava:

<pre>
1 'number' '1' 'string' false
2 'number' '1' 'string' false
3 'number' '1' 'string' false
</pre>

ongelman syy selviää: muuttujassa _id_ on tallennettuna merkkijono '1' kun taas muistiinpanojen id:t ovat numeroita. Javascriptissä === vertailu katsoo kaikki eri tyyppiset arvot oletusarvoisesti erisuuriksi, joten 1 ei ole '1'.

Korjataan ongelma, muuttamalla parametrina oleva merkkijonomuotoinen id [numeroksi](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number):

```js
app.get('/notes/:id', (request, response) => {
  const id = Number(request.params.id)
  const note = notes.find(note => note.id === id)
  response.json(note)
})
```

ja nyt yksittäisen resurssin hakeminen toimii.

![](../images/3/9.png)

Toiminnallisuuteen jää kuitenkin pieni ongelma.

Jos haemme muistiinpanoa sellaisella indeksillä, mitä vastaavaa muistiinpanoa ei ole olemassa, vastaa palvelin seuraavasti

![](../images/3/10.png)

HTTP-statuskoodi on onnistumisesta kertova 200. Vastaukseen ei liity dataa, sillä headerin <i>content-length</i> arvo on 0, ja samaa todistaa selain: mitään ei näy.

Syynä tälle käyttäytymiselle on se, että muuttujan _note_ arvoksi tulee _undefined_ jos muistiinpanoa ei löydy. Tilanne tulisi käsitellä palvelimella järkevämmin, eli statuskoodin 200 sijaan tulee vastata statuskoodilla [404 not found](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.5).

Tehdään koodiin muutos

```js
app.get('/notes/:id', (request, response) => {
  const id = Number(request.params.id)
  const note = notes.find(note => note.id === id)
  
  // highlight-start
  if (note) {
    response.json(note)
  } else {
    response.status(404).end()
  }
  // highlight-end
})
```

Koska vastaukseen ei nyt liity mitään dataa käytetään statuskoodin asettavan metodin [status](http://expressjs.com/en/4x/api.html#res.status) lisäksi metodia [end](http://expressjs.com/en/4x/api.html#res.end) ilmoittamaan siitä, että pyyntöön tulee vastata ilman dataa.

Koodin haarautumisessa hyväksikäytetään sitä, että mikä tahansa Javascript-olio on [truthy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy), eli katsotaan todeksi vertailuoperaatiossa. undefined taas on [falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) eli epätosi.

Nyt sovellus toimii, eli palauttaa oikean virhekoodin. Sovellus ei kuitenkaan palauta mitään käyttäjälle näytettävää kuten web-sovellukset yleensä tekevät jos mennään osoitteeseen jota ei ole olemassa. Emme kuitenkaan tarvitse nyt mitään näytettävää, sillä REST API:t ovat ohjelmalliseen käyttöön tarkoitettuja rajapintoja ja pyyntöön liitetty virheestä kertova statuskoodi on riittävä.

### Resurssin poisto

Toteutetaan seuraavaksi resurssin poistava route. Poisto tapahtuu tekemällä HTTP DELETE -pyyntö resurssin urliin:

```js
app.delete('/notes/:id', (request, response) => {
  const id = Number(request.params.id);
  notes = notes.filter(note => note.id !== id);

  response.status(204).end();
});
```

Jos poisto onnistuu, eli poistettava muistiinpano on olemassa, vastataan statuskoodilla [204 no content](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.2.5) sillä mukaan ei lähetetä mitään dataa.

Ei ole täyttä yksimielisyyttä siitä mikä statuskoodi DELETE-pyynnöstä pitäisi palauttaa jos poistettavaa resurssia ei ole olemassa. Vaihtoehtoja ovat lähinnä 204 ja 404. Yksinkertaisuuden vuoksi sovellus palauttaa nyt molemmissa tilanteissa statuskoodin 204.

### Postman

Herää kysymys miten voimme testata poisto-operaatiota? HTTP GET -pyyntöjä on helppo testata selaimessa. Voisimme toki kirjoittaa Javascript-koodin, joka testaa deletointia, mutta jokaiseen mahdolliseen tilanteeseen testikoodinkaan tekeminen ei ole aina paras ratkaisu.

On olemassa useita backendin testaamista helpottavia työkaluja, eräs näistä on edellisessä osassa nopeasti mainittu komentorivityökalu [curl](https://curl.haxx.se).

Käytetään nyt kuitenkin [postman](https://www.getpostman.com/)-nimistä sovellusta.

Asennetaan postman ja kokeillaan

![](../images/3/11.png)

Postmanin käyttö on tässä tilanteessa suhteellisen yksinkertaista, riittää määritellä url ja valita oikea pyyntötyyppi.

Palvelin näyttää vastaavan oikein. Tekemällä HTTP GET osoitteeseen <http://localhost:3001/notes> selviää että poisto-operaatio oli onnistunut, muistiinpanoa, jonka id on 2 ei ole enää listalla.

Koska muistiinpanot on talletettu palvelimen muistiin, uudelleenkäynnistys palauttaa tilanteen ennalleen.

### Visual Studio Coden REST client

Jos käytät Visual Studio Codea, voit postmanin sijaan käyttää VS Coden
[REST client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) -pluginia.

Kun plugin on asennettu, on sen käyttö erittäin helppoa. Tehdään projektin juureen hakemisto <i>requests</i>, jonka sisään talletetaan REST Client -pyynnöt <i>.rest</i>-päätteisinä tiedostoina.

Luodaan kaikki muistiinpanot hakevan pyynnön määrittelevä tiedosto <i>get\_all\_notes.rest</i>

![](../images/3/12.png)

Klikkaamalla tekstiä <i>Send Request</i>, REST client suorittaa määritellyn HTTP-pyynnön ja palvelimen vastaus avautuu editoriin:

![](../images/3/13.png)

### Datan vastaanottaminen

Toteutetaan seuraavana uusien muistiinpanojen lisäys, joka siis tapahtuu tekemällä HTTP POST -pyyntö osoitteeseen http://localhost:3001/notes ja liittämällä pyynnön mukaan eli [bodyyn](https://www.w3.org/Protocols/rfc2616/rfc2616-sec7.html#sec7) luotavan muistiinpanon tiedot JSON-muodossa.

Jotta pääsisimme pyynnön mukana lähetettyyn dataan helposti käsiksi, tarvitsemme [body-parser](https://github.com/expressjs/body-parser)-kirjaston apua.

Otetaan body-parser käyttöön ja luodaan alustava määrittely HTTP POST -pyynnön käsittelyyn

```js
const express = require('express')
const app = express()
const bodyParser = require('body-parser')

app.use(bodyParser.json())

//...

app.post('/notes', (request, response) => {
  const note = request.body
  console.log(note)

  response.json(note)
})
```

Tapahtumankäsittelijäfunktio pääsee dataan käsiksi olion _request_ kentän <i>body</i> avulla.

Ilman body-parser-käyttöönottoa pyynnön kentän <i>body</i> arvo olisi ollut määrittelemätön. body-parserin toimintaperiaatteena on, että se ottaa pyynnön mukana olevan JSON-muotoisen datan, muuttaa sen Javascript-olioksi ja sijoittaa _request_-olion kenttään <i>body</i> ennen kuin routen käsittelijää kutsutaan.

Toistaiseksi sovellus ei vielä tee vastaanotetulle datalle mitään muuta kuin tulostaa sen konsoliin ja palauttaa sen pyynnön vastauksessa.

Ennen toimintalogiikan viimeistelyä varmistetaan ensin postmanilla, että lähetetty tieto menee varmasti perille. Pyyntötyypin ja urlin lisäksi on määriteltävä myös pyynnön mukana menevä data eli <i>body</i>:

![](../images/3/14.png)

Sovellus tulostaa lähetetyn vastaanottamansa datan terminaaliin:

![](../images/3/15.png)

**HUOM** kun ohjelmoit backendia, <i>pidä sovellusta suorittava konsoli koko ajan näkyvillä</i>. Nodemonin ansiosta sovellus käynnistyy uudelleen jos koodiin tehdään muutoksia. Jos seuraat konsolia, huomaat välittömästi jos sovelluksen koodiin tulee joku perustavanlaatuinen virhe:

![](../images/3/16.png)

Vastaavasti konsolista kannattaa seurata reagoiko backend odotetulla tavalla, esim. kun sovellukselle lähetetään dataa metodilla HTTP POST. Backendiin kannattaa luonnollisesti lisäillä runsaat määrät <em>console.log</em>-komentoja kun sovellus on kehitysvaiheessa. 

Eräs potentiaalinen ongelmanlähde on se, että dataa lähettäessä, sen headerille <i>Content-Type</i> ei aseteta oikeaa arvoa. Näin tapahtuu esim. jos Postmanissa bodyn tyypiä ei määritellä oikein:

![](../images/3/17a.png)

headerin <i>Content-Type</i> arvoksi asettuu <i>text/plain</i>

![](../images/3/18.png)

Palvelin näyttää vastaanottavan ainoastaan tyhjän olion

![](../images/3/19.png)

Ilman oikeaa headerin arvoa palvelin ei osaa parsia dataa oikeaan muotoon. Se ei edes yritä arvailla missä muodossa data on, sillä potentiaalisia datan siirtomuotoja eli <i>Content-Typejä</i> on olemassa [suuri määrä](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types).

Jos käytät VS Codea niin edellisessä luvussa esitelty REST client kannattaa asentaa viimeistään <i>nyt</i>. POST-pyyntö tehdään REST clientillä seuraavasti:

![](../images/3/20.png)

Eli pyyntöä varten on luotu oma tiedosto <i>new\_note.rest</i>. Pyyntö on muotoiltu [dokumentaation ohjetta](https://github.com/Huachao/vscode-restclient/blob/master/README.md#usage) noudatellen.

REST clientin eräs suuri etu Postmaniin verrattuna on se, että pyynnöt saa kätevästi talletettua projektin repositorioon ja tällöin ne ovat helposti koko kehitystiimin käytössä. Postmanillakin on mahdollista tallettaa pyyntöjä, mutta tilanne menee helposti kaaoottiseksi etenkin jos työn alla on useita toisistaan riippumattomia projekteja.

> **Tärkeä sivuhuomio**
>
> Välillä debugatessa tulee vastaan tilanteita, joissa backendissä on tarve selvittää mitä headereja HTTP-pyynnöille on asetettu. Eräs menetelmä tähän on _request_-olion melko kehnosti nimetty metodi [get](http://expressjs.com/en/4x/api.html#req.get), jonka avulla voi selvittää yksittäisen headerin arvon. _request_-oliolla on myös kenttä <i>headers</i>, jonka arvona ovat kaikki pyyntöön liittyvät headerit.
>
> Ongelmia voi esim syntyä jos jätät vahingossa VS REST clientillä ylimmän rivin ja headerit määrittelevien rivien väliin tyhjän rivin. Tällöin REST client tulkitsee, että millekään headerille ei aseteta arvoa ja näin backend ei osaa tulkita pyynnön mukana olevaa dataa JSON:iksi.
>
> Puuttuvan <i>content-type</i>-headerin ongelma selviää kun backendissa tulostaa pyynnön headerit esim. komennolla _console.log(request.headers)_

Palataan taas sovelluksen pariin. Kun tiedämme, että sovellus vastaanottaa tiedon oikein, voimme viimeistellä sovelluslogiikan:

```js
app.post('/notes', (request, response) => {
  const maxId = notes.length > 0
    ? Math.max(...notes.map(n => n.id)) 
    : 0

  const note = request.body
  note.id = maxId + 1

  notes = notes.concat(note)

  response.json(note)
})
```

Uudelle muistiinpanolle tarvitaan uniikki id. Ensin selvitetään olemassaolevista id:istä suurin muuttujaan _maxId_. Uuden muistiinpanon id:ksi asetetaan sitten _maxId + 1_. Tämä tapa ei ole itseasiassa kovin hyvä, mutta emme nyt välitä siitä sillä tulemme pian korvaamaan tavan, jolla muistiinpanot talletetaan.

Tämän hetkisessä versiossa on vielä se ongelma, että voimme HTTP POST -pyynnöllä lisätä mitä tahansa kenttiä sisältäviä olioita. Parannellaan sovellusta siten, että kenttä <i>content</i> ei voi olla tyhjä. Kentille <i>important</i> ja <i>date</i> asetetaan oletusarvot. Kaikki muut kentät hylätään:

```js
const generateId = () => {
  const maxId = notes.length > 0
    ? Math.max(...notes.map(n => n.id))
    : 0
  return maxId + 1
}

app.post('/notes', (request, response) => {
  const body = request.body

  if (!body.content) {
    return response.status(400).json({ 
      error: 'content missing' 
    })
  }

  const note = {
    content: body.content,
    important: body.important || false,
    date: new Date(),
    id: generateId(),
  }

  notes = notes.concat(note)

  response.json(note)
})
```

Tunnisteena toimivan id-kentän arvon generointilogiikka on eriytetty funktioon _generateId_.

Jos vastaanotetulta datalta puuttuu sisältö kentästä <i>content</i>, vastataan statuskoodilla [400 bad request](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.1):

```js
if (!body.content) {
  return response.status(400).json({ 
    error: 'content missing' 
  })
}
```

Huomaa, että returnin kutsuminen on tärkeää, jos sitä ei tapahdu, jatkaa koodi suoritusta metodin loppuun asti ja virheellinen muistiinpano tallettuu!

Jos content-kentällä on arvo, luodaan muistiinpano syötteen perusteella. Kuten edellisessä osassa mainitsimme, aikaleimoja ei kannata luoda selaimen koodissa, sillä käyttäjän koneen kellon aikaan ei voi luottaa. Aikaleiman eli kentän <i>date</i> arvon generointi tapahtuukin nyt palvelimen toimesta.

Jos kenttä <i>important</i> puuttuu, asetetaan sille oletusarvo <i>false</i>. Oletusarvo generoidaan nyt hieman erikoisella tavalla:

```js
important: body.important || false,
```

jos sovelluksen vastaanottamassa muuttujaan _body_ talletetussa datassa on kenttä <i>important</i>, tulee lausekkeelle sen arvo. Jos kenttää ei ole olemassa, tulee lausekkeen arvoksi oikeanpuoleinen osa eli <i>false</i>.

> Jos ollaan tarkkoja, niin kentän <i>important</i> arvon ollessa <i>false</i>, tulee lausekkeen <em>body.important || false</em> arvoksi oikean puoleinen <i>false</i>...

Sovelluksen tämän hetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-1)

Huomaa, että repositorion master-haarassa on myöhemmän vaiheen koodi, tämän hetken koodi on branchissa [part3-1](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-1):

![](../images/3/21.png)

Jos kloonaat projektin itsellesi, suorita komento _npm install_ ennen käynnistämistä eli komentoa _npm start_ tai _npm run watch_.

Vielä pieni huomio ennen tehtäviä. Uuden id:n generoiva funktio näyttää seuraavalta

```js
const generateId = () => {
  const maxId = notes.length > 0
    ? Math.max(...notes.map(n => n.id))
    : 0
  return maxId + 1
}
```

Koodi sisältää hieman erikoisen näköisen rivin

```js
Math.max(...notes.map(n => n.id))
```

Mitä rivillä tapahtuu? <em>notes.map(n => n.id)</em> muodostaa taulukon, joka koostuu muistiinpanojen id-kentisstä. [Math.max](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/max) palauttaa maksimin sille parametrina annetuista luvuista. <em>notes.map(n => n.id)</em> on kuitenkin <i>taulukko</i>, joten se ei kelpaa parametriksi komennolle _Math.max_. Taulukko voidaan muuttaa yksittäisiksi luvuiksi käyttäen taulukon [spread](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)-syntaksia, eli kolmea pistettä <em>... taulukko</em>.

</div>

<div class="tasks">

### Tehtäviä 

**HUOM:** tämän osan tehtäväsarja kannattaa tehdä omaan git-repositorioon, suoraan repositorion juureen! Jos et tee näin, joudut ongelmiin tehtävässä 3.10

**HUOM2:** Koska nyt ei ole kyse frontendista ja Reactista, sovellusta <strong>ei luoda</strong> create-react-app:illa vaan komennolla <em>npm init</em> kuten ylempänä tämän osan materiaalissa.


**Vahva suositus:** kun teet backendin koodia, pidä koko ajan silmällä mitä palvelimen koodia suorittavassa konsolissa tapahtuu.

#### 3.1 puhelinluettelon backend step1

Tee Node-sovellus, joka tarjoaa osoitteessa <http://localhost:3001/api/persons> kovakoodatun taulukon puhelinnumerotietoja:

![](../images/3/22a.png)

Huomaa, että Noden routejen määrittelyssä merkkijonon <i>api/persons</i> vinoviiva käyttäytyy kuten mikä tahansa muu merkki.

Sovellus pitää pystyä käynnistämään komennolla _npm start_.

Komennolla _npm run watch_ käynnistettäessa sovelluksen tulee käynnistyä uudelleen kun koodiin tehdään muutoksia.

#### 3.2: puhelinluettelon backend step2

Tee sovelluksen osoitteeseen <http://localhost:3001/info> suunnilleen seuraavanlainen sivu

![](../images/3/23.png)

eli sivu kertoo pyynnön tekohetken sekä sen kuinka monta puhelinluettelotietoa sovelluksen muistissa olevassa taulukossa on.

#### 3.3: puhelinluettelon backend step3

Toteuta toiminnallisuus yksittäisen puhelinnumerotiedon näyttämiseen. Esim. id:n 5 omaavan numerotiedon url on <http://localhost:3001/api/persons/5>

Jos id:tä vastaavaa puhelinnumerotietoa ei ole, tulee palvelimen vastata asianmukaisella statuskoodilla.

#### 3.4: puhelinluettelon backend step4

Toteuta toiminnallisuus, jonka avulla puhelinnumerotieto on mahdollista poistaa numerotiedon yksilöivään URL:iin tehtävällä HTTP DELETE -pyynnöllä.

Testaa toiminnallisuus Postmanilla tai Visual Studio Coden REST clientillä

#### 3.5: puhelinluettelon backend step5

Laajenna backendia siten, että uusia puhelintietoja on mahdollista lisätä osoitteeseen <http://localhost:3001/api/persons> tapahtuvalla HTTP POST -pyynnöllä.

Generoi uuden puhelintiedon tunniste funktiolla [Math.random](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random). Käytä riittävän isoa arvoväliä jotta arvottu id on riittävän suurella todennäköisyydellä sellainen, joka ei ole jo käytössä.

#### 3.6: puhelinluettelon backend step6

Tee uuden numeron lisäykseen virheiden käsittely. Pyyntö ei saa onnistua, jos
- nimi tai numero puuttuu
- lisättävä nimi on jo luettelossa

Vastaa asiaankuuluvalla statuskoodilla, liitä vastaukseen mukaan myös tieto, joka kertoo virheen syyn, esim:

```js
{ error: 'name must be unique' }
```

</div>

<div class="content">

### Huomioita HTTP pyyntötyyppien käytöstä

[HTTP-standardi](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) puhuu pyyntötyyppien yhteydessä kahdesta ominaisuudesta, **safe** ja **idempotent**.

HTTP-pyynnöistä GET:in tulisi olla <i>safe</i>:

> <i>In particular, the convention has been established that the GET and HEAD methods SHOULD NOT have the significance of taking an action other than retrieval. These methods ought to be considered "safe".</i>

Safety siis tarkoittaa, että pyynnön suorittaminen ei saa aiheuttaa palvelimelle <i>sivuvaikutuksia</i> eli esim. muuttaa palvelimen tietokannan tilaa, pyynnön tulee ainoastaan palauttaa palvelimella olevaa dataa.

Mikään ei automaattisesti takaa, että GET-pyynnöt olisivat luonteeltaan <i>safe</i>, kyseessä onkin HTTP-standardin suositus palvelimien toteuttajille. RESTful-periaatetta noudattaessa GET-pyyntöjä käytetäänkin aina siten, että ne ovat safe.

HTTP-standardi määrittelee myös pyyntötyypin [HEAD](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.4), jonka tulee olla safe. Käytännössä HEAD:in tulee toimia kuten GET, mutta se ei palauta vastauksenaan muuta kuin statuskoodin ja headerit, viestin bodyä HEAD ei palauta ollenkaan.

HTTP-pyynnöistä muiden paitsi POST:in tulisi olla <i>idempotentteja</i>:

> <i>Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request. The methods GET, HEAD, PUT and DELETE share this property</i>

Eli jos pyynnöllä on sivuvaikutuksia, lopputulos on sama suoritetaanko pyyntö yhden tai useamman kerran.

Esim. jos tehdään HTTP PUT pyyntö osoitteeseen <i>/notes/10</i> ja pyynnön mukana on <em>{ content: "ei sivuvaikutuksia", important: true }</em>, on lopputulos sama riippumatta siitä kuinka monta kertaa pyyntö suoritetaan.

Kuten metodin GET <i>safety</i> myös <i>idempotence</i> on HTTP-standardin suositus palvelimien toteuttajille. RESTful-periaatetta noudattaessa GET, HEAD, PUT ja DELETE-pyyntöjä käytetäänkin aina siten, että ne ovat idempotentteja.

HTTP pyyntötyypeistä POST on ainoa joka ei ole <i>safe</i> eikä <i>idempotent</i>. Jos tehdään 5 kertaa HTTP POST -pyyntö osoitteeseen <i>/notes</i> siten että pyynnön mukana on <em>{ content: "monta samaa", important: true }</em>, tulee palvelimelle 5 saman sisältöistä muistiinpanoa.

### Middlewaret

Äsken käyttöönottamamme [body-parser](https://github.com/expressjs/body-parser) on terminologiassa niin sanottu [middleware](http://expressjs.com/en/guide/using-middleware.html).

Middlewaret ovat funktioita, joiden avulla voidaan käsitellä _request_- ja _response_-olioita.

Esim. body-parser ottaa pyynnön mukana tulevan raakadatan _request_-oliosta, parsii sen Javascript-olioksi ja sijoittaa olion _request_:in kenttään <i>body</i>

Middlewareja voi olla käytössä useita, jolloin ne suoritetaan peräkkäin siinä järjestyksessä kun ne on otettu koodissa käyttöön.

Toteutetaan itse yksinkertainen middleware, joka tulostaa konsoliin palvelimelle tulevien pyyntöjen perustietoja.

Middleware on funktio, joka saa kolme parametria:

```js
const requestLogger = (request, response, next) => {
  console.log('Method:', request.method)
  console.log('Path:  ', request.path)
  console.log('Body:  ', request.body)
  console.log('---')
  next()
}
```

Middleware kutsuu lopussa parametrina olevaa funktiota _next_, jolla se siirtää kontrollin seuraavalle middlewarelle.

Middleware otetaan käyttöön seuraavasti:

```js
app.use(requestLogger)
```

Middlewaret suoritetaan siinä järjestyksessä, jossa ne on otettu käyttöön sovellusolion metodilla _use_. Huomaa, että _bodyParser_ tulee ottaa käyttään ennen middlewarea _requestLogger_, muuten <i>request.body</i> ei ole vielä alustettu loggeria suoritettaessa!

Middlewaret tulee ottaa käyttöön ennen routeja jos ne halutaan suorittaa ennen niitä. On myös eräitä tapauksia, joissa middleware tulee määritellä vasta routejen jälkeen, käytännössä tällöin on kyse middlewareista, joita suoritetaan vain, jos mikään route ei käsittele HTTP-pyyntöä.

Lisätään routejen jälkeen seuraava middleware, jonka ansiosta saadaan routejen käsittelemättömistä virhetilanteista JSON-muotoinen virheilmoitus:

```js
const unknownEndpoint = (request, response) => {
  response.status(404).send({ error: 'unknown endpoint' })
}

app.use(unknownEndpoint)
```

Sovelluksen tämän hetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-2), branchissa <i>part3-2</i>.

</div>

<div class="tasks">

### Tehtäviä

#### 3.7: puhelinluettelon backend step7

Lisää sovellukseesi loggausta tekevä middleware [morgan](https://github.com/expressjs/morgan). Konfiguroi se logaamaan konsoliin <i>tiny</i>-konfiguraation mukaisesti.

Morganin ohjeet eivät ole ehkä kaikkein selvimmät ja joudut kenties miettimään hiukan. Toisaalta juuri koskaan dokumentaatio ei ole aivan itsestäänselvää, joten kryptisempiäkin asioita on hyvä oppia tulkitsemaan.

Morgan asennetaan kuten muutkin kirjastot, eli komennolla _npm install_ ja sen käyttöönotto tapahtuu kaikkien middlewarejen tapaan komennolla _app.use_

#### 3.8*: puhelinluettelon backend step8

Konfiguroi morgania siten, että se näyttää myös HTTP POST -pyyntöjen mukana tulevan datan:

![](../images/3/24.png)

Tämä tehtävä on kohtuullisen haastava vaikka koodia ei tarvitakkaan paljoa. 

Tehtävän voi tehdä muutamallakin tavalla, eräs näistä onnistuu hyödyntämällä seuraavia
- [creating new tokens](https://github.com/expressjs/morgan#creating-new-tokens)
- [JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)

</div>
