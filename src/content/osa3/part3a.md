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
We can execute different operations on resources. The operation to be executed is defined by the HTTP <i>verb</i>:

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
This is how we manage to roughly define what REST refers to as a [uniform interface](https://en.wikipedia.org/wiki/Representational_state_transfer#Architectural_constraints), which means a consistent way of defining interfaces that makes it possible for systems to co-operate.

<!-- Tämänkaltaista tapaa tulkita REST:iä on nimitetty kolmiportaisella asteikolla [kypsyystason 2](https://martinfowler.com/articles/richardsonMaturityModel.html) REST:iksi. REST:in kehittäjän Roy Fieldingin mukaan tällöin kyseessä ei vielä ole ollenkaan asia, jota tulisi kutsua [REST-apiksi](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven). Maailman "REST"-apeista valtaosa ei täytäkään puhdasverisen Fieldingiläisen REST-apin määritelmää. -->
This way of interpreting REST falls under the [second level of RESTful maturity](https://martinfowler.com/articles/richardsonMaturityModel.html) in the Richardson Maturity Model. According to the definition provided by Roy Fielding, we have not actually defined a [REST API](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven). In fact, a large majority of the world's purported "REST" API's do not meet Fielding's original criteria outlined in his dissertation.

<!-- Joissain yhteyksissä (ks. esim [Richardsom, Ruby: RESTful Web Services](http://shop.oreilly.com/product/9780596529260.do)) edellä esitellyn kaltaista suoraviivaisehkoa resurssien [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)-tyylisen manipuloinnin mahdollistavaa API:a nimitetään REST:in sijaan [resurssipohjaiseksi](https://en.wikipedia.org/wiki/Resource-oriented_architecture) arkkitehtuurityyliksi. Emme nyt kuitenkaan takerru liian tarkasti määritelmällisiin asioihin vaan jatkamme sovelluksen parissa. -->
In some places (see e.g. [Richardson, Ruby: RESTful Web Services](http://shop.oreilly.com/product/9780596529260.do)) you will see our model for a straightforward [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) API, being referred to as an example of [resource oriented architecture](https://en.wikipedia.org/wiki/Resource-oriented_architecture) instead of REST. We will avoid getting stuck arguing semantics and instead return to working on our application.

<!-- ### Yksittäisen resurssin haku -->
### Fetching a single resource

<!-- Laajennetaan nyt sovellusta siten, että se tarjoaa muistiinpanojen operointiin REST-rajapinnan. Tehdään ensin [route](http://expressjs.com/en/guide/routing.html) yksittäisen resurssin katsomista varten. -->
Let's expand our application so that it offers a REST interface for operating on individual notes. First let's create a [route](http://expressjs.com/en/guide/routing.html) for fetching a single resource.

<!-- Yksittäisen muistiinpanon identifioi URL, joka on muotoa <i>notes/10</i>, missä lopussa oleva numero vastaa resurssin muistiinpanon id:tä. -->
The unique address we will use for an individual note is of the form <i>notes/10</i>, where the number at the end refers to the note's unique id number.

<!-- Voimme määritellä expressin routejen poluille [parametreja](http://expressjs.com/en/guide/routing.html) käyttämällä kaksoispistesyntaksia: -->
We can define [parameters](http://expressjs.com/en/guide/routing.html) for routes in express by using the colon syntax:

```js
app.get('/notes/:id', (request, response) => {
  const id = request.params.id
  const note = notes.find(note => note.id === id)
  response.json(note)
})
```

<!-- Nyt <code>app.get('/notes/:id', ...)</code> käsittelee kaikki HTTP GET -pyynnöt, jotka ovat muotoa <i>note/JOTAIN</i>, missä <i>JOTAIN</i> on mielivaltainen merkkijono. -->
Now <code>app.get('/notes/:id', ...)</code> will handle all HTTP GET requests, that are of the form <i>note/SOMETING</i>, where <i>SOMETHING</i> is an arbitrary string.

<!-- Polun parametrin <i>id</i> arvoon päästään käsiksi pyynnön tiedot kertovan olion [request](http://expressjs.com/en/api.html#req) kautta: -->
The <i>id</i> parameter in the route of a request, can be accessed through the [request](http://expressjs.com/en/api.html#req) object:

```js
const id = request.params.id
```

<!-- Jo tutuksi tulleella taulukon _find_-metodilla haetaan taulukosta parametria vastaava muistiinpano ja palautetaan se pyynnön tekijälle. -->
The now familiar _find_ method of arrays is used to find the note with an id that matches the parameter. The note is then returned to the sender of the request.

<!-- Kun sovellusta testataan menemällä selaimella osoitteeseen <http://localhost:3001/notes/1>, havaitaan että se ei toimi, selain näyttää tyhjältä. Tämä on tietenkin softadevaajan arkipäivää, ja on ruvettava debuggaamaan. -->
When we test our application by going to <http://localhost:3001/notes/1> in our browser, we notice that it does not appear to work, as the browser displays an empty page. This comes as no surprise to us as software developers, and it's time to debug.

<!-- Vanha hyvä keino on alkaa lisäillä koodiin _console.log_-komentoja: -->
Adding _console.log_ commands into our code is a time-proven trick:

```js
app.get('/notes/:id', (request, response) => {
  const id = request.params.id
  console.log(id)
  const note = notes.find(note => note.id === id)
  console.log(note)
  response.json(note)
})
```

<!-- Kun selaimella mennään jälleen osoitteeseen <http://localhost:3001/notes/1> konsoliin, eli siihen terminaaliin, mihin sovellus on käynnistetty tulostuu -->
When we visit <http://localhost:3001/notes/1> again in the browser, the console which is the terminal in this case, will display the following:

![](../images/3/8.png)

<!-- eli halutun muistiinpanon id välittyy sovellukseen aivan oikein, mutta _find_ komento ei löydä mitään. -->
The id parameter from the route is passed to our application but the _find_ method does not find a matching note.

<!-- Päätetään tulostella konsoliin myös _find_-komennon sisällä olevasta vertailijafunktiosta, joka onnistuu helposti kun tiiviissä muodossa oleva funktio <em>note => note.id === id</em> kirjoitetaan eksplisiittisen returnin sisältävässä muodossa: -->
To further our investigation, we also add a console log inside of the comparison function passed to the _find_ method. In order to do this, we have to get rid of the compact arrow function syntax <em>note => note.id === id</em>, and use the syntax with an explicit return statement:

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

<!-- Vierailtaessa jälleen yksittäisen muistiinpanon sivulla jokaisesta vertailufunktion kutsusta tulostetaan nyt monta asiaa. Konsolin tulostus on seuraava: -->
When we visit the URL again in the browser, each call to the comparison function prints a few different things to the console. The console output in its entirety is the following:

<pre>
1 'number' '1' 'string' false
2 'number' '1' 'string' false
3 'number' '1' 'string' false
</pre>

<!-- ongelman syy selviää: muuttujassa _id_ on tallennettuna merkkijono '1' kun taas muistiinpanojen id:t ovat numeroita. Javascriptissä === vertailu katsoo kaikki eri tyyppiset arvot oletusarvoisesti erisuuriksi, joten 1 ei ole '1'. -->
The cause of the bug becomes clear. The _id_ variable contains a string '1', whereas the id's of notes are integers. In JavaScript the "triple equals" comparison === considers all values of different types to not be equal by default, meaning that 1 is not '1'. 

<!-- Korjataan ongelma, muuttamalla parametrina oleva merkkijonomuotoinen id [numeroksi](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number): -->
Let's fix the issue by changing the id parameter from a string into a [number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number):

```js
app.get('/notes/:id', (request, response) => {
  const id = Number(request.params.id)
  const note = notes.find(note => note.id === id)
  response.json(note)
})
```

<!-- ja nyt yksittäisen resurssin hakeminen toimii. -->
Now fetching an individual resource works.

![](../images/3/9.png)

<!-- Toiminnallisuuteen jää kuitenkin pieni ongelma. -->
However, there's another problem with our application.

<!-- Jos haemme muistiinpanoa sellaisella indeksillä, mitä vastaavaa muistiinpanoa ei ole olemassa, vastaa palvelin seuraavasti -->
If we search for a note with an id that does not exist, the server responds with:

![](../images/3/10.png)

<!-- HTTP-statuskoodi on onnistumisesta kertova 200. Vastaukseen ei liity dataa, sillä headerin <i>content-length</i> arvo on 0, ja samaa todistaa selain: mitään ei näy. -->
The HTTP status code that is returned is 200, which means that the response succeeded. There is no data sent back with the response, since the value of the <i>content-length</i> header is 0, and the same can be verified from the browser. 

<!-- Syynä tälle käyttäytymiselle on se, että muuttujan _note_ arvoksi tulee _undefined_ jos muistiinpanoa ei löydy. Tilanne tulisi käsitellä palvelimella järkevämmin, eli statuskoodin 200 sijaan tulee vastata statuskoodilla [404 not found](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.5). -->
The reason for this behavior is that the _note_ variable is set to _undefined_ if no matching note is found. The situation needs to be handled on the server in a better way. If no note is found, the server should respond with the the status code [404 not found](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.5) instead of 200.

<!-- Tehdään koodiin muutos -->
Let's make the following change to our code:

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

<!-- Koska vastaukseen ei nyt liity mitään dataa käytetään statuskoodin asettavan metodin [status](http://expressjs.com/en/4x/api.html#res.status) lisäksi metodia [end](http://expressjs.com/en/4x/api.html#res.end) ilmoittamaan siitä, että pyyntöön tulee vastata ilman dataa. -->
Since no data is attached to the response, we use the [status](http://expressjs.com/en/4x/api.html#res.status) method for setting the status, and the [end](http://expressjs.com/en/4x/api.html#res.end) method for responding to the request without sending any data.

<!-- Koodin haarautumisessa hyväksikäytetään sitä, että mikä tahansa Javascript-olio on [truthy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy), eli katsotaan todeksi vertailuoperaatiossa. undefined taas on [falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) eli epätosi. -->
The if-condition leverages the fact that all JavaScript objects are [truthy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy), meaning that they evaluate to true in a comparison operation. However, _undefined_ is [falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) meaning that it will evaluate to false.

<!-- Nyt sovellus toimii, eli palauttaa oikean virhekoodin. Sovellus ei kuitenkaan palauta mitään käyttäjälle näytettävää kuten web-sovellukset yleensä tekevät jos mennään osoitteeseen jota ei ole olemassa. Emme kuitenkaan tarvitse nyt mitään näytettävää, sillä REST API:t ovat ohjelmalliseen käyttöön tarkoitettuja rajapintoja ja pyyntöön liitetty virheestä kertova statuskoodi on riittävä. -->
Our application works and sends the error status code if no note is found. However, the application doesn't return anything to show to the user, like web applications normally do when we visit a page that does not exist. We do not actually need to display anything in the browser because REST API's are interfaces that are intended for programmatic use, and the error status code is all that is needed.

<!-- ### Resurssin poisto -->
### Deleting resources

<!-- Toteutetaan seuraavaksi resurssin poistava route. Poisto tapahtuu tekemällä HTTP DELETE -pyyntö resurssin urliin: -->
Next let's implement a route for deleting resources. Deletion happens by making an HTTP DELETE request to the url of the resource:

```js
app.delete('/notes/:id', (request, response) => {
  const id = Number(request.params.id);
  notes = notes.filter(note => note.id !== id);

  response.status(204).end();
});
```

<!-- Jos poisto onnistuu, eli poistettava muistiinpano on olemassa, vastataan statuskoodilla [204 no content](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.2.5) sillä mukaan ei lähetetä mitään dataa. -->
If deleting the resource is successful, meaning that the note exists and it is removed, we respond to the request with the status code [204 no content](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.2.5) and return no data with the response.

<!-- Ei ole täyttä yksimielisyyttä siitä mikä statuskoodi DELETE-pyynnöstä pitäisi palauttaa jos poistettavaa resurssia ei ole olemassa. Vaihtoehtoja ovat lähinnä 204 ja 404. Yksinkertaisuuden vuoksi sovellus palauttaa nyt molemmissa tilanteissa statuskoodin 204. -->
There's no consensus on what status code should be returned to a DELETE request if the resource does not exist. Really, the only two options are 204 and 404. For the sake of simplicity our application will respond with 204 in both cases.

### Postman

<!-- Herää kysymys miten voimme testata poisto-operaatiota? HTTP GET -pyyntöjä on helppo testata selaimessa. Voisimme toki kirjoittaa Javascript-koodin, joka testaa deletointia, mutta jokaiseen mahdolliseen tilanteeseen testikoodinkaan tekeminen ei ole aina paras ratkaisu. -->
So how do we test the delete operation? HTTP GET requests are easy to make from the browser. We could write some JavaScript for testing deletion, but writing test code is not always the best solution in every situation.

<!-- On olemassa useita backendin testaamista helpottavia työkaluja, eräs näistä on edellisessä osassa nopeasti mainittu komentorivityökalu [curl](https://curl.haxx.se). -->
Many tools exist for making the testing of backends easier. One of these is the command line program [curl](https://curl.haxx.se) that was mentioned briefly in the previous part of the material.

<!-- Käytetään nyt kuitenkin [postman](https://www.getpostman.com/)-nimistä sovellusta. -->
Instead of curl, we will take a look at using [Postman](https://www.getpostman.com/) for testing the application.

<!-- Asennetaan postman ja kokeillaan -->
Let's install Postman and try it out:

![](../images/3/11.png)

<!-- Postmanin käyttö on tässä tilanteessa suhteellisen yksinkertaista, riittää määritellä url ja valita oikea pyyntötyyppi. -->
Using Postman is quite easy in this situation. It's enough to define the url and then select the correct request type.

<!-- Palvelin näyttää vastaavan oikein. Tekemällä HTTP GET osoitteeseen <http://localhost:3001/notes> selviää että poisto-operaatio oli onnistunut, muistiinpanoa, jonka id on 2 ei ole enää listalla. -->
The backend server appears to respond correctly. By making an HTTP GET request to <http://localhost:3001/notes> we see that the note with the id 2 is no longer in the list, which indicates that the deletion was successful. 

<!-- Koska muistiinpanot on talletettu palvelimen muistiin, uudelleenkäynnistys palauttaa tilanteen ennalleen. -->
Because the notes in the application are only saved to memory, the list of notes will return to its original state when we restsart the application.

<!-- ### Visual Studio Coden REST client -->
### The Visual Studio Code REST client

<!-- Jos käytät Visual Studio Codea, voit postmanin sijaan käyttää VS Coden [REST client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) -pluginia. -->
If you use Visual Studio Code, you can use the VS Code [REST client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) plugin instead of Postman.

<!-- Kun plugin on asennettu, on sen käyttö erittäin helppoa. Tehdään projektin juureen hakemisto <i>requests</i>, jonka sisään talletetaan REST Client -pyynnöt <i>.rest</i>-päätteisinä tiedostoina. -->
Once the plugin is installed, using it is very simple. We make a directory at the root of application named <i>requests</i>. We save all the REST client requests in the directory as files that end with the <i>.rest</i> extension.

<!-- Luodaan kaikki muistiinpanot hakevan pyynnön määrittelevä tiedosto <i>get\_all\_notes.rest</i> -->
Let's create a new <i>get\_all\_notes.rest</i> file and for defining the request that fetches all notes.

![](../images/3/12.png)

<!-- Klikkaamalla tekstiä <i>Send Request</i>, REST client suorittaa määritellyn HTTP-pyynnön ja palvelimen vastaus avautuu editoriin: -->
By clicking the <i>Send Request</i> text, the REST client will execute the HTTP request and response from the server is opened in the editor.

![](../images/3/13.png)

<!-- ### Datan vastaanottaminen -->
### Receiving data

<!-- Toteutetaan seuraavana uusien muistiinpanojen lisäys, joka siis tapahtuu tekemällä HTTP POST -pyyntö osoitteeseen http://localhost:3001/notes ja liittämällä pyynnön mukaan eli [bodyyn](https://www.w3.org/Protocols/rfc2616/rfc2616-sec7.html#sec7) luotavan muistiinpanon tiedot JSON-muodossa. -->
Next, let's make it possible to add new notes to the server. Adding a note happens by making an HTTP POST request to the address http://localhost:3001/notes, and by sending all the information for the new note in the request [body](https://www.w3.org/Protocols/rfc2616/rfc2616-sec7.html#sec7) in the JSON format.

<!-- Jotta pääsisimme pyynnön mukana lähetettyyn dataan helposti käsiksi, tarvitsemme [body-parser](https://github.com/expressjs/body-parser)-kirjaston apua. -->
In order to access the data easily, we need the help of the [body-parser](https://github.com/expressjs/body-parser) library.

<!-- Otetaan body-parser käyttöön ja luodaan alustava määrittely HTTP POST -pyynnön käsittelyyn -->
Let's import body-parser and implement an initial handler for dealing with the HTTP POST requests:

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

<!-- Tapahtumankäsittelijäfunktio pääsee dataan käsiksi olion _request_ kentän <i>body</i> avulla. -->
The event handler function can access the data from the <i>body</i> property of the _request_ object.

<!-- Ilman body-parser-käyttöönottoa pyynnön kentän <i>body</i> arvo olisi ollut määrittelemätön. body-parserin toimintaperiaatteena on, että se ottaa pyynnön mukana olevan JSON-muotoisen datan, muuttaa sen Javascript-olioksi ja sijoittaa _request_-olion kenttään <i>body</i> ennen kuin routen käsittelijää kutsutaan. -->
Without body-parser the <i>body</i> property would be undefined. The body-parser functions so that it takes the JSON data of a request, transforms it into a JavaScript object and then attaches it to the <i>body</i> property of the _request_ object before the route handler is called.

<!-- Toistaiseksi sovellus ei vielä tee vastaanotetulle datalle mitään muuta kuin tulostaa sen konsoliin ja palauttaa sen pyynnön vastauksessa. -->
For the time being the application does not do anything with the received data besides printing it to the console and sending it back in the response.

<!-- Ennen toimintalogiikan viimeistelyä varmistetaan ensin postmanilla, että lähetetty tieto menee varmasti perille. Pyyntötyypin ja urlin lisäksi on määriteltävä myös pyynnön mukana menevä data eli <i>body</i>: -->
Before we implement the rest of the application logic, let's verify with Postman that the data is actually received by the server. In addition to defining the URL and request type in Postman, we also have to define the data sent in the <i>body</i>:

![](../images/3/14.png)

<!-- Sovellus tulostaa lähetetyn vastaanottamansa datan terminaaliin: -->
The application prints the data that we sent in the request to the console:

![](../images/3/15.png)

<!-- **HUOM** kun ohjelmoit backendia, <i>pidä sovellusta suorittava konsoli koko ajan näkyvillä</i>. Nodemonin ansiosta sovellus käynnistyy uudelleen jos koodiin tehdään muutoksia. Jos seuraat konsolia, huomaat välittömästi jos sovelluksen koodiin tulee joku perustavanlaatuinen virhe: -->
**NB** <i>Keep the terminal running the application visible at all times</i> when you are working on the backend. Thanks to Nodemon any changes we make to the code will restart the application. If you pay attention to the console, you will immediately be able to pick up on errors that occur in the application:

![](../images/3/16.png)

<!-- Vastaavasti konsolista kannattaa seurata reagoiko backend odotetulla tavalla, esim. kun sovellukselle lähetetään dataa metodilla HTTP POST. Backendiin kannattaa luonnollisesti lisäillä runsaat määrät <em>console.log</em>-komentoja kun sovellus on kehitysvaiheessa.  -->
Similarly, it is useful to check the console for making sure that the backend behaves like we expect it in different situations, like when we send data with an HTTP POST request. Naturally, it's a good idea to add lots of <em>console.log</em> commands to the code when the application is still being developed.

<!-- Eräs potentiaalinen ongelmanlähde on se, että dataa lähettäessä, sen headerille <i>Content-Type</i> ei aseteta oikeaa arvoa. Näin tapahtuu esim. jos Postmanissa bodyn tyypiä ei määritellä oikein: -->
A potential cause for issues is an incorrectly set <i>Content-Type</i> header in requests. This can happen with Postman if the type of body is not defined correctly:

![](../images/3/17a.png)

<!-- headerin <i>Content-Type</i> arvoksi asettuu <i>text/plain</i> -->
The <i>Contept-Type</i> header is set to <i>text/plain</i>:

![](../images/3/18.png)

<!-- Palvelin näyttää vastaanottavan ainoastaan tyhjän olion -->
The server appears to only receive an empty object:

![](../images/3/19.png)

<!-- Ilman oikeaa headerin arvoa palvelin ei osaa parsia dataa oikeaan muotoon. Se ei edes yritä arvailla missä muodossa data on, sillä potentiaalisia datan siirtomuotoja eli <i>Content-Typejä</i> on olemassa [suuri määrä](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types). -->
The server will not be able to parse the data correctly without the correct value in the header. It won't even try to guess the format of the data, since there's a [massive amount](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types) of potential <i>Content-Types</i>.

<!-- Jos käytät VS Codea niin edellisessä luvussa esitelty REST client kannattaa asentaa viimeistään <i>nyt</i>. POST-pyyntö tehdään REST clientillä seuraavasti: -->
If you are using VS Code, then you should install the REST client form the previous chapter <i>now, if you didn't already</i>. The POST request can be sent with the REST client like this:

![](../images/3/20.png)

<!-- Eli pyyntöä varten on luotu oma tiedosto <i>new\_note.rest</i>. Pyyntö on muotoiltu [dokumentaation ohjetta](https://github.com/Huachao/vscode-restclient/blob/master/README.md#usage) noudatellen. -->
We created a new <i>new\_note.rest</i> file for the request. The request is formatted according to the [instructions in the documentation](https://github.com/Huachao/vscode-restclient/blob/master/README.md#usage).

<!-- REST clientin eräs suuri etu Postmaniin verrattuna on se, että pyynnöt saa kätevästi talletettua projektin repositorioon ja tällöin ne ovat helposti koko kehitystiimin käytössä. Postmanillakin on mahdollista tallettaa pyyntöjä, mutta tilanne menee helposti kaaoottiseksi etenkin jos työn alla on useita toisistaan riippumattomia projekteja. -->
One benefit that the REST client has over Postman is that the requests are handily available at the root of the project repository, and they can be distributed to everyone in the development team. Postman also allows users to save requests, but the situation can get quite chaotic especially when you're working on multiple unrelated projects.

<!-- > **Tärkeä sivuhuomio** -->
> **Important sitenode**
>
<!-- > Välillä debugatessa tulee vastaan tilanteita, joissa backendissä on tarve selvittää mitä headereja HTTP-pyynnöille on asetettu. Eräs menetelmä tähän on _request_-olion melko kehnosti nimetty metodi [get](http://expressjs.com/en/4x/api.html#req.get), jonka avulla voi selvittää yksittäisen headerin arvon. _request_-oliolla on myös kenttä <i>headers</i>, jonka arvona ovat kaikki pyyntöön liittyvät headerit. -->
> Sometimes when you're debugging you may want to find out what headers have been set to the HTTP request. One way of accomplishing this is through the [get](http://expressjs.com/en/4x/api.html#req.get) method of the _request_ object, that can be used for getting the value of a single header. The _request_ object also has the <i>headers</i> property, that contains all of the headers of the application.
>
<!-- > Ongelmia voi esim syntyä jos jätät vahingossa VS REST clientillä ylimmän rivin ja headerit määrittelevien rivien väliin tyhjän rivin. Tällöin REST client tulkitsee, että millekään headerille ei aseteta arvoa ja näin backend ei osaa tulkita pyynnön mukana olevaa dataa JSON:iksi. -->
> Problems can occur with the VS REST client if you accidentally add an empty line between the top row and the row specifying the HTTP headers. In this situation the REST client interprets this to mean that all headers are left empty, which means that the backend server has no way of knowing that the data it has received is in the JSON format.
>
<!-- > Puuttuvan <i>content-type</i>-headerin ongelma selviää kun backendissa tulostaa pyynnön headerit esim. komennolla _console.log(request.headers)_ -->
You will be able to spot this missing <i>Content-Type</i> header if at some point in your code you print all of the request headers with the _console.log(request.headers)_ command.

<!-- Palataan taas sovelluksen pariin. Kun tiedämme, että sovellus vastaanottaa tiedon oikein, voimme viimeistellä sovelluslogiikan: -->
Let's return to the application. Once we know that the application receives data correctly, it's time to finalize the handling of the request:

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

<!-- Uudelle muistiinpanolle tarvitaan uniikki id. Ensin selvitetään olemassaolevista id:istä suurin muuttujaan _maxId_. Uuden muistiinpanon id:ksi asetetaan sitten _maxId + 1_. Tämä tapa ei ole itseasiassa kovin hyvä, mutta emme nyt välitä siitä sillä tulemme pian korvaamaan tavan, jolla muistiinpanot talletetaan. -->
We need a unique id for the note. First we find out the largest id number in the current list and assign it to the _maxId_ variable. The id of the new note is then defined as _maxId + 1_. This method is in fact not recommended, but we will live with it for now as we will replace it soon enough.

<!-- Tämän hetkisessä versiossa on vielä se ongelma, että voimme HTTP POST -pyynnöllä lisätä mitä tahansa kenttiä sisältäviä olioita. Parannellaan sovellusta siten, että kenttä <i>content</i> ei voi olla tyhjä. Kentille <i>important</i> ja <i>date</i> asetetaan oletusarvot. Kaikki muut kentät hylätään: -->
The current version still has the problem that the HTTP POST request can be used to add objects with arbitrary properties. Let's improve the application by defining that the <i>concept</i> property may not be empty. The <i>important</i> and <i>date</i> properties will be given default values. All other properties are discarded:

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

<!-- Tunnisteena toimivan id-kentän arvon generointilogiikka on eriytetty funktioon _generateId_. -->
The logic for generating the new id number for notes has been extracted into a separate _generateId_ function.

<!-- Jos vastaanotetulta datalta puuttuu sisältö kentästä <i>content</i>, vastataan statuskoodilla [400 bad request](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.1): -->
If the received data is missing a value for the <i>content</i> property, the server will respond to the request with the status code [400 bad request](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.1):

```js
if (!body.content) {
  return response.status(400).json({ 
    error: 'content missing' 
  })
}
```

<!-- Huomaa, että returnin kutsuminen on tärkeää, jos sitä ei tapahdu, jatkaa koodi suoritusta metodin loppuun asti ja virheellinen muistiinpano tallettuu! -->
Notice that calling return is crucial, because other the code will execute to the very end and the malformed note gets saved to the application.

<!-- Jos content-kentällä on arvo, luodaan muistiinpano syötteen perusteella. Kuten edellisessä osassa mainitsimme, aikaleimoja ei kannata luoda selaimen koodissa, sillä käyttäjän koneen kellon aikaan ei voi luottaa. Aikaleiman eli kentän <i>date</i> arvon generointi tapahtuukin nyt palvelimen toimesta. -->
If the content property has a value, the note will be based on the received data. As mentioned previously, it is better to generate timestamps on the server than in the browser, since we can't trust that host machine running the browser has its clock set correctly. The generation of the <i>date</i> property is now done by the server.

<!-- Jos kenttä <i>important</i> puuttuu, asetetaan sille oletusarvo <i>false</i>. Oletusarvo generoidaan nyt hieman erikoisella tavalla: -->
If the <i>important</i> property is missing, we will default the value to <i>false</i>. The default value is currently generated in a rather odd-looking way:

```js
important: body.important || false,
```

<!-- jos sovelluksen vastaanottamassa muuttujaan _body_ talletetussa datassa on kenttä <i>important</i>, tulee lausekkeelle sen arvo. Jos kenttää ei ole olemassa, tulee lausekkeen arvoksi oikeanpuoleinen osa eli <i>false</i>. -->
If the data saved in the _body_ variable has the <i>important</i> property, the expression will evaluate to its value. If the property does not exist, then the expression will evaluate to false which is defined on the right-hand side of the the vertical lines.

<!-- > Jos ollaan tarkkoja, niin kentän <i>important</i> arvon ollessa <i>false</i>, tulee lausekkeen <em>body.important || false</em> arvoksi oikean puoleinen <i>false</i>... -->
> To be exact, when the <i>important</i> property is <i>false</i>, then the <em>body.important || false</em> expression will in fact return the <i>false</i> from the right-hand side...

<!-- Sovelluksen tämän hetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-1) -->
You can find the code for our current application in its entirety in the <i>part3-1</i> branch of [this github repository](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-1).

<!-- Huomaa, että repositorion master-haarassa on myöhemmän vaiheen koodi, tämän hetken koodi on branchissa [part3-1](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-1): -->
Notice that the master branch of the repository contains the code from a later version of the application. The code for the current state of the application is specifically in branch [part3-1](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-1).

![](../images/3/21.png)

<!-- Jos kloonaat projektin itsellesi, suorita komento _npm install_ ennen käynnistämistä eli komentoa _npm start_ tai _npm run watch_. -->
If you clone the project, run the _npm install_ command before starting the application with _npm start_ or _npm run watch_.

<!-- Vielä pieni huomio ennen tehtäviä. Uuden id:n generoiva funktio näyttää seuraavalta -->
One more thing before we move onto the exercises. The function for generating id's looks currently like this:

```js
const generateId = () => {
  const maxId = notes.length > 0
    ? Math.max(...notes.map(n => n.id))
    : 0
  return maxId + 1
}
```

<!-- Koodi sisältää hieman erikoisen näköisen rivin -->
The function body contains a row that looks a bit intriguing:

```js
Math.max(...notes.map(n => n.id))
```

<!-- Mitä rivillä tapahtuu? <em>notes.map(n => n.id)</em> muodostaa taulukon, joka koostuu muistiinpanojen id-kentisstä. [Math.max](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/max) palauttaa maksimin sille parametrina annetuista luvuista. <em>notes.map(n => n.id)</em> on kuitenkin <i>taulukko</i>, joten se ei kelpaa parametriksi komennolle _Math.max_. Taulukko voidaan muuttaa yksittäisiksi luvuiksi käyttäen taulukon [spread](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)-syntaksia, eli kolmea pistettä <em>... taulukko</em>. -->
What exactly is happening in that line of code? <em>notes.map(n => n.id)</em> creates a new array that contains all the id's of the notes. [Math.max](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/max) returns the maximum value of the numbers that are passed to it. However, <em>notes.map(n => n.id)</em> is an <i>array</i> so it can't directly be given as a parameter to _Math.max_. The array can be transformed into individual numbers by using the "three dot" [spread](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) syntax <em>... taulukko</em>.

</div>

<div class="tasks">

<!-- ### Tehtäviä  -->
### Exercises

<!-- **HUOM:** tämän osan tehtäväsarja kannattaa tehdä omaan git-repositorioon, suoraan repositorion juureen! Jos et tee näin, joudut ongelmiin tehtävässä 3.10 -->
**NB:** It's recommended to do all of the exercises from this part into a new dedicated git repository, and place your source code right at the root of the repository. Otherwise you will run into problems in exercise 3.10.

<!-- **HUOM2:** Koska nyt ei ole kyse frontendista ja Reactista, sovellusta <strong>ei luoda</strong> create-react-app:illa vaan komennolla <em>npm init</em> kuten ylempänä tämän osan materiaalissa. -->
**NB:** Because this is not a frontend project and we are not working with React, the application <strong>is not created</strong> with create-react-app. You initialize this project with the <em>npm init</em> command that was demonstrated earlier in this part of the material.

<!-- **Vahva suositus:** kun teet backendin koodia, pidä koko ajan silmällä mitä palvelimen koodia suorittavassa konsolissa tapahtuu. -->
**Strong recommendation:** When you are working on backend code, always keep an eye on what's going on in the terminal that is running your application.

<!-- #### 3.1 puhelinluettelon backend step1 -->
#### 3.1: Phonebook backend step1

<!-- Tee Node-sovellus, joka tarjoaa osoitteessa <http://localhost:3001/api/persons> kovakoodatun taulukon puhelinnumerotietoja: -->
Implement a Node application that returns a hardcoded list of phonebook entries from the address <http://localhost:3001/api/persons>:

![](../images/3/22a.png)

<!-- Huomaa, että Noden routejen määrittelyssä merkkijonon <i>api/persons</i> vinoviiva käyttäytyy kuten mikä tahansa muu merkki. -->
Notice that the forward slash in the route <i>api/persons</i> is not a special character, and is just like any other character in thes string. 

<!-- Sovellus pitää pystyä käynnistämään komennolla _npm start_. -->
The application must be started with the command _npm start_.

<!-- Komennolla _npm run watch_ käynnistettäessa sovelluksen tulee käynnistyä uudelleen kun koodiin tehdään muutoksia. -->
The application must also offer an _npm run command_ that will run the application and restart the server whenever changes are made and saved to a file in the source code.

<!-- #### 3.2: puhelinluettelon backend step2 -->
#### 3.2: Phonebook backend step2

<!-- Tee sovelluksen osoitteeseen <http://localhost:3001/info> suunnilleen seuraavanlainen sivu -->
Implement a page at the address <http://localhost:3001/info> that looks roughly like this:

![](../images/3/23.png)

<!-- eli sivu kertoo pyynnön tekohetken sekä sen kuinka monta puhelinluettelotietoa sovelluksen muistissa olevassa taulukossa on. -->
The page has to show the time that the request was received and how many entries are in the phonebook at the time of processing the request.

<!-- #### 3.3: puhelinluettelon backend step3 -->
#### 3.3: Phonebook backend step3

<!-- Toteuta toiminnallisuus yksittäisen puhelinnumerotiedon näyttämiseen. Esim. id:n 5 omaavan numerotiedon url on <http://localhost:3001/api/persons/5> -->
Implement the functionality for displaying the information for a single phonebook entry. The url for getting the data for a person with the id 5 should be <http://localhost:3001/api/persons/5>

<!-- Jos id:tä vastaavaa puhelinnumerotietoa ei ole, tulee palvelimen vastata asianmukaisella statuskoodilla. -->
If an entry for the given id is not found, the server has to respond with the appropriate status code.

<!-- #### 3.4 puhelinluettelon backend step4 -->
#### 3.4: Phonebook backend step4

<!-- Toteuta toiminnallisuus, jonka avulla puhelinnumerotieto on mahdollista poistaa numerotiedon yksilöivään URL:iin tehtävällä HTTP DELETE -pyynnöllä. -->
Implement functionality that makes it possible to delete a single phonebook entry by making an HTTP DELETE request to the unique URL of that phonebook entry.

<!-- Testaa toiminnallisuus Postmanilla tai Visual Studio Coden REST clientillä -->
Test that your functionality works with either Postman or the Visual Studio Code REST client.

<!-- #### 3.5: puhelinluettelon backend step5 -->
#### 3.5: Phonebook backend step5

<!-- Laajenna backendia siten, että uusia puhelintietoja on mahdollista lisätä osoitteeseen <http://localhost:3001/api/persons> tapahtuvalla HTTP POST -pyynnöllä. -->
Expand the backend so that new phonebook entries can be added by making HTTP POST requests to the address <http://localhost:3001/api/persons>.

<!-- Generoi uuden puhelintiedon tunniste funktiolla [Math.random](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random). Käytä riittävän isoa arvoväliä jotta arvottu id on riittävän suurella todennäköisyydellä sellainen, joka ei ole jo käytössä. -->
Generate a new id for the phonebook entry with the [Math.random](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random) function. Use a big enough range for your random values so that the likelihood of creating duplicate id's is small.

<!-- #### 3.6: puhelinluettelon backend step6 -->
#### 3.6: Phonebook backend step6

<!-- Tee uuden numeron lisäykseen virheiden käsittely. Pyyntö ei saa onnistua, jos -->
<!-- - nimi tai numero puuttuu -->
<!-- - lisättävä nimi on jo luettelossa -->
Implement error handling for creating new entries. The request is not allowed to succeed, if:
- The name or number is missing 
- The name already exists in the phonebook

<!-- Vastaa asiaankuuluvalla statuskoodilla, liitä vastaukseen mukaan myös tieto, joka kertoo virheen syyn, esim: -->
Respond to requests like these with the appropriate status code, and also send back information that explains the reason for the error, e.g.:

```js
{ error: 'name must be unique' }
```

</div>

<div class="content">

<!-- ### Huomioita HTTP pyyntötyyppien käytöstä -->
### About HTTP request types

[HTTP-standardi](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) puhuu pyyntötyyppien yhteydessä kahdesta ominaisuudesta, **safe** ja **idempotent**.
[The HTTP standard](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) talks about two qualities related to request types, **safety** and **idempotence**.

<!-- HTTP-pyynnöistä GET:in tulisi olla <i>safe</i>: -->
The HTTP GET request should be <i>safe</i>:

> <i>In particular, the convention has been established that the GET and HEAD methods SHOULD NOT have the significance of taking an action other than retrieval. These methods ought to be considered "safe".</i>

<!-- Safety siis tarkoittaa, että pyynnön suorittaminen ei saa aiheuttaa palvelimelle <i>sivuvaikutuksia</i> eli esim. muuttaa palvelimen tietokannan tilaa, pyynnön tulee ainoastaan palauttaa palvelimella olevaa dataa. -->
Safety means that the executing request must not cause any <i>side effects</i> in the server. By side-effects we mean that the state of the database must not change as a result of the request, and the response must only return data that already exists on the server.

<!-- Mikään ei automaattisesti takaa, että GET-pyynnöt olisivat luonteeltaan <i>safe</i>, kyseessä onkin HTTP-standardin suositus palvelimien toteuttajille. RESTful-periaatetta noudattaessa GET-pyyntöjä käytetäänkin aina siten, että ne ovat safe. -->
Nothing can ever guarantee that a GET request is actually <i>safe</i>, this is in fact just a recommendation that is defined in the HTTP standard. By adhering to RESTful principles in our API, GET requests are in fact always used in a way that they are <i>safe</i>.

<!-- HTTP-standardi määrittelee myös pyyntötyypin [HEAD](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.4), jonka tulee olla safe. Käytännössä HEAD:in tulee toimia kuten GET, mutta se ei palauta vastauksenaan muuta kuin statuskoodin ja headerit, viestin bodyä HEAD ei palauta ollenkaan. -->
The HTTP standard also defines the request type [HEAD](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.4), that ought to be safe. In practice HEAD should work exactly like GET but it does not return anything but the status code and response headers. The response body will not be returned when you make a HEAD request.

<!-- HTTP-pyynnöistä muiden paitsi POST:in tulisi olla <i>idempotentteja</i>: -->
All HTTP requests except POST should be <i>idempotent</i>:

> <i>Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request. The methods GET, HEAD, PUT and DELETE share this property</i>

<!-- Eli jos pyynnöllä on sivuvaikutuksia, lopputulos on sama suoritetaanko pyyntö yhden tai useamman kerran. -->
This means that if a request has side-effects, then the result should be same regardless of how many times the request is sent.

<!-- Esim. jos tehdään HTTP PUT pyyntö osoitteeseen <i>/notes/10</i> ja pyynnön mukana on <em>{ content: "ei sivuvaikutuksia", important: true }</em>, on lopputulos sama riippumatta siitä kuinka monta kertaa pyyntö suoritetaan. -->
If we make an HTTP PUT request to the url <i>/notes/10</i> and with the request we send the data <em>{ content: "ei sivuvaikutuksia", important: true }</em>, the result is the same regardless of many times the request is sent.

<!-- Kuten metodin GET <i>safety</i> myös <i>idempotence</i> on HTTP-standardin suositus palvelimien toteuttajille. RESTful-periaatetta noudattaessa GET, HEAD, PUT ja DELETE-pyyntöjä käytetäänkin aina siten, että ne ovat idempotentteja. -->
Like <i>safety</i> for the GET request, also <i>idempotence</i> is simply a recommendation in the HTTP standard and not something that can be guaranteed simply based on the request type. However, when our API adheres to RESTful principles, then GET, HEAD, PUT, and DELETE requests are used in such a way that they are idempotent.

<!-- HTTP pyyntötyypeistä POST on ainoa joka ei ole <i>safe</i> eikä <i>idempotent</i>. Jos tehdään 5 kertaa HTTP POST -pyyntö osoitteeseen <i>/notes</i> siten että pyynnön mukana on <em>{ content: "monta samaa", important: true }</em>, tulee palvelimelle 5 saman sisältöistä muistiinpanoa. -->
POST is the only HTTP request type that is neither <i>safe</i> nor <i>idempotent</i>. If we make an HTTP POST request five time into the <i>/notes</i> url, so that each time we send the 

<!-- ### Middlewaret -->
### Middleware

<!-- Äsken käyttöönottamamme [body-parser](https://github.com/expressjs/body-parser) on terminologiassa niin sanottu [middleware](http://expressjs.com/en/guide/using-middleware.html). -->
The [body-parser](https://github.com/expressjs/body-parser) we took into use earlier is a so-called [middleware](http://expressjs.com/en/guide/using-middleware.html).

<!-- Middlewaret ovat funktioita, joiden avulla voidaan käsitellä _request_- ja _response_-olioita. -->
Middleware are functions that can be used for handling _request_ and _response_ objects.

<!-- Esim. body-parser ottaa pyynnön mukana tulevan raakadatan _request_-oliosta, parsii sen Javascript-olioksi ja sijoittaa olion _request_:in kenttään <i>body</i> -->
The body-parser we used earlier takes the raw data from the requests that's stored in the _request_ object, parses it into a JavaScript object and assigns it to the _request_ object as a new property <i>body</i>.

<!-- Middlewareja voi olla käytössä useita, jolloin ne suoritetaan peräkkäin siinä järjestyksessä kun ne on otettu koodissa käyttöön. -->
In practice you can use several middleware at the same time. When you have more than one, they're executed one by one in the order that they were taken into use in express.

<!-- Toteutetaan itse yksinkertainen middleware, joka tulostaa konsoliin palvelimelle tulevien pyyntöjen perustietoja. -->
Let's implement our own middleware that prints information about every request that is sent to the server.

<!-- Middleware on funktio, joka saa kolme parametria: -->
Middleware is a function that receives three parameters:

```js
const requestLogger = (request, response, next) => {
  console.log('Method:', request.method)
  console.log('Path:  ', request.path)
  console.log('Body:  ', request.body)
  console.log('---')
  next()
}
```

<!-- Middleware kutsuu lopussa parametrina olevaa funktiota _next_, jolla se siirtää kontrollin seuraavalle middlewarelle. -->
At the end of the function body the _next_ function that was passed as a parameter is called. The _next_ function yields control to the next middleware.

<!-- Middleware otetaan käyttöön seuraavasti: -->
Middleware are taken into use like this:

```js
app.use(requestLogger)
```

<!-- Middlewaret suoritetaan siinä järjestyksessä, jossa ne on otettu käyttöön sovellusolion metodilla _use_. Huomaa, että _bodyParser_ tulee ottaa käyttään ennen middlewarea _requestLogger_, muuten <i>request.body</i> ei ole vielä alustettu loggeria suoritettaessa! -->
Middleware functions are called in the order that they're taken into use with the express server object's _use_ method. Notice that _bodyParser_ is taken into use before the _requestLogger_ middleware, because otherwise <i>request.body</i> will not be initialized when the logger is executed!

<!-- Middlewaret tulee ottaa käyttöön ennen routeja jos ne halutaan suorittaa ennen niitä. On myös eräitä tapauksia, joissa middleware tulee määritellä vasta routejen jälkeen, käytännössä tällöin on kyse middlewareista, joita suoritetaan vain, jos mikään route ei käsittele HTTP-pyyntöä. -->
Middleware functions have to be taken into use before routes if we want them to be executed before the route event handlers are called. There are also situations where we want to define middleware functions after routes. In practice, this means that we are defining middleware functions that are only called if no route handles the HTTP request.

<!-- Lisätään routejen jälkeen seuraava middleware, jonka ansiosta saadaan routejen käsittelemättömistä virhetilanteista JSON-muotoinen virheilmoitus: -->
Let's add the following middleware after our routes, that is used for catching requests made to non-existent routes. For these requests the middleware will return an error message in the JSON format.

```js
const unknownEndpoint = (request, response) => {
  response.status(404).send({ error: 'unknown endpoint' })
}

app.use(unknownEndpoint)
```

<!-- Sovelluksen tämän hetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-2), branchissa <i>part3-2</i>. -->
You can find the code for our current application in its entirety in the <i>part3-2</i> branch of [this github repository](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-2).

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- #### 3.7: puhelinluettelon backend step7 -->
#### 3.7: Phonebook backend step7

<!-- Lisää sovellukseesi loggausta tekevä middleware [morgan](https://github.com/expressjs/morgan). Konfiguroi se logaamaan konsoliin <i>tiny</i>-konfiguraation mukaisesti. -->
Add the [morgan](https://github.com/expressjs/morgan) middleware to your application for logging. Configure it to log messages to your console based on the <i>tiny</i> configuration.

<!-- Morganin ohjeet eivät ole ehkä kaikkein selvimmät ja joudut kenties miettimään hiukan. Toisaalta juuri koskaan dokumentaatio ei ole aivan itsestäänselvää, joten kryptisempiäkin asioita on hyvä oppia tulkitsemaan. -->
The documentation for Morgan is not the best, and you may have to some time figuring out how to configure it correctly. However, most documentation in the world falls under the same category and it's good to learn to decipher and interpret more cryptic documentation.

<!-- Morgan asennetaan kuten muutkin kirjastot, eli komennolla _npm install_ ja sen käyttöönotto tapahtuu kaikkien middlewarejen tapaan komennolla _app.use_ -->
Morgan is installed just like all other libraries with the _npm install_ command. Taking morgan into use happens the same as configuring any other middle by using the _app.use_ command.

<!-- #### 3.8*: puhelinluettelon backend step8 -->
#### 3.8*: Phonebook backend step8

<!-- Konfiguroi morgania siten, että se näyttää myös HTTP POST -pyyntöjen mukana tulevan datan: -->
Configure morgan so that it also shows the data sent in HTTP POST requests:

![](../images/3/24.png)

<!-- Tämä tehtävä on kohtuullisen haastava vaikka koodia ei tarvitakkaan paljoa.  -->
This exercise can be quite challenging even though the solution does not require a lot of code.

<!-- Tehtävän voi tehdä muutamallakin tavalla, eräs näistä onnistuu hyödyntämällä seuraavia -->
This exercise can be completed in a few different ways. One of the possible solutions utilizes these two techniques:
- [creating new tokens](https://github.com/expressjs/morgan#creating-new-tokens)
- [JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)

</div>
