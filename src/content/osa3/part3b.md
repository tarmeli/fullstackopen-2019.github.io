---
mainImage: ../../images/part-3.svg
part: 3
letter: b
---

<div class="content">

<!-- Palataan yritykseemme käyttää nyt tehtyä backendiä [osassa 2](/osa2) tehdyllä React-frontendillä. Aiempi yritys lopahti seuraavaan virheilmoitukseen -->
Let's continue with our efforts to use the new backend with the React-frontend from [part 2](/part2).
Our last attempt failed with the following error message: 

![](../assets/3/3.png)

<!-- Frontendin tekemä GET-pyyntö osoitteeseen <http://localhost:3001/notes> ei jostain syystä toimi. Mistä on kyse? Backend toimii kuitenkin selaimesta ja postmanista käytettäessä ilman ongelmaa. -->
For some reason, the GET-request the frontend sends to the address <http://localhost:3001/notes> is not working. Why is that? The backend works just fine on the browser and with postman. 

### Same origin policy and CORS

<!-- Kyse on asiasta nimeltään CORS eli Cross-origin resource sharing. [Wikipedian](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) sanoin -->
The issue lies with a thing called CORS, or Cross-origin resource sharing. 

According to [Wikipedia](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing):

> <i>Cross-origin resource sharing (CORS) is a mechanism that allows restricted resources (e.g. fonts) on a web page to be requested from another domain outside the domain from which the first resource was served. A web page may freely embed cross-origin images, stylesheets, scripts, iframes, and videos. Certain "cross-domain" requests, notably Ajax requests, are forbidden by default by the same-origin security policy.</i>

<!-- Lyhyesti sanottuna meidän kontekstissa kyse on seuraavasta: websovelluksen selaimessa suoritettava Javascript-koodi saa oletusarvoisesti kommunikoida vain samassa [originissa](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) olevan palvelimen kanssa. Koska palvelin on localhostin portissa 3001 ja frontend localhostin portissa 3000, niiden origin ei ole sama. -->
In our context the problem is, that by default the JavaScript code of an application run on a browser can only communicate with a server in the same [origin](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy). 
Because our server is in localhosts port 3001, and our frontend in localhost port 3000, they are not in the same origin.  

<!-- Korostetaan vielä, että [same origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) ja CORS eivät ole mitenkään React- tai Node-spesifisiä asioita, vaan yleismaailmallisia periaatteita Web-sovellusten toiminnasta. -->
Keep in mind, that [same origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) and CORS are not specific to React or Node. They are in fact universal princibles of the opration of web applications. 

<!-- Voimme sallia muista <i>origineista</i> tulevat pyynnöt käyttämällä Noden [cors](https://github.com/expressjs/cors)-middlewarea. -->
We can allow for requests from other <i>origins</i> by using Nodes [cors](https://github.com/expressjs/cors)-middleware.

<!-- Asennetaan <i>cors</i> komennolla -->
Install <i>cors</i> with the command

```bash
npm install cors --save
```

<!-- Otetaan middleware käyttöön ja sallitaan kaikki origineista tulevat pyynnöt: -->
take the middleware to use and allow for requests from all origins: 

```js
const cors = require('cors')

app.use(cors())
```

<!-- Nyt frontend toimii! Tosin muistiinpanojen tärkeäksi muuttavaa toiminnallisuutta backendissa ei vielä ole. -->
And the frontend works! Although the functionality for changing the importance of notes has not yet been implemented to the backend. 

<!-- CORS:ista voi lukea tarkemmin esim. [Mozillan sivuilta](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS). -->
You can read more about CORS from [Mozillas page](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).

### Application to the Internet

<!-- Kun koko "stäkki" on saatu vihdoin kuntoon, siirretään sovellus internettiin. Käytetään seuraavassa vanhaa kunnon [Herokua](https://www.heroku.com). -->
Now that the whole stack is ready, lets move our application to the internet. We'll use good old [Heroku](https://www.heroku.com) for this.

<!-- > <i>Jos et ole koskaan käyttänyt herokua, löydät käyttöohjeita kurssin [Tietokantasovellus](https://materiaalit.github.io/tsoha-18/viikko1/)-materiaalista ja Googlaamalla...</i> -->
>If you have never used Heroku before, you can find instructions from the course material of [Tietokantasovellus](https://materiaalit.github.io/tsoha-18/viikko1/) course (in Finnish) or by Googling

<!-- Lisätään projektin juureen tiedosto <i>Procfile</i>, joka kertoo Herokulle, miten sovellus käynnistetään -->
Add a file called  <i>Procfile</i> to the projects root to tell Heroku how to start the application. 

```bash
web: node index.js
```

<!-- Muutetaan tiedoston <i>index.js</i> lopussa olevaa sovelluksen käyttämän portin määrittelyä seuraavasti: -->
Change the definition of the port our application uses at the bottom of the <i>index.js</i> file like so: 

```js
const PORT = process.env.PORT || 3001  // highlight-line
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`)
})
```

<!-- Nyt käyttöön tulee [ympäristömuuttujassa](https://en.wikipedia.org/wiki/Environment_variable) _PORT_ määritelty portti tai 3001 jos ympäristömuuttuja _PORT_ ei ole määritelty. Heroku konfiguroi sovelluksen portin ympäristömuuttujan avulla. -->
Now we are using the port defined in [environment variable](https://en.wikipedia.org/wiki/Environment_variable) _PORT_ or port 3001 if the environment variable _PORT_ is undefined. 
Heroku configures application port based on the environment variable. 

<!-- Tehdään projektihakemistosta git-repositorio, lisätään <i>.gitignore</i> jolla seuraava sisältö -->
Create a git-repository in the project directory, and add <i>.gitignore</i> with the following contents

```bash
node_modules
```

<!-- Luodaan heroku-sovellus komennolla <i>heroku create</i>, tehdään sovelluksen hakemistosta git-repositorio, commitoidaan koodi ja siirretään se Herokuun komennolla <i>git push heroku master</i>. -->
Create an heroku-application with the command <i>heroku create</i>, create a git-repository in the application directory, commit the code and move it to Heroku with command <i>git push heroku master</i>.

<!-- Jos kaikki meni hyvin, sovellus toimii: -->
If everything went well, the application works:

![](../images/3/25a.png)

<!-- Jos ei, vikaa voi selvittää herokun lokeja lukemalla, eli komennolla <i>heroku logs</i>. -->
If not, the issue can be found by reading heroku logs with command <i>heroku logs</i>.

<!-- > **HUOM** ainakin alussa on järkevää tarkkailla Herokussa olevan sovelluksen lokeja koko ajan. Parhaiten tämä onnistuu antamalla komento <i>heroku logs -t</i>, jolloin logit tulevat konsoliin sitä mukaan kun palvelimella tapahtuu jotain. -->
>**NB** At least in the beginning it's good to keep an eye on the heroku logs at all times. The best way to do this is with command <i>heroku logs -t</i> which prints the logs to console whenever something happens on the server. 

<!-- Myös frontend toimii Herokussa olevan backendin avulla. Voit varmistaa asian muuttamalla frontendiin määritellyn backendin osoitteen viittaamaan <i>http://localhost:3001</i>:n sijaan Herokussa olevaan backendiin. -->
The frontend also works with the backend on Heroku. You can check this by changing the backends address on the frontend to be the backends address in Heroku instead of <i>http://localhost:3001</i>.

<!-- Seuraavaksi herää kysymys miten saamme myös frontendin internettiin? Vaihtoehtoja on useita, mutta käydään seuraavaksi läpi yksi niistä. -->
The next question is, how do we deploy the frontend to the Internet? We have multiple options. Lets go through one of them next. 

### Frontend production build

<!-- Olemme toistaiseksi suorittaneet React-koodia <i>sovelluskehitysmoodissa</i>, missä sovellus on konfiguroitu antamaan havainnollisia virheilmoituksia, päivittämään koodiin tehdyt muutokset automaattisesti selaimeen ym. -->
So far we have been running React-code in <i>development mode</i>. In development mode the application is configured to give clear error messages, immediately render code changes to the browser and so on. 

<!-- Kun sovellus viedään tuotantoon, täytyy siitä tehdä [production build](https://reactjs.org/docs/optimizing-performance.html#use-the-production-build) -->
<!-- eli tuotantoa varten optimoitu versio. -->
When the application is deployed, we must create a [production build](https://reactjs.org/docs/optimizing-performance.html#use-the-production-build) or a version of the application which is optimized for production. 

<!-- <i>create-react-app</i>:in avulla tehdyistä sovelluksista saadaan muodostettua tuotantoversio komennolla [npm run build](https://github.com/facebookincubator/create-react-app#npm-run-build-or-yarn-build). -->
Production build of applications created with <i>create-react-app</i> can be created with command [npm run build](https://github.com/facebookincubator/create-react-app#npm-run-build-or-yarn-build).

<!-- Suoritetaan nyt komento <i>frontendin projektin juuressa</i>. -->
Lets run this command from the <i>root of the frontend project</i>.

<!-- Komennon seurauksena syntyy hakemiston <i>build</i> (joka sisältää jo sovelluksen ainoan html-tiedoston <i>index.html</i>) sisään hakemisto <i>static</i>, minkä alle generoituu sovelluksen Javascript-koodin [minifioitu](<https://en.wikipedia.org/wiki/Minification_(programming)>) versio. Vaikka sovelluksen koodi on kirjoitettu useaan tiedostoon, generoituu kaikki Javascript yhteen tiedostoon, samaan tiedostoon tulee itseasiassa myös kaikkien sovelluksen koodin tarvitsemien riippuvuuksien koodi. -->
This creates a directory called <i>build</i> (which contains the only html-file of our application, <i>index.html</i> ) which contains the directory <i>static</i>. [Minified](<https://en.wikipedia.org/wiki/Minification_(programming)>) version of our applications JavaScript code will be generated to the <i>static</i>  directory. Even though the application code is in multiple files, all of the JavaScript will be minified into one file. Actually all of the code from all of the applications debendencies will also be minified into this single file. 

<!-- Minifioitu koodi ei ole miellyttävää luettavaa. Koodin alku näyttää seuraavalta: -->
The minified code is not very readable. The beginning of the code looks like this: 

```js
!function(e){function r(r){for(var n,f,i=r[0],l=r[1],a=r[2],c=0,s=[];c<i.length;c++)f=i[c],o[f]&&s.push(o[f][0]),o[f]=0;for(n in l)Object.prototype.hasOwnProperty.call(l,n)&&(e[n]=l[n]);for(p&&p(r);s.length;)s.shift()();return u.push.apply(u,a||[]),t()}function t(){for(var e,r=0;r<u.length;r++){for(var t=u[r],n=!0,i=1;i<t.length;i++){var l=t[i];0!==o[l]&&(n=!1)}n&&(u.splice(r--,1),e=f(f.s=t[0]))}return e}var n={},o={2:0},u=[];function f(r){if(n[r])return n[r].exports;var t=n[r]={i:r,l:!1,exports:{}};return e[r].call(t.exports,t,t.exports,f),t.l=!0,t.exports}f.m=e,f.c=n,f.d=function(e,r,t){f.o(e,r)||Object.defineProperty(e,r,{enumerable:!0,get:t})},f.r=function(e){"undefined"!==typeof Symbol&&Symbol.toStringTag&&Object.defineProperty(e,Symbol.toStringTag,{value:"Module"})
```

### Serving static files from the backend

<!-- Eräs mahdollisuus frontendin tuotantoon viemiseen on kopioida tuotantokoodi, eli hakemisto <i>build</i> backendin repositorion juureen ja määritellä backend näyttämään pääsivunaan frontendin <i>pääsivu</i>, eli tiedosto <i>build/index.html</i>. -->
One option for debloying the frontend is to copy the production build (the <i>build</i> directory) to the root of the backend repository and configure the backend to show the frontends <i>main page</i> (the file <i>build/index.html</i>) as its main page. 

<!-- Aloitetaan kopioimalla frontendin tuotantokoodi backendin alle, projektin juureen. Omalla koneellani kopiointi tapahtuu frontendin hakemistosta käsin komennolla -->
We begin by copying the production build of the frontend to the root of the backend. With my computer the copying can be done from the frontend directory with the command

```bash
cp -r build ../../../osa3/notes-backend
```

<!-- Backendin sisältävän hakemiston tulee nyt näyttää seuraavalta: -->
The backend directory should now look as follows:

![](../images/3/27.png)

<!-- Jotta saamme expressin näyttämään <i>staattista sisältöä</i> eli sivun <i>index.html</i> ja sen lataaman Javascriptin ym. tarvitsemme expressiin sisäänrakennettua middlewarea [static](http://expressjs.com/en/starter/static-files.html). -->
To make express show <i>static content</i>, the page <i>index.html</i> and the JavaScript etc. it fetches, we need build-in middleware from express called [static](http://expressjs.com/en/starter/static-files.html).

<!-- Kun lisäämme muiden middlewarejen määrittelyn yhteyteen seuraavan -->
When we add the following amidst the declarations of middlewares
```js
app.use(express.static('build'))
```

<!-- tarkastaa Express GET-tyyppisten HTTP-pyyntöjen yhteydessä ensin löytyykö pyynnön polkua vastaavan nimistä tiedostoa hakemistosta <i>build</i>. Jos löytyy, palauttaa express tiedoston. -->
whenever express gets a HTTP GET-request it will first check if the <i>build</i> directory contains a file corresponding to the requests address. If a correct file is found, express will return it. 

<!-- Nyt HTTP GET -pyyntö osoitteeseen <i>www.palvelimenosoite.com/index.html</i> tai <i>www.palvelimenosoite.com</i> näyttää Reactilla tehdyn frontendin. GET-pyynnön esim. osoitteeseen <i>www.palvelimenosoite.com/notes</i> hoitaa backendin koodi. -->
Now HTTP GET-request to the address <i>www.serversaddress.com/index.html</i> or <i>www.serversaddress.com</i> will show the React frontend. GET-request to the address <i>www.serversaddress.com/notes</i> will be handled by the backends code.

<!-- Koska tässä tapauksessa sekä frontend että backend toimivat samassa osoitteessa, voidaan React-sovelluksessa tapahtuva backendin _baseUrl_ määritellä [suhteellisena](https://www.w3.org/TR/WD-html40-970917/htmlweb.html#h-5.1.2) URL:ina, eli ilman palvelinta yksilöivää osaa: -->
Because in our situation both the frontend and the backend are at the same address, we can declare _baseUrl_ as a [relative](https://www.w3.org/TR/WD-html40-970917/htmlweb.html#h-5.1.2) URL. This means we can leave out the part declaring the server. 

```js
import axios from 'axios'
const baseUrl = '/notes' // highlight-line

const getAll = () => {
  const request = axios.get(baseUrl)
  return request.then(response => response.data)
}

// ...
```

<!-- Muutoksen jälkeen on luotava uusi production build ja kopioitava se backendin repositorion juureen. -->
After the change we have to create a new production build and copy it to the root of the backend repository. 

<!-- Sovellusta voidaan käyttää nyt <i>backendin</i> osoitteesta <http://localhost:3001>: -->
The application can now be used from the <i>backend</i> address <http://localhost:3001>:

![](../images/3/28.png)

<!-- Sovelluksemme toiminta vastaa nyt täysin osan 0 luvussa [Single page app](/osa0/#single-page-app) läpikäydyn esimerkkisovelluksen toimintaa. -->
Our application now works exactly like the [Single page app](/part0/#single-page-app) example application we studied in part 0. 

<!-- Kun mennään selaimella osoitteeseen <http://localhost:3001> palauttaa palvelin hakemistossa <i>build</i> olevan tiedoston <i>index.html</i>, jonka sisältö hieman tiivistettynä on seuraava: -->
When we use a browser to go to the address <http://localhost:3001>, the server returns the <i>index.html</i> file from the <i>build</i>  repository. Summarized contents of the file are as follows: 

```html
<head>
  <meta charset="utf-8"/>
  <title>React App</title>
  <link href="/static/css/main.f9a47af2.chunk.css" rel="stylesheet">
</head>
<body>
  <div id="root"></div>
  <script src="/static/js/1.578f4ea1.chunk.js"></script>
  <script src="/static/js/main.104ca08d.chunk.js"></script>
</body>
</html>
```

<!-- Sivu sisältää ohjeen ladata sovelluksen tyylit määrittelevän CSS-tiedoston, sekä kaksi <i>script</i>-tagia, joiden ansiosta selain lataa sovelluksen Javascript-koodin, eli varsinaisen React-sovelluksen. -->
The file contains instructions to fetch a CSS-stylesheet defining the styles of the application, and two <i>script</i> tags which instruct the browser to fetch the JavaScript code of the application - the actual React application. 

<!-- React-koodi hakee palvelimelta muistiinpanot osoitteesta <http://localhost:3001/notes> ja renderöi ne ruudulle. Selaimen ja palvelimen kommunikaatio selviää tuttuun tapaan konsolin välilehdeltä <i>Network</i>: -->
The React code fetches notes from the server address http://localhost:3001/notes> and renders them to the screen. The communications between the server and the browser can be seen on the <i>Network</i> tab of the developer console:

![](../images/3/29.png)

<!-- Kun sovelluksen "internettiin vietävä" versio todetaan toimivan paikallisesti, commitoidaan frontendin tuotantoversio backendin repositorioon ja pushataan koodi uudelleen herokuun. -->
After ensuring that the 'online' version of the application works locally, commit the production build of the frontend to the backend repository, and push the code to heroku again. 

<!-- [Sovellus](https://gentle-ravine-74840.herokuapp.com/) toimii moitteettomasti lukuunottamatta vielä backendiin toteuttamatonta muistiinpanon tärkeyden muuttamista: -->
[The application](https://gentle-ravine-74840.herokuapp.com/) works perfectly, except we haven't added the functionality for changing the importance of a note to the backend yet. 

![](../images/3/30.png)

<!-- Sovelluksemme tallettama tieto ei ole ikuisesti pysyvää, sillä sovellus tallettaa muistiinpanot muuttujaan. Jos sovellus kaatuu tai se uudelleenkäynnistetään, kaikki tiedot katoavat. -->
Our application saves the notes to a variable. If the application craches or is restarted, all of the data will dissappear. 

<!-- Tarvitsemme sovelluksellemme tietokannan. Ennen tietokannan käyttöönottoa katsotaan kuitenkin vielä muutamaa asiaa. -->
The application needs a database. Before we introduce one, let's go through a few things. 

###  Streamlining debloying of the frontend 

<!-- Jotta uuden frontendin version generointi onnistuisi jatkossa ilman turhia manuaalisia askelia, tehdään frontendin repositorion juureen yksinkertainen shell-scripti, joka suorittaa uuden tuotantoversion buildaamisen eli komennon _npm run build_ ja sen siirron backendin alle. Annetaan skriptille nimeksi <i>deploy.sh</i>. Sisältö on seuraava -->
To create a new production build of the frontend without extra manual work, let's add a simple shell-script to the root of the frontend repository. The script builds a new  production build with _npm run build_ and moves it to the backend directory. We name the script <i>deploy.sh</i>. It's contents are like so: 

```bash
#!/bin/sh
npm run build
rm -rf ../../osa3/notebackend/build
cp -r build ../../osa3/notebackend/
```

<!-- Skriptille pitää antaa vielä suoritusoikeudet: -->
The script needs permission to execute: 

```bash
chmod u+x deploy.sh
```

<!-- Skripti voidaan suorittaa frontendin juuresta komennolla _./deploy.sh_ -->
You can run the script from the root of the frontend with command _./deploy.sh_. 

### Backend URLs

<!-- Backendin tarjoama muistiinpanojen käsittelyn rajapinta on nyt suoraan sovelluksen URL:in <https://gentle-ravine-74840.herokuapp.com/> alla. Eli <https://gentle-ravine-74840.herokuapp.com//notes> on kaikkien mustiinpanojen lista ym. Koska backendin roolina on tarjota frontendille koneluettava rajapinta, eli API, olisi ehkä parempi erottaa API:n tarjoama osoitteisto selkeämmin, esim. aloittamalla kaikki sanalla _api_. -->
Our backends interface for handling the notes is currently at the applications URL <https://gentle-ravine-74840.herokuapp.com/>. This means that <https://gentle-ravine-74840.herokuapp.com/notes> is the list of all notes and so on. The role of the backend is to offer a machine readable interface, or an API, to the frontend. It might be better to name the API addresses more clearly, for example by starting all of them with the word _api_.

<!-- Tehdään muutos ensin muuttamalla käsin **kaikki backendin routet**: -->
Let's change **all backend routes** by hand: 

```js
//...
app.get('/api/notes', (request, response) => {
  response.json(notes)
});
//...
```

<!-- Frontendin koodiin riittää seuraava muutos -->
Frontend code only requires the following change: 

```js
import axios from 'axios'
const baseUrl = '/api/notes'  // highlight-line

const getAll = () => {
  const request = axios.get(baseUrl)
  return request.then(response => response.data)
}

// ...
```

<!-- Muutosten jälkeen esim. kaikki muistiinpanot tarjoavan API-endpointin osoite on <https://gentle-ravine-74840.herokuapp.com/api/notes> -->
After these changes i.e the API endpoint for all notes is <https://gentle-ravine-74840.herokuapp.com/api/notes>.

![](../images/3/31.png)

<!-- Frontend on edelleen sovelluksen juuressa eli osoitteessa <https://fullstack-notes.herokuapp.com/>. -->
Frontend is still at the root of the application at <https://fullstack-notes.herokuapp.com/>. 


<!-- > Joskus API:n urleissa ilmaistaan myös API:n versio. Eri versioita saatetaan tarvita, jos aikojen kuluessa API:in tehdään laajennuksia, jotka ilman versiointia hajoittaisivat olemassaolevia osia ohjelmista. Versioinnin avulla voidaan tuoda vanhojen rinnalle uusia, hieman eri tavalla toimivia versioita API:sta. -->
<!-- > API:n version ilmaiseminen URL:issa ei kuitenkaan ole välttämättä, ainakaan kaikkien mielestä järkevää vaikka tapaa paljon käytetäänkin. Oikeasta tavasta API:n versiointiin [kiistellään ympäri internettiä](https://stackoverflow.com/questions/389169/best-practices-for-api-versioning). -->


>Sidenote: **API versions**
>
>Sometimes APIs URLs also show the version of the API. Different versions might be needed if the API is extended over time, and the changes could break existing parts of the programs. With versioning new,  slightly different versions of the API can be used alongside of the older versions. 
>
>Not everyone thinks expressing the API version on the URL is a good idea, even though it is done quite often. The right way of versioning APIs is debated [all over the internet](https://stackoverflow.com/questions/389169/best-practices-for-api-versioning).


### Proxy

<!-- Frontendiin tehtyjen muutosten seurauksena on nyt se, että kun suoritamme frontendiä sovelluskehitysmoodissa, eli käynnistämällä sen komennolla _npm start_, yhteys backendiin ei toimi: -->
Changes on the frontend have caused it not to work on development mode (when started with command _npm start_ ) anymore, as the connection to the backend does not work. 

![](../images/3/32.png)

<!-- Syynä tälle on se, että backendin osoite muutettiin suhteellisesti määritellyksi: -->
This is due to changing the backend address to a relative URL: 

```js
const baseUrl = '/api/notes'
```

<!-- Koska frontend toimii osoitteessa <i>localhost:3000</i>, menevät backendiin tehtävät pyynnöt väärään osoitteeseen <i>localhost:3000/api/notes</i>. Backend toimii kuitenkin osoitteessa <i>localhost:3001</i> -->
Because in development mode the frontend is at the address <i>localhost:3000</i>, the requests to the backend go to the wrong address <i>localhost:3000/api/notes</i>. The backend is at <i>localhost:3001</i>. 

<!-- create-react-app:illa luoduissa projekteissa ongelma on helppo ratkaista. Riittää, että frontendin repositorion tiedostoon <i>package.json</i> lisätään seuraava määritelmä: -->
If project was created with create-react-app, this problem is easy to solve. It is enough to add the following declaration to the <i>package.json</i> file at the frontend repository. 

```bash
{
  "dependencies": {
    // ...
  },
  "scripts": {
    // ...
  },
  "proxy": "http://localhost:3001"  // highlight-line
}
```

<!-- Uudelleenkäynnistyksen jälkeen Reactin sovelluskehitysympäristö toimii [proxynä](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#proxying-api-requests-in-development) ja jos React-koodi tekee HTTP-pyynnön palvelimen <i>http://localhost:3000</i> johonkin osoitteeseen joka ei ole React-sovelluksen vastuulla (eli kyse ei ole esim. sovelluksen Javascript-koodin tai CSS:n lataamisesta), lähetetään pyyntö edelleen osoitteessa <i>http://localhost:3001</i> olevalle palvelimelle. -->
After a restart, the react development environment will work as a [proxy](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#proxying-api-requests-in-development). If the React code does a HTTP-request to a server address at <i>http://localhost:3000</i> not managed by the React application itself (so when the request is not about i.e fetching the CSS or JavaScript for the application), the request will be redirected to the server at <i>http://localhost:3001</i>. 

<!-- Nyt myös frontend on kunnossa, se toimii sekä sovelluskehitysmoodissa että tuotannossa yhdessä palvelimen kanssa. -->
Now the frontend is also fine, working with the server both in development- and production mode. 

<!-- Eräs negatiivinen puoli käyttämässämme lähestymistavassa on se, että sovelluksen uuden version tuotantoon vieminen edellyttää frontendin koodin tuotantoversion generoinnista ja sen backendin repositorion kopioimisesta huolehtivan skriptin <i>deploy.sh</i> suorittamisen. Tämä taas hankaloittaa automatisoidun [deployment pipelinen](https://martinfowler.com/bliki/DeploymentPipeline.html) toteuttamista. Deployment pipelinellä tarkoitetaan automatisoitua ja hallittua tapaa viedä koodi sovelluskehittäjän koneelta erilaisten testien ja laadunhallinnallisten vaiheiden kautta tuotantoympäristöön. -->
A negative aspect of our approach is how complicated it is to deploy the frontend. Deploying a new version requires generating new production build of the frontend and running the <i>deploy.sh</i> script to copy it to the backend repository. This makes creating an automated [deployment pipeline](https://martinfowler.com/bliki/DeploymentPipeline.html) more difficult. Deployment pipeline means an automated and controlled way to move the code from the computer of the developer through different tests and quality checks to the production environment. 

<!-- Tähänkin on useita erilaisia ratkaisuja (esim. sekä frontendin että backendin [sijoittaminen samaan repositorioon](https://github.com/mars/heroku-cra-node)), emme kuitenkaan nyt mene niihin. -->
There are multiple ways to achieve this (for example placing both backend and frontend code [to the same repository](https://github.com/mars/heroku-cra-node) ) but we will not go into those now. 

<!-- Myös frontendin koodin deployaaminen omana sovelluksenaan voi joissain tilanteissa olla järkevää. create-react-app:in avulla luotujen sovellusten osalta se on [suoraviivaista](https://github.com/mars/create-react-app-buildpack). -->
In some situations it may be sensible to deploy the frontend code as it's own application. With apps created with create-react-app it is [straightforward](https://github.com/mars/create-react-app-buildpack).

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-3), branchissa <i>part3-3</i>. -->
Current application code can be found from [github](https://github.com/fullstack-hy2019/part3-notes-backend/tree/part3-3).

</div>

<div class="tasks">

### Exercises

<!-- Seuraavissa tehtävissä koodia ei tarvita montaa riviä. Tehtävät ovat kuitenkin haastavia, sillä nyt on tarkaleen hallittava missä tapahtuu mitäkin, ja kaikki konfiguraatiot on tehtävä täsmälleen oikein.  -->
The next exercises don't require many lines of code. They can however be challenging, because you must understand exactly what is happening where, and the configurations must be just right. 

#### 3.9 phonebook backend step9

<!-- Laita backend toimimaan edellisessä osassa tehdyn puhelinluettelon frontendin kanssa muilta osin, paitsi mahdollisen puhelinnumeron muutoksen osalta, jonka vastaava toiminnallisuus toteutetaan backendiin vasta tehtävässä 3.17. -->
Make the backend work with the frontend from the previous part. Do not implement the functionality for making changes to the phone numbers yet, that will be implemented in exercise 3.17. 

<!-- Joudut todennäköisesti tekemään frontendiin erinäisiä pieniä muutoksia ainakin backendin oletettujen urlien osalta. Muista pitää selaimen konsoli koko ajan auki. Jos jotkut HTTP-pyynnöt epäonnistuvat, kannattaa katsoa <i>Network</i>-välilehdeltä mitä tapahtuu. Pidä myös silmällä mitä palvelimen konsolissa tapahtuu. Jos et tehnyt edellistä tehtävää, kannattaa POST-pyyntöä käsittelevässä tapahtumankäsittelijässä tulostaa konsoliin mukana tuleva data eli <i>request.body</i>. -->
You will propably have to do some small changes to the frontend, at least to the URL's to the backend. Remember to keep the developer console open on your browser. If some HTTP-requests fail, you should check from the <i>Network</i>-tab what is going on. Keep an eye on the backends console as well. If you did not do the previous exercise, it is worth it to print the request data or <i>request.body</i> to the console in the event handler responsible for POST-requests. 

#### 3.10 phonebook backend step10

<!-- Vie sovelluksen backend internetiin, esim. Herokuun.  -->
Deploy the backend to the internet, for example to Heroku. 

<!-- **Huom** komento _heroku_ toimii laitoksen koneilla ja fuksikannettavilla. Jos et jostain syystä saa [asennettua](https://devcenter.heroku.com/articles/heroku-cli) herokua koneellesi, voit käyttää komentoa [npx heroku-cli](https://www.npmjs.com/package/heroku-cli). -->
**NB** the command _heroku_ works on the department's computers and the freshman laptops. If for some reason you cannot [install](https://devcenter.heroku.com/articles/heroku-cli) heroku to your computer, you can use the command [npx heroku-cli](https://www.npmjs.com/package/heroku-cli).

<!-- Testaa selaimen ja postmanin tai VS Code REST clientin avulla, että internetissä oleva backend toimii. -->
Test the deployed backend with a browser and postman or VS Code REST client to ensure it works. 

<!-- **PRO TIP:** kun deployaat sovelluksen herokuun, kannattaa ainakin alkuvaiheissa pitää **KOKO AJAN** näkyvillä Herokussa olevan sovelluksen loki antamalla komento <em>heroku logs -t</em>.  -->
**PRO TIP:** When you deploy your application to heroku, it is worth it at least in the beginning to keep an eye on the logs of the heroku application **AT ALL TIMES** with the command <em>heroku logs -t</em>.

<!-- Seuraavassa loki eräästä tyypillisestä ongelmatilanteesta, heroku ei löydä sovelluksen riippuvuutena olevaa moduulia <i>express</i>: -->
The following is a log about one typical problem. Heroku cannot find application debendency <i>express</i>:

![](../images/3/33.png)

<!-- Syynä ongelmalle on se, että <i>expressiä</i> asennettaessa oli unohtunut antaa optio <i>--save</i>, joka tallentaa tiedon riippuvuudesta tiedostoon <i>package.json</i>.  -->
The reason is, that the option <i>--save</i> was forgotten when <i>express</i> was installed, so information about the debendency was not saved to the file <i>package.json</i>.

<!-- Toinen tyypillinen ongelma on se, että sovellusta ei ole konfiguroitu käyttämään ympäristömuuttujana <em>PORT</em> määriteltyä porttia: -->
Another typical problem is, that the application is not configured to use the port set to environment variable <em>PORT</em>: 

![](../images/3/34.png)

<!-- Tee repositorion juureen tiedosto README.md ja lisää siihen linkki internetissä olevaan sovellukseesi. -->
Create a README.md to the root of your repository, and add a link to your online application to it. 

#### 3.11 phonebook full stack

<!-- Generoi frontendistä tuotantoversio ja lisää se internetissä olevaan sovellukseesi tässä osassa esiteltyä menetelmää noudattaen. -->
Generate a production build of your frontend, and add it to the internet application using the method introduced in this part. 

<!-- **Huom** eihän hakemisto <i>build</i> ole gitignoroituna projektissasi? -->
**NB** Make sure the directory <i>build</i> is not gitignored

<!-- Huolehdi myös, että frontend toimii edelleen myös paikallisesti. -->
Also make sure that the frontend still works locally as well. 

</div>
