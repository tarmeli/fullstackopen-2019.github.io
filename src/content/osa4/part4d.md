---
mainImage: ../../images/part-4.svg
part: 4
letter: d
---

<div class="content">

<!-- Käyttäjien tulee pystyä kirjautumaan sovellukseemme ja muistiinpanot pitää automaattisesti liittää kirjautuneen käyttäjän tekemiksi. -->
Users must be able to log into our application, and when a user is logged in, their user information must automatically be attached to any new notes they create. 

<!-- Toteutamme nyt backendiin tuen [token-perustaiselle](https://scotch.io/tutorials/the-ins-and-outs-of-token-based-authentication#toc-how-token-based-works) autentikoinnille. -->
We will now implement support for [token based authentication](https://scotch.io/tutorials/the-ins-and-outs-of-token-based-authentication#toc-how-token-based-works) to the backend. 

<!-- Token-autentikaation periaatetta kuvaa seuraava sekvenssikaavio: -->
The princibles of token based authentication are depicted in the following sequence diagram: 

![](../images/4/16.png)

<!-- - Alussa käyttäjä kirjautuu Reactilla toteutettua kirjautumislomaketta käyttäen -->
- User starts by logging in using a login form implemented with React 
  <!-- - lisäämme kirjautumislomakkeen frontendiin [osassa 5](/osa5) -->
    - We will add the login form to the frontend in [part 5](/part5) 
<!-- - Tämän seurauksena selaimen React-koodi lähettää käyttäjätunnuksen ja salasanan HTTP POST -pyynnöllä palvelimen osoitteeseen <i>/api/login</i> -->
- This causes the React code to send the username and the password to the server address <i>/api/login</i> as a HTTP POST request. 
<!-- - Jos käyttäjätunnus ja salasana ovat oikein, generoi palvelin <i>tokenin</i>, joka yksilöi jollain tavalla kirjautumisen tehneen käyttäjän -->
- If the username and the password are correct, the server generates a <i>token</i> which somehow identifies the logged in user. 
  <!-- - token on digitaalisesti allekirjoitettu, joten sen väärentäminen on (kryptografisesti) mahdotonta -->
    - The token is signed digitally, making it impossible to falsify (with cryptographic means)
<!-- - backend vastaa selaimelle onnistumisesta kertovalla statuskoodilla ja palauttaa tokenin vastauksen mukana -->
- The backend responds with a statuscode indicating the operation was successfull, and returns the token with the response.
<!-- - Selain tallentaa tokenin esimerkiksi React-sovelluksen tilaan -->
- The browser saves the token, for example to the state of a React application. 
<!-- - Kun käyttäjä luo uuden muistiinpanon (tai tekee jonkin operaation, joka edellyttää tunnistautumista), lähettää React-koodi tokenin pyynnön mukana palvelimelle -->
- When the user creates a new note (or does some other operation requiring identification), the React code sends the token to the server with the request.
<!-- - Palvelin tunnistaa pyynnön tekijän tokenin perusteella -->
- The server uses the token to identify the user

<!-- Tehdään ensin kirjautumistoiminto. Asennetaan [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken)-kirjasto, jonka avulla koodimme pystyy generoimaan [JSON web token](https://jwt.io/) -muotoisia tokeneja. -->
Lets first implement the functionality for logging in. Install the [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) library, which allows us to generate [JSON web tokens](https://jwt.io/).

```bash
npm install jsonwebtoken --save
```

<!-- Tehdään kirjautumisesta vastaava koodi tiedostoon _controllers/login.js_ -->
The code for login functionality goes to the file controllers/login.js.


```js
const jwt = require('jsonwebtoken')
const bcrypt = require('bcrypt')
const loginRouter = require('express').Router()
const User = require('../models/user')

loginRouter.post('/', async (request, response) => {
  const body = request.body

  const user = await User.findOne({ username: body.username })
  const passwordCorrect = user === null
    ? false
    : await bcrypt.compare(body.password, user.passwordHash)

  if (!(user && passwordCorrect)) {
    return response.status(401).json({
      error: 'invalid username or password'
    })
  }

  const userForToken = {
    username: user.username,
    id: user._id,
  }

  const token = jwt.sign(userForToken, process.env.SECRET)

  response
    .status(200)
    .send({ token, username: user.username, name: user.name })
})

module.exports = loginRouter
```

<!-- Koodi aloittaa etsimällä pyynnön mukana olevaa <i>usernamea</i> vastaavan käyttäjän tietokannasta. Seuraavaksi katsotaan onko pyynnön mukana oleva <i>password</i> oikea. Koska tietokantaan ei ole talletettu salasanaa, vaan salasanasta laskettu <i>hash</i>, tehdään vertailu metodilla _bcrypt.compare_: -->
The code starts by searching for the user from the database by the <i>username</i> attached to the request. 
Next it checks the <i>password</i>, also attached to the request. 
Because the passwords themselves are not saved to the database, but <i>hashes</i> calculated from the passwords, _bcrypt.compare_ method is used to check if the password is correct: 

```js
await bcrypt.compare(body.password, user.passwordHash)
```

<!-- Jos käyttäjää ei ole olemassa tai salasana on väärä, vastataan kyselyyn statuskoodilla [401 unauthorized](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.2) ja kerrotaan syy vastauksen bodyssä. -->
If the user is not found, or the password is incorrect, the request is responded with statuscode [401 unauthorized](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.2). The reason for the failure is explained in the response body. 

<!-- Jos salasana on oikein, luodaan metodin _jwt.sign_ avulla token, joka sisältää digitaalisesti allekirjoitetussa muodossa käyttäjätunnuksen ja käyttäjän id: -->
If the password is correct, a token is created with method  _jwt.sign_. The token contains the username and the user id in a digitally signed form. 

```js
const userForToken = {
  username: user.username,
  id: user._id,
}

const token = jwt.sign(userForToken, process.env.SECRET)
```

<!-- Token on digitaalisesti allekirjoitettu käyttämällä <i>salaisuutena</i> ympäristömuuttujassa <i>SECRET</i> olevaa merkkijonoa. Digitaalinen allekirjoitus varmistaa sen, että ainoastaan salaisuuden tuntevilla on mahdollisuus generoida validi token. Ympäristömuuttujalle pitää muistaa asettaa arvo tiedostoon <i>.env</i>. -->
The token has been digitally signed using a string from the environment variable <i>SECRET</i> as the <i>secret</i>.
Digital signature ensures, that only parties who know the secret can generate a valid token. 
Value for the environment variable must be set in the <i>.env</i> file. 

<!-- Onnistuneeseen pyyntöön vastataan statuskoodilla <i>200 ok</i> ja generoitu token sekä kirjautuneen käyttäjän käyttäjätunnus ja nimi lähetetään vastauksen bodyssä pyynnön tekijälle. -->
Successfull request is responded with the statuscode <i>200 ok</i>. The generated token and the username of the user are sent back in the response body. 

<!-- Kirjautumisesta huolehtiva koodi on vielä liitettävä sovellukseen lisäämällä tiedostoon <i>app.js</i> muiden routejen käyttöönoton yhteyteen -->
Now the code for login just has to be added to the application by adding the new router to <i>app.js</i>. 

```js
const loginRouter = require('./controllers/login')

//...

app.use('/api/login', loginRouter)
```

<!-- Kokeillaan kirjautumista, käytetään VS Coden REST-clientiä: -->
Lets try logging in using VS Code REST-client: 

![](../images/4/17.png)

<!-- Kirjautuminen ei kuitenkaan toimi, konsoli näyttää seuraavalta: -->
It does not work. The following is printed to console: 

```bash
(node:32911) UnhandledPromiseRejectionWarning: Error: secretOrPrivateKey must have a value
    at Object.module.exports [as sign] (/Users/mluukkai/opetus/_2019fullstack-koodit/osa3/notes-backend/node_modules/jsonwebtoken/sign.js:101:20)
    at loginRouter.post (/Users/mluukkai/opetus/_2019fullstack-koodit/osa3/notes-backend/controllers/login.js:26:21)
(node:32911) UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). (rejection id: 2)
```

<!-- Ongelman aiheuttaa komento _jwt.sign(userForToken, process.env.SECRET)_ sillä ympäristömuuttujalle <i>SECRET</i> on unohtunut määritellä arvo. Kun arvo (joka saa olla mikä tahansa merkkijono) määritellään tiedostoon <i>.env</i>, alkaa kirjautuminen toimia. -->
The command _jwt.sign(userForToken, process.env.SECRET)_ fails. We forgot to set a value to the environment variable <i>SECRET</i>. It can be any string. When we set the value in file <i>.env</i>, the login works. 

<!-- Onnistunut kirjautuminen palauttaa kirjautuneen käyttäjän tiedot ja tokenin: -->
Successful login returns the user details and the token: 

![](../images/4/18.png)

<!-- Virheellisellä käyttäjätunnuksella tai salasanalla kirjautuessa annetaan asianmukaisella statuskoodilla varustettu virheilmoitus -->
Wrong username or password returns an error message and the proper statuscode

![](../images/4/19.png)

### Limiting creating new notes to logged in users

<!-- Muutetaan vielä muistiinpanojen luomista, siten että luominen onnistuu ainoastaan jos luomista vastaavan pyynnön mukana on validi token. Muistiinpano talletetaan tokenin identifioiman käyttäjän tekemien muistiinpanojen listaan. -->
Lets change creating new notes so, that it is only possible if the post request has a valid token attached. 
The note is then saved to the notes list of the user identified by the token. 

<!-- Tapoja tokenin välittämiseen selaimesta backendiin on useita. Käytämme ratkaisussamme [Authorization](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization)-headeria. Tokenin lisäksi headerin avulla kerrotaan mistä [autentikointiskeemasta](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication#Authentication_schemes) on kyse. Tämä voi olla tarpeen, jos palvelin tarjoaa useita eri tapoja autentikointiin. Skeeman ilmaiseminen kertoo näissä tapauksissa palvelimelle, miten mukana olevat kredentiaalit tulee tulkita. -->
There are serveral ways for sending the token from the browser to the server. We will use the [Authorization](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) header. The header also tells which [authentication schema](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication#Authentication_schemes) is used. This can be necessary if the server offers multiple ways to authenticate. 
Identifying the schema tells the server how the attached credentials should be interpreted. 

<!-- Meidän käyttöömme sopii <i>Bearer</i>-skeema. -->
The <i>Bearer</i>-schema is suitable for our needs. 

<!-- Käytännössä tämä tarkoittaa, että jos token on esimerkiksi merkkijono <i>eyJhbGciOiJIUzI1NiIsInR5c2VybmFtZSI6Im1sdXVra2FpIiwiaW</i>, laitetaan pyynnöissä headerin Authorization arvoksi merkkijono -->
In practice this means, that if the token is for example the string <i>eyJhbGciOiJIUzI1NiIsInR5c2VybmFtZSI6Im1sdXVra2FpIiwiaW</i>, the Authorization header will have the value: 

<pre>
Bearer eyJhbGciOiJIUzI1NiIsInR5c2VybmFtZSI6Im1sdXVra2FpIiwiaW
</pre>

<!-- Muistiinpanojen luominen muuttuu seuraavasti: -->
Creating new notes will change like so: 

```js
const jwt = require('jsonwebtoken') //highlight-line

// ...
  //highlight-start
const getTokenFrom = request => {
  const authorization = request.get('authorization')
  if (authorization && authorization.toLowerCase().startsWith('bearer ')) {
    return authorization.substring(7)
  }
  return null
}
  //highlight-end

notesRouter.post('/', async (request, response, next) => {
  const body = request.body

  const token = getTokenFrom(request)

//highlight-start
  try {
    const decodedToken = jwt.verify(token, process.env.SECRET)

    if (!token || !decodedToken.id) {
      return response.status(401).json({ error: 'token missing or invalid' })
    }
//highlight-end

    const user = await User.findById(decodedToken.id)

    const note = new Note({
      content: body.content,
      important: body.important === undefined ? false : body.important,
      date: new Date(),
      user: user._id
    })

    const savedNote = await note.save()
    user.notes = user.notes.concat(savedNote._id) //highlight-line
    await user.save()  //highlight-line
    response.json(savedNote.toJSON())
  } catch(exception) {
    next(exception)
  }
})
```

<!-- Apufunktio _getTokenFrom_ eristää tokenin headerista <i>authorization</i>. Tokenin oikeellisuus varmistetaan metodilla _jwt.verify_. Metodi myös dekoodaa tokenin, eli palauttaa olion, jonka perusteella token on laadittu: -->
Helper function _getTokenFrom_ isolates the token from the <i>authorization</i> header. The validity of the token is checked with _jwt.verify_. The method also decodes the token, or returns the Object which the token was based on: 

```js
const decodedToken = jwt.verify(token, process.env.SECRET)
```

<!-- Tokenista dekoodatun olion sisällä on kentät <i>username</i> ja <i>id</i> eli se kertoo palvelimelle kuka pyynnön on tehnyt. -->
The object decoded from the token contains fields <i>username</i> and <i>id</i>, so it tells the server who made the request. 

<!-- Jos tokenia ei ole tai tokenista dekoodattu olio ei sisällä käyttäjän identiteettiä (eli _decodedToken.id_ ei ole määritelty), palautetaan virheestä kertova statuskoodi [401 unauthorized](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.2) ja kerrotaan syy vastauksen bodyssä: -->
If there is no token, or the object decoded from the token does not contain the users identity (_decodedToken.id_ is undefined), error statuscode [401 unauthorized](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.2) is returned and the reason for the failure is explained in the response body. 

```js
if (!token || !decodedToken.id) {
  return response.status(401).json({
    error: 'token missing or invalid'
  })
}
```

<!-- Kun pyynnön tekijän identiteetti on selvillä, jatkuu suoritus entiseen tapaan. -->
When the identity of the maker of the request is resolved, the execution continues as before. 

<!-- Uuden muistiinpanon luominen onnistuu nyt postmanilla jos <i>authorization</i>-headerille asetetaan oikeanlainen arvo, eli merkkijono <i>bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ</i>, missä loppuosa on <i>login</i>-operaation palauttama token. -->
A new note can now be created using postman if the <i>authorization</i> header is given the correct value, the string <i>bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ</i>, where the second value is the token returned by the <i>login</i> operation. 

<!-- Postmanilla luominen näyttää seuraavalta -->
Using postman this looks as follows: 

![](../images/4/20.png)

<!-- ja Visual Studio Coden REST clientillä -->
and with Visual Studio Code REST client

![](../images/4/21.png)

### Error handling

<!-- Tokenin verifiointi voi myös aiheuttaa poikkeuksen <i>JsonWebTokenError</i>. Jos esim. poistetaan tokenista pari merkkiä, ja yritetään luoda muistiinpano, tapahtuu seuraavasti -->
Token verification can also cause a <i>JsonWebTokenError</i>. If we for example remove a few characters from the token and try creating a new note, this happens: 

```bash
JsonWebTokenError: invalid signature
    at /Users/mluukkai/opetus/_2019fullstack-koodit/osa3/notes-backend/node_modules/jsonwebtoken/verify.js:126:19
    at getSecret (/Users/mluukkai/opetus/_2019fullstack-koodit/osa3/notes-backend/node_modules/jsonwebtoken/verify.js:80:14)
    at Object.module.exports [as verify] (/Users/mluukkai/opetus/_2019fullstack-koodit/osa3/notes-backend/node_modules/jsonwebtoken/verify.js:84:10)
    at notesRouter.post (/Users/mluukkai/opetus/_2019fullstack-koodit/osa3/notes-backend/controllers/notes.js:40:30)
```

<!-- Syynä tokenin dekoodaamisen aiheuttamalle virheelle on monia. Token voi olla viallinen, kuten esimerkissämme, väärennetty tai eliniältään vanhentunut. Laajennetaan virheidenkäsittelymiddlewarea huomioimaan tokenin dekoodaamisen aiheuttamat virheet -->
There are many possible reasons for a decoding error. The token can be faulty (like in our example), falsified, or expired. Lets extend our errorhandlermiddleware to take into account the different decoding errors. 

```js
const unknownEndpoint = (request, response) => {
  response.status(404).send({ error: 'unknown endpoint' })
}

const errorHandler = (error, request, response, next) => {
  if (error.name === 'CastError' && error.kind === 'ObjectId') {
    return response.status(400).send({
      error: 'malformatted id'
    })
  } else if (error.name === 'ValidationError') {
    return response.status(400).json({
      error: error.message 
    })
  } else if (error.name === 'JsonWebTokenError') {  // highlight-line
    return response.status(401).json({ // highlight-line
      error: 'invalid token' // highlight-line
    }) // highlight-line
  }

  logger.error(error.message)

  next(error)
}
```

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-8), branchissä <i>part4-8</i>. -->
Current application code can be found from [github](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part4-8), branch <i>part4-8</i>.

<!-- Jos sovelluksessa on useampia rajapintoja jotka vaativat kirjautumisen, kannattaa JWT:n validointi eriyttää omaksi middlewarekseen, tai käyttää jotain jo olemassa olevaa kirjastoa kuten [express-jwt](https://www.npmjs.com/package/express-jwt). -->
If the application has multiple interfaces requiring identification, JWTs validation should be separated into its own middleware. Some existing library, like [express-jwt](https://www.npmjs.com/package/express-jwt), could also be used. 

### End notes

<!-- Koodissa on tapahtunut paljon muutoksia ja matkan varrella on tapahtunut tyypillinen kiivaasti etenevän ohjelmistoprojektin ilmiö: suuri osa testeistä on hajonnut. Koska kurssin tämä osa on jo muutenkin täynnä uutta asiaa, jätämme testien korjailun vapaaehtoiseksi harjoitustehtäväksi. -->
There has been many changes to the code, which has caused a typical problem for a fast phased software project: most of the tests have broken. Because this part of the course is already jammed with new information, we will leave fixing the tests to a non compulsory exercise. 

<!-- Käyttäjätunnuksia, salasanoja ja tokenautentikaatiota hyödyntäviä sovelluksia tulee aina käyttää salatun [HTTPS](https://en.wikipedia.org/wiki/HTTPS)-yhteyden yli. Voimme käyttää sovelluksissamme Noden [HTTP](https://nodejs.org/docs/latest-v8.x/api/http.html)-serverin sijaan [HTTPS](https://nodejs.org/api/https.html)-serveriä (se vaatii lisää konfiguraatiota). Toisaalta koska sovelluksemme tuotantoversio on Herokussa, sovelluksemme pysyy käyttäjien kannalta suojattuna sen ansiosta, että Heroku reitittää kaiken liikenteen selaimen ja Herokun palvelimien välillä HTTPS:n yli. -->
Usernames, passwords and applications using token authentication must always be used over [HTTPS](https://en.wikipedia.org/wiki/HTTPS). We could use a Node [HTTPS](https://nodejs.org/api/https.html) server in our application instead of the [HTTP](https://nodejs.org/docs/latest-v8.x/api/http.html) server (it requires more configurations). In the other hand the production version of our application is in Heroku, so our applications stays secured because Heroku routes all traffic between a browser and the Heroku server over HTTPS. 

<!-- Toteutamme kirjautumisen frontendin puolelle kurssin [seuraavassa osassa](/osa5). -->
We will implement login to the frontend in the [next part](/part5).

</div>

<div class="tasks">

### Exercises

<!-- Seuraavien tehtävien myötä Blogilistalle luodaan käyttäjienhallinnan perusteet. Varminta on seurata melko tarkkaan osan 4 luvusta [Käyttäjien hallinta](/osa4/kayttajien_hallinta) ja [Token-perustainen kirjautuminen](/osa4/token_perustainen_kirjautuminen) etenevää tarinaa. Toki luovuus on sallittua. -->
In the next exercises, basics of user management will be implemented for the Bloglist application. The safest way is to follow the story from part 4 chapter [User management](/osa4/kayttajien_hallinta) to the chapter [Token-based authorization](/osa4/token_perustainen_kirjautuminen). Creativity is of course allowed. 

<!-- **Varoitus vielä kerran:** jos huomaat kirjoittavasi sekaisin async/awaitia ja _then_-kutsuja, on 99% varmaa, että teet jotain väärin. Käytä siis jompaa kumpaa tapaa, älä missään tapauksessa "varalta" molempia. -->
**One more warning:** If you notice you are mixing async/await and _then_ calls, it is 99% certain you are doing something wrong. Use either or, never both. 

#### 4.15: bloglist expansion, step4

<!-- Tee sovellukseen mahdollisuus luoda käyttäjiä tekemällä HTTP POST -pyyntö osoitteeseen <i>api/users</i>. Käyttäjillä on <i>käyttäjätunnus, salasana ja nimi</i>. -->
Implement a way to create new users by doing a HTTP POST-request to address <i>api/users</i>. Users have <i>username
, password and name</i>.

<!-- Älä talleta tietokantaan salasanoja selväkielisenä vaan käytä osan 4 luvun [Käyttäjien luominen](/osa4/kayttajien_hallinta#kayttajien-luominen) tapaan <i>bcrypt</i>-kirjastoa. -->
Do not save passwords to the database as clear text, but use <i>bcrypt</i>-library like we did in part 4 chapter [Creating new users](/osa4/kayttajien_hallinta#kayttajien-luominen)

<!-- **HUOM** joillain windows-käyttäjillä on ollut ongelmia <i>bcryptin</i> kanssa. Jos törmäät ongelmiin, poista kirjasto komennolla -->
**NB** some windows users have had problems with <i>bcrypt</i>. If you run into problems, remove the library with command 

```bash
npm uninstall bcrypt --save 
```

<!-- ja asenna sen sijaan [bcryptjs](https://www.npmjs.com/package/bcryptjs) -->
and install [bcryptjs](https://www.npmjs.com/package/bcryptjs) instead. 

<!-- Tee järjestelmään myös mahdollisuus katsoa kaikkien käyttäjien tiedot sopivalla HTTP-pyynnöllä. -->
Implement a way to see the details of all users by doing a suitable HTTP-request. 

<!-- Käyttäjien lista voi näyttää esim. seuraavalta: -->
List of users can, for example, look as follows: 

![](../images/4/22.png)

#### 4.16*: bloglist expansion, step5

<!-- Laajenna käyttäjätunnusten luomista siten, että käyttäjätunuksen sekä salasanan tulee olla olla olemassa. ja vähintään 3 merkkiä pitkiä. Käyttäjätunnuksen on oltava järjestelmässä uniikki. -->
Add a feature which adds the following restrictions to creating new users: Both username and password must be given. Both username and password must be at least 3 characters long. Username must be unique. 

<!-- Luomisoperaation tulee palauttaa sopiva statuskoodi ja jonkinlainen virheilmoitus, jos yritetään luoda epävalidi käyttäjä. -->
The operation must respond with a suitable statuscode and some kind of an error message if invalid user is created. 

<!-- **HUOM** älä testaa salasanaan liittyviä ehtoja Mongoosen validointien avulla, se ei ole hyvä idea, sillä backendin vastaanottama salasana ja kantaan tallennettu salasanan tiiviste eivät ole sama asia. Salasanan oikeellisuus kannattaa testata kontrollerissa samoin kun teimme [osassa 3](/osa3/validointi_ja_es_lint) ennen validointien käyttöönottoa. -->
**NB** Do not test password restrictions with Mongoose validations. It is not a good idea because the password received by the backend and the password hash saved to the database are not the same thing. The password length should be validated in the controller like we did in [part 3](/osa3/validointi_ja_es_lint) before using Mongoose validation. 

<!-- Tee myös testit, jotka varmistavat, että virheellisiä käyttäjiä ei luoda, ja että virheellisen käyttäjän luomisoperaatioon vastaus on järkevä statuskoodin ja virheilmoituksen osalta. -->
Implement also tests which test that invalid users are not created and invalid add user operation returns a suitable statuscode and an error message. 

#### 4.17: bloglist expansion, step6

<!-- Laajenna blogia siten, että blogiin tulee tieto sen lisänneestä käyttäjästä. -->
Expand blog so, that each blog contains information on the creator of the blog. 

<!-- Muokkaa blogien lisäystä osan 4 luvun [populate](/osa4/kayttajien_hallinta#populate) tapaan siten, että blogin lisäämisen yhteydessä määritellään blogin lisääjäksi <i>joku</i> järjestelmän tietokannassa olevista käyttäjistä (esim. ensimmäisenä löytyvä). Tässä vaiheessa ei ole väliä kuka käyttäjistä määritellään lisääväksi. Toiminnallisuus viimeistellään tehtävässä 4.19. -->
Modify adding new blogs so, that when a new blog is created,  <i>any</i> user from the database is designated as it's creator (for example the one found first). Implement this according to part 4 chapter [populate](/osa4/kayttajien_hallinta#populate).
Which user is designated as the creator does not matter just yet. The functionality is finished in exercise 4.19. 

<!-- Muokaa kaikkien blogien listausta siten, että blogien yhteydessä näytetään lisääjän tiedot: -->
Modify listing all blogs so that the creators user information is displayed with the blog: 

![](../images/4/23.png)

<!-- ja käyttäjien listausta siten että käyttäjien lisäämät blogit ovat näkyvillä -->
and listing users so that the blogs user has created are listed with the user: 

![](../images/4/24.png)

#### 4.18: bloglist expansion, step7

<!-- Toteuta osan 4 luvun [Kirjautuminen](/osa4#kirjautuminen) tapaan järjestelmään token-perustainen autentikointi. -->
Implement token-based authentication according to part 4 chapter [Authorization](/osa4#kirjautuminen)

#### 4.19: bloglist expansion, step8

<!-- Muuta blogien lisäämistä siten, että se on mahdollista vain, jos lisäyksen tekevässä HTTP POST -pyynnössä on mukana validi token. Tokenin haltija määritellään blogin lisääjäksi. -->
Modify adding new blogs so, that it requires a valid token to be sent with the HTTP POST request. The user identified by the token is designated as the creator of the blog. 

#### 4.20*: bloglist expansion, step9

<!-- Osan 4 [esimerkissä](/osa4#kirjautuminen) token otetaan headereista apufunktion _getTokenFrom_ avulla. -->
[Example](/osa4#kirjautuminen) from part 4 shows taking the token from the header with the _getTokenFrom_ helperfunction.

<!-- Jos käytit samaa ratkaisua, refaktoroi tokenin erottaminen [middlewareksi](/osa3#middlewaret), joka ottaa tokenin <i>Authorization</i>-headerista ja sijoittaa sen <i>request</i>-olion kenttään <i>token</i>. -->
If you used the same solution, refactor taking the token to a [middleware](/osa3#middlewaret). The middleware should take the token from the <i>Authorization</i>-header and place it to <i>token</i> field of the <i>request</i> object. 

<!-- Eli kun rekisteröit middlewaren ennen routeja tiedostossa <i>app.js</i> -->
So when you register middlewares in the <i>app.js</i> file before all routes

```js
app.use(middleware.tokenExtractor)
```

<!-- pääsevät routet tokeniin käsiksi suoraan viittaamalla _request.token_: -->
routes can access the token with _request.token_:
```js
blogsRouter.post('/', async (request, response) => {
  // ..
  const decodedToken = jwt.verify(request.token, process.env.SECRET)
  // ..
})
```

#### 4.21*: bloglist expansion, step10

<!-- Muuta blogin poistavaa operaatiota siten, että poisto onnistuu ainoastaan jos poisto-operaation tekijä (eli se kenen token on pyynnön mukana) on sama kuin blogin lisääjä. -->
Change the delete blog operation so, that deleting a blog is possible only if the user deleting a blog (the user identified by the token) is the same as the blogs creator. 

<!-- Jos poistoa yritetään ilman tokenia tai väärän käyttäjän toimesta, tulee operaation palauttaa asiaan kuuluva statuskoodi. -->
If deleting a blog is attempted without a token or by a wrong user, the operation should return a suitable statuscode. 

<!-- Huomaa, että jos haet blogin tietokannasta -->
Note, that if you fetch a blog from the database

```js
const blog = await Blog.findById(...)
```

<!-- ei kenttä <i>blog.user</i> ole tyypiltään merkkijono vaan <i>object</i>. Eli jos haluat verrata kannasta haetun olion id:tä merkkijonomuodossa olevaan id:hen, ei normaali vertailu toimi. Kannasta haettu id tulee muuttaa vertailua varten merkkijonoksi: -->
the field <i>blog.user</i> does not contain a string, but an Object. So if you want to compare the id of the object fetched from the database and a string id, normal comparison operation does not work. The id fetched from the database must be parsed into a string first. 

```js
if ( blog.user.toString() === userid.toString() ) ...
```

<!---
note left of kayttaja
  käyttäjä täyttää kirjautumislomakkeelle
  käyttäjätunnuksen ja salasanan
end note
kayttaja -> selain: painetaan login-nappia

selain -> backend: HTTP POST /api/login {username, password}
note left of backend
  backend generoi käyttäjän identifioivan TOKENin
end note
backend -> selain: TOKEN palautetaan vastauksen bodyssä
note left of selain
  selain tallettaa TOKENin
end note
note left of kayttaja
  käyttäjä luo uden muistiinpanon
end note
kayttaja -> selain: painetaan create note -nappia
selain -> backend: HTTP POST /api/notes {content} headereissa TOKEN
note left of backend
  backend tunnistaa TOKENin perusteella kuka käyttää kyseessä
end note

backend -> selain: 201 created

kayttaja -> kayttaja:
-->

</div>
