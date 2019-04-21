---
mainImage: ../../images/part-3.svg
part: 3
letter: c
---

<div class="content">

<!-- Ennen kuin siirrymmen osan varsinaiseen aiheeseen, eli tiedon tallettamiseen tietokantaan, tarkastellaan muutamaa tapaa Node-sovellusten debuggaamiseen. -->
Before we move into the main topic of persisting data in a database, we will take a look at a few different ways of debugging Node applications.

<!-- ### Node-sovellusten debuggaaminen -->
### Debugging Node applications

<!-- Nodella tehtyjen sovellusten debuggaaminen on jossain määrin hankalampaa kuin selaimessa toimivan Javascriptin. Vanha hyvä keino on tietysti konsoliin tulostelu. Se kannattaa aina. On mielipiteitä, joiden mukaan konsoliin tulostelun sijaan olisi syytä suosia jotain kehittyneempää menetelmää, mutta en ole ollenkaan samaa mieltä. Jopa maailman aivan eliittiin kuuluvat open source -kehittäjät [käyttävät](https://tenderlovemaking.com/2016/02/05/i-am-a-puts-debuggerer.html) tätä [menetelmää](https://swizec.com/blog/javascript-debugging-slightly-beyond-console-log/swizec/6633). -->
Debugging Node applications is slightly more difficult than debugging JavaScript running in your browser. Printing to the console is a tried and true method, and it's always worth doing. There are people who think that more sophisticated methods should be used instead, but I disagree. Even the world's elite open source developers [use](https://tenderlovemaking.com/2016/02/05/i-am-a-puts-debuggerer.html) this [method](https://swizec.com/blog/javascript-debugging-slightly-beyond-console-log/swizec/6633).

#### Visual Studio Code

<!-- Visual Studio Coden debuggeri voi olla hyödyksi joissain tapauksissa. Saat käynnistettyä sovelluksen debuggaustilassa seuraavasti -->
The Visual Studio Code debugger can be useful in some situations. You can launch the application in debugging mode like this:

![](../images/3/35.png)

<!-- Huomaa, että sovellus ei saa olla samalla käynnissä "normaalisti" konsolista, sillä tällöin sovelluksen käyttämä portti on varattu. -->
Notice that the application shouldn't be running in another console, otherwise the port will already be in use.

<!-- Seuraavassa screenshot, missä koodi on pysäytetty kesken uuden muistiinpanon lisäyksen -->
Below you can see a screenshot where the code execution has been paused in the middle of saving a new note:

![](../images/3/36a.png)

<!-- Koodi on pysähtynyt nuolen osoittaman <i>breakpointin</i> kohdalle ja konsoliin on evaluoitu muuttujan <i>note</i> arvo. Vasemmalla olevassa ikkunassa on nähtävillä myös muuta ohjelman tilaan liittyvää. -->
The execution has stopped at the <i>breakpoint</i> pointed to by the arrow. In the console you can see the value of the <i>note</i> variable. In the top left window you can see other things related to the state of the application.

<!-- Ylhäällä olevista nuolista yms. voidaan kontrolloida debuggauksen etenemistä. -->
The arrows at the top can be used for controlling the flow of the debugger.

<!-- Itse en jostain syystä juurikaan käytä Visual Studio Code debuggeria. -->
For some reason, I don't use the Visual Studio Code debugger a whole lot.

<!-- #### Chromen dev tools -->
#### Chrome dev tools

<!-- Debuggaus onnisuu myös Chromen developer-konsolilla, käynnistämällä sovellus komennolla: -->
Debugging is also possible with the Chrome developer console by starting your application with the command:

```bash
node --inspect index.js
```

<!-- Debuggeriin pääsee käsiksi klikkaamalla chromen devloper-konsoliin ilmestyneestä vihreästä ikonista -->
You can access the debugger by clicking the green icon that appears in the Chrome developer console:

![](../images/3/37.png)

<!-- Debuggausnäkymä toimii kuten React-koodia debugattaessa, <i>Sources</i>-välilehdelle voidaan esim. asettaa breakpointeja, eli kohtia joihin suoritus pysähtyy: -->
The debugging view works the same way as it did with React applications. The <i>Sources</i> tab can be used for setting breakpoints where the execution of the code will be paused.

![](../images/3/38.png)

<!-- Kaikki sovelluksen console.log-tulostukset tulevat debuggerin <i>Console</i>-välilehdelle. Voit myös tutkia siellä muuttujien arvoja ja suorittaa mielivaltaista Javascript-koodia: -->
All of the application's console.log messages will appear in the <i>Console</i> tab of the debugger. You can also inspect values of variables and execute your own JavaScript code.

![](../images/3/39.png)

<!-- #### Epäile kaikkea -->
#### Question everything

<!-- Full Stack -sovellusten debuggaaminen vaikuttaa alussa erittäin hankalalta. Kun kohta kuvaan tulee myös tietokanta ja frontend on yhdistetty backendiin, on potentiaalisia virhelähteitä todella paljon. -->
Debugging Full Stack applications may seem tricky at first. Soon our application will also have a database in addition to the frontend and backend, and there will be many potential areas for bugs in the application.

<!-- Kun sovellus "ei toimi", onkin selvitettävä missä vika on. On erittäin yleistä, että vika on sellaisessa paikassa, mitä ei osaa ollenkaan epäillä, ja menee minuutti-, tunti- tai jopa päiväkausia ennen kuin oikea ongelmien lähde löytyy. -->
When the application "does not work", we have to first figure out where the problem actually occurs. It's very common for the problem to exist in a place where you didn't expect it to, and it can take minutes, hours, or even days before you find the source of the problem.

<!-- Avainasemassa onkin systemaattisuus. Koska virhe voi olla melkein missä vaan, <i>kaikkea pitää epäillä</i>, ja tulee pyrkiä poissulkemaan ne osat tarkastelusta, missä virhe ei ainakaan ole. Konsoliin kirjoitus, Postman, debuggeri ja kokemus auttavat. -->
The key is to be systematic. Since the problem can exist anywhere, <i>you must question everything</i>, and eliminate all possibilities one by one. Logging to the console, Postman, debuggers, and experience will help.

<!-- Virheiden ilmaantuessa <i>ylivoimaisesti huonoin strategia</i> on jatkaa koodin kirjoittamista. Se on tae siitä, että koodissa on pian kymmenen ongelmaa lisää ja niiden syyn selvittäminen on entistäkin vaikeampaa. Toyota Production Systemin periaate [Stop and fix](http://gettingtolean.com/toyota-principle-5-build-culture-stopping-fix/#.Wjv9axP1WCQ) toimii tässäkin yhteydessä paremmin kuin hyvin. -->
When bugs occur, <i>the worst of all possible strategies</i> is to continue writing code. It will guarantee that your code will soon have ten more bugs, and debugging them will be even more difficult. The [stop and fix](http://gettingtolean.com/toyota-principle-5-build-culture-stopping-fix/#.Wjv9axP1WCQ) principle from Toyota Production Systems is very effective in this situation as well.

### MongoDB

<!-- Jotta saisimme talletettua muistiinpanot pysyvästi, tarvitsemme tietokannan. Useimmilla laitoksen kursseilla on käytetty relaatiotietokantoja. Tällä kurssilla käytämme [MongoDB](https://www.mongodb.com/):tä, joka on ns. [dokumenttitietokanta](https://en.wikipedia.org/wiki/Document-oriented_database). -->
In order to store our saved notes indefinitely, we need a database. Most of the courses taught at the University of Helsinki use relational databases. In this course we will use [MongoDB](https://www.mongodb.com/) which is a so-called [document database](https://en.wikipedia.org/wiki/Document-oriented_database).

<!-- Dokumenttitietokannat poikkeavat jossain määrin relaatiotietokannoista niin datan organisointitapansa kuin kyselykielensäkin suhteen. Dokumenttitietokantojen ajatellaan kuuluvan sateenvarjotermin [NoSQL](https://en.wikipedia.org/wiki/NoSQL) alle. Lisää dokumenttitietokannoista ja NoSQL:stä Tietokantojen perusteiden [viikon 7 materiaalista](https://tikape-s18.mooc.fi/part7/). -->
Document databases differ from relational databases in how they organize data as well as the query languages they support. Document databases are usually categorized under the [NoSQL](https://en.wikipedia.org/wiki/NoSQL) umbrella term. You can read more about document databases and NoSQL from the course material for [week 7](https://tikape-s18.mooc.fi/part7/) from the introduction to databases course. Unfortunately the material is currently only available in Finnish.

**Lue nyt Tietokantojen perusteiden dokumenttitietokantoja kuvaava osuus.** Jatkossa oletetaan, että hallitset käsitteet <i>dokumentti</i> ja <i>kokoelma</i> (collection).

<!-- MongoDB:n voi luonnollisesti asentaa omalle koneelle. Internetistä löytyy kuitenkin myös palveluna toimivia Mongoja, joista tämän hetken paras valinta on [MongoDB Atlas](https://www.mongodb.com/cloud/atlas). -->
Naturally, you can install and run MongoDB on your own computer. However, the internet is also full of Mongo database services that you can use. Our preferred MongoDB provider in this course will be [MongoDB Atlas](https://www.mongodb.com/cloud/atlas).

<!-- > <i>Jos ehdit jo konfiguroida itsellesi mlabin, löydät ohjeita sen käyttöön [viime vuoden materiaalista](https://fullstackopen.github.io/osa3/).</i> -->
<i>If you have already configured mlab, you can continue to use it with instructions from [last year's course material](https://fullstackopen.github.io/osa3/).</i>

<!-- Kun käyttäjätili on luotu ja kirjauduttu, Atlas kehoittaa luomaan klusterin: -->
Once you've created and logged into your account, Atlas will recommend creating a cluster:

![](../images/3/57.png)

<!-- Valitaan <i>AWS</i> ja <i>Frankfurt</i> ja luodaan klusteri. -->
Let's choose <i>AWS</i> and <i>Frankfurt</i> and create a cluster.

![](../images/3/58.png)

<!-- Odotetaan että klusteri on valmiina, tähän menee noin 10 minuuttia.  -->
Let's wait for the cluster to be ready for use. This will take approximately 10 minutes.

<!-- **HUOM** älä jatka eteenpäin ennen kun klusteri on valmis! -->
**NB** do not continue before the cluster is ready.

<!-- Luodaan <i>security</i> välilehdeltä tietokantakäyttäjätunnus joka on siis eri tunnus kuin se, jonka avulla kirjaudutaan MongoDB Atlasiin: -->
Let's use the <i>security</i> tab for creating user credentials for the database. Please note that these are not the same credentials you use for logging into MongoDB Atlas.

![](../images/3/59.png)

<!-- annetaan käyttäjälle luku- ja kirjoitustoikeus kaikkiin tietokantoihin -->
Let's grant the user with permissions to read and write to the databases.

![](../images/3/60.png)

<!-- **HUOM** muutamissa tapauksissa uusi käyttäjä ei ole toiminut heti luomisen jälkeen. On saattanut kestää jopa useita minuutteja ennen kuin käyttäjätunnus on ruvennut toimimaan. -->
**NB** for some people the new user credentials have not worked immediately after creation. In some cases it has taken minutes before the credentials have worked.

<!-- Seuraavaksi tulee määritellä ne ip-osoitteet, joista tietokantaan pääsee käsiksi -->
Next we have to define the IP addresses that are allowed access to the database.

![](../images/3/61.png)

<!-- Sallitaan yksinkertaisuuden vuoksi yhteydet kaikkialta: -->
For the sake of simplicity we will allow access from all IP addresses:

![](../images/3/62.png)

<!-- Lopultakin ollaan valmiina ottamaan tietokantayhteyden. Valitaan <i>Connect your application</i> ja <i>Short SRV connection string</i> -->
Finally we are ready to connect to our database. Let's choose <i>Connect your application</i> <i>Short SRV connection string</i>

![](../images/3/64.png)


<!-- Näkymä kertoo <i>MongoDB URI:n</i> eli osoitteen, jonka avulla sovelluksemme käyttämä MongoDB-kirjasto saa yhteyden kantaan. -->
The view displays the <i>MongoDB URI</i>, which is the address of the database that we will supply to the MongoDB client library we will add to our application.

<!-- Osoite näyttää seuraavalta: -->
The address looks like this:

```bash
mongodb+srv://fullstack:<PASSWORD>@cluster0-ostce.mongodb.net/test?retryWrites=true
```

<!-- Olemme nyt valmiina kannan käyttöön. -->
We are now ready to use the database.

<!-- Voisimme käyttää kantaa Javascript-koodista suoraan Mongon virallisen [MongoDB Node.js driver](https://mongodb.github.io/node-mongodb-native/) -kirjaston avulla, mutta se on ikävän työlästä. Käytämmekin hieman korkeammalla tasolla toimivaa [Mongoose](http://mongoosejs.com/index.html)-kirjastoa. -->
We could use the database directly from our JavaScript code with the [official MongoDb Node.js driver](https://mongodb.github.io/node-mongodb-native/) library, but it is quite cumbersome to use. We will instead use the [Mongoose](http://mongoosejs.com/index.html) library that offers a higher level API.

<!-- Mongoosesta voisi käyttää luonnehdintaa <i>object document mapper</i> (ODM), ja sen avulla Javascript-olioiden tallettaminen mongon dokumenteiksi on suoraviivaista. -->
Mongoose could be described as an <i>object document mapper</i> (ODM), and saving JavaScript objects as Mongo documents is straightforward with the library.

<!-- Asennetaan Mongoose: -->
Let's install Mongoose:

```bash
npm install mongoose --save
```

<!-- Ei lisätä mongoa käsittelevää koodia heti backendin koodin sekaan, vaan tehdään erillinen kokeilusovellus tiedostoon <i>mongo.js</i>: -->
Let's not add any code dealing with Mongo to our backend just yet. Instead, let's make a practice application into the file <i>mongo.js</i>:

```js
const mongoose = require('mongoose')

if ( process.argv.length<3 ) {
  console.log('give password as argument')
  process.exit(1)
}

const password = process.argv[2]

const url =
  `mongodb+srv://fullstack:${password}@cluster0-ostce.mongodb.net/test?retryWrites=true`

mongoose.connect(url, { useNewUrlParser: true })

const noteSchema = new mongoose.Schema({
  content: String,
  date: Date,
  important: Boolean,
})

const Note = mongoose.model('Note', noteSchema)

const note = new Note({
  content: 'HTML on helppoa',
  date: new Date(),
  important: true,
})

note.save().then(response => {
  console.log('note saved!');
  mongoose.connection.close();
})
```

<!-- Koodi siis olettaa, että sille annetan parametrina MongoDB Atlasissa luodulle käyttäjälle määritelty salasana. Komentoriviparametriin se pääsee käsiksi seuraavasti -->
The code assumes that it will be passed the password from the credentials we created in MongoDB Atlas as a command line parameter. We can access the command line parameter like this:

```js
const password = process.argv[2]
```

<!-- Kun koodi suoritetaan komennolla <i>node mongo.js salasana</i> lisää Mongoose tietokantaan uuden dokumentin. -->
When the code is run with the command <i>node mongo.js salasana</i>, Mongo will add a new document to the database.

<!-- Voimme tarkastella tietokannan tilaa MongoDB Atlasin hallintanäkymän <i>collections</i>-osasta -->
We can view the current state of the database from the MongoDB Atlas from <i>Collections</i>
in the Overview tab.

![](../images/3/65.png)

<!-- Kuten näkymä kertoo, on muistiinpanoa vastaava <i>dokumentti</i> lisätty tietokannan <i>test</i> kokoelmaan (collection) nimeltään <i>notes</i>. -->
As the view states, the <i>document</i> matching the note has been added to the <i>notes</i> collection in the <i>test</i> database.

![](../images/3/66a.png)

<!-- Tietokanta lienee loogisempaa nimetä paremmin. Kuten dokumentaatio sanoo, kontrolloidaan kannan nimeä tietokanta-URI:in perusteella -->
We should give a better name to the database. Like the documentation says, we can change the name of the database from the URI:

![](../images/3/67.png)

<!-- eli tuhotaan kanta <i>test</i>. Päätetään käyttää tietokannasta nimeä <i>note-app</i> muutetaan siis tietokanta-URI muotoon -->
Let's destroy the <i>test</i> database. Let's change the name of database to <i>note-app</i> instead, by modifying the URI:

```bash
mongodb+srv://fullstack:<PASSWORD>@cluster0-ostce.mongodb.net/note-app?retryWrites=true
```

<!-- Suoritetaan ohjelma uudelleen. -->
Let's run our code again.

![](../images/3/68.png)

<!-- Data on nyt oikeassa kannassa. Hallintanäkymä sisältää myös toiminnon <i>create database</i>, joka mahdollistaa uusien tietokantojenluomisen hallintanäkymän kautta. Kannan luominen etukäteen hallintanäkymässä ei kuitenkaan ole tarpeen, sillä MongoDB Atlasosaa luoda kannan automaattisesti jos sovellus yrittää yhdistää kantaan, jota ei ole vielä olemassa. -->
The data is now stored in the right database. The view also offers the <i>create database</i> functionality, that can be used to create new databases from the website. Creating the database like this is not necessary, since MongoDB Atlas automatically creates a new database when an application tries to connect to a database that does not exist yet.

<!-- ### Skeema -->
### Schema

<!-- Yhteyden avaamisen jälkeen määritellään mustiinpanon [skeema](http://mongoosejs.com/docs/guide.html) ja sitä vastaava [model](http://mongoosejs.com/docs/models.html): -->
After establishing the connection to the database, we define the [schema](http://mongoosejs.com/docs/guide.html) for a note and the matching [model](http://mongoosejs.com/docs/models.html):

```js
const noteSchema = new mongoose.Schema({
  content: String,
  date: Date,
  important: Boolean,
})

const Note = mongoose.model('Note', noteSchema)
```

<!-- Ensin muuttujaan _noteSchema_ määritellään muistiinpanon [skeema](http://mongoosejs.com/docs/guide.html), joka kertoo Mongooselle, miten muistiinpano-oliot tulee tallettaa tietokantaan. -->
First we define the [schema](http://mongoosejs.com/docs/guide.html) of a note that is stored in the _noteSchema_ variable. The schema tells Mongoose how the note objects are to be stored in the database.

<!-- Modelin _Note_ määrittelyssä ensimmäisenä parametrina oleva merkkijono <i>Note</i> määrittelee, että mongoose tallettaa muistiinpanoa vastaavat oliot kokoelmaan nimeltään <i>notes</i>, sillä [Mongoosen konventiona](http://mongoosejs.com/docs/models.html) on määritellä kokoelmien nimet monikossa (esim. <i>notes</i>), kun niihin viitataan skeeman määrittelyssä yksikkömuodossa (esim. <i>Note</i>). -->
In the _Note_ model definition, the first <i>'Note'</i> parameter is the singular name of the model. The name of collection will the lowercased plural <i>notes</i>, because the [Mongoose convention](http://mongoosejs.com/docs/models.html) is to automatically name collections as the plural (e.g. <i>notes</i>) when the schema refers to them in the singular (e.g. <i>Note</i>).

<!-- Dokumenttikannat, kuten Mongo ovat <i>skeemattomia</i>, eli tietokanta itsessään ei välitä mitään sinne talletettavan tiedon muodosta. Samaan kokoelmaankin on mahdollista tallettaa olioita joilla on täysin eri kentät. -->
Document databases like Mongo are <i>schemaless</i>, meaning that the database itself does not  care about the structure of the data that is stored in the database. It is possible to store documents with completely different fields in the same collection.

<!-- Mongoosea käytettäessä periaatteena on kuitenkin se, että tietokantaan talletettavalle tiedolle määritellään <i>sovelluksen koodin tasolla skeema</i>, joka määrittelee minkä muotoisia olioita kannan eri kokoelmiin talletetaan. -->
The idea behind Mongoose is that the data stored in the database is given a <i>schema at the level of the application</i> that defines the shape of the documents stored in any given collection.

<!-- ### Olioiden luominen ja tallettaminen -->
### Creating and saving objects

<!-- Seuraavaksi sovellus luo muistiinpanoa vastaavan [model](http://mongoosejs.com/docs/models.html):in avulla muistiinpano-olion: -->
Next, the application creates a new note object with the help of the <i>Note</i> [model](http://mongoosejs.com/docs/models.html):

```js
const note = new Note({
  content: 'Selain pystyy suorittamaan vain javascriptiä',
  date: new Date(),
  important: false,
})
```

<!-- Modelit ovat ns. <i>konstruktorifunktioita</i>, jotka luovat parametrien perusteella Javascript-olioita. Koska oliot on luotu modelien konstruktorifunktiolla, niillä on kaikki modelien ominaisuudet, eli joukko metodeja, joiden avulla olioita voidaan mm. tallettaa tietokantaan. -->
Models are so-called <i>constructor functions</i> that create new JavaScript objects based on the provided parameters. Since the objects are created with the model's constructor function, they have all the properties of the model, which include methods for saving the object to the database.

<!-- Tallettaminen tapahtuu metodilla _save_. Metodi palauttaa <i>promisen</i>, jolle voidaan rekisteröidä _then_-metodin avulla tapahtumankäsittelijä: -->
Saving the object to the database happens with the appropriately named _save_ method, that can be provided with an event handler with the _then_ method:

```js
note.save().then(result => {
  console.log('note saved!')
  mongoose.connection.close()
})
```

<!-- Kun olio on tallennettu kantaan, kutsutaan _then_:in parametrina olevaa tapahtumankäsittelijää, joka sulkee tietokantayhteyden komennolla <code>mongoose.connection.close()</code>. Ilman yhteyden sulkemista ohjelman suoritus ei pääty. -->
When the object is saved to the database, the event handler provided to _then_  gets called. The event handler closes the database connection with the command <code>mongoose.connection.close()</code>. If the connection is not closed, the program will never finish its execution.

<!-- Tallennusoperaation tulos on takaisinkutsun parametrissa _result_. Yhtä olioa tallentaessamme tulos ei ole kovin mielenkiintoinen, olion sisällön voi esim. tulostaa konsoliin jos haluaa tutkia sitä tarkemmin sovelluslogiikassa tai esim. debugatessa. -->
The result of the save operation is in the _result_ parameter of the event handler. The result is not that interesting when we're storing one object to the database. You can print the object to the console if you want to take a closer look at it while implementing your application or during debugging.

<!-- Talletetaan kantaan myös pari muuta muistiinpanoa muokkaamalla dataa koodista ja suorittamalla ohjelma uudelleen. -->
Let's also save a few more notes by modifying the data in the code and by executing the program again.

<!-- **HUOM** valitettavasti Mongoosen dokumentaatiossa käytetään joka paikassa takaisinkutsufunktioita, joten sieltä ei kannata suoraan copypasteta koodia, sillä promisejen ja vanhanaikaisten callbackien sotkeminen samaan koodiin ei ole kovin järkevää. -->
**NB** unfortunately the Mongoose documentation uses callbacks in its examples, so it is not recommended to copy paste code directly from there. Mixing promises with old-school callbacks in the same code is not recommended. 

<!-- ### Olioiden hakeminen tietokannasta -->
### Fetching objects from the database

<!-- Kommentoidaan koodista uusia muistiinpanoja generoiva osa, ja korvataan se seuraavalla: -->
Let's comment out the code for generating new notes and replace it with the following:

```js
Note.find({}).then(result => {
  result.forEach(note => {
    console.log(note)
  })
  mongoose.connection.close()
})
```

<!-- Kun koodi suoritetaan, kantaan talletetut muistiinpanot tulostuvat. -->
When the code gets executed, all of the notes stored in the database get printed.

<!-- Oliot haetaan kannasta _Note_-modelin metodilla [find](http://mongoosejs.com/docs/api.html#find_find). Metodin parametrina on hakuehto. Koska hakuehtona on tyhjä olio <code>{}</code>, saimme kannasta kaikki _notes_-kokoelmaan talletetut oliot. -->
The objects are retrieved from the database with the [find](http://mongoosejs.com/docs/api.html#find_find) method of the _Note_ model. The parameter of the method is an object expressing search conditions. Since the parameter is an empty object<code>{}</code>, we get all of the notes stored in the  _notes_ collection.

<!-- Hakuehdot noudattavat mongon [syntaksia](https://docs.mongodb.com/manual/reference/operator/). -->
The search conditions adhere to the Mongo search query [syntax](https://docs.mongodb.com/manual/reference/operator/).

<!-- Voisimme hakea esim. ainoastaan tärkeät muistiinpanot seuraavasti: -->
We could restrict our search to only include important notes like this:

```js
Note.find({ important: true }).then(result => {
  // ...
})
```

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- #### 3.12: tietokanta komentoriviltä -->
#### 3.12: Command-line database

<!-- Luo puhelinluettelo-sovellukselle pilvessä oleva mongo Mongo DB Atlaksen avulla. -->
Create a cloud-based MongoDB database for the phonebook application with MongoDB Atlas. 

<!-- Tee projektihakemistoon tiedosto <i>mongo.js</i>, jonka avulla voit lisätä tietokantaan puhelinnumeroja sekä listata kaikki kannassa olevat numerot. -->
Create a <i>mongo.js</i> file in the project directory, that can be used for adding entries to the phonebook, and for listing all of the existing entries in the phonebook.

<!-- **Huom** jos/kun laitat tiedoston Githubiin, älä laita tietokannan salasanaa mukaan! -->
**NB** Do not include the password in the file that you commit and push to GitHub! 

<!-- Ohjelma toimii siten, että jos sille annetaan käynnistäessä kolme komentoriviparametria (joista ensimmäinen on salasana), esim: -->
The application should work as follows. You use the program by passing three command-line arguments (the first is the password), e.g.:

```bash
node mongo.js salasana Joulupukki 040-1234556
```

<!-- Ohjelma tulostaa -->
As a result, the application will print:

```bash
lisätään Joulupukki numero 040-1234556 luetteloon
```

<!-- ja lisää uuden yhteystiedon tietokantaan. Huomaa, että jos nimi sisältää välilyöntejä, on se annettava hipsuissa: -->
The new entry to the phonebook will be saved to the database. Notice that if the name contains whitespace characters, it must be specified in quotes:

```bash
node mongo.js salasana "Arto Vihavainen" 040-1234556
```

<!-- Jos komentoriviparametreina ei ole muuta kuin salasana, eli ohjelma suoritetaan komennolla -->
If the password is the only parameter given to the program, meaning that it is invoked like this:

```bash
node mongo.js salasana
```

<!-- tulostaa ohjelma tietokannassa olevat numerotiedot: -->
Then the program should display all of the entries in the phonebook:

<pre>
puhelinluettelo:
Pekka Mikkola 040-1234556
Arto Vihavainen 045-1232456
Tiina Niklander 040-1231236
</pre>

<!-- Saat selville ohjelman komentoriviparametrit muuttujasta [process.argv](https://nodejs.org/docs/latest-v8.x/api/process.html#process_process_argv) -->
You can get the command-line parameters from the [process.argv](https://nodejs.org/docs/latest-v8.x/api/process.html#process_process_argv) variable.

<!-- **HUOM: älä sulje tietokantayhteyttä väärässä kohdassa**. Esim. seuraava koodi ei toimi -->
**NB: do not close the connection in the wrong place**. E.g. the following code will not work:

```js
Person
  .find({})
  .then(persons=> {
    // ...
  })

mongoose.connection.close()
```

<!-- Koodin suoritus nimittäin etenee siten, että heti operaation <i>Person.find</i> käynnistymisen jälkeen suoritetaan komento <i>mongoose.connection.close()</i> ja tietokantayhteys katkeaa välittömästi. Näin ei koskaan päästä siihen pisteeseen, että <i>Person.find</i>-operaation valmistumisen käsittelevää <i>takaisinkutsufunktiota</i> kutsuttaisiin. -->
In the code above the <i>mongoose.connection.close()</i> command will get executed immediately after the <i>Person.find</i> operation gets started. This means that the database connection will be closed immediately, and the execution will never get to the point where <i>Person.find</i> operation finishes and the <i>callback</i> function gets called.

<!-- Oikea paikka tietokantayhteyden sulkemiselle on takaisinkutsufunktion loppu: -->
The correct place for closing the database connection is at the end of the callback function:

```js
Person
  .find({})
  .then(persons=> {
    // ...
    mongoose.connection.close()
  })
```

<!-- **HUOM2** jos määrittelet modelin nimeksi <i>Person</i>, muuttaa mongoose sen monikkomuotoon <i>people</i>, jota se käyttää vastaavan kokoelman nimenä. -->
**NB2** if you define a model with the name <i>Person</i>, mongoose will automatically name the associated collection as <i>people</i>.

</div>

<div class="content">

<!-- ### Tietokantaa käyttävä backend -->
### Backend connected to a database

<!-- Nyt meillä on periaatteessa hallussamme riittävä tietämys ottaa mongo käyttöön sovelluksessamme. -->
Now we have enough knowledge to start using Mongo in our application.

<!-- Aloitetaan nopean kaavan mukaan, copypastetaan tiedostoon <i>index.js</i> Mongoosen määrittelyt, eli -->
Let's get a quick start by copy pasting the Mongoose definitions to the <i>index.js</i> file:

```js
const mongoose = require('mongoose')

// ÄLÄ KOSKAAN TALLETA SALASANOJA githubiin!
const url =
  'mongodb+srv://fullstack:sekred@cluster0-ostce.mongodb.net/note-app?retryWrites=true'

mongoose.connect(url, { useNewUrlParser: true })

const noteSchema = new mongoose.Schema({
  content: String,
  date: Date,
  important: Boolean,
})

const Note = mongoose.model('Note', noteSchema)
```

<!-- ja muutetaan kaikkien muistiinpanojen hakemisesta vastaava käsittelijä seuraavaan muotoon -->
Let's change the handler for fetching all notes into the following form:

```js
app.get('/api/notes', (request, response) => {
  Note.find({}).then(notes => {
    response.json(notes)
  })
})
```

<!-- Voimme todeta selaimella, että backend toimii kaikkien dokumenttien näyttämisen osalta: -->
We can verify in the browser that the backend works for displaying all of the documents:

![](../images/3/44.png)

<!-- Toiminnallisuus on muuten kunnossa, mutta frontend olettaa, että olioiden yksikäsitteinen tunniste on kentässä <i>id</i>. Emme myöskään halua näyttää frontendille mongon versiointiin käyttämää kenttää <i>\_\_v</i>.  -->
The application works almost perfectly. The frontend assumes that every object has a unique id in the <i>id</i> field. We also don't want to return the mongo versioning field <i>\_\_v</i> to the frontend.

<!-- Eräs tapa muotoilla Mongoosen palauttamat oliot haluttuun muotoon on [muokata](https://stackoverflow.com/questions/7034848/mongodb-output-id-instead-of-id) kannasta haettavilla olioilla olevan _toJSON_-metodin palauttamaa muotoa. Metodin muokkaus onnistuu seuraavasti: -->
One way to format the objects returned by Mongoose is to [modify](https://stackoverflow.com/questions/7034848/mongodb-output-id-instead-of-id) the _toJSON_ method of the objects. Modifying the method happens like this:

```js
noteSchema.set('toJSON', {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString()
    delete returnedObject._id
    delete returnedObject.__v
  }
})
```

<!-- Vaikka Mongoose-olioiden kenttä <i>\_id</i> näyttääkin merkkijonolta, se on todellisuudessa olio. Määrittelemämme metodi _toJSON_ muuttaa sen merkkijonoksi kaiken varalta. Jos emme tekisi muutosta, siitä aiheutuisi ylimääräistä harmia testien yhteydessä. -->
Even though the <i>\_id</i> property of Mongoose objects looks like a string, it is in fact an object. The _toJSON_ method we defined transforms it into a string just to be safe. If we didn't make this change, it would cause more harm for us in the future once we start writing tests.

<!-- Palautetaan HTTP-pyynnön vastauksena _toJSON_-metodin avulla muotoiltuja oliota: -->
Let's respond to the HTTP request with a list of objects formatted with the _toJSON_ method:

```js
app.get('/api/notes', (request, response) => {
  Note.find({}).then(notes => {
    response.json(notes.map(note => note.toJSON()))
  });
});
```

<!-- Nyt siis muuttujassa _notes_ on taulukollinen mongon palauttamia olioita. Kun suoritamme operaation <em>notes.map(note => note.toJSON())</em> seurauksena on uusi taulukko, missä on jokaista alkuperäisen taulukon alkiota vastaava metodin _toJSON_ avulla muodostettu alkio. -->
Now the _notes_ variable is assigned to an array of objects returned by Mongo. When we call <em>notes.map(note => note.toJSON())</em> the result is a new array, where every item from the old one is mapped to a new object with the _toJSON_ method.

<!-- ### Tietokantamäärittelyjen eriyttäminen moduuliksi -->
### Database configuration into its own module

<!-- Ennen kuin täydennämme backendin muutkin osat käyttämään tietokantaa, eriytetään Mongoose-spesifinen koodi omaan moduuliin. -->
Before we refactor the rest of the backend to use the database, let's extract the Mongoose specific code into its own module.

<!-- Tehdään moduulia varten hakemisto <i>models</i> ja sinne tiedosto <i>note.js</i>: -->
Let's create a new directory for the module called <i>models</i>, and add a file called <i>note.js</i>:

```js
const mongoose = require('mongoose')

const url = process.env.MONGODB_URI // highlight-line

console.log('connecting to', url) // highlight-line

mongoose.connect(url, { useNewUrlParser: true })
// highlight-start
  .then(result => {
    console.log('connected to MongoDB')
  })
  .catch((error) => {
    console.log('error connecting to MongoDB:', error.message)
  })
// highlight-end

const noteSchema = new mongoose.Schema({
  content: String,
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

module.exports = mongoose.model('Note', noteSchema) // highlight-line
```

<!-- Noden [moduulien](https://nodejs.org/docs/latest-v8.x/api/modules.html) määrittely poikkeaa hiukan osassa 2 määrittelemistämme frontendin käyttämistä [ES6-moduuleista](/osa2/kokoelmien_renderointi_ja_moduulit#refaktorointia-moduulit). -->
Defining Node [modules](https://nodejs.org/docs/latest-v8.x/api/modules.html) differs slightly from the way pf defining [ES6 modules](/osa2/kokoelmien_renderointi_ja_moduulit#refaktorointia-moduulit) in part 2.

<!-- Moduulin ulos näkyvä osa määritellään asettamalla arvo muuttujalle _module.exports_. Asetamme arvoksi modelin <i>Note</i>. Muut moduulin sisällä määritellyt asiat, esim. muuttujat _mongoose_ ja _url_ eivät näy moduulin käyttäjälle. -->
The public interface of the module is defined by setting a value to the _module.exports_ variable. We will set the value to be the <i>Note</i> model. The other things defined inside of the module, like the variables _mongoose_ and _url_ will not be accessible or visible to users of the module.

<!-- Moduulin käyttöönotto tapahtuu lisäämällä tiedostoon <i>index.js</i> seuraava rivi -->
Importing the module happens by adding the following line to <i>index.js</i>:

```js
const Note = require('./models/note')
```

<!-- Näin muuttuja _Note_ saa arvokseen saman olion, jonka moduuli määrittelee. -->
This way the _Note_ variable will be assigned to the same object that the module defines.

<!-- Yhteyden muodostustavassa on pieni muutos aiempaan: -->
The way that the connection is made has changed slightly:

```js
const url = process.env.MONGODB_URI

console.log('connecting to', url)

mongoose.connect(url, { useNewUrlParser: true })
  .then(result => {
    console.log('connected to MongoDB')
  })
  .catch((error) => {
    console.log('error connecting to MongoDB:', error.message)
  })
```

<!-- Tietokannan osoitetta ei kannata kirjoittaa koodiin, joten osoite annetaan sovellukselle ympäristömuuttujan <em>MONGODB_URI</em> välityksellä.  -->
It's not a good idea to hardcode the address of the database into the code, so instead the address of the database is passed to the application via the <em>MONGODB_URI</em> environment variable.

<!-- Yhteyden muodostavalle metodille on nyt rekisteröity onnistuneen ja epäonnistuneen yhteydenmuodostuksen käsittelevät funktiot, jotka tulostavat konsoliin tiedon siitä, onnistuuko yhteyden muodostaminen: -->
The method for establishing the connection is now given functions for dealing with a successful and unsuccessful connection attempt. Both functions just log a message to the console about the success status:

![](../images/3/45.png)

<!-- On useita tapoja määritellä ympäristömuuttujan arvo, voimme esim. antaa sen ohjelman käynnistyksen yhteydessä seuraavasti -->
There are many ways to define the value of an environment variable. One way would be to define it when the application is started:

```bash
MONGODB_URI=osoite_tahan npm run watch
```

<!-- Eräs kehittyneempi tapa on käyttää [dotenv](https://github.com/motdotla/dotenv#readme)-kirjastoa. Asennetaan kirjasto komennolla -->
A more sophisticated way is to use the [dotenv](https://github.com/motdotla/dotenv#readme) library. You can install the library with the command:

```bash
npm install dotenv --save
```

<!-- Sovelluksen juurihakemistoon tehdään sitten tiedosto nimeltään <i>.env</i>, minne tarvittavien ympäristömuuttujien arvot määritellään. Tiedosto näyttää seuraavalta -->
To use the library, we create a <i>.env</i> file at the root of the project. The environment variables are defined inside of the file, and it can look like this:

```bash
MONGODB_URI=mongodb+srv://fullstack:sekred@cluster0-ostce.mongodb.net/note-app?retryWrites=true
PORT=3001
```

<!-- Määrittelimme samalla aiemmin kovakoodaamamme sovelluksen käyttämän portin eli ympäristömuuttujan <em>PORT</em>. -->
We also added the hardcoded port of the server into the <em>PORT</em> environment variable.

<!-- **Tiedosto <i>.env</i> tulee heti gitignorata sillä emme halua julkaista tiedoston sisältöä verkkoon!** -->
**The <i>.env</i> file should be gitignored right away, since we do not want to publish any confidential information publicly online!**

<!-- dotenvissä määritellyt ympäristömuuttujat otetaan koodissa käyttöön komennolla <em>require('dotenv').config()</em> ja niihin viitataan Nodessa kuten "normaaleihin" ympäristömuuttujiin syntaksilla <em>process.env.MONGODB_URI</em>. -->
The enviroment variables defined in the dotenv file can be taken into use with the command <em>require('dotenv').config()</em> and you can reference them in your code just like you would reference normal environment variables, with the familiar <em>process.env.MONGODB_URI</em> syntax.

<!-- Muutetaan nyt tiedostoa <i>index.js</i> seuraavasti -->
Let's change the <i>index.js</i> file in the following way:

```js
require('dotenv').config() // highlight-line
const express = require('express')
const bodyParser = require('body-parser') 
const app = express()
const Note = require('./models/note') // highlight-line

// ..

const PORT = process.env.PORT // highlight-line
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`)
})
```

<!-- On tärkeää, että <i>dotenv</i> otetaan käyttöön ennen modelin <i>note</i> importtaamista, tällöin varmistutaan siitä, että tiedostossa <i>.env</i> olevat ympäristömuuttujat ovat alustettuja kun moduulin koodia importoidaan. -->
It's important that <i>dotenv</i> gets imported before the <i>note</i> model is imported. This ensures that the environment variables from the <i>.env</i> file are available globally before the code from the other modules are imported.

<!-- ### Muut operaatiot -->
### Other operations

<!-- Muutetaan nyt kaikki operaatiot tietokantaa käyttävään muotoon. -->
Next, let's change the rest of the backend functionality to use the database.

<!-- Uuden muistiinpanon luominen tapahtuu seuraavasti: -->
Creating a new note is accomplished like this:

```js
app.post('/api/notes', (request, response) => {
  const body = request.body

  if (body.content === undefined) {
    return response.status(400).json({ error: 'content missing' })
  }

  const note = new Note({
    content: body.content,
    important: body.important || false,
    date: new Date(),
  })

  note.save().then(savedNote => {
    response.json(savedNote.toJSON())
  })
})
```

<!-- Muistiinpano-oliot siis luodaan _Note_-konstruktorifunktiolla. Pyyntöön vastataan _save_-operaation takaisinkutsufunktion sisällä. Näin varmistutaan, että operaation vastaus tapahtuu vain jos operaatio on onnistunut. Palaamme virheiden käsittelyyn myöhemmin. -->
The note objects are created with the _Note_ constructor function. The response for the request is sent inside of the callback function for the _save_ operation. This ensures that the response is sent only if the operation succeeded. We will discuss error handling a little bit later.

<!-- Takaisinkutsufunktion parametrina oleva _savedNote_ on talletettu muistiinpano. HTTP-pyyntöön palautetaan kuitenkin siitä metodilla _toJSON_formatoitu muoto: -->
The _savedNote_ parameter in the callback function is the saved and newly created note. The data sent back in the response is the formatted version created with the _toJSON_ method:

```js
response.json(savedNote.toJSON())
```

<!-- Yksittäisen muistiinpanon tarkastelu muuttuu muotoon -->
Fetching an individual note gets changed into the following:

```js
app.get('/api/notes/:id', (request, response) => {
  Note.findById(request.params.id).then(note => {
    response.json(note.toJSON())
  })
})
```

<!-- ### Frontendin ja backendin yhteistoiminnallisuuden varmistaminen -->
### Verifying frontend and backend integration

<!-- Kun backendia laajennetaan, kannattaa sitä testailla aluksi **ehdottomasti selaimella, postmanilla tai VS Coden REST clientillä**. Seuraavassa kokeillaan uuden muistiinpanon luomista tietokannan käyttöönoton jälkeen: -->
When the backend gets expanded, it's a good idea to test the backend first with **the browser, Postman or the VS Code REST client**. Next, let's try creating a new note after taking the database into use:

![](../images/3/46.png)

<!-- Vasta kun kaikki on todettu toimivaksi, kannattaa siirtyä testailemaan että muutosten jälkeinen backend toimii yhdessä myös frontendin kanssa. Kaikkien kokeilujen tekeminen ainoastaan frontendin kautta on todennäköisesti varsin tehotonta. -->
Only once everything has been verified to work in the backend, is it a good idea to test that the frontend works with the backend. It is highly inefficient to test things exclusively through the frontend.

<!-- Todennäköisesti voi olla kannattavaa edetä frontin ja backin integroinnissa toiminnallisuus kerrallaan, eli ensin voidaan toteuttaa esim. kaikkien muistiinpanojen näyttäminen backendiin ja testata että toiminnallisuus toimii selaimella. Tämän jälkeen varmistetaan, että frontend toimii yhteen muutetun backendin kanssa. Kun kaikki on todettu olevan kunnossa, siirrytään seuraavan ominaisuuden toteuttamiseen. -->
It's probably a good idea to integrate the frontend and backend one functionality at a time. First we could implement fetching all of the notes from the database and test it through the backend endpoint in the browser. After this we could verify that the frontend works with the new backend. Once everything seems to work, we would move onto the next feature.

<!-- Kun kuvioissa on mukana tietokanta, on tietokannan tilan tarkastelu MongoDB Atlasin hallintanäkymästä varsin hyödyllistä, usein myös suoraan tietokantaa käyttävät Node-apuohjelmat, kuten tiedostoon <i>mongo.js</i> kirjoittamamme koodi auttavat sovelluskehityksen edetessä. -->
Once we introduce a database into the mix, it is useful to inspect the state persisted in the database, e.g. from the control panel in MongoDB Atlas. Quite often little Node helper programs like the <i>mongo.js</i> program we wrote earlier can be very helpful during development.

<!-- Sovelluksen tämän hetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-3), branchissa <i>part3-3</i>. -->
You can find the code for our current application in its entirety in the <i>part3-3</i> branch of [this github repository](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-3).

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- Seuraavat tehtävät saattavat olla melko suoraviivaisia, tosin jos frontend-koodissasi sattuu olemaan bugeja tai epäyhteensopivuutta backendin kanssa, voi seurauksena olla myös mielenkiintoisia bugeja. -->
The following exercises are pretty straightforward, but if your frontend stops working with the backend, then finding and fixing the bugs can be quite interesting. 

<!-- #### 3.13: puhelinluettelo ja tietokanta, step1 -->
#### 3.13: Phonebook database, step1

<!-- Muuta backendin kaikkien puhelintietojen näyttämistä siten, että se <i>hakee näytettävät puhelintiedot tietokannasta</i>. -->
Change the fetching of all phonebook entries so that the data is <i>fetched from the database</i>.

<!-- Varmista, että frontend toimii muutosten jälkeen. -->
Verify that the frontend works after the changes have been made.

<!-- Tee tässä ja seuraavissa tehtävissä Mongoose-spesifinen koodi omaan moduuliin samaan tapaan kuin luvussa [Tietokantamäärittelyjen eriyttäminen moduuliksi](/osa3/tietojen_tallettaminen_mongo_db_tietokantaan#tietokantamaarittelyjen-eriyttaminen-moduuliksi). -->
In the following exercises, write all Mongoose-specific code into its own module, just like we did in the chapter [Database configuration into its own module](/osa3/tietojen_tallettaminen_mongo_db_tietokantaan#tietokantamaarittelyjen-eriyttaminen-moduuliksi)

<!-- #### 3.14: puhelinluettelo ja tietokanta, step2 -->
#### 3.13: Phonebook database, step2

<!-- Muuta backendiä siten, että uudet numerot <i>tallennetaan tietokantaan</i>. Varmista, että frontend toimii muutosten jälkeen. -->
Change the backend so that new numbers are <i>saved to the database</i>. Verify that your frontend still works after the changes.

<!-- <i>**Tässä vaiheessa voit olla välittämättä siitä, onko tietokannassa jo henkilöä jolla on sama nimi kuin lisättävällä.**</i> -->
At this point you can choose to simply allow users to create all phonebook entries. At this stage, the phonebook can have multiple entries for a person with the same name. 

</div>

<div class="content">

<!-- ### Virheiden käsittely -->
### Error handling

<!-- Jos yritämme mennä selaimella sellaisen yksittäisen muistiinpanon sivulle, jota ei ole olemassa, eli esim. urliin <http://localhost:3001/api/notes/5c41c90e84d891c15dfa3431> missä <i>5a3b80015b6ec6f1bdf68d</i> ei ole minkään tietokannassa olevan muistiinpanon tunniste, jää selain "jumiin" sillä palvelin ei vastaa pyyntöön koskaan. -->
If we try to visit the URL of a note with an id that does not actually exist e.g. <http://localhost:3001/api/notes/5c41c90e84d891c15dfa3431> where <i>5a3b80015b6ec6f1bdf68d</i> is not an id stored in the database, then the browser will simply get "stuck" since the server never responds to the request.

<!-- Palvelimen konsolissa näkyykin virheilmoitus: -->
We can see the following error message appear in the logs for the backend:

![](../images/3/47.png)

<!-- Kysely on epäonnistunut ja kyselyä vastaava promise mennyt tilaan <i>rejected</i>. Koska emme käsittele promisen epäonnistumista, ei pyyntöön vastata koskaan. Osassa 2 tutustuimme jo [promisejen virhetilanteiden käsittelyyn](/osa2/palvelimella_olevan_datan_muokkaaminen#promise-ja-virheet). -->
The request has failed and the associated Promise has been <i>rejected</i>. Since we don't handle the rejection of the promise, the request never gets a response. In part 2 we already acquainted ourselves [handling errors in promises](/osa2/palvelimella_olevan_datan_muokkaaminen#promise-ja-virheet).

<!-- Lisätään tilanteeseen yksinkertainen virheidenkäsittelijä: -->
Let's add a simple error handler:

```js
app.get('/api/notes/:id', (request, response) => {
  Note.findById(request.params.id)
    .then(note => {
      response.json(note.toJSON())
    })
    .catch(error => {
      console.log(error);
      response.status(404).end()
    })
})
```

<!-- Kaikissa virheeseen päättyvissä tilanteissa HTTP-pyyntöön vastataan statuskoodilla 404 not found. Konsoliin tulostetaan tarkempi tieto virheestä. -->
Every request that leads to an error will be responded to with the HTTP status code 404 not found. The console displays more detailed information about the error.

<!-- Tapauksessamme on itseasiassa olemassa kaksi erityyppistä virhetilannetta. Toinen vastaa sitä, että yritetään hakea muistiinpanoa virheellisen muotoisella _id_:llä, eli sellasiella mikä ei vastaa mongon id:iden muotoa. -->
There's actually two different types of error situations. In one of the situations we are trying to fetch a note with a wrong kind of _id_, meaning an _id_ that doesn't match the mongo identifier format.

<!-- Jos teemme näin tulostuu konsoliin: -->
If we make the following request we will get the error message shown below:

<pre>
Method: GET
Path:   /api/notes/5a3b7c3c31d61cb9f8a0343
Body:   {}
---
{ CastError: Cast to ObjectId failed for value "5a3b7c3c31d61cb9f8a0343" at path "_id"
    at CastError (/Users/mluukkai/opetus/_fullstack/osa3-muisiinpanot/node_modules/mongoose/lib/error/cast.js:27:11)
    at ObjectId.cast (/Users/mluukkai/opetus/_fullstack/osa3-muisiinpanot/node_modules/mongoose/lib/schema/objectid.js:158:13)
    ...
</pre>

<!-- Toinen virhetilanne taas vastaa tilannetta, missä haettavan muistiinpanon id on periaatteessa oikeassa formaatissa, mutta tietokannasta ei löydy indeksillä mitään: -->
The other error situation is related a situation where the id is in the correct format, but no note is found from the database for that id.

<pre>
Method: GET
Path:   /api/notes/5a3b7c3c31d61cbd9f8a0343
Body:   {}
---
TypeError: Cannot read property 'toJSON' of null
    at Note.findById.then.note (/Users/mluukkai/opetus/_2019fullstack-koodit/osa3/notes-backend/index.js:27:24)
    at process._tickCallback (internal/process/next_tick.js:178:7)
</pre>

<!-- Nämä tilanteet on syytä erottaa toisistaan, ja itseasiassa jälkimmäinen poikkeus on oman koodimme aiheuttama. -->
We should distinguish between these two different types of error situations. The latter is in fact an error caused by our own code.

<!-- Muutetaan koodia seuraavasti: -->
Let's change the code in the following way:

```js
app.get('/api/notes/:id', (request, response) => {
  Note.findById(request.params.id)
    .then(note => {
      // highlight-start
      if (note) {
        response.json(note.toJSON())
      } else {
        response.status(404).end() 
      }
      // highlight-end
    })
    .catch(error => {
      console.log(error)
      response.status(400).send({ error: 'malformatted id' }) // highlight-line
    })
})
```

<!-- Jos kannasta ei löydy haettua olioa, muuttujan _note_ arvo on _undefined_ ja koodi ajautuu _else_-haaraan. Siellä vastataan kyselyyn <i>404 not found_</i> -->
If no matching object is found in the database, the value of _note_ will be undefined and the _else_ block gets executed. This results in a response with the status code <i>404 not found</i>.

<!-- Jos id ei ole hyväksyttävässä muodossa, ajaudutaan _catch_:in avulla määriteltyyn virheidenkäsittelijään. Sopiva statuskoodi on [400 bad request](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.1) koska kyse on juuri siitä: -->
If the format of the id is incorrect, then we will end up in the error handler defined in the _catch_ block. The appropriate status code for the situation is [400 bad request](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.1), because the situation fits the description perfectly:

> <i>The request could not be understood by the server due to malformed syntax. The client SHOULD NOT repeat the request without modifications.</i>

<!-- Vastaukseen on lisätty myös hieman dataa kertomaan virheen syystä. -->
We have also added some data to the response to shed some light on the cause of the error.

<!-- Promisejen yhteydessä kannattaa melkeinpä aina lisätä koodiin myös virhetilainteiden käsittely, muuten seurauksena on usein hämmentäviä vikoja. -->
When dealing with Promises it's almost always a good idea to add error and exception handling, because otherwise you will find yourself dealing with strange bugs.

<!-- Ei ole koskaan huono idea tulostaa poikkeuksen aiheuttanutta olioa konsoliin virheenkäsittelijässä: -->
It's never a bad idea to print the object that caused the exception to the console in the error handler:

```js
.catch(error => {
  console.log(error)
  response.status(400).send({ error: 'malformatted id' })
})
```

<!-- Virheenkäsittelijään joutumisen syy voi olla joku ihan muu kuin mitä on tullut alunperin ajatelleeksi. Jos virheen tulostaa konsoliin, voi säästyä pitkiltä ja turhauttavilta väärää asiaa debuggaavilta sessioilta. -->
The reason the error handler gets called might be something completely different than what you had anticipated. If you log the error to the console, you may save yourself from long and frustrating debugging sessions.

<!-- Aina kun ohjelmoit ja projektissa on mukana backend <i>**tulee ehdottomasti koko ajan pitää silmällä backendin konsolin tulostuksia**</i>. Jos työskentelet pienellä näytöllä, riittää että konsolista on näkyvissä edes pieni kaistale: -->
Every time you're working on a project with a backend, <i>it is critical to keep an eye on the console output of the backend</i>. If you are working on a small screen, it is enough to just see a tiny slice of the output in the background. Any error messages will catch your attention even when the console is far back in the background:

![](../images/3/15b.png)

<!-- ### Virheidenkäsittelyn keskittäminen middlewareen -->
### Moving error handling into middleware

<!-- Olemme kirjoittaneet poikkeuksen aiheuttavan virhetilanteen käsittelevän koodin muun koodin sekaan. Se on välillä ihan toimiva ratkaisu, mutta on myös tilanteita, joissa on järkevämpää keskittää virheiden käsittely yhteen paikkaan. Tästä on huomattava etu esim. jos virhetilanteiden yhteydessä virheen aiheuttaneen pyynnön tiedot logataan tai lähetetään johonkin virhediagnostiikkajärjestelmään, esim. [Sentryyn](https://sentry.io/welcome/).  -->
We have written the code for the error handler among the rest of our code. This can be a reasonable solution at times, but there are cases where it is better to implement all error handling in a single place. This can be particularly useful if we later on want to report data related to errors to an external error tracking system like [Sentry](https://sentry.io/welcome/).

<!-- Muutetaan routen <i>/api/notes/:id</i> käsittelijää siten, että se <i>siirtää virhetilanteen käsittelyn eteenpäin</i> funktiolla <em>next</em> jonka se saa <i>kolmantena</i> parametrina: -->
Let's change the handler for the <i>/api/notes/:id</i> route, so that it passes the error forward with the <em>next</em> function. The next function is passed to the handler as the third parameter:

```js
app.get('/api/notes/:id', (request, response, next) => {
  Note.findById(request.params.id)
    .then(note => {
      if (note) {
        response.json(note.toJSON())
      } else {
        response.status(204).end()
      }
    })
    .catch(error => next(error))
})
```

<!-- Eteenpäin siirrettävä virhe annetaan funktiolle <em>next</em> parametrina. Jos funktiota <em>next</em> kutsuttaisiin ilman parametria, käsittely siirtyisi ainoastaan eteenpäin seuraavaksi määritellylle routelle tai middlewarelle. Jos funktion <em>next</em> kutsussa annetaan parametri, siirtyy käsittely <i>virheidenkäsittelymiddlewarelle</i>. -->
The error that is passed forward, is given to the <em>next</em> function as a parameter. If <em>next</em> was called without a parameter, then the execution would simply move onto the next route or middleware. If the <em>next</em> function is called with a parameter, then the execution will continue to the <i>error handler middleware</i>.

<!-- Expressin [virheidenkäsittelijät](https://expressjs.com/en/guide/error-handling.html) ovat middlewareja, joiden määrittelevällä funktiolla on <i>neljä parametria</i>. Virheidenkäsittelijämme näyttää seuraavalta: -->
Express [error handlers](https://expressjs.com/en/guide/error-handling.html) are middleware that are defined with a function that accepts <i>four parameters</i>. Our error handler looks like this:

```js
const errorHandler = (error, request, response, next) => {
  console.error(error.message)

  if (error.name === 'CastError' && error.kind == 'ObjectId') {
    return response.status(400).send({ error: 'malformatted id' })
  } 

  next(error)
}

app.use(errorHandler)
```

<!-- Virhekäsittelijä tarkastaa onko kyse <i>CastError</i>-poikkeuksesta, eli virheellisestä olioid:stä, jos on, se lähettä pyynnön tehneelle selaimelle vastauksen käsittelijän parametrina olevan response-olion avulla. Muussa tapauksessa se siirtää funktiolla <em>next</em> virheen käsittelyn Expressin oletusarvoisen virheidenkäsittelijän hoidettavavksi. -->
The error handler checks if the error was a <i>CastError</i> exception, and if was, then we know that the error was caused by an invalid object id for Mongo. In this situation the error handler will send a response to the browser with the response object passed as a parameter. In all other error situations the middleware passes the error forward to the default Express error handler. 

<!-- ### Middlewarejen käyttöönottojärjestys -->
### The order of middleware loading

<!-- Koska middlewaret suoritetaan siinä järjestyksessä, missä ne on otettu käyttöön funktiolla _app.use_ on niiden määrittelyn kanssa oltava tarkkana. -->
The execution order of middleware is the same as the order that they are loaded into express with the _app.use_ function. For this reason it is important to be careful when defining middleware.

<!-- Oikeaoppinen järjestys seuraavassa: -->
The correct order is the following:

```js
app.use(express.static('build'))
app.use(bodyParser.json())
app.use(logger)

app.post('/api/notes', (request, response) => {
  const body = request.body
  // ...
})

const unknownEndpoint = (request, response) => {
  response.status(404).send({ error: 'unknown endpoint' })
}

// olemattomien osoitteiden käsittely
app.use(unknownEndpoint)

const errorHandler = (error, request, response, next) => {
  // ...
}

// virheellisten pyyntöjen käsittely
app.use(errorHandler)
```

<!-- _bodyParser_ on syytä ottaa käyttöön melkeimpä ensimmäisenä. Jos järjestys olisi seuraava -->
The _bodyParser_ middleware should be among the very first middleware loaded into Express. If the order was the following:

```js
app.use(logger) // request.body on tyhjä

app.post('/api/notes', (request, response) => {
  // request.body on tyhjä
  const body = request.body
  // ...
})

app.use(bodyParser.json())
```

<!-- ei HTTP-pyynnön mukana oleva data olisi loggerin eikä POST-pyynnön käsittelyn aikana käytettävissä, kentässä _request.body_ olisi tyhjä olio. -->
Then the JSON data sent with the HTTP requests would not be available for the logger middleware the or POST route handler, since the _request.body_ would be an empty object.

<!-- Tärkeää on myös ottaa käyttöön olemattomien osoitteiden käsittely viimeisenä. -->
It's also important that the middleware for handling unsupported routes is the last middleware that is loaded into Express.

<!-- Myös seuraava järjestys aiheuttaisi ongelman -->
The following loading order would also cause an issue:

```js
const unknownEndpoint = (request, response) => {
  response.status(404).send({ error: 'unknown endpoint' })
}

// olemattomien osoitteiden käsittely
app.use(unknownEndpoint)

app.get('/api/notes', (request, response) => {
  // ...
})
```

<!-- Nyt olemattomien osoitteiden käsittely on sijoitettu <i>ennen HTTP GET -pyynnön käsittelyä</i>. Koska olemattomien osoitteiden käsittelijä vastaa kaikkiin pyyntöihin <i>404 unknown endpoint</i>, ei mihinkään sen jälkeen määriteltyyn reittiin tai middlewareen (poikkeuksena virheenkäsittelijä) enää mennä.  -->
Now the handling of unknown endpoints is ordered <i>before the HTTP request handler</i>. Since the unknown endpoint handler responds to all requests with <i>404 unknown endpoint</i>, no routes or middleware will be called after the response has been sent by unknown endpoint middleware. The only exception to this is the error handler.

<!-- ### Muut operaatiot -->
### Other operations

<!-- Toteutetaan vielä jäljellä olevat operaatiot, eli yksittäisen muistiinpanon poisto ja muokkaus. -->
Let's add the missing functionality to our application, i.e. deleting and updating an individual note.

<!-- Poisto onnistuu helpoiten metodilla [findByIdAndRemove](https://mongoosejs.com/docs/api.html#model_Model.findByIdAndRemove): -->
The easiest way to delete a note from the database is with the [findByIdAndRemove](https://mongoosejs.com/docs/api.html#model_Model.findByIdAndRemove) method:

```js
app.delete('/api/notes/:id', (request, response, next) => {
  Note.findByIdAndRemove(request.params.id)
    .then(result => {
      response.status(204).end()
    })
    .catch(error => next(error))
})
```

<!-- Vastauksena on statauskoodi <i>204 no content</i> molemmissa "onnistuneissa" tapauksissa, eli jos olio poistettiin tai olioa ei ollut mutta <i>id</i> oli periaatteessa oikea. Takaisinkutsun parametrin _result_ perusteella olisi mahdollisuus haarautua ja palauttaa tilanteissa eri statuskoodi jos sille on tarvetta. Mahdollinen poikkeus siirretään jälleen virheenkäsittelijälle. -->
In both of the "successful" cases of deleting a resource, the backend responds with the status code <i>204 no content</i>. The two different cases are deleting a note that exists, and deleting a note that does not exist in the database. The _result_ callback parameter could be used for checking if a resource actually was deleted, and we could use that information for returning different status codes for the two cases if we deemed it necessary. Any exception that occurs is passed onto the error handler.

<!-- Muistiinpanon tärkeyden muuttamisen mahdollistava olemassaolevan muistiinpanon päivitys onnistuu helposti metodilla [findByIdAndUpdate](https://mongoosejs.com/docs/api.html#model_Model.findByIdAndUpdate).  -->
The toggling of the importance of a note can be easily accomplished with the [findByIdAndUpdate](https://mongoosejs.com/docs/api.html#model_Model.findByIdAndUpdate) method.

```js
app.put('/api/notes/:id', (request, response, next) => {
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
```

<!-- Operaatio mahdollistaa myös muistiinpanon sisällön editoinnin. Päivämäärän muuttaminen ei ole mahdollista. -->
In the code above we also allow the content of the note to be edited. However, we will not support changing the creation date for obvious reasons.

<!-- Huomaa, että metodin <em>findByIdAndUpdate</em> parametrina tulee antaa normaali Javascript-olio, eikä uuden olion luomisessa käytettävä <em>Note</em>-konstruktorifunktiolla luotu olio. -->
Notice that the <em>findByIdAndUpdate</em> method receives a regular JavaScript object as its parameter, and not a new note object created with the <em>Note</em> constructor function.

<!-- Pieni, mutta tärkeä detalji liittyen operaatioon <em>findByIdAndUpdate</em>. Oletusarvoisesti tapahtumankäsittelijä saa parametrikseen <em>updatedNote</em> päivitetyn olion [ennen muutosta](https://mongoosejs.com/docs/api.html#model_Model.findByIdAndUpdate) olleen tilan. Lisäsimme operaatioon parametrin <code>{ new: true }</code> jotta saamme muuttuneen olion palautetuksi kutsujalle. -->
There is one important detail regarding the use of the <em>findByIdAndUpdate</em> method. By default, the <em>updatedNote</em> parameter of the event handler receives the original document [without the modifications](https://mongoosejs.com/docs/api.html#model_Model.findByIdAndUpdate). We added the optional <code>{ new: true }</code> parameter, which will cause our event handler to be called with the new modified document instead of the original.

<!-- Backend vaikuttaa toimivan postmanista ja VS Code REST clientistä tehtyjen kokeilujen perusteella ja myös frontend toimii moitteettomasti tietokantaa käyttävän backendin kanssa. -->
After testing the backend directly with Postman and the VS Code REST client, we can verify that it seems to work. The frontend also appears to work with the backend using the database. 

<!-- Kun muutamme muistiinpanon tärkeyttä, tulostuu backendin konsoliin ikävä varoitus -->
When we toggle the importance of a note, we see the following worrisome error message in the console:

![](../images/3/48.png)

<!-- Googlaamalla virheilmoitusta löytyy [ohje](https://stackoverflow.com/questions/52572852/deprecationwarning-collection-findandmodify-is-deprecated-use-findoneandupdate) ongelman korjaamiseen. Eli kuten [mongoosen dokumentaatio kehottaa](https://mongoosejs.com/docs/deprecations.html) lisätään tiedostoon <i>note.js</i> yksi rivi: -->
Googling the error message will lead to [instructions](https://stackoverflow.com/questions/52572852/deprecationwarning-collection-findandmodify-is-deprecated-use-findoneandupdate) for fixing the problem. Following [the suggestion in the Mongoose documentation](https://mongoosejs.com/docs/deprecations.html), we add the following line to the <i>note.js</i> file:

```js
const mongoose = require('mongoose')

mongoose.set('useFindAndModify', false) // highlight-line

// ...
  
module.exports = mongoose.model('Note', noteSchema) 
```

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-4), branchissa <i>part3-4</i>. -->
You can find the code for our current application in its entirety in the <i>part3-4</i> branch of [this github repository](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-4).

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- #### 3.15: puhelinluettelo ja tietokanta, step3 -->
#### 3.15: Phonebook database, step3

<!-- Muuta backendiä siten, että numerotietojen poistaminen päivittyy tietokantaan. -->
Change the backend so that deleting phonebook entries is reflected in the database.

<!-- Varmista, että frontend toimii muutosten jälkeen. -->
Verify that the frontend still works after making the changes.

<!-- #### 3.16: puhelinluettelo ja tietokanta, step3 -->
#### 3.16: Phonebook database, step3

<!-- Keskitä sovelluksen virheidenkäsittely middlewareen. -->
Move the error handling of the application to a new error handler middleware. 

<!-- #### 3.17*: puhelinluettelo ja tietokanta, step4 -->
#### 3.17*: Phonebook database, step4

<!-- Jos frontendissä annetaan numero henkilölle, joka on jo olemassa, päivittää frontend tehtävässä 2.18 tehdyn toteutuksen ansiosta tiedot uudella numerolla tekemällä HTTP PUT -pyynnön henkilön tietoja vastaavaan url:iin. -->
If the user tries to create a new phonebook entry for a person whose name is already in the phonebook, the frontend will try to update the phone number of the existing entry by making an HTTP PUT request to the entry's unique URL.

<!-- Laajenna backendisi käsittelemään tämä tilanne. -->
Modify the backend to support this request.

<!-- Varmista, että frontend toimii muutosten jälkeen. -->
Verify that the frontend works after making your changes.

<!-- #### 3.18*: puhelinluettelo ja tietokanta, step5 -->
#### 3.18*: Phonebook database step5

<!-- Päivitä myös polkujen <i>api/persons/:id</i> ja <i>info</i> käsittely, ja varmista niiden toimivuus suoraan selaimella, postmanilla tai VS Coden REST clientillä. -->
Also update the handling of the <i>api/persons/:id</i> and <i>info</i> routes to use the database, and verify that they work directly with the browser, Postman, or VS Code REST client.

<!-- Selaimella tarkastellen yksittäisen numerotiedon tulisi näyttää seuraavalta: -->
Inspecting an individual phonebook entry from the browser should look like this:

![](../images/3/49.png)

</div>
