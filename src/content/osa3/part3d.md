---
mainImage: ../../images/part-3.svg
part: 3
letter: d
---

<div class="content">

<!-- Sovelluksen tietokantaan tallettamalle datan muodolle on usein tarve asettaa joitain ehtoja. Sovelluksemme ei esim. hyväksy muistiinpanoja, joiden sisältö eli <i>content</i> kenttä puuttuu. Muistiinpanon oikeellisuus tallennetaan sen luovassa metodissa: -->
There are usually constraints that we want to apply to the data that is stored in our application's database. Our application shouldn't accept notes that have a missing or empty <i>content</i> property. The validity of the note is checked in the route handler:

```js
app.post('/api/notes', (request, response) => {
  const body = request.body
  // highlight-start
  if (body.content === undefined) {
    return response.status(400).json({ error: 'content missing' })
  }
  // highlight-end

  // ...
})
```

<!-- Eli jos muistiinpanolla ei ole kenttää <i>content</i>, vastataan pyyntöön statuskoodilla <i>400 bad request</i>.  -->
If the note does not have the <i>content</i> property, we respond to the request with the status code <i>400 bad request</i>.

<!-- Routejen tapahtumakäsittelijöissä tehtävää tarkastusta järkevämpi tapa tietokantaan talletettavan tiedon oikean muodon määrittelylle ja tarkastamiselle on Mongoosen [validointitoiminnallisuuden](https://mongoosejs.com/docs/validation.html) käyttö. -->
One smarter way of validating the format of the data before it is stored in the database, is to use the [validation](https://mongoosejs.com/docs/validation.html) functionality available in Mongoose.

<!-- Kullekin talletettavan datan kentälle voidaan määritellä validointisääntöjä skeemassa: -->
We can define specific validation rules for each field in the schema:

```js
const noteSchema = new mongoose.Schema({
  // highlight-start
  content: {
    type: String,
    minlength: 5,
    required: true
  },
  date: { 
    type: Date,
    required: true
  }
    // highlight-end
  important: Boolean
})
```

<!-- Kentän <i>content</i> pituuden vaaditaan nyt olevan vähintään 5 merkkiä. Kentälle <i>data</i> taas on asetettu ehdoksi että sillä on oltava joku arvo, eli kenttä ei voi olla tyhjä. Sama ehto on asetettu myös kentälle <i>content</i>, sillä minimipituuden tarkistava ehto ei huomioi tilannetta, missä kentällä ei ole mitään arvoa. Kentälle <i>important</i> ei ole asetettu mitään ehtoa, joten se on määritelty edelleen yksinkertaisemmassa muodossa. -->
The <i>content</i> field is now required to be at least five characters long. The <i>data</i> field is set as required, meaning that it can not be missing. The same constraint is also implicitly applied to the <i>content</i> field, since the minimum length constraint by default requires the field to not be missing. We have not added any constraints to the <i>important</i> field, so its definition in the schema has not changed.

<!-- Esimerkissä käytetyt validaattorit <i>minlength</i> ja <i>required</i> ovat Mongooseen [sisäänrakennettuja](https://mongoosejs.com/docs/validation.html#built-in-validators) validointisääntöjä. Mongoosen [custom validator](https://mongoosejs.com/docs/validation.html#custom-validators) -ominaisuus mahdollistaa mielivaltaisten validaattorien toteuttamisen jos valmiiden joukosta ei löydy tarkoitukseen sopivaa. -->
The <i>minlength</i> and <i>required</i> validators are [built-in](https://mongoosejs.com/docs/validation.html#built-in-validators) and provided by Mongoose. The Mongoose [custom validator](https://mongoosejs.com/docs/validation.html#custom-validators) functionality allows us to create new validators, if none of the built-in ones cover our needs.

<!-- Jos tietokantaan yritetään tallettaa validointisäännön rikkova olio, heittää tallennusoperaatio poikkeuksen. Muutetaan uuden muistiinpanon luomisesta huolehtivaa käsittelijää siten, että se välittää mahdollisen poikkeuksen virheenkäsittelijämiddlewaren huolehdittavaksi:   -->
If we try to store an object in the database that breaks one of the constraints, the operation will throw an exception. Let's change our handler for creating a new note so that it passes any potential exceptions to the error handler middleware:

```js
app.post('/api/notes', (request, response, next) => {
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
    .catch(error => next(error)) // highlight-line
})
```

<!-- Laajennetaan virheenkäsittelijää huomioimaan validointivirheet: -->
Let's expand the error handler to deal with these validation errors:

```js
const errorHandler = (error, request, response, next) => {
  console.error(error.message)

  if (error.name === 'CastError' && error.kind == 'ObjectId') {
    return response.status(400).send({ error: 'malformatted id' })
  } else if (error.name === 'ValidationError') { // highlight-line
    return response.status(400).json({ error: error.message }) // highlight-line
  }

  next(error)
}
```

<!-- Validoinnin epäonnistuessa palautetaan validaattorin oletusarvoinen virheviesti: -->
When validating an object fails, we return the following default error message from Mongoose:

![](../images/3/50.png)

<!-- ### Promisejen ketjutus -->
### Promise chaining

<!-- Useat routejen tapahtumankäsittelijöistä muuttivat palautettavan datan oikeaan formaattiin kutsumalla palautetuille olioille niiden metodia _toJSON_. Esimimerkiksi uuden muistiinpanon luomisessa metodia kutsutaan _then_:in parametrina palauttamalle oliolle: -->
Many of the route handlers changed the response data into the right format by calling the _toJSON_ method. When we created a new note, the _toJSON_ method was called for the object passed as a parameter to _then_:

```js
app.post('/api/notes', (request, response, next) => {
  // ...

  note.save()
    .then(savedNote => {
      response.json(savedNote.toJSON())
    })
    .catch(error => next(error)) 
})
```

<!-- Voisimme tehdä saman myös hieman tyylikkäämmin [promiseja ketjuttamalla](https://javascript.info/promise-chaining): -->
We can accomplish the same functionality in a much cleaner way with [promise chaining](https://javascript.info/promise-chaining):

```js
app.post('/api/notes', (request, response) => {
  // ...

  note
    .save()
    // highlight-start
    .then(savedNote => {
      return savedNote.toJSON()
    })
    .then(savedAndFormattedNote => {
      response.json(savedAndFormattedNote)
    }) 
    // highlight-end
    .catch(error => next(error)) 
})
```

<!-- Eli ensimmäisen _then_:in takaisinkutsussa otamme Mongoosen palauttaman olion _savedNote_ ja formatoimme sen. Operaation tulos palautetaan returnilla. Kuten osassa 2 [todettiin](/osa2/palvelimella_olevan_datan_muokkaaminen#palvelimen-kanssa-tapahtuvan-kommunikoinnin-eristaminen-omaan-moduuliin), promisen then-metodi palauttaa myös promisen. Eli kun palautamme _savedNote.toJSON()_:n takaisinkutsufunktiosta, syntyy promise, jonka arvona on formatoitu muistiinpano. Pääsemme käsiksi arvoon rekisteröimällä _then_-kutsulla uuden tapahtumankäsittelijän. -->
In the first _then_ we receive _savedNote_ object returned by Mongoose and format it. The result of the operation is returned. Then as [we discussed earlier](/osa2/palvelimella_olevan_datan_muokkaaminen#palvelimen-kanssa-tapahtuvan-kommunikoinnin-eristaminen-omaan-moduuliin), the _then_ method of a promise also returns a promise. This means that when we return _savedNote.toJSON()_ from the callback function, we are actually creating a promise that receives the formatted note as its value. We can access the formatted note by registering a new callback function with the _then_ method.

<!-- Selviämme vieläkin tiiviimmällä koodilla käyttämällä nuolifunktion lyhempää muotoa: -->
We can clean up our code even more by using the more compact syntax for arrow functions:

```js
app.post('/api/notes', (request, response) => {
  // ...

  note
    .save()
    .then(savedNote => savedNote.toJSON()) // highlight-line
    .then(savedAndFormattedNote => {
      response.json(savedAndFormattedNote)
    }) 
    .catch(error => next(error)) 
})
```

<!-- Esimerkkimme tapauksessa promisejen ketjutuksesta ei ole suurta hyötyä. Tilanne alkaa muuttua jos joudumme tekemään useita peräkkäisiä asynkronisia operaatiota. Emme kuitenkaan mene asiaan sen tarkemmin. Tutustumme seuraavassa osassa Javascriptin <i>async/await</i>-syntaksiin, jota käyttämällä peräkkäisten asynkronisten operaatioiden tekeminen helpottuu olellisesti. -->
In this example, Promise chaining does not provide much of a benefit. The situation would change if there were many asynchronous operations that had to be done in sequence. We will not delve further into the topic. In the next part of the course we will learn about the <i>async/await</i> syntax in JavaScript, that will make writing subsequent asynchronous operations a lot easier.

<!-- ### Tietokantaa käyttävän version vieminen tuotantoon -->
### Deploying the database backend to production

<!-- Sovelluksen pitäisi toimia tuotannossa, eli Herokussa lähes sellaisenaan. Frontendin muutosten takia on tehtävä siitä uusi tuotantoversio ja kopioitava se backendiin. Toinen muutos on se, että <i>emme halua</i> Herokussa olevan version käyttävän tiedostossa <i>.env</i> määriteltyjä ympäristömuuttujia. Tämän takia tiedoston <i>index.js</i> alussa oleva rivi -->
The application should work almost as-is in Heroku. We do have to generate a new production build of the frontend due to the changes that we have made to our frontend. <i>We don't</i> want the version in Heroku to use the environment variables defined in the <i>.env</i> file. For this reason, the first line of code in <i>index.js</i>:

```js
require('dotenv').config()
```

<!-- on muutettava muotoon -->
Must be changed to:

```js
if (process.env.NODE_ENV !== 'production') {
  require('dotenv').config()
}
```

<!-- Nyt dotenvissä olevat ympäristömuuttujat otetaan käyttöön ainoastaan silloin kun sovellus ei ole <i>production</i>- eli tuotantomoodissa (kuten esim. Herokussa). -->
How the environment variables defined in dotenv will only be used when the backend is not in <i>production mode</i>, i.e. Heroku.

<!-- Tietokantaurlin kertovan ympäristömuuttujan arvo asetetaan Herokuun komennolla _heroku config:set_ -->
The environment variable that defines the database URL can be set to Heroku with the _heroku config:set_ command.

```bash
heroku config:set MONGODB_URI=mongodb+srv://fullstack:secred@cluster0-ostce.mongodb.net/note-app?retryWrites=true
```

<!-- Sovelluksen pitäisi toimia muutosten jälkeen. Aina kaikki ei kuitenkaan mene suunnitelmien mukaan. Jos ongelmia ilmenee, <i>heroku logs</i> auttaa. Oma sovellukseni ei toiminut muutoksen jälkeen. Loki kertoi seuraavaa -->
The application should work after making these changes. Sometimes things don't go according to plan. If there are problems, <i>heroku logs</i> will be there to help. My own application did not work after the making the changes. The logs showed the following:

![](../images/3/51a.png)

<!-- eli tietokannan osoite olikin jostain syystä määrittelemätön. Komento <i>heroku config</i> paljasti että olin vahingossa määritellyt ympäristömuuttujan <em>MONGO\_URL</em> kun koodi oletti sen olevan nimeltään <em>MONGODB\_URI</em>. -->
For some reason the URL of the database was undefined. The <i>heroku config</i> command revealed that I had accidentally defined the URL to the <em>MONGO\_URL</em> environment variable, when the code expected it to be in <em>MONGODB\_URI</em>.

<!-- Sovelluksen tämän hetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-5), branchissä <i>part3-5</i>. -->
You can find the code for our current application in its entirety in the <i>part3-5</i> branch of [this github repository](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-5).
</div>

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- #### 3.19: puhelinluettelo ja tietokanta, step7 -->
#### 3.19: Phonebook database, step7

<!-- Toteuta sovelluksellesi validaatio, joka huolehtii, että backendiin voi lisätä yhdelle nimelle ainoastaan yhden numeron. Frontendin nykyisestä versiosta ei duplikaatteja voi luoda, mutta suoraan Postmanilla tai VS Coden REST clientillä se onnistuu. -->
Add validation to your application, that will make sure that you can only add one number for a person in the phonebook. Our current frontend won't allow users to try and create duplicates, but we can attempt to create them directly with Postman or the VS Code REST client.

<!-- Mongoose ei tarjoa tilanteeseen sopivaa valmista validaattoria. Käytä npm:llä asennettavaa pakettia [mongoose-unique-validator](https://github.com/blakehaswell/mongoose-unique-validator#readme). -->
Mongoose does not offer a built-in validator for this purpose. Install the [mongoose-unique-validator](https://github.com/blakehaswell/mongoose-unique-validator#readme) package with npm and use it instead.

<!-- Jos HTTP POST -pyyntö yrittää lisätä nimeä, joka on jo puhelinluettelossa, tulee vastata sopivalla statuskoodilla ja lisätä vastaukseen asianmukainen virheilmoitus. -->
If an HTTP POST request tries to add a name that is already in the phonebook, the server must respond with an appropriate status code and error message.

<!-- #### 3.20*: puhelinluettelo ja tietokanta, step8 -->
#### 3.20*: Phonebook database, step8

<!-- Laajenna validaatiota siten, että tietokantaan talletettavan nimen on oltava pituudeltaan vähintään 3 merkkiä ja puhelinnumeron vähitään 8 merkkiä.  -->
Expand the validation so that the name stored in the database has to be at least three characters long, and the phone number must have at least 8 digits.

<!-- Laajenna sovelluksen frontendia siten, että se antaa jonkinlaisen virheilmoituksen validoinnin epäonnistuessa. Virheidenkäsittely hoidetaan lisäämällä <em>catch</em>-lohko uuden henkilön lisäämisen yhteyteen: -->
Expand the frontend so that it displays some form of error message when a validation error occurs. Error handling can be implemented by adding a <em>catch</em> block as shown below:


```js
personService
    .create({ ... })
    .then(createdPerson => {
      // ...
    })
    .catch(error => {
      // pääset käsiksi palvelimen palauttamaan virheilmoitusolioon näin
      console.log(error.response.data)
    })
```

<!-- Voit näyttää frontendissa käyttäjälle Mongoosen validoinnin oletusarvoisen virheilmoituksen vaikka ne eivät olekaan luettavuudeltaan parhaat mahdolliset: -->
You can display the default error message returned by Mongoose, even though they are not as readable as they could be:

![](../images/3/56.png)

<!-- #### 3.21 tietokantaa käyttävä versio internettiin -->
#### 3.21 Deploying the database backend to production

<!-- Generoi päivitetystä sovelluksesta "full stack"-versio, eli tee frontendista uusi production build ja kopioi se backendin repositorioon. Varmista että kaikki toimii paikallisesti käyttämällä koko sovellusta backendin osoitteesta <https://localhost:3001>. -->
Generate a new "full stack" version of the application by creating a new production build of the frontend, and copy it to the backend repository. Verify that everything works locally by using the entire application from the address <https://localhost:3001>.

<!-- Pushaa uusi versio Herokuun ja varmista, että kaikki toimii myös siellä. -->
Push the latest version to Heroku and verify that everything works there as well.

</div>

<div class="content">

<!-- ### Lint -->
### Lint

<!-- Ennen osan lopetusta katsomme vielä nopeasti paitsioon jäänyttä tärkeää työkalua [lintiä](<https://en.wikipedia.org/wiki/Lint_(software)>). Wikipedian sanoin: -->
Before we move onto the next part, we will take a look at an important tool called [lint](<https://en.wikipedia.org/wiki/Lint_(software)>). Wikipedia says the following about lint:

> <i>Generically, lint or a linter is any tool that detects and flags errors in programming languages, including stylistic errors. The term lint-like behavior is sometimes applied to the process of flagging suspicious language usage. Lint-like tools generally perform static analysis of source code.</i>

<!-- Staattisesti tyypitetyissä, käännettävissä kielissä esim. Javassa ohjelmointiympäristöt, kuten NetBeans osaavat huomautella monista koodiin liittyvistä asioista, sellaisistakin, jotka eivät ole välttämättä käännösvirheitä. Erilaisten [staattisen analyysin](https://en.wikipedia.org/wiki/Static_program_analysis) lisätyökalujen, kuten [checkstylen](http://checkstyle.sourceforge.net/) avulla voidaan vielä laajentaa Javassa huomautettavien asioiden määrää koskemaan koodin tyylillisiä seikkoja, esim. sisentämistä. -->
In compiled statically typed languages like Java, IDEs like NetBeans can point out errors in the code, even ones that are more than just compile errors. Additional tools for performing [static analysis](https://en.wikipedia.org/wiki/Static_program_analysis) like [checkstyle](http://checkstyle.sourceforge.net/), can be used for expanding the capabilities of the IDE to also point out problems related to style, like indentation.


<!-- Javascript-maailmassa tämän hetken johtava työkalu staattiseen analyysiin, eli "linttaukseen" on [ESlint](https://eslint.org/). -->
In the JavaScript university, the current leading tool for static analysis aka. "linting" is [ESlint](https://eslint.org/).

<!-- Asennetaan ESlint backendiin kehitysaikaiseksi riippuvuudeksi komennolla -->
Let's install ESlint as a development dependency to the backend project with the command:

```bash
npm install eslint --save-dev
```

<!-- Tämän jälkeen voidaan muodostaa alustava ESlint-konfiguraatio komennolla -->
After this we can initialize a default ESlint configuration with the command:

```bash
node_modules/.bin/eslint --init
```

<!-- Vastaillaan kysymyksiin: -->
We will answer all of the questions:

![](../images/3/52.png)

<!-- Konfiguraatiot tallentuvat tiedostoon _.eslintrc.js_: -->
The configuration will be saved in the _.eslintrc.js_ file:

```js
module.exports = {
    "env": {
        "es6": true,
        "node": true
    },
    "extends": "eslint:recommended",
    "parserOptions": {
        "ecmaVersion": 2018
    },
    "rules": {
        "indent": [
            "error",
            4
        ],
        "linebreak-style": [
            "error",
            "unix"
        ],
        "quotes": [
            "error",
            "single"
        ],
        "semi": [
            "error",
            "never"
        ]
    }
};
```

<!-- Muutetaan heti konfiguraatioista sisennystä määrittelevä sääntö, siten että sisennystaso on 2 välilyöntiä -->
Let's immediately change the rule concerning indentation, so that the indentation level is two spaces.

```js
"indent": [
    "error",
    2
],
```

<!-- Esim tiedoston _index.js_ tarkastus tapahtuu komennolla -->
Inspecting and validating a file like _index.js_ can be done with the following command:

```bash
node_modules/.bin/eslint index.js
```

<!-- Kannattaa ehkä tehdä linttaustakin varten _npm-skripti_: -->
It is recommended to create a separate _npm script_ for linting:

```json
{
  // ...
  "scripts": {
    "start": "node index.js",
    "watch": "nodemon index.js",
    "lint": "eslint ."
  },
  // ...
}
```

<!-- Nyt komennot _npm run lint_ suorittaa tarkastukset koko projektille. -->
Now the _npm run lint_ command will check every file in the project.

<!-- Myös hakemistossa <em>build</em> oleva frontendin tuotantoversio tulee näin tarkastettua. Sitä emme kuitenkaan halua, eli tehdään projektin juureen tiedosto [.eslintignore](https://eslint.org/docs/user-guide/configuring#ignoring-files-and-directories) ja sille seuraava sisältö -->
Also the files in the <em>build</em> directory get checked when the command is run. We do not want this to happen, and we can accomplish this by creating an [.eslintignore](https://eslint.org/docs/user-guide/configuring#ignoring-files-and-directories) file in the project's root with the following contents:

```bash
build
```

<!-- Näin koko hakemiston <em>build</em> sisältö jätetään huomioimatta linttauksessa. -->
This causes the entire <em>build</em> directory to not be checked by ESlint.

<!-- Lintillä on jonkin verran huomautettavaa koodistamme: -->
Lint has quite a lot to say about our code:

![](../images/3/53.png)

<!-- Ei kuitenkaan korjata ongelmia vielä. -->
Let's not fix these issues just yet.

<!-- Parempi vaihtoehto kuin linttauksen suorittaminen komentoriviltä on konfiguroida editorille _lint-plugin_, joka suorittaa linttausta koko ajan. Näin pääset korjaamaan pienet virheet välittömästi. Tietoja esim. Visual Studion ESlint-pluginsta [täällä](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint). -->
A better alternative to executing the linter from the command line is to configure a _lint plugin_ to the editor, that runs the linter continuously. By using the plugin you will see errors in your code immediately. You can find more information about the Visual Studio ESLint plugin [here](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint).

<!-- VS Coden ESlint-plugin alleviivaa tyylisääntöjä rikkovat kohdat punaisella: -->
The VS Code ESlint plugin will underline style violations with a red line:

![](../images/3/54a.png)

<!-- Näin ongelmat on helppo korjata koodiin heti. -->
This makes errors easy to spot and fix right away.

<!-- ESlintille on määritelty suuri määrä [sääntöjä](https://eslint.org/docs/rules/), joita on helppo ottaa käyttöön muokkaamalla tiedostoa <i>.eslintrc.js</i>. -->
ESlint has a vast array of [rules](https://eslint.org/docs/rules/) that are easy to take into use by editing the <i>.eslintrc.js</i> file.

<!-- Otetaan käyttöön sääntö [eqeqeq](https://eslint.org/docs/rules/eqeqeq) joka varoittaa, jos koodissa yhtäsuuruutta verrataan muuten kuin käyttämällä kolmea = -merkkiä. Sääntö lisätään konfiguraatiotiedostoon kentän <i>rules</i> alle. -->
Let's add the [eqeqeq](https://eslint.org/docs/rules/eqeqeq) rule that warns us, if equality is checked with anything but the the triple equals operator. The rule is added under the <i>rules</i> field in the configuration file.

```json
{
  // ...
  "rules": {
    // ...
    "eqeqeq": "error"
  },
}
```

<!-- Tehdään samalla muutama muukin muutos tarkastettaviin sääntöihin. -->
While we're at it, let's make a few other changes to the rules.

<!-- Estetään rivien lopussa olevat [turhat välilyönnit](https://eslint.org/docs/rules/no-trailing-spaces), vaaditaan että [aaltosulkeiden edessä/jälkeen on aina välilyönti](https://eslint.org/docs/rules/object-curly-spacing) ja vaaditaan myös konsistenttia välilyöntien käyttöä [nuolifunktioiden parametrien suhteen](https://eslint.org/docs/rules/arrow-spacing): -->
Let's prevent unnecessary [trailing spaces](https://eslint.org/docs/rules/no-trailing-spaces) at the ends of lines, let's require that [there is always a space before and after curly braces](https://eslint.org/docs/rules/object-curly-spacing), and let's also demand a consistent use of whitespaces in the function parameters of arrow functions.

```json
{
  // ...
  "rules": {
    // ...
    "eqeqeq": "error",
    "no-trailing-spaces": "error",
    "object-curly-spacing": [
        "error", "always"
    ],
    "arrow-spacing": [
        "error", { "before": true, "after": true }
    ]
  },
}
```

<!-- Oletusarvoinen konfiguraatiomme ottaa käyttöön joukon valmiiksi määriteltyjä sääntöjä <i>eslint:recommended</i> -->
Our default configuration takes a bunch of predetermined rules into use from <i>eslint:recommended</i>:

```bash
"extends": "eslint:recommended",
```

<!-- Mukana on myös _console.log_-komennoista varoittava sääntö. Yksittäisen sääntö on helppo kytkeä [pois päältä](https://eslint.org/docs/user-guide/configuring#configuring-rules) määrittelemällä sen "arvoksi" konfiguraatiossa 0. Tehdään toistaiseksi näin säännölle <i>no-console</i>. -->
This includes a rule that warns about _console.log_ commands. [Disabling](https://eslint.org/docs/user-guide/configuring#configuring-rules) a rule can be accomplished by defining its "value" as 0 in the configuration file. Let's do this for the <i>no-console</i> rule in the meantime.

```json
{
  // ...
  "rules": {
    // ...
    "eqeqeq": "error",
    "no-trailing-spaces": "error",
    "object-curly-spacing": [
        "error", "always"
    ],
    "arrow-spacing": [
        "error", { "before": true, "after": true }
    ],
    "no-console": 0 // highlight-line
  },
}
```

<!-- **HUOM** kun teet muutoksia tiedostoon <i>.eslintrc.js</i>, kannattaa muutosten jälkeen suorittaa linttaus komentoriviltä ja varmistaa että konfiguraatio ei ole viallinen: -->
**NB** when you make changes to the <i>.eslintrc.js</i> file, it is recommended to run the linter from the command line. This will verify that the configuration file is correctly formatted:

![](../images/3/55.png)

<!-- Jos konfiguraatiossa on jotain vikaa, voi editorin lint-plugin näyttää mitä sattuu. -->
If there is something wrong in your configuration file, the lint plugin can behave quite erratically.

<!-- Monissa yrityksissä on tapana määritellä yrityksen laajuiset koodausstandardit ja näiden käyttöä valvova ESlint-konfiguraatio. Pyörää ei kannata välttämättä keksiä uudelleen ja voi olla hyvä idea ottaa omaan projektiin käyttöön joku jossain muualla hyväksi havaittu konfiguraatio. Viime aikoina monissa projekteissa on omaksuttu AirBnB:n [Javascript](https://github.com/airbnb/javascript)-tyyliohjeet ottamalla käyttöön firman määrittelemä [ESLint](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb)-konfiguraatio. -->
Many companies define coding standards that are enforced throughout the organization through the ESlint configuration file. It is not recommended to keep reinventing the wheel over and over again, and it can be a good idea to adopt a ready-made configuration from someone else's project into yours. Recently many projects have adopted the AirBnB [Javascript style guide](https://github.com/airbnb/javascript) by taking AirBnB's [ESlint](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb) configuration into use.

<!-- Sovelluksen tämän hetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-6), branchissa <i>part3-6</i>. -->
You can find the code for our current application in its entirety in the <i>part3-6</i> branch of [this github repository](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-6).
</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- #### 3.22: lint-konfiguraatio -->
#### 3.22: Lint configuration

<!-- Ota sovellukseesi käyttöön ESlint. -->
Add ESlint to your application.

</div>
