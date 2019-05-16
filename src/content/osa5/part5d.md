---
mainImage: ../../images/part-5.svg
part: 5
letter: d
---

<div class="content">

<!-- React tarjoaa yhteensä 10 erilaista [valmista hookia](https://reactjs.org/docs/hooks-reference.html), näistä ylivoimaisesti eniten käytetyt ovat meillekin jo tutut [useState](https://reactjs.org/docs/hooks-reference.html#usestate) ja [useEffect](https://reactjs.org/docs/hooks-reference.html#useeffect).  -->
React offers 10 different [built-in hooks](https://reactjs.org/docs/hooks-reference.html), of which the most popular ones are the [useState](https://reactjs.org/docs/hooks-reference.html#usestate) and [useEffect](https://reactjs.org/docs/hooks-reference.html#useeffect) hooks, that we have already been using extensively.

<!-- ### Custom hookit -->
### Custom hooks

<!-- React tarjoaa mahdollisuuden myös omien eli [custom](https://reactjs.org/docs/hooks-custom.html)-hookien määrittelyyn. Customhookien pääasiallinen tarkoitus on Reactin dokumentaation mukaan mahdollistaa komponenttien logiikan uusiokäyttö. -->
React offers the option to create our own [custom](https://reactjs.org/docs/hooks-custom.html) hooks. According to React, the primary purpose of custom hooks is to facilitate the reuse of the logic used in components.

> <i>Building your own Hooks lets you extract component logic into reusable functions.</i>

<!-- Custom hookit ovat tavallisia Javascripfunktioita, jotka voivat kutsua mitä tahansa muita hookeja kunhan vain toimivat [hookien sääntöjen](osa1/monimutkaisempi_tila_reactin_debuggaus#hookien-saannot) puitteissa. Custom hookin nimen täytyy alkaa sanalla _use_. -->
Custom hooks are regular JavaScript functions that can use any other hooks, as long as they adhere to the [rules of hooks](osa1/monimutkaisempi_tila_reactin_debuggaus#hookien-saannot). Additionally, the name of custom hooks must start with the word _use_.

<!-- Teimme [osassa 1](/osa1/komponentin_tila_ja_tapahtumankasittely#tapahtumankasittely) laskurin, jonka arvoa voi kasvattaa, vähentää ja nollata. Sovelluksen koodi on seuraava -->
We implemented a counter application in [part 1](/osa1/komponentin_tila_ja_tapahtumankasittely#tapahtumankasittely), that can have its value incremented, decremented, or reset. The code of the application is as follows:

```js  
import React, { useState } from 'react'
const App = (props) => {
  const [counter, setCounter] = useState(0)

  return (
    <div>
      <div>{counter}</div>
      <button onClick={() => setCounter(counter + 1)}>
        plus
      </button>
      <button onClick={() => setCounter(counter - 1)}>
        minus
      </button>      
      <button onClick={() => setCounter(0)}>
        zero
      </button>
    </div>
  )
}
```

<!-- Eriytetään laskurilogiikka custom hookiksi. Hookin koodi on seuraavassa -->
Let's extract the counter logic into its own custom hook. The code for the hook is as follows:

```js
const useCounter = () => {
  const [value, setValue] = useState(0)

  const increase = () => {
    setValue(value + 1)
  }

  const decrease = () => {
    setValue(value - 1)
  }

  const zero = () => {
    setValue(0)
  }

  return {
    value, 
    increase,
    decrease,
    zero
  }
}
```

<!-- Hook siis käyttää sisäisesti _useState_-hookia luomaan itselleen tilan. Hook palauttaa olion, joka sisältää kenttinään hookin tilan arvon sekä funktiot hookin tallettaman arvon muuttamiseen. -->
Our custom hook uses the _useState_ hook internally to create its own state. The hook returns an object, that's properties contain the value of the counter as well as functions for manipulating the value.

<!-- React-komponentti käyttää hookia seuraavaan tapaan: -->
React components can use the hook as shown below:

```js
const App = (props) => {
  const counter = useCounter()

  return (
    <div>
      <div>{counter.value}</div>
      <button onClick={counter.increase}>
        plus
      </button>
      <button onClick={counter.decrease}>
        minus
      </button>      
      <button onClick={counter.zero}>
        zero
      </button>
    </div>
  )
}
```

<!-- Näin komponentin _App_ tila ja sen manipulointi on siirretty kokonaisuudessaan hookin _useCounter_ vastuulle.  -->
By doing this we can extract the state of the _App_ component and its manipulation entirely into the _useCounter_ hook. Managing the counter state and logic is now the responsibility of the custom hook.

<!-- Samaa hookia voitaisiin <i>uusiokäyttää</i> sovelluksessa joka laski vasemman ja oikean napin painalluksia: -->
The same hook could be <i>reused</i> in the application that was keeping track of the amount of clicks made to the left and right buttons:

```js

const App = () => {
  const left = useCounter()
  const right = useCounter()

  return (
    <div>
      {left.value}
      <button onClick={left.increase}>
        left
      </button>
      <button onClick={right.increase}>
        right
      </button>
      {right.value}
    </div>
  )
}
```

<!-- Nyt sovellus luo <i>kaksi</i> erillistä laskuria, toisen käsittelyfunktioineen se tallentaa muuttujaan _left_ ja toisen muuttujaan _right_.  -->
The application creates <i>two</i> completely separate counters. The first one is assigned to the variable _left_ and the other to the variable _right_.

<!-- Lomakkeiden käsittely on Reactissa jokseenkin vaivalloista. Seuraavassa sovellus, joka pyytää lomakkeella käyttäjän nimen, syntymäajan ja pituuden: -->
Dealing with forms in React is somewhat tricky. The following application presents the user with a form that requests the user to input their name, birthday, and height:

```js
const App = () => {
  const [name, setName] = useState('')
  const [born, setBorn] = useState('')
  const [length, setLength] = useState('')

  return (
    <div>
      <form>
        nimi: 
        <input
          type='text'
          value={name}
          onChange={(event) => setName(event.target.value)} 
        /> 
        <br/> 
        syntymäaika:
        <input
          type='date'
          value={born}
          onChange={(event) => setBorn(event.target.value)}
        />
        <br /> 
        pituus:
        <input
          type='number'
          value={length}
          onChange={(event) => setLength(event.target.value)}
        />
      </form>
      <div>
        {name} {born} {length} 
      </div>
    </div>
  )
}
```

<!-- Jokaista lomakkeen kenttää varten on oma tilansa. Jotta tila pysyy synkroonissa lomakkeelle syötettyjen tietojen kanssa, on jokaiselle <i>input</i>-elementille rekisteröity sopiva <i>onChange</i>-käsittelijä. -->
Every field of the form has its own state. In order to keep the state of the form synchronized with the data provided by the user, we have to register an appropriate <i>onChange</i> handler for each of the <i>input</i> elements.

<!-- Määritellään custom hook _useField_, joka yksinkertaistaa lomakkeen tilan hallintaa: -->
Let's define our own custom _useField_ hook, that simplifies the state management of the form:

```js
const useField = (type) => {
  const [value, setValue] = useState('')

  const onChange = (event) => {
    setValue(event.target.value)
  }

  return {
    type,
    value,
    onChange
  }
}
```

<!-- Hook-funktio saa parametrina kentän tyypin. Funktio palauttaa kaikki <i>input</i>-kentän tarvitsemat attribuutit eli tyypin, kentän arvon sekä onChange-tapahtumankäsittelijän. -->
The hook function receives the type of the input field as a parameter. The function returns all of the attributes required by the <i>input</i> which are its type, value and the onChange handler.

<!-- Hookia voidaan käyttää seuraavalla tavalla: -->
The hook can be used in the following way:

```js
const App = () => {
  const name = useField('text')
  // ...

  return (
    <div>
      <form>
        <input
          type={name.type}
          value={name.value}
          onChange={name.onChange} 
        /> 
        // ...
      </form>
    </div>
  )
}
```

<!-- ### Spread-attribuutit -->
### Spread attributes

<!-- Pääsemme itseasiassa helpommalla. Koska oliolla _name_ on nyt täsmälleen ne kentät, mitä <i>input</i>-komponentti odottaa saavansa propsina, voimme välittää propsit hyödyntäen [spread-syntaksia](https://reactjs.org/docs/jsx-in-depth.html#spread-attributes), seuraavasti: -->
We make simplify things a bit further. Since the _name_ object has exactly all of the attributes that the <i>input</i> element excepts to receive as props, we can pass the props to the element with the [spread syntax](https://reactjs.org/docs/jsx-in-depth.html#spread-attributes) in the following way:

```js
<input {...name} /> 
```

<!-- Eli kuten Reactin dokumentaation [esimerkki](https://reactjs.org/docs/jsx-in-depth.html#spread-attributes) sanoo, seuraavat kaksi tapaa välittää propseja komponentille tuottavat saman lopputuloksen: -->
As the [example](https://reactjs.org/docs/jsx-in-depth.html#spread-attributes) in the React documentation states, the following two ways of passing props to a component achieve the exact same result:

```js
<Greeting firstName='Arto' lastName='Hellas' />

const person = {
  firstName: 'Arto',
  lastName: 'Hella'
}

<Greeting {...person} />
```

<!-- Sovellus pelkistyy muotoon -->
The application gets simplified into the following format:

```js
const App = () => {
  const name = useField('text')
  const born = useField('date')
  const length = useField('number')

  return (
    <div>
      <form>
        nimi: 
        <input  {...name} /> 
        <br/> 
        syntymäaika:
        <input {...born} />
        <br /> 
        pituus:
        <input {...length} />
      </form>
      <div>
        {name.value} {born.value} {length.value}
      </div>
    </div>
  )
}
```

<!-- Lomakkeiden käsittely yksinkertaistuu huomattavasti kun ikävät tilan synkronoimiseen liittyvät detaljit on kapseloitu custom hookin vastuulle. -->
Dealing with forms is greatly simplified when the unpleasant nitty-gritty details related to synchronizing the state of the form is encapsulated inside of our custom hook.

<!-- Custom hookit eivät selvästikään ole pelkkä uusiokäytön väline, ne mahdollistavat myös entistä paremman tavan jakaa koodia pienempiin, modulaarisiin osiin. -->
Custom hooks are clearly not only a tool for reuse, they also provide a better way for dividing our code into smaller modular parts.

<!-- ### Hookien säännöt revisited -->
### Rules of hooks, revisited

<!-- Kuten [osassa 1](/osa1/monimutkaisempi_tila_reactin_debuggaus#hookien-saannot)  mainittiin, on hookeja käytettävä tiettyjä [rajoituksia](https://reactjs.org/docs/hooks-rules.html) noudattaen. Seuraavassa vielä hookien käytön säännöt suoraan Reactin dokumentaatiosta kopioituna: -->
As mentioned in [part 1](/osa1/monimutkaisempi_tila_reactin_debuggaus#hookien-saannot), when using hooks we have to adhere to certain [rules or limitations](https://reactjs.org/docs/hooks-rules.html). Let's recap the rules of using hooks, copied verbatim from the official React documentation:

**Don’t call Hooks inside loops, conditions, or nested functions.** Instead, always use Hooks at the top level of your React function. 

**Don’t call Hooks from regular JavaScript functions.** Instead, you can:

- Call Hooks from React function components.
- Call Hooks from custom Hooks

<!-- On olemassa [ESlint](https://www.npmjs.com/package/eslint-plugin-react-hooks)-sääntö, jonka avulla voidaa varmistaa, että sovellus käyttää hookeja oikein. Valitettavasti sääntöä ei voi vielä tässä vaiheessa (29.1.2019) käyttää create-react-app:issa sovelluksissa. -->
There's an existing [ESlint](https://www.npmjs.com/package/eslint-plugin-react-hooks) rule that can be used to verify that the application uses hooks correctly. Unfortunately the rule can not be currently (29.1.2019) used with create-react-app applications.

<!-- ### Lisää hookeista -->
### More about hooks

<!-- Internetistä alkaa löytyä yhä enenevissä määrin hyödyllistä hookeihin liittyvä materiaalia, esim. seuraavia kannattaa vilkaista -->
The internet is starting to fill up with more and more helpful material related to hooks. The following sources are worth checking out:

* [Awesome React Hooks Resouces](https://github.com/rehooks/awesome-react-hooks)
* [Why Do React Hooks Rely on Call Order?](https://overreacted.io/why-do-hooks-rely-on-call-order/)
* [Easy to understand React Hook recipes by Gabe Ragland](https://usehooks.com/)

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- #### 5.18: blogilista ja hookit step1 -->
#### 5.18: Blog list and hooks step1

<!-- Yksinkertaista sovelluksesi kirjautumislomakkeen käyttöä äsken määritellyn _useField_ custom hookin avulla. -->
Simplify the login form of your application with the _useField_ custom hook we defined earlier.

<!-- Luonteva paikka tallentaa hook on tiedosto <i>/src/hooks/index.js</i>.  -->
One natural place to save the custom hooks of your application is in the <i>/src/hooks/index.js</i> file.

<!-- Jos käytät normaalisti käyttämämme default exportin sijaan [nimettyä exportia](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export#Description) -->
If you use the [named export](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export#Description) instead of the default export:

```js
import { useState } from 'react'

export const useField = (type) => { // highlight-line
  const [value, setValue] = useState('')

  const onChange = (event) => {
    setValue(event.target.value)
  }

  return {
    type,
    value,
    onChange
  }
}

// moduulissa voi olla monta nimettyä eksportia
export const useAnotherHook = () => { // highlight-line
  // ...
}
```

<!-- [importtaus](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) tapahtuu seuraavasti -->
Then [importing](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) happens in the following way:

```js
import  { useField } from './hooks'

const App = () => {
  // ...
  const username = useField('text')
  // ...
}
```

<!-- #### 5.19: blogilista ja hookit step2 -->
#### 5.19: Blog list and hooks step2

<!-- <i>useField</i>-hookissa on pieni epäkohta. Se ei mahdollista lomakkeen syötekentän tyhjentämistä. Laajenna hookia siten, että se tarjoaa operaation <i>reset</i> kentän tyhjentämiseen. Ota hook käyttöön myös uuden blogin luovassa formissa. -->
The <i>useField</i> hook has a defect. It does not offer any way of clearing the form field. Expand the functionality of the hook so that it offers a new <i>reset</i> operation for clearing the field. Also take the improved hook into use in the form for creating a new blog post.

<!-- Lisäyksen jälkeen lomakeet toimivat edelleen, mutta riippuen ratkaisustasi konsoliin saattaa ilmestyä ikävä varoitus: -->
The forms of your application will work after making the necessary changes. However, depending on your solution you may see the following warning in your console:

![](../images/5/22.png)

<!-- Ei välitetä virheestä vielä tässä tehtävässä. -->
We will return to this warning in the next exercise.

<!-- #### 5.20: blogilista ja hookit step3 -->
#### 5.20: Blog list and hooks step3

<!-- Jos ratkaisusi ei aiheuttanut warningia, ei sinun tarvitse tehdä tässä tehtävässä mitään. -->
If your solution did not cause a warning to appear in the console you have already finished this exercise.

<!-- Muussa tapauksessa tee sovellukseen korjaus, joka poistaa varoituksen `Invalid value for prop reset' on <input> tag`.  -->
If you see the warning in the console, make the necessary changes to get rid of the `Invalid value for prop reset' on <input> tag` console warning. 

<!-- Warningin syynä on siis se, että edellisen tehtävän laajennuksen jälkeen seuraava -->
The reason for this warning is that after making the changes to your application, the following expression:

```js
<input {...username}/>
```

<!-- tarkoittaa samaa kuin -->
Essentially, is the same as this:

```js
<input
  value={username.value} 
  type={username.type}
  onChange={username.onChange}
  reset={username.reset} // highlight-line
/>
```

<!-- Elementille <i>input</i> ei kuitenkaan kuuluisi antaa propsia <i>reset</i> -->
The <i>input</i> element should not be given a <i>reset</i> attribute.

<!-- Yksinkertainen korjaus olisi tietysti olla käyttämättä spread-syntaksia ja kirjoittaa kaikki lomakkeet seuraavasti -->
One simple fix would be to not use the spread syntax and write all of the forms like this:

```js
<input
  value={username.value} 
  type={username.type}
  onChange={username.onChange}
/>
```

<!-- Tällöin menettäisimme suurelta osin <i>useField</i>-hookin edut. Eli keksi tähän tehtävään spread-syntaksia edelleen käyttävä helppokäyttöinen ratkaisu ongelman kiertämiseen. -->
If we were to do this we would lose much of the benefit provided by the <i>useField</i> hook. Instead, come up with a solution that fixes the issue, but is still easy to use with spread syntax.

#### 5.21*: ultimate hooks

<!-- Tämän osan materiaalissa jatkokehitetyn muistiinpanosovelluksen palvelimen kanssa keskusteleva koodi näyttää seuraavalta: -->
The code of the application responsible for communicating with the backend of the note application looks like this:

```js
import axios from 'axios'
const baseUrl = '/api/notes'

let token = null

const setToken = newToken => {
  token = `bearer ${newToken}`
}

const getAll = () => {
  const request = axios.get(baseUrl)
  return request.then(response => response.data)
}

const create = async newObject => {
  const config = {
    headers: { Authorization: token },
  }

  const response = await axios.post(baseUrl, newObject, config)
  return response.data
}

const update = (id, newObject) => {
  const request = axios.put(`${ baseUrl } /${id}`, newObject)
  return request.then(response => response.data)
}

export default { getAll, create, update, setToken }
```

<!-- Huomaamme, että koodi ei itseasiassa välitä ollenkaan siitä että se käsittelee nimenomaan muistiinpanoja. Muuttujan _baseUrl_ arvoa lukuunottamatta käytännössä sama koodi voi hoitaa blogisovelluksen frontendin ja backendin kommunikointia.  -->
We notice that the code is in no way specific to the fact that our application deals with notes. Excluding the value of the _baseUrl_ variable, the same code could be reused in the blog post application for dealing with the communication with the backend.

<!-- Eristä kommunikoiva koodi hookiksi _useResource_. Riittää, että kaikkien olioiden haku ja uuden olion luominen onnistuvat. -->
Extract the code for communicating with the backend into its own _useResource_ hook. It is sufficient to implement fetching all resources and creating a new resource.

<!-- Voit tehdä tehtävän repositoriosta https://github.com/fullstack-hy2019/custom-hooks löytyvään projektiin. Projektin komponentti <i>App</i> on seuraavassa: -->
You can do the exercise for the project found in the https://github.com/fullstack-hy2019/custom-hooks repository. The <i>App</i> component for the project is the following:

```js
const App = () => {
  const content = useField('text')
  const name = useField('text')
  const number = useField('text')

  const [notes, noteService] = useResource('http://localhost:3005/notes')
  const [persons, personService] = useResource('http://localhost:3005/persons')

  const handleNoteSubmit = (event) => {
    event.preventDefault()
    noteService.create({ content: content.value })
  }
 
  const handlePersonSubmit = (event) => {
    event.preventDefault()
    personService.create({ name: name.value, number: number.value})
  }

  return (
    <div>
      <h2>notes</h2>
      <form onSubmit={handleNoteSubmit}>
        <input {...content} />
        <button>create</button>
      </form>
      {notes.map(n => <p key={n.id}>{n.content}</p>)}

      <h2>persons</h2>
      <form onSubmit={handlePersonSubmit}>
        name <input {...name} /> <br/>
        number <input {...number} />
        <button>create</button>
      </form>
      {persons.map(n => <p key={n.id}>{n.name} {n.number}</p>)}
    </div>
  )
}
```

<!-- Custom hook _useResource_ siis palauttaa (tilahookien tapaan) kaksialkioisen taulukon. Taulukon ensimmäinen alkio sisältää resurssin kaikki oliot ja toisena alkiona on olio, jonka kautta resurssia on mahdollista manipuloida, mm lisäämällä uusia olioita.  -->
The _useResource_ custom hook returns an array of two items just like the state hooks. The first item of the array contains all of the individual resources and the second item of the array is an object that can be used for manipulating the resource collection, like creating new ones.

<!-- Jos toteutit hookin oikein, mahdollistaa sovellus blogien ja puhelinnumeroiden yhtäaikaisen käsittelyn (käynnistä backend porttiin 3005 komennolla _npm run server_) -->
If you implement the hook correctly, it can be used for both notes and phone numbers (start the server with the _npm run server_ command at the port 3005).

![](../images/5/21.png)

</div>
