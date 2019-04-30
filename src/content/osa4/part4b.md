---
mainImage: ../../images/part-4.svg
part: 4
letter: b
---

<div class="content">

<!-- Ruvetaan nyt tekemään testejä backendille. Koska backend ei sisällä monimutkaista laskentaa, ei yksittäisiä funktioita testaavia [yksikkötestejä](https://en.wikipedia.org/wiki/Unit_testing) oikeastaan kannata tehdä. Ainoa potentiaalinen yksikkötestattava asia olisi muistiinpanojen metodi _toJSON_.  -->
We will now start writing tests for the backend. Since the backend does not contain any complicated logic, it doesn't make sense to write [unit tests](https://en.wikipedia.org/wiki/Unit_testing) for it. The only potential thing we could unit test is the _toJSON_ method that is used for formatting notes.

<!-- Joissain tilanteissa voisi olla mielekästä suorittaa ainakin osa backendin testauksesta siten, että oikea tietokanta eristettäisiin testeistä ja korvattaisiin "valekomponentilla" eli mockilla. Eräs tähän sopiva ratkaisu olisi [mongo-mock](https://github.com/williamkapke/mongo-mock). -->
In some situations it can be beneficial to implement a part of the backend's tests by mocking the database instead of using a real database. One library that could be used for this is [mongo-mock](https://github.com/williamkapke/mongo-mock).

<!-- Koska sovelluksemme backend on koodiltaan kuitenkin suhteellisen yksinkertainen, päätämme testata sitä kokonaisuudessaan sen tarjoaman REST-apin tasolta, siten että myös testeissä käytetään tietokantaa. Tämän kaltaisia, useita sovelluksen komponentteja yhtäaikaa käyttäviä testejä voi luonnehtia [integraatiotesteiksi](https://en.wikipedia.org/wiki/Integration_testing). -->
Since our application's backend is still relatively simple, we will make the decision to test the entire application through its REST API, so that the database is also included. This kind of testing where multiple components of the system are being tested as a group, can be described as [integration testing](https://en.wikipedia.org/wiki/Integration_testing).

<!-- ### test-ympäristö -->
### Test environment

<!-- Edellisen osan luvussa [Tietokantaa käyttävän version vieminen tuotantoon](/osa3/validointi_ja_es_lint#tietokantaa-kayttavan-version-vieminen-tuotantoon) mainitsimme, että kun sovellusta suoritetaan Herokussa, on se <i>production</i>-moodissa. -->
In one of the previous chapters of the course material, we mentioned that when your backend server is running in Heroku, it is in <i>production</i> mode.

<!-- Noden konventiona on määritellä projektin suoritusmoodi ympäristömuuttujan <i>NODE\_ENV</i> avulla. Lataammekin sovelluksen nykyisessä versiossa tiedostossa <i>.env</i> määritellyt ympäristömuuttujat ainoastaan jos sovellus <i>ei ole</i> production moodissa: -->
The convention in Node is to define the execution mode of the application with the <i>NODE\_ENV</i> environment variable. In our current application we only load the environment variables defined in the <i>.env</i> file is the application is <i>not</i> in production mode.

```js
if (process.env.NODE_ENV !== 'production') {
  require('dotenv').config()
}
```

<!-- Yleinen käytäntö on määritellä sovelluksille omat moodinsa myös sovelluskehitykseen ja testaukseen. -->
It is common practice to define separate modes for development and testing.

<!-- Määritellään nyt tiedostossa <i>package.json</i>, että testejä suoritettaessa sovelluksen <i>NODE\_ENV</i> saa arvokseen <i>test</i>: -->
Next, let's change the scripts in our <i>package.json</i> so that when tests are run, <i>NODE\_ENV</i> gets the value <i>test</i>:

```json
{
  // ...
  "scripts": {
    "start": "NODE_ENV=production node index.js",
    "watch": "NODE_ENV=development nodemon index.js",
    "test": "NODE_ENV=test jest --verbose --runInBand",
    "lint": "eslint ."
  },
  // ...
}
```

<!-- Lisäsimme testit suorittavaan npm-skriptiin myös määreen [runInBand](https://jestjs.io/docs/en/cli.html#runinband), joka estää testien rinnakkaisen suorituksen. Tämä tarkennus on viisainta tehdä sitten, kun testimme tulevat käyttämään tietokantaa. -->
We also added the [runInBand](https://jestjs.io/docs/en/cli.html#runinband) option to the npm script that executes the tests. This option will prevent Jest from running tests in parallel and we will discuss its significance once our tests start using the database.

<!-- Samalla määriteltiin, että suoritettaessa sovellusta komennolla _npm run watch_ eli nodemonin avulla, on sovelluksen moodi <i>development</i>. Jos sovellusta suoritetaan normaalisti Nodella, on moodiksi määritelty <i>production</i>. -->
We specified the mode of the application to be <i>development</i> in the _npm run watch_ script that uses nodemon. We also specified that the default _npm start_ command will define the mode as <i>production</i>.

<!-- Määrittelyssämme on kuitenkin pieni ongelma: se ei toimi Windowsilla. Tilanne korjautuu asentamalla kirjasto [cross-env](https://www.npmjs.com/package/cross-env) komennolla -->
There is a slight issue in the way that we have specified the mode of the application in our scripts: it will not work on Windows. We can correct this by installing the [cross-env](https://www.npmjs.com/package/cross-env) package with the command:

```bash
npm install --save-dev cross-env
```

<!-- ja muuttamalla <i>package.js</i> kaikilla käyttöjärjestelmillä toimivaan muotoon -->
We can then achieve cross-platform compatibility by using the cross-env library in our npm scripts defined in <i>package.json</i>:

```json
{
  // ...
  "scripts": {
    "start": "cross-env NODE_ENV=production node index.js",
    "watch": "cross-env NODE_ENV=development nodemon index.js",
    "test": "cross-env NODE_ENV=test jest --verbose --runInBand",
    "lint": "eslint ."
  },
  // ...
}
```

<!-- Nyt sovelluksen toimintaa on mahdollista muokata sen suoritusmoodiin perustuen. Eli voimme määritellä, esim. että testejä suoritettaessa ohjelma käyttää erillistä, testejä varten luotua tietokantaa. -->
Now we can modify the way that our application runs in different modes. As an example of this, we could define the application to use a separate test database when it is running tests.

<!-- Sovelluksen testikanta voidaan luoda tuotantokäytön ja sovelluskehityksen tapaan Mongo DB Atlasiin. Ratkaisu ei ole optimaalinen erityisesti, jos sovellusta on tekemässä yhtä aikaa useita henkilöitä. Testien suoritus nimittäin yleensä edellyttää, että samaa tietokantainstanssia ei ole yhtä aikaa käyttämässä useampia testiajoja. -->
We can create our separate test database in Mongo DB Atlas. This is not an optimal solution in situations where there are many people developing the same application. Test execution in particular typically requires that a single database instance is not used by tests that are running concurrently.

<!-- Testaukseen kannattaisikin käyttää verkossa olevan jaetun tietokannan sijaan mieluummin sovelluskehittäjän paikallisella koneella olevaa tietokantaa. Optimiratkaisu olisi tietysti se, että jokaista testiajoa varten olisi käytettävissä oma tietokanta, sekin periaatteessa onnistuu "suhteellisen helposti" mm. [keskusmuistissa toimivan Mongon](https://docs.mongodb.com/manual/core/inmemory/) ja [docker](https://www.docker.com)-kontainereiden avulla. Etenemme kuitenkin nyt lyhyemmän kaavan mukaan ja käytetään testikantana normaalia Mongoa. -->
It would be better run our tests using a database that is installed and running in the developer's local machine. The optimal solution would be to have every test execution use its own separate database. This is "relatively simple" to achieve by [running Mongo in-memory](https://docs.mongodb.com/manual/core/inmemory/) or by using [Docker](https://www.docker.com) containers. We will not complicate things and will instead continue to use the MongoDB Atlas database.

<!-- Muutetaan konfiguraatiot suorittavaa moduulia seuraavasti: -->
Let's make some changes to the module that defines the application's configuration:

```js
if (process.env.NODE_ENV !== 'production') {
  require('dotenv').config()
}

let port = process.env.PORT
let mongoUrl = process.env.MONGODB_URI

// highlight-start
if (process.env.NODE_ENV === 'test') {
  mongoUrl = process.env.TEST_MONGODB_URI
}
// highlight-end

module.exports = {
  mongoUrl,
  port,
}
```

<!-- Koodi siis lataa ympäristömuuttujat tiedostosta <i>.env</i> jos se <i>ei ole</i> tuotantomoodissa. Tuotantomoodissa käytetään Herokuun asetettuja ympäristömuuttujia. -->
The code imports the environment variables from the <i>.env</i> file if <i>it is not</i> in production mode. In production mode our application will use the environment variables defined in Heroku.

<!-- Tiedostossa <i>.env</i> on nyt määritelty <i>erikseen</i> sekä sovelluskehitysympäristön että testausympäristön tietokannan osoite (esimerkissä molemmat ovat sovelluskehityskoneen lokaaleja mongo-kantoja): -->
The <i>.env</i> file has <i>separate variables</i> for the database addresses of the development and test databases (the addresses shown in the example are for locally running databases):

```bash
MONGODB_URI=mongodb+srv://fullstack:secred@cluster0-ostce.mongodb.net/note-app?retryWrites=true
PORT=3001

// highlight-start
TEST_MONGODB_URI=mongodb+srv://fullstack:fullstack@cluster0-ostce.mongodb.net/note-app-test?retryWrites=true
// highlight-end
```

<!-- Oma tekemämme eri ympäristöjen konfiguroinnista huolehtiva _config_-moduuli toimii hieman samassa hengessä kuin [node-config](https://github.com/lorenwest/node-config)-kirjasto. Oma tekemä konfigurointiympäristö sopii tarkoitukseemme, sillä sovellus on yksinkertainen ja oman konfiguraatio-moduulin tekeminen on myös jossain määrin opettavaista. Isommissa sovelluksissa kannattaa harkita valmiiden kirjastojen, kuten [node-config](https://github.com/lorenwest/node-config):in käyttöä. -->
The _config_ module that we have implemented slightly resembles the [node-config](https://github.com/lorenwest/node-config) package. Writing our own implementation is justified since our application is simple and also because doing so teaches us valuable lessons.

<!-- Muualle koodiin ei muutoksia tarvita. -->
These are the only changes we need to make to our application's code.

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-2), branchissä <i>part4-2</i>. -->
You can find the code for our current application in its entirety in the <i>part4-2</i> branch of [this github repository](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-2).


<!-- ### supertest -->
### supertest

<!-- Käytetään API:n testaamiseen Jestin apuna [supertest](https://github.com/visionmedia/supertest)-kirjastoa. -->
Let's use the [supertest](https://github.com/visionmedia/supertest) package to help us write our tests for testing the API.

<!-- Kirjasto asennetaan kehitysaikaiseksi riippuvuudeksi komennolla -->
We will install the package as a development dependency:

```bash
npm install --save-dev supertest
```

<!-- Luodaan heti ensimmäinen testi tiedostoon <i>tests/note_api.test.js</i> -->
Let's write our first test in the <i>tests/note_api.test.js</i> file:

```js
const mongoose = require('mongoose')
const supertest = require('supertest')
const app = require('../app')

const api = supertest(app)

test('notes are returned as json', async () => {
  await api
    .get('/api/notes')
    .expect(200)
    .expect('Content-Type', /application\/json/)
})

afterAll(() => {
  mongoose.connection.close()
})
```

<!-- Testi importtaa tiedostoon <i>app.js</i> määritellyn Express-sovelluksen ja käärii sen  funktion <i>supertest</i> avulla ns. [superagent](https://github.com/visionmedia/superagent)-olioksi. Tämä olio sijoitetaan muuttujaan <i>api</i> ja sen kautta testit voivat tehdä HTTP-pyyntöjä backendiin. -->
The test imports the Express application from the <i>app.js</i> module and wraps it with the <i>supertest</i> function into a so-called [superagent](https://github.com/visionmedia/superagent) object. This object is assigned to the <i>api</i> variable and tests can use it for making HTTP requests to the backend.

<!-- Testimetodi tekee HTTP GET -pyynnön osoitteeseen <i>api/notes</i> ja varmistaa, että pyyntöön vastataan statuskoodilla 200 ja että data palautetaan oikeassa muodossa, eli että <i>Content-Type</i>:n arvo on <i>application/json</i>. -->
Our test makes an HTTP GET request to the <i>api/notes</i> url and verifies that the request is responded to with the status code 200. The test also verifies that the <i>Content-Type</i> header is set to <i>application/json</i> indicating that the data is in the desired format.

<!-- Testissä on muutama detalji joihin tutustumme vasta [hieman myöhemmin](/osa4/backendin_testaaminen#async-await) tässä osassa. Testikoodin määrittelevä nuolifunktio alkaa sanalla <i>async</i> ja <i>api</i>-oliolle tehtyä metodikutsua edeltää sama <i>await</i>. Teemme ensin muutamia testejä ja tutustumme sen jälkeen async/await-magiaan. Tällä hetkellä niistä ei tarvitse välittää, kaikki toimii kun kirjoitat testimetodit esimerkin mukaan. Async/await-syntaksin käyttö liittyy siihen, että palvelimelle tehtävät pyynnöt ovat <i>asynkronisia</i> operaatioita. [Async/await-kikalla](https://facebook.github.io/jest/docs/en/asynchronous.html) saamme pyynnön näyttämään koodin tasolla synkroonisesti toimivalta. -->
The test contains some details that we will explore [a bit later on](/osa4/backendin_testaaminen#async-await). The arrow function that defines the test is preceded by the <i>async</i> keyword and the method call for the <i>api</i> object is preceded by the <i>await</i> keyword. We will write a few tests and then take a closer look at this async/await magic. Do not concern yourself with them for now, just be assured that the example tests work correctly. The async/await syntax is related to the fact that making a request to the API is an <i>asynchronous</i> operation. The [Async/await syntax](https://facebook.github.io/jest/docs/en/asynchronous.html) can be used for writing asynchronous code with the appearance of synchronous code.

<!-- Kaikkien testien (joita siis tällä kertaa on vain yksi) päätteeksi on vielä lopputoimenpiteenä katkaistava Mongoosen käyttämä tietokantayhteys. Tämä onnistuu helposti metodissa [afterAll](https://facebook.github.io/jest/docs/en/api.html#afterallfn-timeout): -->
Once all the tests (there is currently only one) have finished running we have to close the database connection used by Mongoose. This can be easily achieved with the [afterAll](https://facebook.github.io/jest/docs/en/api.html#afterallfn-timeout) method:

```js
afterAll(() => {
  mongoose.connection.close()
})
```

<!-- Testejä suorittaessa saattaa tulla seuraava ilmoitus -->
When running your tests you may run across the following console warning:

![](../images/4/8.png)

<!-- Jos näin käy, toimitaan [ohjeen](https://mongoosejs.com/docs/jest.html) mukaan ja lisätään projektin hakemiston juureen tiedosto <i>jest.config.js</i> jolla on seuraava sisältö: -->
If this occurs, let's follow the [instructions](https://mongoosejs.com/docs/jest.html) and add a <i>jest.config.js</i> file at the root of the project with the following content:

```js
module.exports = {
  testEnvironment: 'node'
}
```

<!-- Pieni mutta tärkeä huomio: eristimme tämän osan [alussa](/osa4/sovelluksen_rakenne_ja_testauksen_alkeet#sovelluksen-rakenne) Express-sovelluksen tiedostoon <i>app.js</i> ja tiedoston <i>index.js</i> rooliksi jäi sovelluksen käynnistäminen määriteltyyn porttiin Noden <i>http</i>-olion avulla: -->
One tiny but important detail: at the [beginning](/osa4/sovelluksen_rakenne_ja_testauksen_alkeet#sovelluksen-rakenne) of this part we extracted the Express application into the <i>app.js</i> file, and the role of the <i>index.js</i> file was changed to launch the application at the specified port with Node's built-in <i>http</i> object:

```js
const app = require('./app') // varsinainen Express-sovellus
const http = require('http')
const config = require('./utils/config')

const server = http.createServer(app)

server.listen(config.PORT, () => {
  console.log(`Server running on port ${config.PORT}`)
})
```

<!-- Testit käyttävät ainoastaan tiedostossa <i>app.js</i> määriteltyä express-sovellusta: -->
The tests only use the express application defined in the <i>app.js</i> file:

```js
const mongoose = require('mongoose')
const supertest = require('supertest')
const app = require('../app') // highlight-line

const api = supertest(app) // highlight-line

// ...
```

<!-- Supertestin dokumentaatio toteaa seuraavasti -->
The documentation for supertest says the following:

> <i>if the server is not already listening for connections then it is bound to an ephemeral port for you so there is no need to keep track of ports.</i>

<!-- eli Supertest huolehtii testattavan sovelluksen käynnistämisestä sisäisesti käyttämäänsä porttiin. -->
In other words, supertest takes care that the application being tested is started at the port that it uses internally.

<!-- Tehdään pari testiä lisää: -->
Let's write a few more tests:

```js
test('there are five notes', async () => {
  const response = await api.get('/api/notes')

  expect(response.body.length).toBe(3)
})

test('the first note is about HTTP methods', async () => {
  const response = await api.get('/api/notes')

  expect(response.body[0].content).toBe('HTML on helppoa')
})
```

<!-- Molemmat testit sijoittavat pyynnön vastauksen muuttujaan _response_ ja toisin kuin edellinen testi, joka käytti _supertestin_ mekanismeja statuskoodin ja vastauksen headereiden oikeellisuuden varmistamiseen, tällä kertaa tutkitaan vastauksessa olevan datan, eli <i>response.body</i>:n oikeellisuutta Jestin [expect](https://facebook.github.io/jest/docs/en/expect.html#content):in avulla. -->
Both tests store the response of the request to the _response_ variable, and unlike the previous test that used the methods provided by _supertest_ for verifying the status code and headers, this time we are inspecting the response data stored in <i>response.body</i> property. Our tests verify the format and content of the response data with the [expect](https://facebook.github.io/jest/docs/en/expect.html#content) method of Jest.

<!-- Async/await-kikan hyödyt tulevat nyt selkeästi esiin. Normaalisti tarvitsisimme asynkronisten pyyntöjen vastauksiin käsille pääsemiseen promiseja ja takaisinkutsuja, mutta nyt kaikki menee mukavasti: -->
The benefit of using the async/await syntax is starting to become evident. Normally we would have to use callback functions to access the data returned by promises, but with the new syntax things are a lot more comfortable:

```js
const res = await api.get('/api/notes')

// tänne tullaan vasta kun edellinen komento eli HTTP-pyyntö on suoritettu
// muuttujassa res on nyt HTTP-pyynnön tulos
expect(res.body.length).toBe(3)
```

<!-- ### Loggeri -->
### Logger

<!-- HTTP-pyyntöjen tiedot konsoliin kirjoittava middleware häiritsee hiukan testien tulostusta. Konsoliin tulostaminen onkin viisainta estää silloin kuin sovellus on testausmoodissa. Eristetään samalla kaikki konsoliin tulostelu omaan moduliinsa <i>utils/logger.js</i>: -->
The logger middleware that outputs information about the HTTP requests is obstructing the test execution output. It's a good idea to prevent logging when the application is in test mode. While we're at it, let's extract console logging into its own <i>utils/logger.js</i> module:

```js
const info = (...params) => {
  if (process.env.NODE_ENV !== 'test') {
    console.log(...params)
  }
}

const error = (...params) => {
  console.error(...params)
}

module.exports = {
  info, error
}
```

<!-- Loggeri tarjoaa kaksi funktiota, joista _info_ ei tulosta mitään sovelluksen ollessa testausmoodissa. Virhetilanteisiin tarkoitettu funktio _error_ tulostaa konsoliin myös testausmoodissa. -->
The logger offers two functions. The _info_ function does not print anything if the application is in test mode. The _error_ function intended for error logging will still print to console in test mode.

<!-- Otetaan loggeri käyttöön muualla sovelluksessa. Muutoksia tulee middlewaret määrittelevään tiedostoon -->
Let's take the logger into use in the application. We will need to make changes to the file that defines the middleware of the application:

```js
const logger = require('./logger') // highlight-line

const requestLogger = (request, response, next) => {
  // highlight-start
  logger.info('Method:', request.method)
  logger.info('Path:  ', request.path)
  logger.info('Body:  ', request.body)
  logger.info('---')
  // highlight-end
  next()
}

const unknownEndpoint = (request, response) => {
  response.status(404).send({ error: 'unknown endpoint' })
}

const errorHandler = (error, request, response, next) => {
  logger.error(error.message) // highlight-line

  if (error.name === 'CastError' && error.kind === 'ObjectId') {
    return response.status(400).send({ error: 'malformatted id' })
  } else if (error.name === 'ValidationError') {
    return response.status(400).json({ error: error.message })
  }

  next(error)
}

module.exports = {
  requestLogger,
  unknownEndpoint,
  errorHandler
}
```

<!-- ja express-sovelluksen määrittelevään tiedostoon <i>app.js</i>: -->
We also have to make changes to the <i>app.js</i> file that creates the Express application:

```js 
// ...
const logger = require('./utils/logger') // highlight-line

logger.info('connecting to', config.MONGODB_URI) // highlight-line

mongoose.connect(config.MONGODB_URI, { useNewUrlParser: true })
  .then(() => {
    logger.info('connected to MongoDB') // highlight-line
  })
  .catch((error) => {
    logger.error('error connection to MongoDB:', error.message) // highlight-line
  })

// ...
```

<!-- Logauksen eristäminen omaan moduulinsa vastuulle on monellakin tapaa järkevää. Jos esim. päätämme ruveta kirjoittamaan logeja tiedostoon tai keräämään ne johonkin ulkoiseen palveuun kuten [graylog](https://www.graylog.org/) tai [papertrail](https://papertrailapp.com), on muutos helppo tehdä yhteen paikkaan. -->
Extracting logging into its own module is a good idea in more ways than one. If we wanted to start writing logs to a file or send them to an external logging service like [graylog](https://www.graylog.org/) or [papertrail](https://papertrailapp.com) we would only have to make changes in one place.

<!-- ### Tietokannan alustaminen ennen testejä -->
### Initializing the database before tests

<!-- Testaus vaikuttaa helpolta ja testit menevät läpi. Testimme ovat kuitenkin huonoja, niiden läpimeno riippuu tietokannan tilasta (joka sattuu omassa testikannassani olemaan sopiva). Jotta saisimme robustimmat testit, tulee tietokannan tila nollata testien alussa ja sen jälkeen laittaa kantaan hallitusti testien tarvitsema data. -->
Testing appears to be easy and our tests are currently passing. However, our tests are bad as they are dependent on the state of the database (that happens to be correct in my test database). In order to make our tests more robust we have to reset the database and generate the needed test data in a controlled manner before we run the tests.

<!-- Testimme käyttää jo jestin metodia [afterAll](https://facebook.github.io/jest/docs/en/api.html#afterallfn-timeout) sulkemaan tietokannan testien suoritusten jälkeen. Jest tarjoaa joukon muitakin [funktioita](https://facebook.github.io/jest/docs/en/setup-teardown.html#content), joiden avulla voidaan suorittaa operaatioita ennen yhdenkään testin suorittamista tai ennen jokaisen testin suoritusta. -->
Our tests are already using the [afterAll](https://facebook.github.io/jest/docs/en/api.html#afterallfn-timeout) function of Jest to close the connection to the database after the tests are finished executing. Jest offers many other [functions](https://facebook.github.io/jest/docs/en/setup-teardown.html#content) that can be used for executing operations once before any test is run, or every time before a test is run.

<!-- Päätetään alustaa tietokanta ennen <i>jokaisen testin suoritusta,</i> eli funktiossa [beforeEach](https://jestjs.io/docs/en/api.html#aftereachfn-timeout): -->
Let's initialize the database <i>before every test</i> with the [beforeEach](https://jestjs.io/docs/en/api.html#aftereachfn-timeout) function:

```js
const supertest = require('supertest')
const app = require('../app')
const api = supertest(app)
const Note = require('../models/note')

const initialNotes = [
  {
    content: 'HTML on helppoa',
    important: false,
  },
  {
    content: 'HTTP-protokollan tärkeimmät metodit ovat GET ja POST',
    important: true,
  },
]

beforeEach(async () => {
  await Note.deleteMany({})

  let noteObject = new Note(initialNotes[0])
  await noteObject.save()

  noteObject = new Note(initialNotes[1])
  await noteObject.save()
})
```

<!-- Tietokanta siis tyhjennetään aluksi ja sen jälkeen kantaan lisätään kaksi taulukkoon _initialNotes_ talletettua muistiinpanoa. Näin testien suoritus aloitetaan aina hallitusti samasta tilasta. -->
The database is cleared out at the beginning and after that we save the two notes stored in the _initialNotes_ array to the database. By doing this we ensure that the database is in the same state before every test is run.

<!-- Muutetaan kahta jälkimmäistä testiä vielä seuraavasti: -->
Let's also make the following changes to the last two tests:

```js
test('all notes are returned', async () => {
  const response = await api.get('/api/notes')

  expect(response.body.length).toBe(initialNotes.length) // highlight-line
})

test('a specific note is within the returned notes', async () => {
  const response = await api.get('/api/notes')

  const contents = response.body.map(r => r.content) // highlight-line

  expect(contents).toContain(
    'HTTP-protokollan tärkeimmät metodit ovat GET ja POST' // highlight-line
  )
})
```

<!-- Huomaa jälkimmäisen testin ekspektaatio. Komennolla <code>response.body.map(r => r.content)</code> muodostetaan taulukko API:n palauttamien muistiinpanojen sisällöistä. Jestin [toContain](https://facebook.github.io/jest/docs/en/expect.html#tocontainitem)-ekspektaatiometodilla tarkistetaan että parametrina oleva muistiinpano on kaikkien API:n palauttamien muistiinpanojen joukossa. -->
Pay special attention to the expect in the latter test. The <code>response.body.map(r => r.content)</code> command is used to create an array containing the content of every note returned by the API. The [toContain](https://facebook.github.io/jest/docs/en/expect.html#tocontainitem) method is used for checking that the note given to it as a parameter is in the list of notes returned by the API.

<!-- ### Testien suorittaminen yksitellen -->
### Running tests one by one

<!-- Komento _npm test_ suorittaa projektin kaikki testit. Kun olemme vasta tekemässä testejä, on useimmiten järkevämpää suorittaa kerrallaan ainoastaan yhtä tai muutamaa testiä. Jest tarjoaa tähän muutamia vaihtoehtoja. Eräs näistä on komennon [only](https://jestjs.io/docs/en/api#testonlyname-fn-timeout) käyttö. Jos testit on kirjoitettu useaan tiedotoon, ei menetelmä ole kovin hyvä. -->
The _npm test_ command executes all of the tests of the application. When we are writing tests it is usually wise to only execute one or two tests. Jest offers a few different ways of accomplishing this, one of which is the [only](https://jestjs.io/docs/en/api#testonlyname-fn-timeout) method. If tests are written across many files this method is not great.

<!-- Parempi vaihtoehto on käyttää jestiä suoraan, ilman npm:ää. Tällöin on mahdollista määritellä tarkasti mitä testejä jest suorittaa. Seuraava komento suorittaa ainoastaan tiedostossa <i>tests/note_api.test.js</i> olevat testit -->
A better option is to use Jest directly without npm. This way we can specify what the tests that we want to run with Jest. The following command only runs the tests found in the <i>tests/note_api.test.js</i> file:

```js
npx jest tests/note_api.test.js --runInBand
```

<!-- Parametrin <i>-t</i> avulla voidaan suorittaa testejä nimen perusteella: -->
The <i>-t</i> option can be used for running tests with a specific name:

```js
npx jest -t 'a specific note is within the returned notes'
```

<!-- Parametri voi viitata testin tai describe-lohkon nimeen. Parametrina voidaan antaa myös nimen osa. Seuraava komento suorittaisi kaikki testit, joiden nimessä on sana <i>notes</i>: -->
The provided parameter can refer to the name of the test or the describe block. The parameter can also contain just a part of the name. The following command will run all of the tests that contain <i>notes</i> in their name:

```js
npx jest -t 'notes' --runInBand
```

<!-- Jos asennat koneellesi jestin <i>globaalisti</i>, eli komennolla -->
If you install Jest <i>globally</i> with the command:

```js
npm install -g jest
```

<!-- testien suoritus onnistuu suoraan komennolla _jest_. Globaaliin asennukseen tarvitset pääkäyttäjän oikeudet. -->
Then you can run tests directly with the _jest_ command. To install packages globally you need to have admin rights.

### async/await

<!-- Ennen kuin teemme lisää testejä, tarkastellaan tarkemmin mitä _async_ ja _await_ tarkoittavat. -->
Before we write more tests let's take a look at the _async_ and _await_ keywords. 

<!-- Async- ja await ovat ES7:n mukanaan tuoma uusi syntaksi, joka mahdollistaa <i>promisen palauttavien asynkronisten funktioiden</i> kutsumisen siten, että kirjoitettava koodi näyttää synkroniselta. -->
The async/await syntax that was introduced in ES7 makes it possible to use <i>asynchronous functions that return a promise</i> in a way that makes the code look synchronous.

<!-- Esim. muistiinpanojen hakeminen tietokannasta hoidetaan promisejen avulla seuraavasti: -->
As an example, the fetching notes from the database with promises looks like this:

```js
Note.find({}).then(notes => {
  console.log('operaatio palautti seuraavat muistiinpanot', notes)
})
```

<!-- Metodikutsu _Note.find()_ palauttaa promisen, ja saamme itse operaation tuloksen rekisteröimällä promiselle tapahtumankäsittelijän metodilla _then_. -->
The _Note.find()_ method returns a promise and we can access the result of the operation by registering a callback function with the _then_ method.

<!-- Kaikki operaation suorituksen jälkeinen koodi kirjoitetaan tapahtumankäsittelijään. Jos haluaisimme tehdä peräkkäin useita asynkronisia funktiokutsuja, menisi tilanne ikävämmäksi. Joutuisimme tekemään kutsut tapahtumankäsittelijästä. Näin syntyisi potentiaalisesti monimutkaista koodia, pahimmassa tapauksessa jopa niin sanottu [callback-helvetti](http://callbackhell.com/). -->
All of the code we want to execute once the operation finishes is written in the callback function. If we wanted make several asynchronous function calls in sequence, the situation would start to become painful. The asynchronous calls would have to be made in the callback. This would likely lead to complicated code and could potentially give birth to a so-called [callback hell](http://callbackhell.com/).

<!-- [Ketjuttamalla promiseja](https://javascript.info/promise-chaining) tilanne pysyy jollain tavalla hallinnassa, callback-helvetin eli monien sisäkkäisten callbackien sijaan saadaan aikaan siistihkö _then_-kutsujen ketju. Olemmekin nähneet jo kurssin aikana muutaman sellaisen. Seuraavassa vielä erittäin keinotekoinen esimerkki, joka hakee ensin kaikki muistiinpanot ja sitten tuhoaa niistä ensimmäisen: -->
By [chaining promises](https://javascript.info/promise-chaining) we could keep the situation somewhat under control, and avoid callback hell by creating a fairly clean chain of _then_ method calls. We have seen a few of these during the course. To illustrate this, you can view an artificial example of a function that fetches all notes and then deletes the first one:

```js
Note.find({})
  .then(notes => {
    return notes[0].remove()
  })
  .then(response => {
    console.log('the first note is removed')
    // more code here
  })
```

<!-- Then-ketju on ok, mutta parempaankin pystytään. Jo ES6:ssa esitellyt [generaattorifunktiot](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator) mahdollistivat [ovelan tavan](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20%26%20performance/ch4.md#iterating-generators-asynchronously) määritellä asynkronista koodia siten että se "näyttää synkroniselta". Syntaksi ei kuitenkaan ole täysin luonteva ja sitä ei käytetä kovin yleisesti. -->
The then-chain is alright but we can do better. The [generator functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator) introduced in ES6 provided a [clever way](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20%26%20performance/ch4.md#iterating-generators-asynchronously) of writing asynchronous code in a way that "looks synchronous". The syntax is a bit clunky and not widely used.

<!-- ES7:ssa _async_ ja _await_ tuovat generaattoreiden tarjoaman toiminnallisuuden ymmärrettävästi ja syntaksin puolesta selkeällä tavalla koko Javascript-kansan ulottuville. -->
The _async_ and _await_ keywords introduced in ES7 bring the same functionality as the generators, but in an understandable and syntactically cleaner way to the hands of all citizens of the JavaScript world.

<!-- Voisimme hakea tietokannasta kaikki muistiinpanot [await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)-operaattoria hyödyntäen seuraavasti: -->
We could fetch all of the notes in the database by utilizing the [await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) operator like this:

```js
const notes = await Note.find({})

console.log('operaatio palautti seuraavat muistiinpanot ', notes)
```

<!-- Koodi siis näyttää täsmälleen synkroniselta koodilta. Suoritettavan koodinpätkän suhteen tilanne on se, että suoritus pysähtyy komentoon <em>const notes = await Note.find({})</em> ja jatkuu kyselyä vastaavan promisen <i>fulfillmentin</i> eli onnistuneen suorituksen jälkeen seuraavalta riviltä. Kun suoritus jatkuu, promisea vastaavan operaation tulos on muuttujassa _notes_. -->
The code looks exactly like synchronous code. The execution of code pauses at <em>const notes = await Note.find({})</em> and waits until the related promise is <i>fulfilled</i>, and then continues its execution to the next line. When the execution continues, the result of the operation that returned a promise is assigned to the _notes_ variable.

<!-- Ylempänä oleva monimutkaisempi esimerkki suoritettaisiin awaitin avulla seuraavasti: -->
The slightly complicated example presented above could be implemented by using await like this:

```js
const notes = await Note.find({})
const response = await notes[0].remove()

console.log('the first note is removed')
```

<!-- Koodi siis yksinkertaistuu huomattavasti verrattuna promiseja käyttävään then-ketjuun. -->
Thanks to the new syntax the code is a lot simpler than the previous then-chain.

<!-- Awaitin käyttöön liittyy parikin tärkeää seikkaa. Jotta asynkronisia operaatioita voi kutsua awaitin avulla, niiden täytyy palauttaa promiseja. Tämä ei sinänsä ole ongelma, sillä myös "normaaleja" callbackeja käyttävä asynkroninen koodi on helppo kääriä promiseksi. -->
There are a few important details to pay attention to when using async/await syntax. In order to use the await operator with asynchronous operations they have to return a promise. This is not a problem as such, as regular asynchronous functions using callbacks are easy to wrap around promises.

<!-- Mistä tahansa kohtaa Javascript-koodia ei awaitia kuitenkaan pysty käyttämään. Awaitin käyttö onnistuu ainoastaan jos ollaan [async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)-funktiossa. -->
The await keyword can't be used just anywhere in JavaScript code. Using await is possible only inside of an [async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) function.

<!-- Eli jotta edelliset esimerkit toimisivat, on ne suoritettava async-funktioiden sisällä, huomaa funktion määrittelevä rivi: -->
This means that in order for the previous examples to work they have to be using async functions. Notice the first line in the arrow function definition:

```js
const main = async () => { // highlight-line
  const notes = await Note.find({})
  console.log('operaatio palautti seuraavat muistiinpanot', notes)

  const notes = await Note.find({})
  const response = await notes[0].remove()

  console.log('the first note is removed')
}

main() // highlight-line
```

<!-- Koodi määrittelee ensin asynkronisen funktion, joka sijoitetaan muuttujaan _main_. Määrittelyn jälkeen koodi kutsuu metodia komennolla <code>main()</code> -->
The code declares that the function assigned to _main_ is asynchronous. After this the code calls the function with <code>main()</code>.

<!-- ### async/await backendissä -->
### async/await in the backend

<!-- Muutetaan nyt backend käyttämään asyncia ja awaitia. Koska kaikki asynkroniset operaatiot tehdään joka tapauksessa funktioiden sisällä, awaitin käyttämiseen riittää, että muutamme routejen käsittelijät async-funktioiksi. -->
Let's change the backend to async and await. As all of the asynchronous operations are currently done inside of a function, it is enough to change the route handler functions into async functions.

<!-- Kaikkien muistiinpanojen hakemisesta vastaava route muuttuu seuraavasti: -->
The route for fetching all notes gets changed to the following:

```js
notesRouter.get('/', async (request, response) => { 
  const notes = await Note.find({})
  response.json(notes.map(note => note.toJSON()))
})
```

<!-- Voimme varmistaa refaktoroinnin onnistumisen selaimella, sekä suorittamalla juuri määrittelemämme testit. -->
We can verify that our refactoring was successful by testing the endpoint through the browser and by running the tests that we wrote earlier.

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-3), branchissa <i>part4-3</i>. -->
You can find the code for our current application in its entirety in the <i>part4-3</i> branch of [this github repository](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-3).

<!-- ### Lisää testejä ja backendin refaktorointia -->
### More tests and refactoring the backend

<!-- Koodia refaktoroidessa vaanii aina [regression](https://en.wikipedia.org/wiki/Regression_testing) vaara, eli on olemassa riski, että jo toimineet ominaisuudet hajoavat. Tehdäänkin muiden operaatioiden refaktorointi siten, että ennen koodin muutosta tehdään jokaiselle API:n routelle sen toiminnallisuuden varmistavat testit. -->
When code gets refactored there is always the risk of [regression](https://en.wikipedia.org/wiki/Regression_testing), meaning that existing functionality may break. Let's refactor the remaining operations by first writing a test for each route of the API.

<!-- Aloitetaan lisäysoperaatiosta. Tehdään testi, joka lisää uuden muistiinpanon ja tarkistaa, että API:n palauttamien muistiinpanojen määrä kasvaa, ja että lisätty muistiinpano on palautettujen joukossa: -->
Let's start with the operation for adding a new note. Let's write a test that adds a new note and verifies that the amount of notes returned by the API increases, and that the newly added note is in the list.

```js
test('a valid note can be added ', async () => {
  const newNote = {
    content: 'async/await yksinkertaistaa asynkronisten funktioiden kutsua',
    important: true,
  }

  await api
    .post('/api/notes')
    .send(newNote)
    .expect(200)
    .expect('Content-Type', /application\/json/)

  const response = await api.get('/api/notes')

  const contents = response.body.map(r => r.content)

  expect(response.body.length).toBe(initialNotes.length + 1)
  expect(contents).toContain(
    'async/await yksinkertaistaa asynkronisten funktioiden kutsua'
  )
})
```

<!-- Kuten odotimme ja toivoimme, menee testi läpi. -->
The test passes just like we hoped and expected it to.

<!-- Tehdään myös testi, joka varmistaa, että muistiinpanoa, jolle ei ole asetettu sisältöä, ei talleteta -->
Let's also write a test that verifies that a note without content will not be saved into the database.

```js
test('note without content is not added', async () => {
  const newNote = {
    important: true
  }

  await api
    .post('/api/notes')
    .send(newNote)
    .expect(400)

  const response = await api.get('/api/notes')

  expect(response.body.length).toBe(initialNotes.length)
})
```

<!-- Molemmat testit tarkastavat lisäyksen jälkeen mihin tilaan tietokanta on päätynyt hakemalla kaikki sovelluksen muistiinpanot -->
Both tests check the state stored in the database after the saving operation, by fetching all the notes of the application.  

```js
const response = await api.get('/api/notes')
```

<!-- Sama tulee toistumaan myöhemminkin monissa testeissä ja operaatio kannattaakin eristää apufunktioon. Sijoitetaan se testien yhteyteen tiedostoon <i>tests/test_helper.js</i>  -->
The same verification steps will repeat in other tests later on, and it is a good idea to extract these steps into helper functions. Let's add the function into a new file called <i>tests/test_helper.js</i> that is in the same directory as the test file.

```js
const Note = require('../models/note')

const initialNotes = [
  {
    content: 'HTML on helppoa',
    important: false
  },
  {
    content: 'HTTP-protokollan tärkeimmät metodit ovat GET ja POST',
    important: true
  }
]

const nonExistingId = async () => {
  const note = new Note({ content: 'willremovethissoon' })
  await note.save()
  await note.remove()

  return note._id.toString()
}

const notesInDb = async () => {
  const notes = await Note.find({})
  return notes.map(note => note.toJSON())
}

module.exports = {
  initialNotes, nonExistingId, notesInDb
}
```

<!-- Moduuli määrittelee funktion _notesInDb_, jonka avulla voidaan tarkastaa sovelluksen tietokannassa olevat muistiinpanot. Tietokantaan alustettava sisältö _initialNotes_ on siirretty samaan tiedostoon. Määrittelimme myös tulevan varalta funktion _nonExistingId_, jonka avulla on mahdollista luoda tietokantaid, joka ei kuulu millekään kannassa olevalle oliolle. -->
The module defines the _notesInDb_ function that can be used for checking the notes stored in the database. The _initialNotes_ array containing the initial database state is also in the module. We also define the _nonExistingId_ function ahead of time, that can be used for creating a database object ID that does not belong to any note object in the database.

<!-- Testit muuttuvat muotoon -->
Our tests can now use helper module and be changed like this:

```js
const supertest = require('supertest')
const mongoose = require('mongoose')
const helper = require('./test_helper') // highlight-line
const app = require('../app')
const api = supertest(app)

const Note = require('../models/note')

beforeEach(async () => {
  await Note.deleteMany({})

  let noteObject = new Note(helper.initialNotes[0]) // highlight-line
  await noteObject.save()

  noteObject = new Note(helper.initialNotes[1]) // highlight-line
  await noteObject.save()
})

test('notes are returned as json', async () => {
  await api
    .get('/api/notes')
    .expect(200)
    .expect('Content-Type', /application\/json/)
})

test('all notes are returned', async () => {
  const response = await api.get('/api/notes')

  expect(response.body.length).toBe(helper.initialNotes.length) // highlight-line
})

test('a specific note is within the returned notes', async () => {
  const response = await api.get('/api/notes')

  const contents = response.body.map(r => r.content)
  expect(contents).toContain(
    'HTTP-protokollan tärkeimmät metodit ovat GET ja POST'
  )
})

test('a valid note can be added ', async () => {
  const newNote = {
    content: 'async/await yksinkertaistaa asynkronisten funktioiden kutsua',
    important: true,
  }

  await api
    .post('/api/notes')
    .send(newNote)
    .expect(200)
    .expect('Content-Type', /application\/json/)


  const notesAtEnd = await helper.notesInDb() // highlight-line
  expect(notesAtEnd.length).toBe(helper.initialNotes.length + 1) // highlight-line

  const contents = notesAtEnd.map(n => n.content) // highlight-line
  expect(contents).toContain(
    'async/await yksinkertaistaa asynkronisten funktioiden kutsua'
  )
})

test('note without content is not added', async () => {
  const newNote = {
    important: true
  }

  await api
    .post('/api/notes')
    .send(newNote)
    .expect(400)

  const notesAtEnd = await helper.notesInDb() // highlight-line

  expect(notesAtEnd.length).toBe(helper.initialNotes.length) // highlight-line
})

afterAll(() => {
  mongoose.connection.close()
}) 
```

<!-- Promiseja käyttävä koodi toimii nyt ja testitkin menevät läpi. Olemme valmiit muuttamaan koodin käyttämään async/await-syntaksia. -->
The code using promises works and the tests pass. We are ready to refactor our code to use the async/await syntax.

<!-- Koodi muuttuu seuraavasti (huomaa, että käsittelijän alkuun on laitettava määre _async_): -->
We make the following changes to our code (notice that the route handler definition is preceded by the _async_ keyword):

```js
notesRouter.post('/', async (request, response, next) => {
  const body = request.body

  const note = new Note({
    content: body.content,
    important: body.important === undefined ? false : body.important,
    date: new Date(),
  })

  const savedNote = await note.save()
  response.json(savedNote.toJSON())
})
```

<!-- Koodiin jää kuitenkin pieni ongelma: virhetilanteita ei nyt käsitellä ollenkaan. Miten niiden suhteen tulisi toimia? -->
There's a slight problem with our code: we don't handle error situations. How should we deal with them?

<!-- ### virheiden käsittely ja async/await -->
### Error handling and async/await

<!-- Jos sovellus POST-pyyntöä käsitellessään aiheuttaa jonkinlaisen ajonaikaisen virheen, syntyy jälleen tuttu tilanne: -->
If there's an exception while handling the POST request we end up in a familiar situation:

![](../images/4/6.png)

<!-- eli käsittelemätön promisen rejektoituminen. Pyyntöön ei vastata tilanteessa mitenkään. -->
In other words we end up with an unhandled promise rejection, and the request never receives a response.

<!-- Async/awaitia käyttäessä kannattaa käyttää vanhaa kunnon _try/catch_-mekanismia virheiden käsittelyyn: -->
With async/await the recommended way of dealing with exceptions is the old and familiar _try/catch_ mechanism:

```js
notesRouter.post('/', async (request, response, next) => {
  const body = request.body

  const note = new Note({
    content: body.content,
    important: body.important === undefined ? false : body.important,
    date: new Date(),
  })
  // highlight-start
  try { 
    const savedNote = await note.save()
    response.json(savedNote.toJSON())
  } catch(exception) {
    next(exception)
  }
  // highlight-end
})
```

<!-- Catch-lohkossa siis ainoastaan kutsutaan funktiota _next_ siirretään poikkeuksen käsittely virheidenkäsittelymiddlewarelle. -->
The catch block simply calls the _next_ function, which passes the request handling to the error handling middleware.

<!-- Muutoksen jälkeen testit menevät läpi. -->
After making the change, all of our tests will pass once again.

<!-- Tehdään sitten testit yksittäisen muistiinpanon tietojen katsomiselle ja muistiinpanon poistolle: -->
Next, let's write tests for fetching and removing an individual note:

```js
test('a specific note can be viewed', async () => {
  const notesAtStart = await helper.notesInDb()

  const noteToView = notesAtStart[0]

// highlight-start
  const resultNote = await api
    .get(`/api/notes/${noteToView.id}`)
    .expect(200)
    .expect('Content-Type', /application\/json/)
// highlight-end

  expect(resultNote.body).toEqual(noteToView)
})

test('a note can be deleted', async () => {
  const notesAtStart = await helper.notesInDb()
  const noteToDelete = notesAtStart[0]

// highlight-start
  await api
    .delete(`/api/notes/${noteToDelete.id}`)
    .expect(204)
// highlight-end

  const notesAtEnd = await helper.notesInDb()

  expect(notesAtEnd.length).toBe(
    helper.initialNotes.length - 1
  )

  const contents = notesAtEnd.map(r => r.content)

  expect(contents).not.toContain(noteToDelete.content)
})
```

<!-- Molemmat testit ovat rakenteeltaan samankaltaisia. Alustusvaiheessa ne hakevat kannasta yksittäisen muistiinpanon. Tämän jälkeen on itse testattava operaatio, joka on koodissa korostettuna. Lopussa tarkastetaan, että operaation tulos on haluttu.  -->
Both tests share a similar structure. In the initialization phase they fetch a note from the database. After this the tests call the actual operation being tested, which is highlighted in the code block. Lastly, the tests verify that the outcome of the operation is as expected.

<!-- Testit menevät läpi, joten voimme turvallisesti refaktoroida testatut routet käyttämään async/awaitia: -->
The tests pass and we can safely refactor the tested routes to use async/await:

```js
notesRouter.get('/:id', async (request, response, next) => {
  try{
    const note = await Note.findById(request.params.id)
    if (note) {
      response.json(note.toJSON())
    } else {
      response.status(404).end()
    }
  } catch(exception) {
    next(exception)
  }
})

notesRouter.delete('/:id', async (request, response, next) => {
  try {
    await Note.findByIdAndRemove(request.params.id)
    response.status(204).end()
  } catch (exception) {
    next(exception)
  }
})
```

<!-- Async/await ehkä selkeyttää koodia jossain määrin, mutta saavutettava hyöty ei ole sovelluksessamme vielä niin iso mitä se tulee olemaan jos asynkronisia kutsuja on tehtävä useampia. Async/awaitin 'hinta' on poikkeusten käsittelyn edellyttämä iso <i>try/catch</i>-rakenne. Kaikki routejen käsittelijät noudattavatkin samaa kaavaa -->
The async/await syntax makes the code somewhat clearer, but the experienced benefit of the syntax is not as great as it would be in the case of multiple asynchronous function calls. The price of the async/await syntax is the syntactically heavy <i>try/catch</i> block. All of the route handlers indeed share the structure:

```js
try {
  // do the async operations here
} catch(exception) {
  next(exception)
}
```

<!-- Mieleen herää kysymys, olisiko koodia mahdollista refaktoroida siten, että <i>catch</i> saataisiin refaktoroitua ulos metodeista? Siihen on olemassa eräitä ratkaisuja mutta koska ne muuttavat koodia kompleksisemmaksi, jätämme sen ennalleen. -->
This begs the question, would it be possible to refactor the repeated <i>catch</i> block out from the route handler functions? There are some existing ways of accomplishing this, but we will skip them due to their complexity.

<!-- Kaikki eivät ole vakuuttuneita siitä, että async/await on hyvä lisä Javascriptiin, lue esim. [ES7 async functions - a step in the wrong direction](https://spion.github.io/posts/es7-async-await-step-in-the-wrong-direction.html) -->
Not everyone is convinced that the async/await syntax is a good addition to JavaScript. To provide an example, you can read [ES7 async functions - a step in the wrong direction](https://spion.github.io/posts/es7-async-await-step-in-the-wrong-direction.html).

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-4), haarassa <i>part4-4</i>. Samassa on "vahingossa" mukana testeistä seuraavan luvun jälkeinen paranneltu versio. -->
You can find the code for our current application in its entirety in the <i>part4-4</i> branch of [this github repository](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-4). The same branch also contains a slightly improved version of tests from the next part of the course material.

<!-- ### Testin beforeEach-metodin optimointi -->
### Optimizing the beforeEach function

<!-- Palataan takaisin testien pariin, ja tarkastellaan määrittelemäämme testit alustavaa funktiota _beforeEach_: -->
Let's return to writing our tests and take a closer look at the _beforeEach_ function that sets up the tests:

```js
beforeEach(async () => {
  await Note.deleteMany({})

  let noteObject = new Note(helper.initialNotes[0])
  await noteObject.save()

  noteObject = new Note(helper.initialNotes[1])
  await noteObject.save()
})
```

<!-- Funktio tallettaa tietokantaan taulukon _helper.initialNotes_ nollannen ja ensimmäisen alkion, kummankin erikseen taulukon alkioita indeksöiden. Ratkaisu on ok, mutta jos haluaisimme tallettaa alustuksen yhteydessä kantaan useampia alkioita, olisi toisto parempi ratkaisu: -->
The function saves the first two notes from the   _helper.initialNotes_ array into the database with two separate operations. The solution is alright, but there's a better way of saving multiple objects to the database:

```js
beforeEach(async () => {
  await Note.deleteMany({})
  console.log('cleared')

  helper.initialNotes.forEach(async (note) => {
    let noteObject = new Note(note)
    await noteObject.save()
    console.log('saved')
  })
  console.log('done')
})

test('notes are returned as json', async () => {
  console.log('entered test')
  // ...
}
```

<!-- Talletamme siis taulukossa olevat muistiinpanot tietokantaan _forEach_-loopissa. Testeissä kuitenkin ilmenee jotain häikkää, ja sitä varten koodin sisään on lisätty aputulosteita. -->
We save the notes stored in the array into the database inside of a _forEach_ loop. The tests don't quite seem to work however, so we have added some console logs to help us find the problem. 

<!-- Konsoliin tulostuu -->
The console displays the following output:

<pre>
cleared
done
entered test
saved
saved
</pre>

<!-- Yllättäen ratkaisu ei async/awaitista huolimatta toimi niin kuin oletamme, testin suoritus aloitetaan ennen kuin tietokannan tila on saatu alustettua! -->
Despite our use of the async/await syntax our solution does not work like we expected it to. The test execution begins before the database is initialized!

<!-- Ongelma on siinä, että jokainen forEach-loopin läpikäynti generoi oman asynkronisen operaation ja _beforeEach_ ei odota näiden suoritusta. Eli forEach:in sisällä olevat _await_-komennot eivät ole funktiossa _beforeEach_ vaan erillisissä funktioissa, joiden päättymistä _beforeEach_ ei odota. -->
The problem is that every iteration of the forEach loop generates its own asynchronous operation and _beforeEach_ won't wait for them to finish executing. In other words, the _await_ commands defined inside of the _forEach_ loop are not in the _beforeEach_ function, but in separate functions that _beforeEach_ will not wait for.

<!-- Koska testien suoritus alkaa heti _beforeEach_ metodin suorituksen jälkeen, testien suoritus ehditään aloittamaan ennen kuin tietokanta on alustettu toivottuun alkutilaan. -->
Since the execution of tests begins immediately after _beforeEach_ has finished executing, the execution of tests begins before the database state is initialized.

<!-- Toimiva ratkaisu tilanteessa on odottaa asynkronisten talletusoperaatioiden valmistumista _beforeEach_-funktiossa, esim. metodin [Promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) avulla: -->
One way of fixing this is to wait for all of the asynchronous operations to finish executing with the [Promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) method:

```js
beforeEach(async () => {
  await Note.remove({})

  const noteObjects = helper.initialNotes
    .map(note => new Note(note))
  const promiseArray = noteObjects.map(note => note.save())
  await Promise.all(promiseArray)
})
```

<!-- Ratkaisu on varmasti aloittelijalle tiiviydestään huolimatta hieman haastava. Taulukkoon _noteObjects_ talletetaan taulukkossa _helper.initialNotes_ olevia Javascript-oliota vastaavat _Note_-konstruktorifunktiolla generoidut Mongoose-oliot. Seuraavalla rivillä luodaan uusi taulukko, joka <i>muodostuu promiseista</i>, jotka saadaan kun jokaiselle _noteObjects_ taulukon alkiolle kutsutaan metodia _save_, eli ne talletetaan kantaan. -->
The solution is quite advanced despite its compact appearance. The _noteObject_ variable is assigned to an array of Mongoose objects that are created with the _Note_ constructor for each of the notes in the _helper.initialNotes_ array. The next line of code creates a new array that <i>consists of promises</i>, that are created by calling the _save_ method of each item in the _noteObjects_ array. In other words, it is an array of promises for saving each of the items to the database.

<!-- Metodin [Promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) avulla saadaan koostettua taulukollinen promiseja yhdeksi promiseksi, joka valmistuu, eli menee tilaan <i>fulfilled</i> kun kaikki sen parametrina olevan taulukon promiset ovat valmistuneet. Siispä viimeinen rivi, <em>await Promise.all(promiseArray)</em> odottaa, että kaikki tietokantaan talletetusta vastaavat promiset ovat valmiina, eli alkiot on talletettu tietokantaan. -->
The [Promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) method can be used for transforming an array of promises into a single promise, that will be <i>fulfilled</i> once every promise in the array passed to it is as a parameter is resolved. The last line of code <em>await Promise.all(promiseArray)</em> waits that every promise for saving a note is finished, meaning that the database has been initialized.

<!-- > Promise.all-metodia käyttäessä päästään tarvittaessa käsiksi sen parametrina olevien yksittäisten promisejen arvoihin, eli promiseja vastaavien operaatioiden tuloksiin. Jos odotetaan promisejen valmistumista _await_-syntaksilla <em>const results = await Promise.all(promiseArray)</em> palauttaa operaatio taulukon, jonka alkioina on _promiseArray_:n promiseja vastaavat arvot samassa järjestyksessä kuin promiset ovat taulukossa. -->
> The returned values of each promise in the array can still be accessed when using the Promise.all method. If we wait for the promises to be resolved with the _await_ syntax <em>const results = await Promise.all(promiseArray)</em>, the operation will return an array that contains the resolved values for each promise in the _promiseArray_, and they appear in the same order as the promises in the array.

<!-- Promise.all suorittaa kaikkia syötteenä saamiaan promiseja rinnakkain. Jos operaatioiden suoritusjärjestyksellä on merkitystä, voi tämä aiheuttaa ongelmia. Tällöin asynkroniset operaatiot on mahdollista määrittää [for...of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of) lohkon sisällä, jonka suoritusjärjestys on taattu. -->
Promise.all executes the promises it receives in parallel. If the promises need to be executed in a particular order, this will be problematic. In situations like this, the operations can be executed inside of a [for...of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of) block, that guarantees a specific execution order.

```js
beforeEach(async () => {
  await Note.remove({})

  for (let note of initialNotes) {
    let noteObject = new Note(note)
    await noteObject.save()
  }
})
```

<!-- Javascriptin asynkroninen suoritusmalli aiheuttaakin siis helposti yllätyksiä ja myös async/await-syntaksin kanssa pitää olla koko ajan tarkkana. Vaikka async/await peittää monia promisejen käsittelyyn liittyviä seikkoja, promisejen toiminta on syytä tuntea mahdollisimman hyvin! -->
The asynchronous nature of JavaScript can lead to surprising behavior and for this reason it is important to pay careful attention when using the async/await syntax. Even though the syntax makes it easier to deal with promises, it is still necessary to understand how promises work!

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

**Huom:** materiaalissa käytetään muutamaan kertaan matcheria [toContain](https://facebook.github.io/jest/docs/en/expect.html#tocontainitem) tarkastettaessa, että jokin arvo on taulukossa. Kannattaa huomata, että metodi käyttää samuuden vertailuun ===-operaattoria ja olioiden kohdalla tämä ei ole useinkaan se mitä halutaan ja parempi vaihtoehto onkin [toContainEqual](https://facebook.github.io/jest/docs/en/expect.html#tocontainequalitem). Tosin mallivastauksissa ei vertailla kertaakaan olioita matcherien avulla, joten ilmankin selviää varsin hyvin.

**NB:** the material uses the [toContain](https://facebook.github.io/jest/docs/en/expect.html#tocontainitem) matcher in several places to verify that an array contains a specific element. It's worth noting that the method uses the === operator for comparing and matching elements, which means that it is often not well-suited for matching objects. In most cases the appropriate method for verifying objects in arrays is the [toContainEqual](https://facebook.github.io/jest/docs/en/expect.html#tocontainequalitem) matcher. The model solutions don't check for objects in arrays with matchers, so using the method is not required for solving the exercises.

<!-- **Varoitus:** Jos huomaat kirjoittavasi sekaisin async/awaitia ja <i>then</i>-kutsuja, on 99% varmaa, että teet jotain väärin. Käytä siis jompaa kumpaa tapaa, älä missään tapauksessa "varalta" molempia. -->
**Warning:** If you find yourself using async/await and <i>then</i> methods in the same code, it is almost guaranteed that you are doing something wrong. Use one or the other and don't mix the two.

<!-- #### 4.8: blogilistan testit, step 1 -->
#### 4.8: Blog list tests, step1

<!-- Tee supertest-kirjastolla testit blogilistan osoitteeseen <i>/api/blogs</i> tapahtuvalle HTTP GET -pyynnölle. Testaa, että sovellus palauttaa oikean määrän JSON-muotoisia blogeja.  -->
Use the supertest package for writing a test that makes an HTTP GET request to the <i>/api/blogs</i> url. Verify that the blog list application returns the correct amount of blog posts in the JSON format.

<!-- Kun testi on valmis, refaktoroi operaatio käyttämään promisejen sijaan async/awaitia. -->
Once the test is finished, refactor the route handler to use the async/await syntax instead of promises.

<!-- Huomaa, että joudut tekemään koodiin [materiaalin tapaan](/osa4/backendin_testaaminen#test-ymparisto) hieman muutoksia (mm. testausympäristön määrittely), jotta saat järkevästi tehtyä omaa tietokantaa käyttäviä API-tason testejä. -->
Notice that you will have to make similar changes to the code that were made [in the material](/osa4/backendin_testaaminen#test-ymparisto), like defining the test environment so that you can write tests that use their own separate database.

<!-- **Huom1:** Testejä suorittaessa saatat törmätä seuraavaan varoitukseen -->
**NB:** When running the tests you may run into the following warning:

![](../images/4/8a.png)

<!-- Jos näin käy, toimi [ohjeen](https://mongoosejs.com/docs/jest.html) mukaan ja lisää projektin hakemiston juureen tiedosto <i>jest.config.js</i> jolla on seuraava sisältö: -->
If this happens, follow the [instructions](https://mongoosejs.com/docs/jest.html) and create a new <i>jest.config.js</i> file at the root of the project with the following contents:

```js
module.exports = {
  testEnvironment: 'node'
}
```

<!-- **Huom2:** testien kehitysvaiheessa yleensä **<i>ei kannata suorittaa joka kerta kaikkia testejä</i>**, vaan keskittyä yhteen testiin kerrallaan. Katso lisää [täältä](/osa4/backendin_testaaminen#testien-suorittaminen-yksitellen). -->

**NB:** when yo are writing your tests **<i>it is better to not execute all of your tests</i>**, only execute the ones you are working on. Read more about this [here](/osa4/backendin_testaaminen#testien-suorittaminen-yksitellen).

<!-- #### 4.9*: blogilistan testit, step2 -->
#### 4.9*: Blog list tests, step2

<!-- Tee testi, joka varmistaa että palautettujen blogeien identifioivan kentän tulee olla nimeltään <i>id</i>,  oletusarvoisestihan tietokantaan talletettujen olioiden tunnistekenttä on <i>_id</i>. Olion kentän olemassaolon tarkastaminen onnistuu jestin matcherillä [toBeDefined](https://jestjs.io/docs/en/expect#tobedefined) -->
Write a test that verifies that the unique identifier property of the blog posts is named <i>id</i>, by default the database names the property <i>_id</i>. Verifying the existence of a property is easily done with Jest's [toBeDefined](https://jestjs.io/docs/en/expect#tobedefined) matcher:

<!-- Muuta koodia siten, että testi menee läpi. Osassa 3 käsitelty [toJSON](osa3/tietojen_tallettaminen_mongo_db_tietokantaan#tietokantaa-kayttava-backend) on sopiva paikka parametrin <i>id</i> määrittelyyn.  -->
Make the required changes to the code so that it passes the test. The [toJSON](osa3/tietojen_tallettaminen_mongo_db_tietokantaan#tietokantaa-kayttava-backend) method discussed in part 3 is an appropriate place for defining the <i>id</i> parameter.

<!-- #### 4.10: blogilistan testit, step3 -->
#### 4.10: Blog list tests, step3

<!-- Tee testi joka varmistaa että sovellukseen voi lisätä blogeja osoitteeseen <i>/api/blogs</i> tapahtuvalla HTTP POST -pyynnölle. Testaa ainakin, että blogien määrä kasvaa yhdellä. Voit myös varmistaa, että oikeansisältöinen blogi on lisätty järjestelmään. -->
Write a test that verifies that making an HTTP POST request to the <i>/api/blogs</i> url successfully creates a new blog post. At the very least, verify that the total number of blogs in the system is increased by one. You can also verify that the content of the blog post is saved correctly to the database.

<!-- Kun testi on valmis, refaktoroi operaatio käyttämään promisejen sijaan async/awaitia. -->
Once the test is finished, refactor the operation to use async/await instead of promises.

<!-- #### 4.11*: blogilistan testit, step4 -->
#### 4.11*: Blog list tests, step4

<!-- Tee testi joka varmistaa, että jos kentälle <i>likes</i> ei anneta arvoa, asetetaan sen arvoksi 0. Muiden kenttien sisällöstä ei tässä tehtävässä vielä välitetä. -->
Write a test that verifies that if the <i>likes</i> property is missing from the request, it will default to the value 0. Do not test the other properties of the created notes yet.

<!-- Laajenna ohjelmaa siten, että testi menee läpi. -->
Make the required changes to the code so that it passes the test.

<!-- #### 4.12*: blogilistan testit, step5 -->
#### 4.12*: Blog list tests, step5

<!-- Tee testit blogin lisäämiselle, eli osoitteeseen <i>/api/blogs</i> tapahtuvalle HTTP POST -pyynnölle, joka varmistaa, että jos uusi blogi ei sisällä kenttiä <i>title</i> ja <i>url</i>, pyyntöön vastataan statuskoodilla <i>400 Bad request</i> -->
Write a test related to creating new notes via the <i>/api/blogs</i> endpoint, that verifies that if the <i>title</i> and <i>url</i> properties are missing from the request data, the backend responds to the request with the status code <i>400 Bad Request</i>.

<!-- Laajenna toteutusta siten, että testit menevät läpi. -->
Make the required changes to the code so that it passes the test.

</div>

<div class="content">

<!-- ### Testien refaktorintia -->
### Refactoring tests

<!-- Testit ovat tällä hetkellä osittain epätäydelliset, esim. reittejä <i>GET /api/notes/:id</i> ja <i>DELETE /api/notes/:id</i> ei tällä hetkellä testata epävalidien id:iden osalta. Myös testien organisoinnissa on hieman toivomisen varaa, sillä kaikki on kirjoitettu suoraan testifunktion "päätasolle", parempaan luettavuuteen pääsisimme eritellessä loogisesti toisiinsa liittyvät testit <i>describe</i>-lohkoihin. -->
Our test coverage is currently lacking. Some requests like <i>GET /api/notes/:id</i> and <i>DELETE /api/notes/:id</i> aren't tested when the request is sent with an invalid id. The grouping and organization of tests could also use some improvement, as all tests exist on the same "top level" in the test file. The readability of the test would improve, if we grouped related tests with <i>describe</i> blocks.

<!-- Jossain määrin parannellut testit seuraavassa: -->
Below is an example of the test file after making some minor improvements:

```js
const supertest = require('supertest')
const mongoose = require('mongoose')
const helper = require('./test_helper')
const app = require('../app')
const api = supertest(app)

const Note = require('../models/note')

describe('when there is initially some notes saved', async () => {
  beforeEach(async () => {
    await Note.deleteMany({})

    const noteObjects = helper.initialNotes
      .map(note => new Note(note))
    const promiseArray = noteObjects.map(note => note.save())
    await Promise.all(promiseArray)
  })

  test('notes are returned as json', async () => {
    await api
      .get('/api/notes')
      .expect(200)
      .expect('Content-Type', /application\/json/)
  })

  test('all notes are returned', async () => {
    const response = await api.get('/api/notes')

    expect(response.body.length).toBe(helper.initialNotes.length)
  })

  test('a specific note is within the returned notes', async () => {
    const response = await api.get('/api/notes')

    const contents = response.body.map(r => r.content)
    expect(contents).toContain(
      'HTTP-protokollan tärkeimmät metodit ovat GET ja POST'
    )
  })

  describe('viewing a specifin note', async () => {

    test('succeeds with a valid id', async () => {
      const notesAtStart = await helper.notesInDb()

      const noteToView = notesAtStart[0]

      const resultNote = await api
        .get(`/api/notes/${noteToView.id}`)
        .expect(200)
        .expect('Content-Type', /application\/json/)

      expect(resultNote.body).toEqual(noteToView)
    })

    test('fails with statuscode 404 if note does not exist', async () => {
      const validNonexistingId = await helper.nonExistingId()

      console.log(validNonexistingId)

      await api
        .get(`/api/notes/${validNonexistingId}`)
        .expect(404)
    })

    test('fails with statuscode 400 id is invalid invalid', async () => {
      const invalidId = '5a3d5da59070081a82a3445'

      await api
        .get(`/api/notes/${invalidId}`)
        .expect(400)
    })
  })

  describe('addition of a new note', async () => {
    test('succeeds with valid data', async () => {
      const newNote = {
        content: 'async/await yksinkertaistaa asynkronisten funktioiden kutsua',
        important: true,
      }

      await api
        .post('/api/notes')
        .send(newNote)
        .expect(200)
        .expect('Content-Type', /application\/json/)


      const notesAtEnd = await helper.notesInDb()
      expect(notesAtEnd.length).toBe(helper.initialNotes.length + 1)

      const contents = notesAtEnd.map(n => n.content)
      expect(contents).toContain(
        'async/await yksinkertaistaa asynkronisten funktioiden kutsua'
      )
    })

    test('fails with status code 400 if data invaild', async () => {
      const newNote = {
        important: true
      }

      await api
        .post('/api/notes')
        .send(newNote)
        .expect(400)

      const notesAtEnd = await helper.notesInDb()

      expect(notesAtEnd.length).toBe(helper.initialNotes.length)
    })
  })

  describe('deletion of a note', async () => {
    test('succeeds with status code 200 if id is valid', async () => {
      const notesAtStart = await helper.notesInDb()
      const noteToDelete = notesAtStart[0]

      await api
        .delete(`/api/notes/${noteToDelete.id}`)
        .expect(204)

      const notesAtEnd = await helper.notesInDb()

      expect(notesAtEnd.length).toBe(
        helper.initialNotes.length - 1
      )

      const contents = notesAtEnd.map(r => r.content)

      expect(contents).not.toContain(noteToDelete.content)
    })
  })
})

afterAll(() => {
  mongoose.connection.close()
})
```

<!-- Testien raportointi tapahtuu <i>describe</i>-lohkojen ryhmittelyn mukaan: -->
The test output is grouped according to the <i>describe</i> blocks:

![](../images/4/7.png)

<!-- Testeihin jää vielä parannettavaa mutta on jo aika siirtyä eteenpäin. -->
There is still room for improvement but it is time to move forward.

<!-- Käytetty tapa API:n testaamiseen, eli HTTP-pyyntöinä tehtävät operaatiot ja tietokannan tilan tarkastelu Mongoosen kautta ei ole suinkaan ainoa tai välttämättä edes paras tapa tehdä API-tason integraatiotestausta. Universaalisti parasta tapaa testien tekoon ei ole, vaan kaikki on aina suhteessa käytettäviin resursseihin ja testattavaan ohjelmistoon. -->
This way of testing the API, by making HTTP requests and inspecting the database with Mongoose, is by no means the only nor the best way of conducting API-level integration tests for server applications. There is no universal best way of writing tests, as it all depends on the application being tested and available resources.

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-5), branchissa <i>part4-5</i> -->
You can find the code for our current application in its entirety in the <i>part4-5</i> branch of [this github repository](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-5).

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- #### 4.13 blogilistan laajennus, step1 -->
#### 4.13 Blog list expansions, step1

<!-- Toteuta sovellukseen mahdollisuus yksittäisen blogin poistoon. -->
Implement functionality for deleting a single blog post resource.

<!-- Käytä async/awaitia. Noudata operaation HTTP-rajapinnan suhteen [RESTful](/osa3/node_js_ja_express#rest)-käytänteitä. -->
Use the async/await syntax. Follow [RESTful](/osa3/node_js_ja_express#rest) conventions when defining the HTTP API.

<!-- Saat toteuttaa ominaisuudelle testit jos haluat. Jos et, varmista ominaisuuden toimivuus esim. Postmanilla. -->
Feel free to implement tests for the functionality if you want to. Otherwise verify that the functionality works with Postman or some other tool.

<!-- #### 4.14* blogilistan laajennus, step2 -->
#### 4.13 Blog list expansions, step2

<!-- Toteuta sovellukseen mahdollisuus yksittäisen blogin muokkaamiseen. -->
Implement functionality for updating the information of an individual blog post.

<!-- Käytä async/awaitia. -->
Use async/await.

<!-- Tarvitsemme muokkausta lähinnä <i>likejen</i> lukumäärän päivittämiseen. Toiminnallisuuden voi toteuttaa samaan tapaan kuin muistiinpanon päivittäminen toteutettiin [osassa 3](/osa3/tietojen_tallettaminen_mongo_db_tietokantaan#muut-operaatiot). -->
The application mostly needs to update the amount of <i>likes</i> for a blog post. You can implement this functionality the same way that we implemented updating notes in [part 3](/osa3/tietojen_tallettaminen_mongo_db_tietokantaan#muut-operaatiot).

<!-- Saat toteuttaa ominaisuudelle testit jos haluat. Jos et, varmista ominaisuuden toimivuus esim. Postmanilla. -->
Feel free to implement tests for the functionality if you want to. Otherwise verify that the functionality works with Postman or some other tool.

</div>
