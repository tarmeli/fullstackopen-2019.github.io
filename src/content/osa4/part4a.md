---
mainImage: ../../images/part-4.svg
part: 4
letter: a
---

<div class="content">

<!-- Jatketaan [osassa 3](/osa3) tehdyn muistiinpanosovelluksen backendin kehittämistä. -->
Let's continue our work on the backend of the notes application we started in [part 3](/osa3). 

<!-- ### Sovelluksen rakenne -->
### Project structure

<!-- Ennen osan ensimmäistä isoa teemaa eli testaamista, muutetaan sovelluksen rakennetta noudattamaan paremmin Noden [parhaita käytänteitä](https://github.com/i0natan/nodebestpractices/tree/master/sections/projectstructre). -->
Before we move into the topic of testing, we will modify the structure of our project to adhere to Node.js [best practices](https://github.com/i0natan/nodebestpractices/tree/master/sections/projectstructre).

<!-- Seuraavassa läpikäytävien muutosten jälkeen sovelluksemme hakemistorakenne näyttää seuraavalta -->
After making the changes to the directory structure of our project, we end up with the following structure:

```bash
├── index.js
├── app.js
├── build
│   ├── ...
├── controllers
│   └── notes.js
├── models
│   └── note.js
├── package-lock.json
├── package.json
├── utils
│   ├── config.js
│   └── middleware.js  
```

<!-- Sovelluksen käynnistystiedosto <i>index.js</i> pelkistyy seuraavaan muotoon: -->
The contents of the <i>index.js</i> file used for starting the application gets simplified as follows:

```js
const app = require('./app') // varsinainen Express-sovellus
const http = require('http')
const config = require('./utils/config')

const server = http.createServer(app)

server.listen(config.PORT, () => {
  console.log(`Server running on port ${config.PORT}`)
})
```

<!-- <i>index.js</i> ainoastaan importaa tiedostossa <i>app.js</i> olevan varsinaisen sovelluksen ja käynnistää sen. Sovelluksen käynnistäminen tapahtuu nyt <em>server</em>-muuttujassa olevan olion kautta.  -->
The <i>index.js</i> file only imports the actual application from the <i>app.js</i> file and then starts the application.

<!-- Ympäristömuuttujien käsittely on eriytetty moduulin <i>utils/config.js</i> vastuulle: -->
The handling of environment variables is extracted into a separate <i>utils/config.js</i> file:

```js
if (process.env.NODE_ENV !== 'production') {
  require('dotenv').config()
}

let PORT = process.env.PORT
let MONGODB_URI = process.env.MONGODB_URI

module.exports = {
  MONGODB_URI,
  PORT
}
```

<!-- Sovelluksen muut osat pääsevät ympäristömuuttujiin käsiksi importtaamalla konfiguraatiomoduulin -->
The other parts of the application can access the environment variables by importing the configuration module:

```js
const config = require('./utils/config')

console.log(`Server running on port ${config.PORT}`)
```

<!-- Routejen määrittely siirretään omaan tiedostoonsa, eli myös siitä tehdään moduuli. Routejen tapahtumankäsittelijöitä kutsutaan usein <i>kontrollereiksi</i>. Sovellukselle onkin luotu hakemisto <i>controllers</i> ja sinne tiedosto <i>notes.js</i>, johon kaikki muistiinpanoihin liittyvien reittien määrittelyt on siirretty. -->
The route handlers have also been moved into a dedicated module. The event handlers of routes are commonly referred to as <i>controllers</i>, and for this reason we have created a new <i>controllers</i> directory. All of the routes related to notes are now in the <i>notes.js</i> module under the <i>controllers</i> directory.

<!-- Tiedoston sisältö on seuraava: -->
The contents of the <i>notes.js</i> module are the following:

```js
const notesRouter = require('express').Router()
const Note = require('../models/note')

notesRouter.get('/', (request, response) => {
  Note.find({}).then(notes => {
    response.json(notes.map(note => note.toJSON()))
  })
})

notesRouter.get('/:id', (request, response, next) => {
  Note.findById(request.params.id)
    .then(note => {
      if (note) {
        response.json(note.toJSON())
      } else {
        response.status(404).end()
      }
    })
    .catch(error => next(error))
})

notesRouter.post('/', (request, response, next) => {
  const body = request.body

  const note = new Note({
    content: body.content,
    important: body.important || false,
    date: new Date(),
  })

  note.save()
    .then(savedNote => {
      response.json(savedNote.toJSON())
    })
    .catch(error => next(error))
})

notesRouter.delete('/:id', (request, response, next) => {
  Note.findByIdAndRemove(request.params.id)
    .then(() => {
      response.status(204).end()
    })
    .catch(error => next(error))
})

notesRouter.put('/:id', (request, response, next) => {
  const body = request.body

  const note = {
    content: body.content,
    important: body.important,
  }

  Note.findByIdAndUpdate(request.params.id, note, { new: true })
    .then(updatedNote => {
      response.json(updatedNote.toJSON())
    })
    .catch(error => next(error))
})

module.exports = notesRouter
```

<!-- Kyseessä on käytännössä melkein suora copypaste tiedostosta <i>index.js</i>. -->
The is almost an exact copy-paste of our previous <i>index.js</i> file.

<!-- Muutoksia on muutama. Tiedoston alussa luodaan [router](http://expressjs.com/en/api.html#router)-olio: -->
However, there are a few significant changes. At the very beginning of the file we create a new [router](http://expressjs.com/en/api.html#router) object:

```js
const notesRouter = require('express').Router()

//...

module.exports = notesRouter
```

<!-- Tiedosto eksporttaa moduulin käyttäjille määritellyn routerin. -->
The module exports the router to be available for all consumers of the module.

<!-- Kaikki määriteltävät routet liitetään router-olioon, samaan tapaan kuin aiemmassa versiossa routet liitettiin sovellusta edustavaan olioon. -->
All routes are now defined for the router object, in a similar fashion to what we had previously done with the object representing the entire application.

<!-- Huomioinarvoinen seikka routejen määrittelyssä on se, että polut ovat typistyneet, aiemmin määrittelimme esim. -->
It's worth noting that the paths in the route handlers have shortened. In the previous version, we had:

```js
app.delete('/api/notes/:id', (request, response) => {
```

<!-- nyt riittää määritellä -->
And in the current version, we have:

```js
notesRouter.delete('/:id', (request, response) => {
```

<!-- Mistä routereissa oikeastaan on kyse? Expressin manuaalin sanoin -->
So what are these router objects exactly? The Express manual provides the following explanation:

> <i>A router object is an isolated instance of middleware and routes. You can think of it as a “mini-application,” capable only of performing middleware and routing functions. Every Express application has a built-in app router.</i>

<!-- Router on siis <i>middleware</i>, jonka avulla on mahdollista määritellä joukko "toisiinsa liittyviä" routeja yhdessä paikassa, yleensä omassa moduulissaan. -->
The router is in fact a <i>middleware</i>, that can be used for defining "related routes" in a single place, that is typically placed in its own module.

<!-- Varsinaisen sovelluslogiikan määrittelevä tiedosto <i>app.js</i> ottaa määrittelemämme routerin käyttöön seuraavasti: -->
The <i>app.js</i> file that creates the actual application, takes the router into use as shown below:

```js
const notesRouter = require('./controllers/notes')
app.use('/api/notes', notesRouter)
```

<!-- Näin määrittelemäämme routeria käytetään <i>jos</i> polun alkuosa on <i>/api/notes</i>. notesRouter-olion sisällä täytyy tämän takia käyttää ainoastaan polun loppuosia, eli tyhjää polkua <i>/</i> tai pelkkää parametria <i>/:id</i>. -->
The router we defined earlier is used <i>if</i> the URL of the request starts with <i>/api/notes</i>. For this reason, the notesRouter object must only define the relative parts of the routes, i.e. the empty path <i>/</i> or just the parameter <i>/:id</i>.

<!-- Sovelluksen määrittelevä <i>app.js</i> näyttää muutosten jälkeen seuraavalta: -->
After making these changes, our <i>app.js</i> file looks like this:

```js
const config = require('./utils/config')
const express = require('express')
const bodyParser = require('body-parser')
const app = express()
const notesRouter = require('./controllers/notes')
const middleware = require('./utils/middleware')
const mongoose = require('mongoose')

console.log('commecting to', config.MONGODB_URI)

mongoose.connect(config.MONGODB_URI, { useNewUrlParser: true })
  .then(() => {
    console.log('connected to MongoDB')
  })
  .catch((error) => {
    console.log('error connection to MongoDB:', error.message)
  })

app.use(express.static('build'))
app.use(bodyParser.json())
app.use(middleware.requestLogger)

app.use('/api/notes', notesRouter)

app.use(middleware.unknownEndpoint)
app.use(middleware.errorHandler)

module.exports = app
```

<!-- Tiedostossa siis otetaan käyttöön joukko middlewareja, näistä yksi on polkuun <i>/api/notes</i> kiinnitettävä <i>notesRouter</i> (tai notes-kontrolleri niin kuin jotkut sitä kutsuisivat). -->
The file takes different middleware into use, and one of these is the <i>notesRouter</i> that is attached to the <i>/api/notes</i> route.

<!-- Itse toteutettujen middlewarejen määritelty on siirretty tiedostoon <i>utils/middleware.js</i>: -->
Our custom middleware has been moved to a new <i>utils/middleware.js</i> module:

```js
const requestLogger = (request, response, next) => {
  console.log('Method:', request.method)
  console.log('Path:  ', request.path)
  console.log('Body:  ', request.body)
  console.log('---')
  next()
}

const unknownEndpoint = (request, response) => {
  response.status(404).send({ error: 'unknown endpoint' })
}

const errorHandler = (error, request, response, next) => {
  console.error(error.message)

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

<!-- Koska tietokantayhteyden muodostaminen on siirretty tiedoston <i>app.js</i>:n vastuulle. Hakemistossa <i>models</i> oleva tiedosto <i>note.js</i> sisältää nyt ainoastaan muistiinpanojen skeeman määrittelyn. -->
The responsibility of establishing the connection to the database has been given to the  <i>app.js</i> module. The <i>note.js</i> file under the <i>models</i> directory only defines the Mongoose schema for notes.

```js
const mongoose = require('mongoose')

const noteSchema = new mongoose.Schema({
  content: {
    type: String,
    required: true,
    minlength: 5
  },
  date: Date,
  important: Boolean,
})

noteSchema.set('toJSON', {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id
    delete returnedObject._id
    delete returnedObject.__v
  }
})

module.exports = mongoose.model('Note', noteSchema)
```

<!-- Sovelluksen hakemistorakenne siis näyttää refaktoroinnin jälkeen seuraavalta: -->
To recap, the directory structure looks like this after the changes have been made:

```bash
├── index.js
├── app.js
├── build
│   ├── ...
├── controllers
│   └── notes.js
├── models
│   └── note.js
├── package-lock.json
├── package.json
├── utils
│   ├── config.js
│   └── middleware.js  
```

<!-- Jos sovellus on pieni, ei rakenteella ole kovin suurta merkitystä. Sovelluksen kasvaessa kannattaa sille muodostaa jonkinlainen rakenne eli arkkitehtuuri, ja jakaa erilaisten vastuut omiin moduuleihin. Tämä helpottaa huomattavasti ohjelman jatkokehitystä. -->
For smaller applications the structure does not matter that much. Once the application starts to grow in size, you are going to have establish some kind of a structure, and separate the different responsibilities of the application into separate modules. This will make developing the application much easier.

<!-- Express-sovelluksien rakenteelle, eli hakemistojen ja tiedostojen nimennälle ei ole olemassa mitään yleismaailmallista standardia samaan tapaan kuin esim. Ruby on Railsissa. Tässä käyttämämme malli noudattaa eräitä internetissä vastaan tulevia hyviä käytäntöjä. -->
There is no strict directory structure or file naming convention that is required for Express applications. To contrast this, Ruby on Rails does require a specific structure. Our current structure simply follows some of the best practices you can come across on the internet.

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-1), branchissa <i>part4-1</i>: -->
You can find the code for our current application in its entirety in the <i>part4-1</i> branch of [this github repository](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-1).


<!-- Jos kloonaat projektin itsellesi, suorita komento _npm install_ ennen käynnistämistä eli komentoa _npm start_. -->
If you clone the project for yourself, run the _npm install_ command before starting the application with _npm start_.

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- Rakennamme tämän osan tehtävissä <i>blogilistasovellusta</i>, jonka avulla käyttäjien on mahdollista tallettaa tietoja internetistä löytämistään mielenkiintoisista blogeista. Kustakin blogista talletetaan sen kirjoittaja (author), aihe (title), url sekä blogilistasovelluksen käyttäjien antamien äänien määrä. -->
In the exercises for this part we will be building a <i>blog list application</i>, that allows users to save information about interesting blogs they have stumbled across on the internet. For each listed blog we will save the author, title, url, and amount of upvotes from users of the application.

<!-- #### 4.1 blogilista, step1 -->
#### 4.1 Blog list, step1

<!-- Kuvitellaan tilanne, jossa saat sähköpostitse seuraavan, yhteen tiedostoon koodatun sovellusrungon: -->
Let's imagine a situation, where you receive an email that contains the following application body:

```js
const http = require('http')
const express = require('express')
const app = express()
const bodyParser = require('body-parser')
const cors = require('cors')
const mongoose = require('mongoose')

const blogSchema = mongoose.Schema({
  title: String,
  author: String,
  url: String,
  likes: Number
})

const Blog = mongoose.model('Blog', blogSchema)

const mongoUrl = 'mongodb://localhost/bloglist'
mongoose.connect(mongoUrl, { useNewUrlParser: true })

app.use(cors())
app.use(bodyParser.json())

app.get('/api/blogs', (request, response) => {
  Blog
    .find({})
    .then(blogs => {
      response.json(blogs)
    })
})

app.post('/api/blogs', (request, response) => {
  const blog = new Blog(request.body)

  blog
    .save()
    .then(result => {
      response.status(201).json(result)
    })
})

const PORT = 3003
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`)
})
```

<!-- Tee sovelluksesta toimiva <i>npm</i>-projekti. Jotta sovelluskehitys olisi sujuvaa, konfiguroi sovellus suoritettavaksi <i>nodemonilla</i>. Voit luoda sovellukselle uuden tietokannan MongoDB Atlasiin tai käyttää edellisen osan sovelluksen tietokantaa. -->
Turn the application into a functioning <i>npm</i> project. In order to keep your development productive, configure the application to be executed with <i>nodemon</i>. You can create a new database for your application with MongoDB Atlas, or use the same database from the previous part's exercises.

<!-- Varmista, että sovellukseen on mahdollista lisätä blogeja Postmanilla tai VS Code REST clientilla, ja että sovellus näyttää lisätyt blogit. -->
Verify that it is possible to add blogs to list with Postman or the VS Code REST client and that the application returns the added blogs for the correct endpoint.

<!-- #### 4.2 blogilista, step2 -->
#### 4.2 Blog list, step2

<!-- Jaa sovelluksen koodi tämän osan alun tapaan useaan moduuliin. -->
Refactor the application into separate modules as shown earlier in this part of the course material.

<!-- **HUOM** etene todella pienin askelin, varmistaen että kaikki toimii koko ajan. Jos yrität "oikaista" tekemällä monta asiaa kerralla, on [Murphyn lain](https://fi.wikipedia.org/wiki/Murphyn_laki) perusteella käytännössä varmaa, että jokin menee pahasti pieleen ja "oikotien" takia maaliin päästään paljon myöhemmin kuin systemaattisin pienin askelin. -->
**NB** refactor your application in baby steps and verify that the application works after every change you make. If you try to take a "shortcut" by refactoring many things at once, then [Murphy's law](https://fi.wikipedia.org/wiki/Murphyn_laki) will kick in and it is almost certain that something will break in your application. The "shortcut" will end up taking more time than moving forward slowly and systematically.

<!-- Paras käytänne on commitoida koodi aina stabiilissa tilanteessa, tällöin on helppo palata aina toimivaan tilanteeseen jos koodi menee liian solmuun. -->
One best practice is to commit your code every time it is in a stable state. This makes it easy to rollback to a situation where the application still works.


</div>

<div class="content">

<!-- ### Node-sovellusten testaaminen -->
### Testing Node applications

<!-- Olemme laiminlyöneet ikävästi yhtä oleellista ohjelmistokehityksen osa-aluetta, automatisoitua testaamista. -->
We have completely neglected one essential area of software development, and that is automated testing.

<!-- Aloitamme yksikkötestauksesta. Sovelluksemme logiikka on sen verran yksinkertaista, että siinä ei ole juurikaan mielekästä yksikkötestattavaa. Luodaan tiedosto <i>utils/for_testing.js</i> ja määritellään sinne pari yksinkertaista funktiota testattavaksi: -->
Let's start our testing journey by looking at unit tests. The logic of our application is so simple, that there is not much that makes sense to test with unit tests. Let's create a new file <i>utils/for_testing.js</i> and write a couple of simple functions that we can use for test writing practice:

```js
const palindrom = string => {
  return string
    .split('')
    .reverse()
    .join('')
}

const average = array => {
  const reducer = (sum, item) => {
    return sum + item
  }

  return array.reduce(reducer, 0) / array.length
}

module.exports = {
  palindrom,
  average,
}
```

<!-- > Metodi _average_ käyttää taulukoiden metodia [reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce). Jos metodi ei ole vieläkään tuttu, on korkea aika katsoa Youtubesta [Functional Javascript](https://www.youtube.com/watch?v=BMUiFMZr7vk&list=PL0zVEGEvSaeEd9hlmCXrk5yUyqUag-n84) -sarjasta ainakin kolme ensimmäistä videoa. -->
> The _average_ function uses the array [reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) method. If the method is not familiar to you yet, then now is a good time to watch the the first three videos from the [Functional Javascript](https://www.youtube.com/watch?v=BMUiFMZr7vk&list=PL0zVEGEvSaeEd9hlmCXrk5yUyqUag-n84) series on Youtube.

<!-- Javascriptiin on tarjolla runsaasti erilaisia testikirjastoja eli <i>test runnereita</i>. Käytämme tällä kurssilla Facebookin kehittämää ja sisäisesti käyttämää [jest](https://jestjs.io/):iä, joka on toiminnaltaan ja syntaksiltaankin hyvin samankaltainen kuin testikirjastojen entinen kuningas [Mocha](https://mochajs.org/). Muitakin mahdollisuuksia olisi, esim. eräissä piireissä suosiota nopeasti saavuttanut [ava](https://github.com/avajs/ava). -->
There are many different testing libraries or <i>test runners</i> available for JavaScript. In this course we will be using a testing library developed and used internally by Facebook called [jest](https://jestjs.io/), that resembles the previous king of JavaScript testing libraries [Mocha](https://mochajs.org/). Other alternatives do exist, like [ava](https://github.com/avajs/ava) that has gained popularity in some circles.

<!-- Jest on tälle kurssille luonteva valinta, sillä se sopii hyvin backendien testaamiseen, mutta suorastaan loistaa Reactilla tehtyjen frontendien testauksessa. -->
Jest is a natural choice for this course, as it works well for testing backends, and it shines when it comes to testing React applications. 

<!-- > <i>**Huomio Windows-käyttäjille:**</i> jest ei välttämättä toimi, jos projektin hakemistopolulla on hakemisto, jonka nimessä on välilyöntejä. -->
> <i>**Windows users:**</i> Jest may not work if the path of the project directory contains a directory that has spaces in its name.

<!-- Koska testejä on tarkoitus suorittaa ainoastaan sovellusta kehitettäessä, asennetaan <i>jest</i> kehitysaikaiseksi riippuvuudeksi komennolla -->
Since tests are only executed during the development of our application, we will install <i>jest</i> as a development dependency with the command:

```bash
npm install --save-dev jest
```

<!-- määritellään <i>npm skripti _test</i> suorittamaan testaus jestillä ja raportoimaan testien suorituksesta <i>verbose</i>-tyylillä: -->
Let's define the <i>npm script _test_</i> to execute tests with Jest and to report about the test execution with the <i>verbose</i> style:

```bash
{
  //...
  "scripts": {
    "start": "node index.js",
    "watch": "nodemon index.js",
    "lint": "eslint .",
    "test": "jest --verbose"
  },
  //...
}
```

<!-- Jestin uudemmissa versioissa näyttäisi olevan tarve kertoa, että suoritusympäristönä on käytössä Node. Tämä tapahtuu esim. lisäämällä <i>package.json</i> tiedoston loppuun: -->
I newer versions of Jest, there appears to be the need to specify that the execution environment is Node. This can be done by adding the following to the end of <i>package.json</i>:

```js
{
 //...
 "jest": {
   "testEnvironment": "node"
 }
}
```

<!-- Tai vaihtoehtoisesti Jest löytää myös oletuksena asetustiedoston nimellä <i>jest.config.js</i>, jonne suoritusympäristön määrittely tapahtuu seuraavasti: -->
Alternatively, Jest can look for a configuration file with the default name <i>jest.config.js</i>, where we can define the execution environment like this:

```js
module.exports = {
  testEnvironment: 'node',
};
```

<!-- Tehdään testejä varten hakemisto <i>tests</i> ja sinne tiedosto <i>palindrom.test.js</i>, jonka sisältö on seuraava -->
Let's create a separate directory for our tests called <i>tests</i> and create a new file called <i>palindrom.test.js</i> with the following contents:

```js
const palindrom = require('../utils/for_testing').palindrom

test('palindrom of a', () => {
  const result = palindrom('a')

  expect(result).toBe('a')
})

test('palindrom of react', () => {
  const result = palindrom('react')

  expect(result).toBe('tcaer')
})

test('palindrom of saippuakauppias', () => {
  const result = palindrom('saippuakauppias')

  expect(result).toBe('saippuakauppias')
})
```

<!-- Edellisessä osassa käyttöön ottamamme ESlint valittaa testien käyttämistä komennoista _test_ ja _expect_ sillä käyttämämme konfiguraatio kieltää <i>globaalina</i> määriteltyjen asioiden käytön. Poistetaan valitus lisäämällä <i>.eslintrc.js</i>-tiedoston kenttään <i>env</i> arvo <i>"jest": true</i>. Näin kerromme ESlintille, että käytämme projektissamme Jestiä ja sen globaaleja muuttujia. -->
The ESLint configuration we added to the project in the previous part complains about the _test_ and _expect_ commands in our test file, since the configuration does not allow <i>globals</i>. Let's get rid of the complaints by adding <i>"jest": true</i> to the <i>env</i> property in the <i>.eslintrc.js</i> file.

```js
module.exports = {
  "env": {
      "es6": true,
      "node": true,
      "jest": true 
  },
  "extends": "eslint:recommended",
  "rules": {
    // ...
  },
};
```

<!-- Testi ottaa ensimmäisellä rivillä käyttöön testattavan funktion sijoittaen sen muuttujaan _palindrom_: -->
In the first row, the test file imports the function to be tested and assigns it to a variable called _palindrom_:

```js
const palindrom = require('../utils/for_testing').palindrom
```

<!-- Yksittäiset testitapaukset määritellään funktion _test_ avulla. Ensimmäisenä parametrina on merkkijonomuotoinen testin kuvaus. Toisena parametrina on <i>funktio</i>, joka määrittelee testitapauksen toiminnallisuuden. Esim. toisen testitapauksen toiminnallisuus näyttää seuraavalta: -->
Individual test cases are defined with the _test_ function. The first parameter of the function is the test description as a string. The second parameter is a <i>function</i>, that defines the functionality for the test case. The functionality for the second test case looks like this:

```js
() => {
  const result = palindrom('react')

  expect(result).toBe('tcaer')
}
```

<!-- Ensin suoritetaan testattava koodi, eli generoidaan merkkijonon <i>react</i> palindromi. Seuraavaksi varmistetaan tulos metodin [expect](https://facebook.github.io/jest/docs/en/expect.html#content) avulla. Expect käärii tuloksena olevan arvon olioon, joka tarjoaa joukon <i>matcher</i>-funktioita, joiden avulla tuloksen oikeellisuutta voidaan tarkastella. Koska kyse on kahden merkkijonon samuuden vertailusta, sopii tilanteeseen matcheri [toBe](https://facebook.github.io/jest/docs/en/expect.html#tobevalue). -->
First we execute the code to be tested, meaning that we generate a palindrome for the string <i>react</i>. Next we verify the results with the [expect](https://facebook.github.io/jest/docs/en/expect.html#content) function. Expect wraps the resulting value into an object that offers a collection of <i>matcher</i> functions, that can be used for verifying the correctness of the result. Since in this test case we are comparing two strings, we can use the [toBe](https://facebook.github.io/jest/docs/en/expect.html#tobevalue) matcher.

<!-- Kuten odotettua, testit menevät läpi: -->
As expected, all of the tests pass:

![](../images/4/1.png)

<!-- Jest olettaa oletusarvoisesti, että testitiedoston nimessä on merkkijono <i>.test</i>. Käytetään kurssilla konventiota, millä testitiedostojen nimen loppu on <i>.test.js</i> -->
Jest expects by default that the names of test files contain <i>.test</i>. In this course, we will follow the convention of naming our tests files with the extension <i>test.js</i>.

<!-- Jestin antamat virheilmoitukset ovat hyviä, rikotaan testi -->
Jest has excellent error messages, let's break the test to demonstrate this:

```js
test('palindrom of react', () => {
  const result = palindrom('react')

  expect(result).toBe('tkaer')
})
```

<!-- seurauksena on seuraava virheilmotus -->
Running the tests above results in the following error message:

![](../images/4/2.png)

<!-- Lisätään muutama testi metodille _average_, tiedostoon <i>tests/average.test.js</i>. -->
Let's add a few tests for the _average_ function, into a new file <i>tests/average.test.js</i>.

```js
const average = require('../utils/for_testing').average

describe('average', () => {
  test('of one value is the value itself', () => {
    expect(average([1])).toBe(1)
  })

  test('of many is calculated right', () => {
    expect(average([1, 2, 3, 4, 5, 6])).toBe(3.5)
  })

  test('of empty array is zero', () => {
    expect(average([])).toBe(0)
  })
})
```

<!-- Testi paljastaa, että metodi toimii väärin tyhjällä taulukolla (sillä nollallajaon tulos on Javascriptissä <i>NaN</i>): -->
The test reveals that the function does not work correctly with an empty array (this is because in JavaScript diving by zero results in <i>NaN</i>):

![](../images/4/3.png)

<!-- Metodi on helppo korjata -->
Fixing the function is quite easy:

```js
const average = array => {
  const reducer = (sum, item) => {
    return sum + item
  }
  return array.length === 0
    ? 0 
    : array.reduce(reducer, 0) / array.length
}
```

<!-- Eli jos taulukon pituus on 0, palautetaan 0 ja muussa tapauksessa palautetaan metodin _reduce_ avulla laskettu keskiarvo. -->
If the length of the array is 0 then we return 0, and in all other cases we use the _reduce_ method to calculate the average.

<!-- Pari huomiota keskiarvon testeistä. Määrittelimme testien ympärille nimellä _average_ varustetun <i>describe</i>-lohkon. -->
There's a few things to notice about the tests that we just wrote. We defined a <i>describe</i> block around the tests that was given the name _average_:

```js
describe('average', () => {
  // testit
})
```

<!-- Describejen avulla yksittäisessä tiedostossa olevat testit voidaan jaotella loogisiin kokonaisuuksiin. Testituloste hyödyntää myös describe-lohkon nimeä: -->
Describe blocks can be used for grouping tests into logical collections. The test output of Jest also uses the name of the describe block:

![](../images/4/4.png)

<!-- Kuten myöhemmin tulemme näkemään, <i>describe</i>-lohkot ovat tarpeellisia siinä vaiheessa, jos haluamme osalle yksittäisen testitiedoston testitapauksista jotain yhteisiä alustus- tai lopetustoimenpiteitä. -->
As we will see later on <i>describe</i> blocks are necessary when we want to run some shared setup or teardown operations for a group of tests.

<!-- Toisena huomiona se, että kirjoitimme testit aavistuksen tiiviimmässä muodossa, ottamatta testattavan metodin tulosta erikseen apumuuttujaan: -->
Another thing to notice is that we wrote the tests in quite a compact way, without assigning the output of the function being tested to a variable:

```js
test('of empty array is zero', () => {
  expect(average([])).toBe(0)
})
```

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- Tehdään joukko blogilistan käsittelyyn tarkoitettuja apufunktioita. Tee funktiot esim. tiedostoon <i>utils/list_helper.js</i>. Tee testit sopivasti nimettyyn tiedostoon hakemistoon <i>tests</i>. -->
Let's create a collection of helper functions that are meant to assist dealing with the blog list. Create the functions into a file called <i>utils/list_helper.js</i>. Write your tests into an appropriately named test file under the <i>tests</i> directory.

<!-- #### 4.3: apufunktioita ja yksikkötestejä, step1 -->
#### 4.3: helper functions and unit tests, step1

<!-- Määrittele ensin funktio _dummy_ joka saa parametrikseen taulukollisen blogeja ja palauttaa aina luvun 1. Tiedoston <i>list_helper.js</i> sisällöksi siis tulee tässä vaiheessa -->
First define a _dummy_ function that receives an array of blog posts as a parameter and always returns the value 1. The contents of the <i>list_helper.js</i> file at this point should be the following:

```js
const dummy = (blogs) => {
  // ...
}

module.exports = {
  dummy
}
```

<!-- Varmista testikonfiguraatiosi toimivuus seuraavalla testillä: -->
Verify that your test configuration works with the following test:

```js
const listHelper = require('../utils/list_helper')

test('dummy returns one', () => {
  const blogs = []

  const result = listHelper.dummy(blogs)
  expect(result).toBe(1)
})
```

<!-- #### 4.4: apufunktioita ja yksikkötestejä, step2 -->
#### 4.4: helper functions and unit tests, step2

<!-- Määrittele funktio _totalLikes_ joka saa parametrikseen taulukollisen blogeja. Funktio palauttaa blogien yhteenlaskettujen tykkäysten eli <i>likejen</i> määrän. -->
Define a new _totalLikes_ function that receives a list of blog posts as a parameter. The function returns the total sum of <i>likes</i> in all of the blog posts.

<!-- Määrittele funktiolle sopivat testit. Funktion testit kannattaa laittaa <i>describe</i>-lohkoon jolloin testien tulostus ryhmittyy miellyttävästi: -->
Write appropriate tests for the function. It's recommended to put the tests inside of a <i>describe</i> block, so that the test report output gets grouped nicely:

![](../images/4/5.png)

<!-- Testisyötteiden määrittely onnistuu esim. seuraavaan tapaan: -->
Defining test inputs for the function can be done like this:

```js
describe('total likes', () => {
  const listWithOneBlog = [
    {
      _id: '5a422aa71b54a676234d17f8',
      title: 'Go To Statement Considered Harmful',
      author: 'Edsger W. Dijkstra',
      url: 'http://www.u.arizona.edu/~rubinson/copyright_violations/Go_To_Considered_Harmful.html',
      likes: 5,
      __v: 0
    }
  ]

  test('when list has only one blog equals the likes of that', () => {
    const result = listHelper.totalLikes(listWithOneBlog)
    expect(result).toBe(5)
  })
})
```

<!-- Jos et viitsi itse määritellä testisyötteenä käytettäviä blogeja, saat valmiin listan [täältä.](https://github.com/fullstack-hy2019/misc/blob/master/blogs_for_test.md) -->
If defining your own test input list of blogs is too much work, you can use the ready-made list [here](https://github.com/fullstack-hy2019/misc/blob/master/blogs_for_test.md).

<!-- Törmäät varmasti testien tekemisen yhteydessä erinäisiin ongelmiin. Pidä mielessä osassa 3 käsitellyt [debuggaukseen](osa3/tietojen_tallettaminen_mongo_db_tietokantaan#node-sovellusten-debuggaaminen) liittyvät asiat, voit testejäkin suorittaessasi printtailla konsoliin komennolla _console.log_. Myös debuggerin käyttö testejä suorittaessa on mahdollista, ohje [täällä](https://jestjs.io/docs/en/troubleshooting). -->
You are bound to run into problems while writing tests. Remember the things that we learned about [debugging](osa3/tietojen_tallettaminen_mongo_db_tietokantaan#node-sovellusten-debuggaaminen) in part 3. You can print things to the console with _console.log_ even during test execution. It is even possible to use the debugger while running tests, you can find instructions for that [here](https://jestjs.io/docs/en/troubleshooting).

<!-- **HUOM:** jos jokin testi ei mene läpi, ei ongelmaa korjatessa kannata suorittaa kaikkia testejä, vaan ainoastaan rikkinäistä testiä hyödyntäen [only](https://facebook.github.io/jest/docs/en/api.html#testonlyname-fn-timeout)-metodia.  -->
**NB:** if some test is failing, then it is recommended to only run that test while you are fixing the issue. You can run a single test with the [only](https://facebook.github.io/jest/docs/en/api.html#testonlyname-fn-timeout) method.

<!-- Toinen tapa suorittaa yksittäinen testi (tai describe-lohko) on kutsua jestiä suoraan ja määritellä sille suoritettava testi argumentin [-t](https://jestjs.io/docs/en/cli.html) avulla: -->
Another way of running a single test (or describe block) is to run Jest from the command line and to specify the name of the test to be run with the [-t](https://jestjs.io/docs/en/cli.html) flag:

```js
npx jest -t 'when list has only one blog equals the likes of that'
```

<!-- #### 4.5*: apufunktioita ja yksikkötestejä, step3 -->
#### 4.5*: helper functions and unit tests, step3

<!-- Määrittele funktio _favoriteBlog_ joka saa parametrikseen taulukollisen blogeja. Funktio selvittää millä blogilla on eniten likejä. Jos suosikkeja on monta, riittää että funktio palauttaa niistä jonkun. -->
Define a new _favoriteBlog_ function that receives a list of blogs as a parameter. The function finds out which blog has most likes. If there are many top favorites, it is enough to return one of them.

<!-- Paluuarvo voi olla esim. seuraavassa muodossa: -->
The value returned by the function could be in the following format:

```js
{
  title: "Canonical string reduction",
  author: "Edsger W. Dijkstra",
  likes: 12
}
```

<!-- **Huom**, että kun vertailet olioita, metodi [toEqual](https://jestjs.io/docs/en/expect#toequalvalue) on todennäköisesti se mitä haluat käyttää sillä [toBe](https://jestjs.io/docs/en/expect#tobevalue)-vertailu, joka sopii esim. lukujen ja merkkijonojen vertailuun vaatisi olioiden vertailussa, että oliot ovat samat, pelkkä sama sisältöisyys ei riitä. -->
**NB** when you are comparing objects, the [toEqual](https://jestjs.io/docs/en/expect#toequalvalue) method is probably what you want to use, since the [toBe](https://jestjs.io/docs/en/expect#tobevalue) tries to verify that the two values are the same value, and not just that they contain the same properties.

<!-- Tee myös tämän ja seuraavien kohtien testit kukin oman <i>describe</i>-lohkon sisälle. -->
Write the tests for this exercise inside of a new <i>describe</i> block. Do the same for the remaining exercises as well.

#### 4.6*: apufunktioita ja yksikkötestejä, step4

<!-- Tämä ja seuraava tehtävä ovat jo hieman haastavampia. Tehtävien tekeminen ei ole osan jatkon kannalta oleellista, eli voi olla hyvä idea palata näihin vasta kun muu osa on kahlattu läpi. -->
This and the next exercise are a little bit more challenging. Finishing these two exercises is not required in order to advance in the course material, so it may be a good idea to return to these once you're done going through the material for this part in its entirety.

<!-- Tehtävän tekeminen onnistuu hyvin ilman mitään kirjastojakin, mutta tämä saattaa olla hyvä paikka tutustua kokoelmien käsittelyä suuresti helpottavaan [Lodash](https://lodash.com/)-kirjastoon. -->
Finishing this exercise can be done without the use of additional libraries. However, this exercise is a great opportunity to learn how to use the [Lodash](https://lodash.com/) library.

<!-- Määrittele funktio _mostBlogs_ joka saa parametrikseen taulukollisen blogeja. Funktio selvittää <i>kirjoittajan</i>, kenellä on eniten blogeja. Funktion paluuarvo kertoo myös ennätysblogaajan blogien määrän: -->
Define a function called _mostBlogs_ that receives an array of blogs as a parameter. The function return the <i>author</i> who has the largest amount of blogs. The return value also contains the number of blogs the top author has:

```js
{
  author: "Robert C. Martin",
  blogs: 3
}
```

<!-- Jos ennätysblogaajia on monta, riittää että funktio palauttaa niistä jonkun. -->
If there are many top bloggers, then it is enough to return any one of them.

<!-- #### 4.7*: apufunktioita ja yksikkötestejä, step5 -->
#### 4.7*: helper functions and unit tests, step5

<!-- Määrittele funktio _mostLikes_ joka saa parametrikseen taulukollisen blogeja. Funktio selvittää kirjoittajan, kenen blogeilla on eniten likejä. Funktion paluuarvo kertoo myös suosikkiblogaajan likejen yhteenlasketun määrän: -->
Define a function called _mostLikes_ that receives an array of blogs as its parameter. The function returns the author, whose blog posts have the largest amount of likes. The return value also contains the total number of likes that the author has received:

```js
{
  author: "Edsger W. Dijkstra",
  likes: 17
}
```

<!-- Jos suosikkiblogaajia on monta, riittää että funktio palauttaa niistä jonkun. -->
If there are many top bloggers, then it is enough to show any one of them.

</div>
