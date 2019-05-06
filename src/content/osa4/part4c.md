---
mainImage: ../../images/part-4.svg
part: 4
letter: c
---

<div class="content">

<!-- Haluamme toteuttaa sovellukseemme käyttäjien hallinnan. Käyttäjät tulee tallettaa tietokantaan ja jokaisesta muistiinpanosta tulee tietää sen luonut käyttäjä. Muistiinpanojen poisto ja editointi tulee olla sallittua ainoastaan muistiinpanot tehneelle käyttäjälle. -->
We want to add user authentication and authorization to our application. Users should be stored in the database and every note should be linked to the user who created it. Deleting and editing a note should only be allowed for the user who created the note.

<!-- Aloitetaan lisäämällä tietokantaan tieto käyttäjistä. Käyttäjän (<i>User</i>) ja muistiinpanojen (<i>Note</i>) välillä on yhden suhde moneen -yhteys: -->
Let's start by adding information about users to the database. There is a one-to-many relationship between the user (<i>User</i>) and notes (<i>Note</i>):

![](https://yuml.me/a187045b.png)

<!-- Relaatiotietokantoja käytettäessä ratkaisua ei tarvitsisi juuri miettiä. Molemmille olisi oma taulunsa ja muistiinpanoihin liitettäisiin sen luonutta käyttäjää vastaava id vierasavaimeksi (foreign key). -->
If we were working with a relational database the implementation would be self-evident. Both resources would have separate database tables and the id of the user who created the note would be stored in the notes table as a foreign key.

<!-- Dokumenttitietokantoja käytettäessä tilanne on kuitenkin toinen, erilaisia tapoja mallintaa tilanne on useita. -->
When working with document databases the situation is a bit different, as there are many different ways of modeling the situation.

<!-- Olemassaoleva ratkaisumme tallentaa jokaisen luodun muistiinpanon tietokantaan <i>notes</i>-kokoelmaan eli <i>collectioniin</i>. Jos emme halua muuttaa tätä, lienee luontevinta tallettaa käyttäjät omaan kokoelmaansa, esim. nimeltään <i>users</i>. -->
The existing solution saves every note in the <i>notes collection</i> in the database. If we do not want to change this existing collection, then the natural choice is to save users in their own collection called <i>users</i>, for example.

<!-- Mongossa voidaan kaikkien dokumenttitietokantojen tapaan käyttää olioiden id:itä viittaamaan muissa kokoelmissa talletettaviin dokumentteihin, vastaavasti kuten viiteavaimia käytetään relaatiotietokannoissa. -->
Like all document databases, we can use object id's in Mongo to reference documents in other collections. This is similar to using foreign keys in relational databases.

<!-- Dokumenttitietokannat kuten Mongo eivät kuitenkaan tue relaatiotietokantojen <i>liitoskyselyitä</i> vastaavaa toiminnallisuutta, joka mahdollistaisi useaan kokoelmaan kohdistuvan tietokantahaun. Tämä ei tarkalleen ottaen enää välttämättä pidä paikkaansa, sillä versiosta 3.2. alkaen Mongo on tukenut useampaan kokoelmaan kohdistuvia [lookup-aggregaattikyselyitä](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/). Emme kuitenkaan käsittele niitä kurssilla. -->
Document databases like Mongo do not support similar <i>join queries</i> that are available in relational databases, that are used for aggregating data from multiple tables. This is not strictly true, however. Starting from version 3.2. Mongo has supported [lookup aggregation queries](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/). We will not be taking a look at this functionality in this course.

<!-- Jos tarvitsemme liitoskyselyitä vastaavaa toiminnallisuutta, tulee se toteuttaa sovelluksen tasolla, eli käytännössä tekemällä tietokantaan useita kyselyitä. Tietyissä tilanteissa mongoose-kirjasto osaa hoitaa liitosten tekemisen, jolloin kysely näyttää mongoosen käyttäjälle toimivan liitoskyselyn tapaan. Mongoose tekee kuitenkin näissä tapauksissa taustalla useamman kyselyn tietokantaan. -->
If we need functionality similar to join queries, we will implement it in our application code by making multiple queries. In certain situations Mongoose can take care of joining and aggregating data which gives the appearance of a join query. However, even in these situations Mongoose makes multiple queries to the database.

<!-- ### Viitteet kokoelmien välillä -->
### References across collections

<!-- Jos käyttäisimme relaatiotietokantaa, muistiinpano sisältäisi <i>viiteavaimen</i> sen tehneeseen käyttäjään. Dokumenttitietokannassa voidaan toimia samoin. -->
If we were using a relational database the note would contain a <i>reference key</i> to the user who created it. In document databases we can do the same thing. 

<!-- Oletetaan että kokoelmassa <i>users</i> on kaksi käyttäjää: -->
Let's assume that the <i>users</i> collection contains two users:

```js
[
  {
    username: 'mluukkai',
    _id: 123456,
  },
  {
    username: 'hellas',
    _id: 141414,
  },
];
```

<!-- Kokoelmassa <i>notes</i> on kolme muistiinpanoa, kaikkien kenttä <i>user</i> viittaa <i>users</i>-kentässä olevaan käyttäjään: -->
The <i>notes</i> collection contains three notes that all have a <i>user</i> field that references a user in the <i>users</i> collection:

```js
[
  {
    content: 'HTML on helppoa',
    important: false,
    _id: 221212,
    user: 123456,
  },
  {
    content: 'HTTP-protokollan tärkeimmät metodit ovat GET ja POST',
    important: true,
    _id: 221255,
    user: 123456,
  },
  {
    content: 'Java on kieli, jota käytetään siihen asti kunnes aurinko sammuu',
    important: false,
    _id: 221244,
    user: 141414,
  },
];
```

<!-- Mikään ei kuitenkaan määrää dokumenttitietokannoissa, että viitteet on talletettava muistiinpanoihin, ne voivat olla <i>myös</i> (tai ainoastaan) käyttäjien yhteydessä: -->
Document databases do not demand the foreign key to be stored in the note resources, they could <i>also</i> be stored in the users collection, or even both:

```js
[
  {
    username: 'mluukkai',
    _id: 123456,
    notes: [221212, 221255],
  },
  {
    username: 'hellas',
    _id: 141414,
    notes: [141414],
  },
];
```

<!-- Koska käyttäjiin liittyy potentiaalisesti useita muistiinpanoja, niiden id:t talletetaan käyttäjän kentässä <i>notes</i> olevaan taulukkoon. -->
Since users can have many notes, the related ids are stored in an array in the <i>notes</i> field.

<!-- Dokumenttitietokannat tarjoavat myös radikaalisti erilaisen tavan datan organisointiin; joissain tilanteissa saattaisi olla mielekästä tallettaa muistiinpanot kokonaisuudessa käyttäjien sisälle: -->
Document databases offer a radically different way of organizing data.  In some situations it might even be beneficial to nest the entire notes array as a part of the documents in the users collection:

```js
[
  {
    username: 'mluukkai',
    _id: 123456,
    notes: [
      {
        content: 'HTML on helppoa',
        important: false,
      },
      {
        content: 'HTTP-protokollan tärkeimmät metodit ovat GET ja POST',
        important: true,
      },
    ],
  },
  {
    username: 'hellas',
    _id: 141414,
    notes: [
      {
        content:
          'Java on kieli, jota käytetään siihen asti kunnes aurinko sammuu',
        important: false,
      },
    ],
  },
];
```

<!-- Muistiinpanot olisivat tässä skeemaratkaisussa siis yhteen käyttäjään alisteisia kenttiä, niillä ei olisi edes omaa identiteettiä, eli id:tä tietokannan tasolla. -->
In this schema notes would be tightly nested under users and the database would not generate ids for them.

<!-- Dokumenttitietokantojen yhteydessä skeeman rakenne ei siis ole ollenkaan samalla tavalla ilmeinen kuin relaatiotietokannoissa, ja valittava ratkaisu kannattaa määritellä siten että se tukee parhaalla tavalla sovelluksen käyttötapauksia. Tämä ei luonnollisestikaan ole helppoa, sillä järjestelmän kaikki käyttötapaukset eivät yleensä ole selvillä kun projektin alkuvaiheissa mietitään datan organisointitapaa. -->
The structure and schema of the database is not as self-evident as it was with relational databases. The choice of schema must be one that supports the use cases of the application the best. This is not a simple design decision to make, as all use cases of the applications are not known when the design decision is made.

<!-- Hieman paradoksaalisesti tietokannan tasolla skeematon Mongo edellyttääkin projektin alkuvaiheissa jopa radikaalimpien datan organisoimiseen liittyvien ratkaisujen tekemistä kuin tietokannan tasolla skeemalliset relaatiotietokannat, jotka tarjoavat keskimäärin kaikkiin tilanteisiin melko hyvin sopivan tavan organisoida dataa. -->
Paradoxically, schema-less databases like Mongo require developers to make far more radical design decisions about data organization at the beginning of the project than relational databases with schemas. On average, relational databases offer a more-or-less suitable way of organizing data for many applications.

<!-- ### Käyttäjien mongoose-skeema -->
### Mongoose schema for users

<!-- Päätetään tallettaa käyttäjän yhteyteen myös tieto käyttäjän luomista muistiinpanoista, eli käytännössä muistiinpanojen id:t. Määritellään käyttäjää edustava model tiedostoon <i>models/user.js</i> -->
In this case, we make the decision to store the ids of the notes created by the user in the user document. Let's define the model for representing a user in the <i>models/user.js</i> file:

```js
const mongoose = require('mongoose')

const userSchema = mongoose.Schema({
  username: String,
  name: String,
  passwordHash: String,
  notes: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Note'
    }
  ],
})

userSchema.set('toJSON', {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString()
    delete returnedObject._id
    delete returnedObject.__v
    // suodatetaan passwordHash eli salasanan tiiviste pois näkyviltä
    delete returnedObject.passwordHash
  }
})

const User = mongoose.model('User', userSchema)

module.exports = User
```

<!-- Muistiinpanojen id:t on talletettu käyttäjien sisälle taulukkona mongo-id:itä. Määrittely on seuraava -->
The ids of the notes are stored within the user document as an array of Mongo ids. The definition is as follows:

```js
{
  type: mongoose.Schema.Types.ObjectId,
  ref: 'Note'
}
```

<!-- kentän tyyppi on <i>ObjectId</i> joka viittaa <i>note</i>-tyyppisiin dokumentteihin. Mongo ei itsessään tiedä mitään siitä, että kyse on kentästä, joka viittaa nimenomaan muistiinpanoihin, kyseessä onkin puhtaasti mongoosen syntaksi. -->
The type of the field is <i>ObjectId</i> that references <i>note</i>-style documents. Mongo does not inherently know that this is a field that references notes, this syntax is purely related to and defined by Mongoose.

<!-- Laajennetaan tiedostossa <i>model/note.js</i> olevaa muistiinpanon skeemaa siten, että myös muistiinpanossa on tieto sen luoneesta käyttäjästä -->
Let's expand the schema of the note defined in the <i>model/note.js</i> file, so that the note contains information about the user who created it:

```js
const noteSchema = new mongoose.Schema({
  content: {
    type: String,
    required: true,
    minlength: 5
  },
  date: Date,
  important: Boolean,
  // highlight-start
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
  // highlight-end
})
```

<!-- Relaatiotietokantojen käytänteistä poiketen <i>viitteet on nyt talletettu molempiin dokumentteihin</i>, muistiinpano viittaa sen luoneeseen käyttäjään ja käyttäjä sisältää taulukollisen viitteitä sen luomiin muistiinpanoihin. -->
In stark contrast to the conventions of relational databases, <i>references are now stored in both documents</i>, the note references the user who created it, and the user has an array of references to all of the notes created by the user.

<!-- ### Käyttäjien luominen -->
### Creating users

<!-- Toteutetaan seuraavaksi route käyttäjien luomista varten. Käyttäjällä on siis <i>username</i> jonka täytyy olla järjestelmässä yksikäsitteinen, nimi eli <i>name</i> sekä <i>passwordHash</i>, eli salasanasta [yksisuuntaisen funktion](https://en.wikipedia.org/wiki/Cryptographic_hash_function) perusteella laskettu tunniste. Salasanojahan ei ole koskaan viisasta tallentaa tietokantaan selväsanaisena! -->
Let's implement a route for creating new users. Users have a unique <i>username</i>, a <i>name</i> and something called a <i>passwordHash</i>. The password hash is the output of a [one-way hash function](https://en.wikipedia.org/wiki/Cryptographic_hash_function) applied to the user's password. It is never wise to store unencrypted plaintext passwords in the database!

<!-- Asennetaan salasanojen hashaamiseen käyttämämme [bcrypt](https://github.com/kelektiv/node.bcrypt.js)-kirjasto: -->
Let's install the [bcrypt](https://github.com/kelektiv/node.bcrypt.js) package for generating the password hash for users:

```bash
npm install bcrypt --save
```

<!-- Käyttäjien luominen tapahtuu osassa 3 läpikäytyjä [RESTful](/osa3/node_js_ja_express#rest)-periaatteita seuraten tekemällä HTTP POST -pyyntö polkuun <i>users</i>. -->
Creating new users happens in compliance with the RESTful conventions discussed in [part 3](/osa3/node_js_ja_express#rest), by making an HTTP POST request to the <i>users</i> path.

<!-- Määritellään käyttäjienhallintaa varten oma <i>router</i> tiedostoon <i>controllers/users.js</i>, ja liitetään se <i>app.js</i>-tiedostossa huolehtimaan polulle <i>/api/users/</i> tulevista pyynnöistä: -->
Let's define a separate <i>router</i> for dealing with users in a new <i>controllers/users.js</i> file. Let's take the router into use in our application in the <i>app.js</i> file, so that it handles requests made to the <i>/api/users</i> url:

```js
const usersRouter = require('./controllers/users')

// ...

app.use('/api/users', usersRouter)
```

<!-- Routerin alustava sisältö on seuraava: -->
The contents of the file that define the router are as follows:

```js
const bcrypt = require('bcrypt')
const usersRouter = require('express').Router()
const User = require('../models/user')

usersRouter.post('/', async (request, response, next) => {
  try {
    const body = request.body

    const saltRounds = 10
    const passwordHash = await bcrypt.hash(body.password, saltRounds)

    const user = new User({
      username: body.username,
      name: body.name,
      passwordHash,
    })

    const savedUser = await user.save()

    response.json(savedUser)
  } catch (exception) {
    next(exception)
  }
})

module.exports = usersRouter
```

<!-- Tietokantaan siis <i>ei</i> talleteta pyynnön mukana tulevaa salasanaa, vaan funktion _bcrypt.hash_ avulla laskettu <i>hash</i>. -->
The password sent in the request is <i>not</i> stored in the database. We store the <i>hash</i> of the password that is generated with the _bcrypt.hash_ function.

<!-- Materiaalin tilamäärä ei valitettavasti riitä käsittelemään sen tarkemmin salasanojen [tallennuksen perusteita](https://codahale.com/how-to-safely-store-a-password/), esim. mitä maaginen luku 10 muuttujan [saltRounds](https://github.com/kelektiv/node.bcrypt.js/#a-note-on-rounds) arvona tarkoittaa. Lue linkkien takaa lisää. -->
The fundamentals of [storing passwords](https://codahale.com/how-to-safely-store-a-password/) is outside the scope of this course material. We will not discuss what the magic number 10 assigned to the [saltRounds](https://github.com/kelektiv/node.bcrypt.js/#a-note-on-rounds) variable means, but you can read more about it in the linked material.

<!-- Koodissa ei tällä hetkellä ole mitään virheidenkäsittelyä eikä validointeja, eli esim. käyttäjätunnuksen ja salasanan halutun muodon tarkastuksia. -->
Our current code does not contain any error handling or input validation for verifying that the username and password are in a desired format.

<!-- Uutta ominaisuutta voidaan ja kannattaakin joskus testailla käsin esim. postmanilla. Käsin tapahtuva testailu muuttuu kuitenkin nopeasti työlääksi, etenkin kun tulemme pian vaatimaan, että samaa käyttäjätunnusta ei saa tallettaa kantaan kahteen kertaan. -->
The new feature can and should initially be tested manually with a tool like Postman. Testing things manually will quickly become too cumbersome, especially once we implement functionality that enforces usernames to be unique.

<!-- Pienellä vaivalla voimme tehdä automaattisesti suoritettavat testit, jotka helpottavat sovelluksen kehittämistä merkittävästi. -->
It takes much less effort to write automated tests, that will make the development of our application much easier.

<!-- Alustava testi näyttää seuraavalta: -->
Our initial tests could look like this:

```js
const User = require('../models/user')

//...

describe('when there is initially one user at db', async () => {
  beforeEach(async () => {
    await User.deleteMany({})
    const user = new User({ username: 'root', password: 'sekret' })
    await user.save()
  })

  test('creation succeeds with a fresh username', async () => {
    const usersAtStart = await helper.usersInDb()

    const newUser = {
      username: 'mluukkai',
      name: 'Matti Luukkainen',
      password: 'salainen',
    }

    await api
      .post('/api/users')
      .send(newUser)
      .expect(200)
      .expect('Content-Type', /application\/json/)

    const usersAtEnd = await helper.usersInDb()
    expect(usersAtEnd.length).toBe(usersAtStart.length + 1)

    const usernames = usersAtEnd.map(u => u.username)
    expect(usernames).toContain(newUser.username)
  })
})
```

<!-- Testit käyttävät myös tiedostossa <i>tests/test_helper.js</i> määriteltyä apufunktiota <i>usersInDb()</i> tarkastamaan lisäysoperaation jälkeisen tietokannan tilan: -->
The tests use the <i>usersInDb()</i> helper function that we imlemented in the <i>tests/test_helper.js</i> file. The function is used to help us verify the state of the database after a user is created:

```js
const User = require('../models/user')

// ...

const usersInDb = async () => {
  const users = await User.find({})
  return users.map(u => u.toJSON())
}

module.exports = {
  initialNotes,
  nonExistingId,
  notesInDb,
  usersInDb,
}
```

<!-- Lohkon <i>beforeEach</i> lisää kantaan käyttäjän, jonka username on <i>root</i>. Voimmekin tehdä uuden testin, jolla varmistetaan, että samalla käyttäjätunnuksella ei voi luoda uutta käyttäjää: -->
The <i>beforeEach</i> block adds a user with the username <i>root</i> to the database. We can write a new test that verifies that a new user with the same username can not be created:

```js
describe('when there is initially one user at db', async () => {
  // ...

  test('creation fails with proper statuscode and message if username already taken', async () => {
    const usersAtStart = await helper.usersInDb()

    const newUser = {
      username: 'root',
      name: 'Superuser',
      password: 'salainen',
    }

    const result = await api
      .post('/api/users')
      .send(newUser)
      .expect(400)
      .expect('Content-Type', /application\/json/)

    expect(result.body.error).toContain('`username` to be unique')

    const usersAtEnd = await helper.usersInDb()
    expect(usersAtEnd.length).toBe(usersAtStart.length)
  })
})
```

<!-- Testi ei tietenkään mene läpi tässä vaiheessa. Toimimme nyt oleellisesti [TDD:n eli test driven developmentin](https://en.wikipedia.org/wiki/Test-driven_development) hengessä, uuden ominaisuuden testi on kirjoitettu ennen ominaisuuden ohjelmointia. -->
The test case obviously will not pass at this point. We are essentially practicing [test-driven development (TDD)](https://en.wikipedia.org/wiki/Test-driven_development), where tests for new functionality are written before the functionality is implemented.

<!-- Hoidetaan uniikkiuden tarkastaminen Mongoosen validoinnin avulla. Kuten edellisen osan tehtävässä [3.19](/osa3/validointi_ja_es_lint#tehtavia) mainittiin, Mongoose ei tarjoa valmista validaattoria kentän uniikkiuden tarkastamiseen. Tilanteeseen ratkaisun tarjoaa npm-pakettina asennettava[mongoose-unique-validator](https://www.npmjs.com/package/mongoose-unique-validator). Suoritetaan asennus -->
Let's validate the uniqueness of the username with the help of Mongoose validators. As we mentioned in exercise [3.19](/osa3/validointi_ja_es_lint#tehtavia), Mongoose does not have a built-in validator for checking the uniqueness of a field. We can find a ready-made solution for this from the [mongoose-unique-validator](https://www.npmjs.com/package/mongoose-unique-validator) npm package. Let's install it:

```bash
npm install --save mongoose-unique-validator
```

<!-- Käyttäjän skeemaa tiedostossa <i>models/user.js</i> tulee muuttaa seuraavasti seuraavasti: -->
We must make the following changes to the schema defined in the <i>models/user.js</i> file:

```js
const mongoose = require('mongoose')
const uniqueValidator = require('mongoose-unique-validator') // highlight-line

const userSchema = mongoose.Schema({
  username: {
    type: String,
    unique: true  // highlight-line
  },
  name: String,
  passwordHash: String,
  notes: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Note'
    }
  ],
})

userSchema.plugin(uniqueValidator) // highlight-line

// ...
```

<!-- Voisimme toteuttaa käyttäjien luomisen yhteyteen myös muita tarkistuksia, esim. onko käyttäjätunnus tarpeeksi pitkä, koostuuko se sallituista merkeistä ja onko salasana tarpeeksi hyvä. Jätämme ne kuitenkin vapaaehtoiseksi harjoitustehtäväksi. -->
We could also implement other validations into the creation of a user. We could check that the username is long enough, that the username only consists of permitted characters, or that the password is strong enough. Implementing this functionality is left as an optional exercise.

<!-- Ennen kuin menemme eteenpäin, lisätään sovellukseen alustava versio palauttaa kaikki käyttäjät palauttavasta käsittelijäfunktiosta: -->
Before we move onward, let's add an initial implementation of a route handler that returns all of the users in the application:

```js
usersRouter.get('/', async (request, response) => {
  const users = await User.find({})
  response.json(users.map(u => u.toJSON()))
})
```

<!-- Lista näyttää seuraavalta -->
The list looks like this:

![](../images/4/9.png)

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-6), branchissä <i>part4-6</i>. -->
You can find the code for our current application in it entirety in the <i>part4-6</i> branch of [this github repository](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-6).

<!-- ### Muistiinpanon luominen -->
### Creating a new note

<!-- Muistiinpanot luovaa koodia on nyt mukautettava siten, että uusi muistiinpano tulee liitetyksi sen luoneeseen käyttäjään. -->
The code for creating a new note has to be updated so that it is assigned to the user that created it.

<!-- Laajennetaan ensin olemassaolevaa toteutusta siten, että tieto muistiinpanon luovan käyttäjän id:stä lähetetään pyynnön rungossa kentän <i>userId</i> arvona: -->
Let's expand our current implementation, so that the information about the user who created the note is sent in the <i>userId</i> field of the request body:

```js
const User = require('../models/user')

//...

notesRouter.post('/', async (request, response, next) => {
  const body = request.body

  const user = await User.findById(body.userId) //highlight-line

  const note = new Note({
    content: body.content,
    important: body.important === undefined ? false : body.important,
    date: new Date(),
    user: user._id //highlight-line
  })

  try {
    const savedNote = await note.save()
    user.notes = user.notes.concat(savedNote._id) //highlight-line
    await user.save()  //highlight-line
    response.json(savedNote.toJSON())
  } catch(exception) {
    next(exception)
  }
})
```

<!-- Huomionarvoista on nyt se, että myös <i>user</i>-olio muuttuu. Sen kenttään <i>notes</i> talletetaan luodun muistiinpanon <i>id</i>: -->
It's worth noting that the <i>user</i> object also changes. The <i>id</i> of the note is stored in the <i>notes</i> field:

```js
const user = User.findById(userId)

user.notes = user.notes.concat(savedNote._id)
await user.save()
```

<!-- Kokeillaan nyt lisätä uusi muistiinpano -->
Let's try to create a new note

![](../images/4/10.png)

<!-- Operaatio vaikuttaa toimivan. Lisätään vielä yksi muistiinpano ja mennään kaikkien käyttäjien sivulle: -->
The operation appears to work. Let's add one more note and then visit the route for fetching all users:

![](../images/4/11.png)

<!-- Huomaamme siis, että käyttäjällä on kaksi muistiinpanoa. -->
We can see that the user has two notes. 

<!-- Muistiinpanon luoneen käyttäjän id näkyviin muistiinpanon yhteyteen: -->
Likewise, the id of the user who created the note can be seen when we visit the route for fetching all notes:

![](../images/4/12.png)

<!-- ### populate -->
### Populate

<!-- Haluaisimme API:n toimivan siten, että haettaessa esim. käyttäjien tiedot polulle <i>/api/users</i> tehtävällä HTTP GET -pyynnöllä tulisi käyttäjien tekemien muistiinpanojen id:iden lisäksi näyttää niiden sisältö. Relaatiotietokannoilla toiminnallisuus toteutettaisiin <i>liitoskyselyn</i> avulla. -->
We would like our API to work in such a way, that when an HTTP GET request is made to the <i>/api/users</i> route, the user objects would also contain the content of the user's notes, and not just their id. In a relational database, this functionality would be implemented with a <i>join query</i>.

<!-- Kuten aiemmin mainittiin, eivät dokumenttitietokannat tue (kunnolla) eri kokoelmien välisiä liitoskyselyitä. Mongoose-kirjasto osaa kuitenkin tehdä liitoksen puolestamme. Mongoose toteuttaa liitoksen tekemällä useampia tietokantakyselyitä, joten siinä mielessä kyseessä on täysin erilainen tapa kuin relaatiotietokantojen liitoskyselyt, jotka ovat <i>transaktionaalisia</i>, eli liitoskyselyä tehdessä tietokannan tila ei muutu. Mongoosella tehtävä liitos taas on sellainen, että mikään ei takaa sitä, että liitettävien kokoelmien tila on konsistentti, toisin sanoen jos tehdään users- ja notes-kokoelmat liittävä kysely, kokoelmien tila saattaa muuttua kesken Mongoosen liitosoperaation. -->
As previously mentioned, document databases do not properly support join queries between collections, but the Mongoose library can do some of these joins for us. Mongoose accomplishes the join by doing multiple queries, which is different from join queries in relational databases that are <i>transactional</i>, meaning that the state of the database does not change during the time that the query is made. With join queries in Mongoose, nothing can guarantee that the state between the collections being joined is consistent, meaning that if we make a query that joins the user and notes collections, the state of the collections may change during the query.

<!-- Liitoksen tekeminen suoritetaan Mongoosen komennolla [populate](http://mongoosejs.com/docs/populate.html). Päivitetään ensin kaikkien käyttäjien tiedot palauttava route: -->
The Mongoose join is done with the [populate](http://mongoosejs.com/docs/populate.html) method. Let's update the route that returns all users first:

```js
usersRouter.get('/', async (request, response) => {
  const users = await User  // highlight-line
    .find({}).populate('notes') // highlight-line

  response.json(users.map(u => u.toJSON()))
})
```

<!-- Funktion [populate](http://mongoosejs.com/docs/populate.html) kutsu siis ketjutetaan kyselyä vastaavan metodikutsun (tässä tapauksessa <i>find_</i> perään. Populaten parametri määrittelee, että <i>user</i>-dokumenttien <i>notes</i>-kentässä olevat <i>note</i>-olioihin viittaavat <i>id</i>:t korvataan niitä vastaavilla dokumenteilla. -->
The [populate](http://mongoosejs.com/docs/populate.html) method is chained after the <i>find</i> method that makes the initial query. The parameter given to the populate method defines that the <i>ids</i> referencing <i>note</i> objects in the <i>notes</i> field of the <i>user</i> document, will be replaced by the referenced <i>note</i> documents.

<!-- Lopputulos on jo melkein haluamamme kaltainen: -->
The result is almost exactly what we wanted:

![](../images/4/13.png)

<!-- Populaten yhteydessä on myös mahdollista rajata mitä kenttiä sisällytettävistä dokumenteista otetaan mukaan. Rajaus tapahtuu Mongon [syntaksilla](https://docs.mongodb.com/manual/tutorial/project-fields-from-query-results/#return-the-specified-fields-and-the-id-field-only): -->
We can use the same populate parameter for choosing the fields we want to include from the documents. The selection of fields is done with the Mongo [syntax](https://docs.mongodb.com/manual/tutorial/project-fields-from-query-results/#return-the-specified-fields-and-the-id-field-only):

```js
usersRouter.get('/', async (request, response) => {
  const users = await User
    .find({}).populate('notes', { content: 1, date: 1 })

  response.json(users.map(u => u.toJSON()))
});
```

<!-- Tulos on täsmälleen sellainen kuin haluamme -->
The result is now exactly like we want it to be:

![](../images/4/14.png)

<!-- Lisätään sopiva käyttäjän tietojen populointi muistiinpanojen yhteyteen: -->
Let's also add a suitable population of user information to notes:

```js
notesRouter.get('/', async (request, response) => {
  const notes = await Note
    .find({}).populate('user', { username: 1, name: 1 })

  response.json(notes.map(note => note.toJSON()))
});
```

<!-- Nyt käyttäjän tiedot tulevat muistiinpanon kenttään <i>user</i>. -->
Now the user's information is added to the <i>user</i> field of note objects.

![](../images/4/15.png)

<!-- Korostetaan vielä, että tietokannan tasolla ei siis ole mitään määrittelyä siitä, että esim. muistiinpanojen kenttään <i>user</i> talletetut id:t viittaavat käyttäjä-kokoelman dokumentteihin. -->
It's important to understand that the database does not actually know that the ids stored in the <i>user</i> field of notes reference documents in the user collection.

<!-- Mongoosen <i>populate</i>-funktion toiminnallisuus perustuu siihen, että olemme määritelleet viitteiden "tyypit" olioiden Mongoose-skeemaan <i>ref</i>-kentän avulla: -->
The functionality of the <i>populate</i> method of Mongoose is based on the fact that we have defined "types" to the references in the Mongoose schema with the <i>ref</i> option:

```js
const noteSchema = new mongoose.Schema({
  content: {
    type: String,
    required: true,
    minlength: 5
  },
  date: Date,
  important: Boolean,
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
})
```

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-7), branchissä <i>part4-7</i>. -->
You can find the code for our current application in it entirety in the <i>part4-7</i> branch of [this github repository](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-7).

</div>
