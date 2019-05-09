---
mainImage: ../../images/part-5.svg
part: 5
letter: c
---

<div class="content">

<!-- Reactilla tehtyjen frontendien testaamiseen on monia tapoja. Aloitetaan niihin tutustuminen nyt. -->
There are many different ways of testing React applications. Let's take a look at them next.

<!-- Testit tehdään samaan tapaan kuin edellisessä osassa eli Facebookin [Jest](http://jestjs.io/)-kirjastolla. Jest onkin valmiiksi konfiguroitu create-react-app:illa luotuihin projekteihin. -->
Tests will be implemented with the same [Jest](http://jestjs.io/) testing library developed by Facebook that was used in the previous part. Jest is actually configured by default to applications created with create-react-app.

<!-- Tarvitsemme Jestin lisäksi testaamiseen apukirjaston, jonka avulla React-komponentteja voidaan renderöidä testejä varten. Tähän tarkoitukseen ehdottomasti paras vaihtoehto vielä viime syksyyn asti oli AirBnB:n kehittämä [enzyme](https://github.com/airbnb/enzyme)-kirjasto. Enzyme ei kuitenkaan tue kunnolla Reactin hookeja, joten käytämme Enzymen sijaan viime aikoina nopeasti suosiota kasvattanutta kirjastoa [react-testing-library](https://github.com/kentcdodds/react-testing-library). Jestin ilmaisuvoimaa kannattaa myös laajentaa kirjastolla [jest-dom](https://www.npmjs.com/package/jest-dom). -->
In addition to Jest, we also need another testing library that will help us render components for testing purposes. The best option for this used to be the [enzyme](https://github.com/airbnb/enzyme) library developed by AirBnB. Unfortunately Enzyme does not support React hooks properly, so we will instead use [react-testing-library](https://github.com/kentcdodds/react-testing-library) which has seen a rapid growth in popularity in recent times.

<!-- Asennetaan kirjastot komennolla: -->
Let's install the library with the command:

```js
npm install --save-dev react-testing-library jest-dom
```

<!-- Testataan aluksi muistiinpanon renderöivää komponenttia: -->
Let's first write tests for the component that is responsible for rendering a note:

```js
const Note = ({ note, toggleImportance }) => {
  const label = note.important
    ? 'make not important'
    : 'make important'

  return (
    <li className='note'> // highlight-line
      {note.content}
      <button onClick={toggleImportance}>{label}</button>
    </li>
  )
}
```

<!-- Huomaa, että muistiinpanon sisältävällä <i>li</i>-elementillä on [CSS](https://reactjs.org/docs/dom-elements.html#classname)-luokka <i>note</i>, pääsemme sen avulla muistiinpanoon käsiksi testistä. -->
Notice that the <i>li</i> element has the [CSS](https://reactjs.org/docs/dom-elements.html#classname) classname <i>note</i>, that is used to access the component in our tests.

<!-- ### Komponentin renderöinti testiä varten -->
### Rendering the component for tests

<!-- Tehdään testi tiedostoon <i>src/components/Note.test.js</i>, eli samaan hakemistoon, missä komponentti itsekin sijaitsee. -->
We will write our test in the <i>src/components/Note.test.js</i> file, which is in the same directory as the component itself.

<!-- Ensimmäinen testi varmistaa, että komponentti renderöi muistiinpanon sisällön: -->
The first test verifies that the component renders the contents of the note:

```js
import React from 'react'
import 'jest-dom/extend-expect'
import { render, cleanup } from 'react-testing-library'
import Note from './Note'

afterEach(cleanup)

test('renders content', () => {
  const note = {
    content: 'Komponenttitestaus tapahtuu react-testing-library:llä',
    important: true
  }

  const component = render(
    <Note note={note} />
  )

  expect(component.container).toHaveTextContent(
    'Komponenttitestaus tapahtuu react-testing-library:llä'
  )
})
```

<!-- Alun konfiguroinnin jälkeen testi renderöi komponentin metodin react-testing-library-kirjaston tarjoaman [render](https://testing-library.com/docs/react-testing-library/api#render) avulla: -->
After the initial configuration, the test renders the component with the [render](https://testing-library.com/docs/react-testing-library/api#render) method provided by the react-testing-library:

```js
const component = render(
  <Note note={note} />
)
```

<!-- Normaalisti React-komponentit renderöityvät <i>DOM</i>:iin. Nyt kuitenkin renderöimme komponentteja testeille sopivaan muotoon laittamatta niitä DOM:iin.  -->
Normally React components are rendered to the <i>DOM</i>. The render method we used renders the components in a format that is suitable for tests without rendering them to the DOM.

<!-- _render_ palauttaa olion, jolla on useita kenttiä. Yksi kentistä on <i>container</i>, se sisältää koko komponentin renderöimän HTML:n. -->
_render_ returns an object that has several properties. One of the properties is called <i>container</i> and it contains all of the HTML rendered by the component.

<!-- Ekspektaatiossa varmistamme, että komponenttiin on renderöitynyt oikea teksti, eli muistiinpanon sisältö: -->
In the expectation we verify that the component renders the correct text, which in this case is the content of the note:

```js
expect(component.container).toHaveTextContent(
  'Komponenttitestaus tapahtuu react-testing-library:llä'
)
```

<!-- ### Testien suorittaminen -->
### Running tests

<!-- Create-react-app:issa on konfiguroitu testit oletusarvoisesti suoritettavaksi ns. watch-moodissa, eli jos suoritat testit komennolla _npm test_, jää konsoli odottamaan koodissa tapahtuvia muutoksia. Muutosten jälkeen testit suoritetaan automaattisesti ja Jest alkaa taas odottamaan uusia muutoksia koodiin. -->
Create-react-app configures the tests to be run in watch mode by default, which means that the _npm test_ command will not exit once the tests have finished, and will instead wait for changes to be made to the code. Once new changes to the code are saved, the tests are executed automatically after which Jest goes back to waiting for new changes to be made.

<!-- Jos haluat ajaa testit "normaalisti", se onnistuu komennolla -->
If you want to run tests "normally", you can do so with the command:

```js
CI=true npm test
```

<!-- **HUOM:** konsoli saattaa herjata virhettä, jos sinulla ei ole asennettuna watchmania. Watchman on Facebookin kehittämä tiedoston muutoksia tarkkaileva ohjelma. Ohjelma nopeuttaa testien ajoa ja ainakin osx sierrasta ylöspäin jatkuva testien vahtiminen aiheuttaa käyttäjillä virheilmoituksia. Näistä ilmoituksista pääsee eroon asentamalla Watchmanin. -->
**NB:** the console may issue a warning if you have not installed Watchman. Watchman is an application developed by Facebook that watches for changes that are made to files. The program speeds up the execution of tests and at least starting from macOS Sierra, running tests in watch mode issues some warnings to the console, that can be gotten rid of by installing Watchman.

<!-- Ohjeet ohjelman asentamiseen eri käyttöjärjestelmille löydät Watchmanin sivulta: https://facebook.github.io/watchman/ -->
Instructions for installing Watchman on different operating systems can be found from the official Watchman website: https://facebook.github.io/watchman/

<!-- ### Testien sijainti -->
### Test file location

<!-- Reactissa on (ainakin) [kaksi erilaista](https://medium.com/@JeffLombardJr/organizing-tests-in-jest-17fc431ff850) konventiota testien sijoittamiseen. Sijoitimme testit ehkä vallitsevan tavan mukaan, eli samaan hakemistoon missä testattava komponentti sijaitsee. -->
In React there are (at least) [two different conventions](https://medium.com/@JeffLombardJr/organizing-tests-in-jest-17fc431ff850) for test file location. We created our test files according to the current standard by placing them in the same directory as the component being tested.

<!-- Toinen tapa olisi sijoittaa testit "normaaliin" tapaan omaan erilliseen hakemistoon. Valitaanpa kumpi tahansa tapa, on varmaa että se on jonkun mielestä täysin väärä. -->
The other convention is to store the test files "normally" in their own separate directory. Whichever convention we choose, it is almost guaranteed to be wrong according to someone's opinion.

<!-- Itse en pidä siitä, että testit ja normaali koodi ovat samassa hakemistossa. Noudatamme kuitenkin nyt tätä tapaa, sillä se on oletusarvo create-react-app:illa konfiguroiduissa sovelluksissa. -->
Personally, I do not like this way of storing tests and application code in the same directory. The reason we choose to follow this convention is because it is configured by default in applications created by create-react-app.

<!-- ### Sisällön etsiminen testattavasta komponentista -->
### Searching for content in a component

<!-- react-testing-library-kirjasto tarjoaa runsaasti tapoja, miten voimme tutkia testattavan komponentin sisältöä. Laajennetaan testiämme hiukan: -->
The react-testing-library package offers many different ways for investigating the content of the component being tested. Let's expand our test slightly:

```js
test('renders content', () => {
  const note = {
    content: 'Komponenttitestaus tapahtuu react-testing-library:llä',
    important: true
  }

  const component = render(
    <Note note={note} />
  )

  // tapa 1
  expect(component.container).toHaveTextContent(
    'Komponenttitestaus tapahtuu react-testing-library:llä'
  )

  // tapa 2
  const element = component.getByText('Komponenttitestaus tapahtuu react-testing-library:llä')
  expect(element).toBeDefined()

  // tapa 3
  const div = component.container.querySelector('.note')
  expect(div).toHaveTextContent(
    'Komponenttitestaus tapahtuu react-testing-library:llä'
  )
})
```

<!-- Ensimmäinen tapa siis etsii tiettyä tekstiä koko komponentin renderöimästä HTML-koodista.  -->
The first way searches for a matching text from the entire HTML code rendered by the component.

<!-- Toisena käytimme render-metodin palauttamaan olioon liitettyä [getByText](https://testing-library.com/docs/api-queries#getbytext)-metodia, joka palauttaa sen elementin, jolla on määritelty teksti. Jos elementtiä ei ole, tapahtuu poikkeus. Eli mitään ekspektaatiota ei välttämättä edes tarvittaisi. -->
The second way uses the [getByText](https://testing-library.com/docs/api-queries#getbytext) method of the object returned by the render method. The method returns the element that contains the given text. An exception occurs if no such element exists. For this reason we would technically not need to specify any additional expectation.

<!-- Kolmas tapa on etsiä komponentin sisältä tietty elementti metodilla [querySelector](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector), joka saa parametrikseen [CSS-selektorin](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors). -->
The third way is to search for a specific element that is rendered by the component with the [querySelector](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector) method that receives a [CSS-selektorin](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors) as its parameter.

<!-- ### Testien debuggaaminen -->
### Debugging tests

<!-- Testejä tehdessä törmäämme tyypillisesti erittäin moniin ongelmiin.  -->
We typically run into many different kinds of problems when writing our tests.

<!-- Renderin palauttaman olion metodilla [debug](https://testing-library.com/docs/react-testing-library/api#debug) voimme tulostaa komponentin tuottaman HTML:n konsoliin, eli kun muutamme testiä seuraavasti, -->
The object returned by the render method has a [debug](https://testing-library.com/docs/react-testing-library/api#debug) method that can be used to print the HTML rendered by the component to the console. Let's try this out by making the following changes to our code:

```js
test('renders content', () => {
  const note = {
    content: 'Komponenttitestaus tapahtuu react-testing-library:llä',
    important: true
  }

  const component = render(
    <Note note={note} />
  )

  component.debug() // highlight-line

  // ...
})
```

<!-- konsoliin tulostuu komponentin generoima HTML: -->
We can see the HTML generated by the component in the console:

```js
console.log node_modules/react-testing-library/dist/index.js:64
  <body>
    <div>
      <li
        class="note"
      >
        Komponenttitestaus tapahtuu react-testing-library:llä
        <button>
          make not important
        </button>
      </li>
    </div>
  </body>
```

<!-- On myös mahdollista etsiä komponentista pienempi osa, ja tulostaa sen HTML-koodi, tällöin tarvitsemme metodia _prettyDOM_, joka löytyy react-testing-library:n mukana tulevasta kirjastosta <i>dom-testing-library</i>: -->
It is also possible to search for a smaller part of the component and print its HTML code. In order to do this we need the _prettyDOM_ method, that can be imported from the <i>dom-testing-library</i> package that is automatically installed with react-testing-library:

```js
import React from 'react'
import 'jest-dom/extend-expect'
import { render, cleanup } from 'react-testing-library'
import { prettyDOM } from 'dom-testing-library' // highlight-line
import Note from './Note'

test('renders content', () => {
  const note = {
    content: 'Komponenttitestaus tapahtuu react-testing-library:llä',
    important: true
  }

  const component = render(
    <Note note={note} />
  )
  const li = component.container.querySelector('li')
  
  console.log(prettyDOM(li)) // highlight-line
})
```

<!-- Eli haimme selektorin avulla komponentin sisältä <i>li</i>-elementin ja tulostimme sen HTML:n konsoliin: -->
We used the selector to find the <i>li</i> element inside of the component, and printed its HTML to the console:

```js
console.log src/components/Note.test.js:38
  <li
    class="note"
  >
    Komponenttitestaus tapahtuu react-testing-library:llä
    <button>
      make not important
    </button>
  </li>
```

### setup

<!-- react-testing-library:n manuaali kehoittaa kutsumaan jokaisen testin jälkeen metodia [cleanup](https://testing-library.com/docs/react-testing-library/api#cleanup). Hoidimme asian lisäämällä testitiedostoon [afterEach](https://jestjs.io/docs/en/setup-teardown)-määreen, joka kutsuu metodia: -->
The manual for the react-testing-library recommends to call the [cleanup](https://testing-library.com/docs/react-testing-library/api#cleanup) method after every test. We can accomplish with the [afterEach](https://jestjs.io/docs/en/setup-teardown) function, that calls the method:

```js 
import React from 'react'
import 'jest-dom/extend-expect' // highlight-line
import { render, cleanup } from 'react-testing-library'
import { prettyDOM } from 'dom-testing-library'
import Note from './Note'

afterEach(cleanup)  // highlight-line
```

<!-- Voisimme toistaa saman kaikkiin testitiedostoihin. Parempi vaihtoehto on kuitenkin [konfiguroida](https://testing-library.com/docs/react-testing-library/setup) cleanup tapahtumaan automaattisesti. Tehdään konfiguraatiota varten tiedosto <i>src/setupTests.js</i> jolla on seuraava sisältö: -->
We could repeat the same cleanup in all of our test files. A better option is to [configure](https://testing-library.com/docs/react-testing-library/setup) the cleanup to be done automatically. Let's create a new <i>src/setupTests.js</i> file for this configuration with the following contents:

```js
import 'jest-dom/extend-expect'
import 'react-testing-library/cleanup-after-each'
```

<!-- Nyt pääsemme eroon molemmista ylläolevan testikoodin korostetuista riveistä. -->
Now we can get rid of both of the highlighted rows of the test code.

<!-- **HUOM** mikäli testejä suoritettaessa ei löydetä tiedostossa <i>src/setupTests.js</i> tehtyjä konfiguraatioita, auttaa seuraavan asetuksen lisääminen tiedostoon package-lock.json: -->
**NB** if the configuration defined in the <i>src/setupTests.js</i> file is not used when the tests are run, it may help to add the following configuration to the <i>package.json</i> file:

```
  "jest": {
    ...
    "setupFiles": [
      "<rootDir>/src/setupTests.js"
    ],
    ...
  }
```

<!-- ### Nappien painelu testeissä -->
### Clicking buttons in tests

<!-- Sisällön näyttämisen lisäksi toinen <i>Note</i>-komponenttien vastuulla oleva asia on huolehtia siitä, että painettaessa noten yhteydessä olevaa nappia, tulee propsina välitettyä tapahtumankäsittelijäfunktiota _toggleImportance_ kutsua. -->
In addition to displaying content, the <i>Note</i> component also makes sure that when the button associated to the note is pressed, the _toggleImportance_ event handler functions gets called.

<!-- Testaus onnistuu seuraavasti: -->
Testing this functionality can be accomplished like this:

```js
import React from 'react'
import { render, fireEvent } from 'react-testing-library' // highlight-line
import { prettyDOM } from 'dom-testing-library' 
import Note from './Note'

// ...

it('clicking the button calls event handler once', async () => {
  const note = {
    content: 'Komponenttitestaus tapahtuu jestillä ja enzymellä',
    important: true
  }

  const mockHandler = jest.fn()

  const { getByText } = render(
    <Note note={note} toggleImportance={mockHandler} />
  )

  const button = getByText('make not important')
  fireEvent.click(button)

  expect(mockHandler.mock.calls.length).toBe(1)
})
```

<!-- Testissä on muutama mielenkiintoinen seikka. Tapahtumankäsittelijäksi annetaan Jestin avulla määritelty [mock](https://facebook.github.io/jest/docs/en/mock-functions.html)-funktio: -->
There's a few interesting things related to this test. The event handler is [mock](https://facebook.github.io/jest/docs/en/mock-functions.html) function defined with Jest:

```js
const mockHandler = jest.fn()
```

<!-- Testi hakee renderöidystä komponentista napin <i>tekstin perusteella</i> ja klikkaa sitä: -->
The test finds the button <i>based on the text</i> from the rendered component and clicks the element:

```js
const button = getByText('make not important')
fireEvent.click(button)
```

<!-- Klikkaaminen tapahtuu metodin [fireEvent](https://testing-library.com/docs/api-events#fireevent) avulla. -->
Clicking happens with the [fireEvent](https://testing-library.com/docs/api-events#fireevent) method.

<!-- Testin ekspektaatio varmistaa, että <i>mock-funktiota</i> on kutsuttu täsmälleen kerran: -->
The expectation of the test verifies that the <i>mock function</i> has been called exactly once.

```js
expect(mockHandler.mock.calls.length).toBe(1)
```

<!-- [Mockoliot ja -funktiot](https://en.wikipedia.org/wiki/Mock_object) ovat testauksessa yleisesti käytettyjä valekomponentteja, joiden avulla korvataan testattavien komponenttien riippuvuuksia, eli niiden tarvitsemia muita komponentteja. Mockit mahdollistavat mm. kovakoodattujen syötteiden palauttamisen sekä niiden metodikutsujen lukumäärän sekä parametrien testauksen aikaisen tarkkailun. -->
[Mock objects and functions](https://en.wikipedia.org/wiki/Mock_object) are commonly used stub components in testing, that are used for replacing dependencies of the components being tested. Mocks make it possible to return hardcoded responses, and to verify the amount of times the mock functions are called and with what parameters.

<!-- Esimerkissämme mock-funktio sopi tarkoitukseen erinomaisesti, sillä sen avulla on helppo varmistaa, että metodia on kutsuttu täsmälleen kerran. -->
In our example the mock function is a perfect choice, since it can be easily used for verifying that the method gets called exactly once.

<!-- ### Komponentin <i>Togglable</i> testit -->
### Tests for the <i>Togglable</i> component

<!-- Tehdään komponentille <i>Togglable</i> muutama testi. Lisätään komponentin lapset renderöivään div-elementtiin CSS-luokka <i>togglableContent</i>: -->
Let's write a few tests for the <i>Togglable</i> component. Let's add the <i>togglableContent</i> CSS classname to the div that returns the child components.

```js
const Togglable = React.forwardRef((props, ref) => {
  // ...

  return (
    <div>
      <div style={hideWhenVisible}>
        <button onClick={toggleVisibility}>
          {props.buttonLabel}
        </button>
      </div>
      <div style={showWhenVisible} className="togglableContent"> // highlight-line
        {props.children}
        <button onClick={toggleVisibility}>cancel</button>
      </div>
    </div>
  )
})
```

<!-- Testit ovat seuraavassa -->
The tests are shown below:

```js
import React from 'react'
import { render, fireEvent } from 'react-testing-library'
import Togglable from './Togglable'

describe('<Togglable />', () => {
  let component

  beforeEach(() => {
    component = render(
      <Togglable buttonLabel="show...">
        <div className="testDiv" />
      </Togglable>
    )
  })

  it('renders its children', () => {
    component.container.querySelector('.testDiv')
  })

  it('at start the children are not displayed', () => {
    const div = component.container.querySelector('.togglableContent')

    expect(div).toHaveStyle('display: none')
  })

  it('after clicking the button, children are displayed', () => {
    const button = component.getByText('show...')
    fireEvent.click(button)

    const div = component.container.querySelector('.togglableContent')
    expect(div).not.toHaveStyle('display: none')
  })

})
```

<!-- Ennen jokaista testiä suoritettava _beforeEach_ renderöi <i>Togglable</i>-komponentin muuttujaan _component_. -->
The _beforeEach_ function gets called before each test, which then renders the <i>Togglable</i> component into the _component_ variable 

<!-- Ensimmäinen testi tarkastaa, että <i>Togglable</i> renderöi sen lapsikomponentin `<div className="testDiv" />`.  -->
The first test verifies that the <i>Togglable</i> component renders its child component `<div className="testDiv" />`.

<!-- Loput testit varmistavat metodia [toHaveStyle](https://www.npmjs.com/package/jest-dom#tohavestyle) käyttäen, että Togglablen sisältämä lapsikomponentti on alussa näkymättömissä, eli sen sisältävään <i>div</i>-elementtiin liittyy tyyli `{ display: 'none' }`, ja että nappia painettaessa komponentti näkyy, eli näkymättömäksi tekevää tyyliä <i>ei</i> enää ole.  -->
The remaining tests use the [toHaveStyle](https://www.npmjs.com/package/jest-dom#tohavestyle) method to verify that the child component of the <i>Togglable</i> component is not visible initially, by checking that the style of the <i>div</i> element contains `{ display: 'none' }`. Another test verifies that when the button is pressed the component is visible, meaning that the style for hiding the component <i>is no longer</i> assigned to the component.

<!-- Nappi etsitään jälleen nappiin liittyvän tekstin perusteella. Nappi oltaisiin voitu etsiä myös CSS-selektorin avulla -->
The button is searched for once again based on the text that it contains. The button could have been located also with the help of a CSS selector:

```js
const button = component.container.querySelector('button')
```

<!-- Komponentissa on kaksi nappia, mutta koska [querySelector](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector) palauttaa <i>ensimmäisen</i> löytyvän napin, löytyy napeista oikea. -->
The component contains two buttons but since [querySelector](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector) returns the <i>first</i> matching button, we happen to get the button that we wanted.

<!-- Lisätään vielä mukaan testi, joka varmistaa että auki togglattu sisältö saadaan piilotettua painamalla komponentin toisena olevaa nappia -->
Let's also add a test that can be used to verify that the visible content can be hidden by clicking the second button of the component:

```js
it('toggled content can be closed', () => {
  const button = component.querySelector('button')
  fireEvent.click(button)

  const closeButton = component.container.querySelector('button:nth-child(2)')
  fireEvent.click(closeButton)

  const div = component.container.querySelector('.togglableContent')
  expect(div).toHaveStyle('display: none')
})
```

<!-- eli määrittelimme selektorin, joka palauttaa toisena olevan napin `button:nth-child(2)`. Testeissä ei kuitenkaan ole viisasta olla riippuvainen komponentin nappien järjestyksestä, joten parempi onkin hakea napit niiden tekstin perusteella: -->
We defined a selector that returns the second button `button:nth-child(2)`. It's not a wise move to depend on the order of the buttons in the component, and it is recommended to find the elements based on their text:

```js
it('toggled content can be closed', () => {
  const button = component.getByText('show...')
  fireEvent.click(button)

  const closeButton = component.getByText('cancel')
  fireEvent.click(closeButton)

  const div = component.container.querySelector('.togglableContent')
  expect(div).toHaveStyle('display: none')
})
```

<!-- Käyttämämme _getByText_ on vain yksi monista [queryistä](https://testing-library.com/docs/api-queries#queries), joita <i>react-testing-library</i> tarjoaa. -->
The _getByText_ method that we used, is just one of the many [queries](https://testing-library.com/docs/api-queries#queries) <i>react-testing-library</i> offers.

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part2-notes/tree/part5-7), branchissa <i>part5-7</i>. -->
You can find the code for our current application in its entirety in the <i>part5-7</i> branch of [this github repository](https://github.com/fullstack-hy2019/part2-notes/tree/part5-7).
</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- #### 5.13: blogilistan testit, step1 -->
#### 5.13: Blog list tests, step1

<!-- Lisää sovellukseesi tilapäisesti seuraava komponentti -->
Add the following component temporarily to your application:

```js
import React from 'react'

const SimpleBlog = ({ blog, onClick }) => (
  <div>
    <div>
      {blog.title} {blog.author}
    </div>
    <div>
      blog has {blog.likes} likes
      <button onClick={onClick}>like</button>
    </div>
  </div>
)

export default SimpleBlog
```

<!-- Tee testi, joka varmistaa, että komponentti renderöi blogin titlen, authorin ja likejen määrän. -->
Write a test that verifies that the component renders the title, author and amount of likes for the blog post.

<!-- Lisää komponenttiin tarvittaessa testausta helpottavia CSS-luokkia. -->
If necessary, add CSS classes to the component for making testing easier.

<!-- #### 5.14: blogilistan testit, step2 -->
#### 4.14: Blog list tests, step2

<!-- Tee testi, joka varmistaa, että jos komponentin <i>like</i>-nappia painetaan kahdesti, komponentin propsina saamaa tapahtumankäsittelijäfunktiota kutsutaan kaksi kertaa. -->
Write a test that verifies that if the <i>like</i> button of a component is pressed twice, the event handler function passed in the component's props is called twice.

<!-- #### 5.15*: blogilistan testit, step3 -->
#### 5.15*: Blog list tests, step3

<!-- Tee oman sovelluksesi komponentille <i>Blog</i> testit, jotka varmistavat, että oletusarvoisesti blogista on näkyvissä ainoastaan nimi ja kirjoittaja, ja että klikkaamalla niitä saadaan näkyviin myös muut osat blogin tiedoista. -->
Write tests for the <i>Blog</i> component of your application, that verify that by default only the name and author of the blog post is shown. Also verify that when the blog post is clicked, the other information of the blog post become visible.

</div>

<div class="content">

<!-- ### Lomakkeiden testaus -->
### Testing forms

<!-- Käytimme jo edellisissä testeissä [fireEvent](https://testing-library.com/docs/api-events#fireevent)-funktiota nappien klikkaamiseen: -->
In the previous tests we used the [fireEvent](https://testing-library.com/docs/api-events#fireevent) function for clicking buttons:

```js
const button = component.getByText('show...')
fireEvent.click(button)
```

<!-- Käytännössä siis loimme <i>fireEventin</i> avulla tapahtuman <i>click</i> nappia vastaavalle komponentille. Voimme myös simuloida lomakkeisiin kirjoittamista <i>fireEventin</i> avulla. -->
In practice we created used <i>fireEventin</i> to create a <i>click</i> event for the button component. We can also use <i>fireEvent</i>  to simulate filling in forms.

<!-- Tehdään testi komponentille <i>NoteForm</i>. Lomakkeen koodi näyttää seuraavalta -->
Let's write a test for the <i>NoteForm</i> component. The code for the form looks like this:

```js
const NoteForm = ({ onSubmit, handleChange, value }) => {
  return (
    <div>
      <h2>Luo uusi muistiinpano</h2>

      <form onSubmit={onSubmit}>
        <input
          value={value}
          onChange={handleChange}
        />
        <button type="submit">tallenna</button>
      </form>
    </div>
  )
}
```

<!-- Lomakkeen toimintaperiaatteena on synkronoida syötekentän tila sen ulkopuolella olevan React-komponentin tilaan. Lomakettamme on jossain määrin vaikea testata yksistään. -->
The operating principle of the form is to synchronize the state of the input with the state of its parent React component. It is quite difficult to test the form on its own.

<!-- Teemmekin testejä varten apukomponentin <i>Wrapper</i>, joka renderöi <i>NoteForm</i>:in ja hallitsee lomakkeen tilaa parametrinaan saamansa propsin <i>state</i> avulla: -->
For this reason we will create a helper <i>Wrapper</i> component that renders the <i>NoteForm</i> and manages its state with the <i>state</i> prop that it receives:

```js
const Wrapper = (props) => {

  const onChange = (event) => {
    props.state.value = event.target.value
  }

  return (
    <NoteForm
      value={props.state.value}
      onSubmit={props.onSubmit}
      handleChange={onChange}
    />
  )
} 

```

<!-- Testi on seuraavassa: -->
Our test is as follows:

```js
import React from 'react'
import { render, fireEvent } from 'react-testing-library'
import NoteForm from './NoteForm'

const Wrapper = (props) => {
  // ...
}

test('<NoteForm /> updates parent state and calls onSubmit', () => {
  const onSubmit = jest.fn()
  const state = {
    value: ''
  }

  const component = render(
    <Wrapper onSubmit={onSubmit} state={state} />
  )

  const input = component.container.querySelector('input')
  const form = component.container.querySelector('form')

  fireEvent.change(input, { target: { value: 'lomakkeiden testaus on hankalaa' } })
  fireEvent.submit(form)

  expect(onSubmit.mock.calls.length).toBe(1)
  expect(state.value).toBe('lomakkeiden testaus on hankalaa')  
})
```

<!-- Testi luo <i>Wrapper</i>-komponentin, jolle se välittää propseina mockatun funktion _onSubmit_ sekä tilaa edustavan olion <i>state</i>. -->
The test creates a <i>Wrapper</i> component that is passed an _onSubmit_ mock function and a <i>state</i> object for representing the state.

<!-- Wrapper välittää funktion edelleen <i>NoteFormille</i> tapahtuman <i>onSubmit</i> käsittelijäksi ja saamansa propsin <i>state</i> kentän <i>value</i> syötekentän <i>input</i> arvoksi.  -->
The wrapper passes the function on to the <i>NoteForm</i> as its <i>onSubmit</i> event handler, and the the <i>value</i> property of the <i>state</i> prop as the value of the <i>input</i> element.

<!-- Syötekenttään <i>input</i> kirjoittamista simuloidaan tekemällä syötekenttään tapahtuma <i>change</i> ja määrittelemällä sopiva olio, joka määrittelee syötekenttään 'kirjoitetun' sisällön. -->
We simulate writing text into the <i>input</i> element by creating a <i>change</i> event for the input, and by defining a suitable object that defines the content we want to write.

<!-- Lomake lähetetään simuloimalla tapahtuma <i>submit</i> lomakkeelle. -->
The form is submitted by simulating a <i>submit</i> event for the form.

<!-- Testin ensimmäinen ekspektaatio varmistaa, että lomakkeen lähetys on aikaansaanut tapahtumankäsittelijän kutsumisen. Toinen ekspektaatio tutkii komponentille <i>Wrapper</i> propsina välitettyä muuttujaa <i>state</i>, ja varmistaa, että lomakkeelle kirjoitettu teksti on siirtynyt tilaan.  -->
The first expect of the test verifies that submitting the form results in a call to the event handler. The second expect inspects the <i>state</i> object that was passed to the <i>Wrapper</i> component, and verifies that the value that was written in the input is reflected in the state.

<!-- ### Frontendin integraatiotestaus -->
### Frontend integration tests

<!-- Suoritimme edellisessä osassa backendille integraatiotestejä, jotka testasivat backendin tarjoaman API:n läpi backendia ja tietokantaa. Backendin testauksessa tehtiin tietoinen päätös olla kirjoittamatta yksikkötestejä sillä backendin koodi on melko suoraviivaista ja ongelmat tulevatkin esiin todennäköisemmin juuri monimutkaisemmissa skenaarioissa, joita integraatiotestit testaavat hyvin. -->
In the previous part of the course material, we wrote integration tests for the backend that tested its logic and connected database through the API provided by the backend. When writing these tests we made the conscious decision not to write unit tests, as the code for that backend is fairly simple, and it is likely that bugs in our application occur in more complicated scenarios that integration tests are well suited for.

<!-- Toistaiseksi kaikki frontendiin tekemämme testit ovat olleet yksittäisten komponenttien oikeellisuutta valvovia yksikkötestejä. Yksikkötestaus on toki välillä hyödyllistä, mutta kattavinkaan yksikkötestaus ei riitä antamaan riittävää luotettavuutta sille, että järjestelmä toimii kokonaisuudessaan. -->
So far all of our tests for the frontend have been unit tests that have validated the correct functioning of individual components. Unit testing is useful at times but even a comprehensive suite of unit tests is not enough to validate that the application works as a whole.

<!-- Tehdään nyt sovellukselle yksi integraatiotesti. Integraatiotestaus on huomattavasti komponenttien yksikkötestausta hankalampaa. Erityisesti sovelluksemme kohdalla ongelmia aiheuttaa kaksi seikkaa: sovellus hakee näytettävät muistiinpanot palvelimelta <i>ja</i> sovellus käyttää local storagea kirjautuneen käyttäjän tietojen tallettamiseen. -->
Next let's write a single integration test for the application. Writing integration tests is considerably more difficult than writing unit tests for individual components. There are two challenges specific to our application: the application fetches the notes from the backend <i>and</i> the application uses local storage for storing information about the user who is logged in.

<!-- Local storage ei ole oletusarvoiseti käytettävissä testejä suorittaessa, sillä kyseessä on selaimen tarjoama toiminnallisuus ja testit ajetaan selaimen ulkopuolella. Ongelma on helppo korjata määrittelemällä testien suorituksen ajaksi <i>mock</i> joka matkii local storagea. Tapoja tähän on [monia](https://stackoverflow.com/questions/32911630/how-do-i-deal-with-localstorage-in-jest-tests). -->
Local storage is not available to our tests by default, as it is functionality provided by the browser and our tests are not running in the browser. It is quite easy to overcome this challenge by defining a <i>mock</i> that mimics the functionality of the local storage. There are  [many](https://stackoverflow.com/questions/32911630/how-do-i-deal-with-localstorage-in-jest-tests) ways to accomplish this.

<!-- Koska testimme ei edellytä local storagelta juuri mitään toiminnallisuutta, teemme tiedostoon [src/setupTests.js](https://github.com/facebookincubator/create-react-app/blob/ed5c48c81b2139b4414810e1efe917e04c96ee8d/packages/react-scripts/template/README.md#initializing-test-environment) hyvin yksinkertaisen mockin -->
As our tests do not rely on any actual local storage functionality, we will write a very simple mock in the [src/setupTests.js](https://github.com/facebookincubator/create-react-app/blob/ed5c48c81b2139b4414810e1efe917e04c96ee8d/packages/react-scripts/template/README.md#initializing-test-environment) file:

```js
let savedItems = {}

const localStorageMock = {
  setItem: (key, item) => {
    savedItems[key] = item
  },
  getItem: (key) => savedItems[key],
  clear: savedItems = {}
}

window.localStorage = localStorageMock
```

<!-- Toinen ongelmistamme on se, että sovellus hakee näytettävät muistiinpanot palvelimelta. Muistiinpanojen haku tapahtuu heti komponentin <i>App</i> luomisen jälkeen suoritettavassa effect hookissa: -->
Our second challenge is that the application fetches its notes from the server. Fetching the notes happens immediately after the <i>App</i> component is created in the effect hook:


```js
const App = () => {
  // ...

  useEffect(() => {
    noteService
      .getAll().then(initialNotes => {
        setNotes(initialNotes)
      })
  }, [])

// ...
}
```

<!-- Jestin [manual mock](https://facebook.github.io/jest/docs/en/manual-mocks.html#content) -konsepti tarjoaa tilanteeseen hyvän ratkaisun. Manual mockien avulla voidaan kokonainen moduuli, tässä tapauksessa _noteService_ korvata testien ajaksi vaihtoehtoisella esim. kovakoodattua dataa tarjoavalla toiminnallisuudella. -->
The [manual mock](https://facebook.github.io/jest/docs/en/manual-mocks.html#content) concept from Jest provides us with a good solution. With manual mocks we can replace an entire module like _noteService_ with an alternative module that can mock the functionality of the module e.g. by returning hardcoded data. 

<!-- Luodaan Jestin ohjeiden mukaisesti hakemistoon <i>src/services</i> alihakemisto <i>\_\_mocks\_\_</i> (alussa ja lopussa kaksi alaviivaa) ja sinne tiedosto <i>notes.js</i> jonka määrittelemä metodi <i>getAll</i> palauttaa kovakoodatun listan muistiinpanoja: -->
We follow the instructions provided by Jest and create a <i>\_\_mocks\_\_</i> subdirectory under the <i>src/services</i> directory, and inside the directory we create a new <i>notes.js</i> file that defines a <i>getAll</i> function that returns a hardcoded list of notes:

```js
const notes = [
  {
    id: "5a451df7571c224a31b5c8ce",
    content: "HTML on helppoa",
    date: "2019-01-28T16:38:15.541Z",
    important: false,
    user: {
      _id: "5a437a9e514ab7f168ddf138",
      username: "mluukkai",
      name: "Matti Luukkainen"
    }
  },
  {
    id: "5a451e21e0b8b04a45638211",
    content: "Selain pystyy suorittamaan vain javascriptiä",
    date: "2019-01-28T16:38:57.694Z",
    important: true,
    user: {
      _id: "5a437a9e514ab7f168ddf138",
      username: "mluukkai",
      name: "Matti Luukkainen"
    }
  },
  {
    id: "5a451e30b5ffd44a58fa79ab",
    content: "HTTP-protokollan tärkeimmät metodit ovat GET ja POST",
    date: "2019-01-28T16:39:12.713Z",
    important: true,
    user: {
      _id: "5a437a9e514ab7f168ddf138",
      username: "mluukkai",
      name: "Matti Luukkainen"
    }
  }
]

const getAll = () => {
  return Promise.resolve(notes)
}

export default { getAll }
```

<!-- Määritelty metodi _getAll_ palauttaa muistiinpanojen listan käärittynä promiseksi metodin [Promise.resolve](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve) avulla sillä käytettäessä metodia, oletetaan sen paluuarvon olevan promise: -->
The _getAll_ function returns a list of notes wrapped inside of a promise with the [Promise.resolve](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve) method. This is done because our application expects a promise when it uses the method.

<!-- Olemme valmiina määrittelemään testin. Koska kyseessä on koko sovellusta koskeva testi, tehdään se tiedostoon <i>App.test.js</i>: -->
We are ready to write our test. Since this test deals with the entire application, we will write our test in a new <i>App.test.js</i> file:

```js
import React from 'react'
import { render,  waitForElement } from 'react-testing-library'
jest.mock('./services/notes')
import App from './App'

describe('<App />', () => {
  it('renders all notes it gets from backend', async () => {
    const component = render(
      <App />
    )
    component.rerender(<App />)
    await waitForElement(
      () => component.container.querySelector('.note')
    )

    const notes = component.container.querySelectorAll('.note')
    expect(notes.length).toBe(3) 

    expect(component.container).toHaveTextContent(
      'HTML on helppoa'
    )
    expect(component.container).toHaveTextContent(
      'Selain pystyy suorittamaan vain javascriptiä'
    )
    expect(component.container).toHaveTextContent(
      'HTTP-protokollan tärkeimmät metodit ovat GET ja POST'
    )
  })
})
```

<!-- Komennolla <i>jest.mock('./services/notes')</i> otetaan juuri määritelty mock käyttöön. Loogisempi paikka komennolle olisi kenties testien määrittelyt tekevä tiedosto <i>src/setupTests.js</i>. -->
The <i>jest.mock('./services/notes')</i> command takes our mock into use in the test. Perhaps a more logical place for this command would be in the <i>src/setupTests.js</i> file that configures our tests.

<!-- Testi aloittaa renderöimällä komponentin uudelleen `component.rerender(<App />)`, näin varmistetaan että kaikki efektit suoritetaan. Voi kyllä olla, että komento ei ole enää tarpeen uusimpien Reactin versioiden kanssa. -->
The test start by re-rendering the component `component.rerender(<App />)`, this is done to ensure that all of the effects are executed. It is possible that this command is no longer necessary with newer versions of React.

<!-- Koska efektin käynnistämä muistiinpanojen haku palvelimelta on [asynkroninen](https://testing-library.com/docs/api-async) tapahtuma, varmistamme funktion [waitForElement](https://testing-library.com/docs/api-async#waitforelement) avulla, että <i>App</i> ehtii renderöidä muistiinpanot -->
Since the action for fetching notes from the server is an [asynchronous](https://testing-library.com/docs/api-async) event, we use the [waitForElement](https://testing-library.com/docs/api-async#waitforelement) function for verifying that the <i>App</i> component renders all of the notes.

```js
await waitForElement(() => component.container.querySelector('.note'))
```

<!-- Tämän jälkeen teemme varsinaiset expektaatiot, eli varmistetaan että sovelluksessa on kolme CSS-luokalla <i>note</i> merkittyä elementtiä, ja että kaikkien muistiinpanojen sisältö on renderöity. -->
After this we implement the actual expect validations for the test, by verifying that the application has three elements that contain the CSS classname <i>note</i>, and that the content of each note is rendered.

<!-- ### Testauskattavuus -->
### Test coverage

<!-- [Testauskattavuus](https://github.com/facebookincubator/create-react-app/blob/ed5c48c81b2139b4414810e1efe917e04c96ee8d/packages/react-scripts/template/README.md#coverage-reporting) saadaan helposti selville suorittamalla testit komennolla -->
[The test coverage](https://github.com/facebookincubator/create-react-app/blob/ed5c48c81b2139b4414810e1efe917e04c96ee8d/packages/react-scripts/template/README.md#coverage-reporting) of our suite of tests is easy to discover by running the tests with the following command:

```js
CI=true npm test -- --coverage
```

![](../images/5/18.png)

<!-- Melko primitiivinen HTML-muotoinen raportti generoituu hakemistoon <i>coverage/lcov-report</i>. HTML-muotoinen raportti kertoo mm. yksittäisen komponenttien testaamattomat koodirivit: -->
The result is a fairly primitive HTML report that gets generated inside of the <i>coverage/lcov-report</i> directory. The HTML report tells us the untested lines of code in a component:

![](../images/5/19.png)

<!-- Huomaamme, että parannettavaa jäi vielä runsaasti. -->
As we can see, there is a lot of room for improvement.

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part2-notes/tree/part5-8), branchissa <i>part5-8</i>. -->
You can find the code for our current application in its entirety in the <i>part5-8</i> branch of [this github repository](https://github.com/fullstack-hy2019/part2-notes/tree/part5-8).

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- #### 5.16*: blogilistan testit, step4 -->
#### 5.16*: Blog list tests, step4

<!-- Tee sovelluksesi integraatiotesti, joka varmistaa, että jos käyttäjä ei ole kirjautunut järjestelmään, näyttää sovellus ainoastaan kirjautumislomakkeen, eli yhtään blogia ei vielä renderöidä. -->
Write an integration test for your application that verifies that if the user is not logged into the application, the application only displays a login form and no blogs are rendered.

<!-- Testi voi odottaa komponentin sisällön renderöitymistä funktiolla _waitForElement_ -->
The test can wait for the content of the component to render with the _waitForElement_ function.

```js
import React from 'react'
import { 
  render, waitForElement 
} from 'react-testing-library' // highlight-line
jest.mock('./services/blogs')
import App from './App'

describe('<App />', () => {
  it('if no user logged, notes are not rendered', async () => {
    const component = render(
      <App />
    )
    component.rerender(<App />)

// highlight-start
    await waitForElement(
      () => component.getByText('kirjaudu')
    ) 
    // highlight-end

    // expectations here
  })
})
```

<!-- **VAROITUS** kun tein tehtävää, esiintyi testeissä ajoittain epästabiiliutta sen suhteen, toimiko _waitForElement_ tai joku sitä vastaavista asynkronisten operaatioiden odottamiseen tarkoitetuista metodeista. -->
**WARNING** when I was piloting this exercise, there was occasional instability related to _waitForElement_ or any other method intended for waiting for asynchronous operations to finish.

<!-- #### 5.17*: blogilistan testit, step5 -->
#### 5.17*: Blog list tests, step5

<!-- Tee myös testi, joka varmistaa, että kun käyttäjä on kirjautuneena, blogit renderöityvät sivulle. -->
Writer another test that verifies that when the user is logged in, the blog posts are rendered to the page.

<!-- **Vihje:** -->
**Hint:*

<!-- Kirjautuminen kannattaa toteuttaa manipuloimalla testeissä local storagea. Jos määrittelet testeille mock-localstoragen ylläolevaa materiaalia seuraten, voit käyttää testikoodissa local storagea seuraavasti: -->
Logging in is best to implement by mocking and manipulating local storage in the tests. If you define a mocked local storage for the tests according to the instructions, you can then use it in your tests like this:

```js
const user = {
  username: 'tester',
  token: '1231231214',
  name: 'Teuvo Testaaja'
}

localStorage.setItem('loggedBlogAppUser', JSON.stringify(user))
```

</div>

<div class="content">

<!-- ### Snapshot-testaus -->
### Snapshot testing

<!-- Jest tarjoaa "perinteisen" testaustavan lisäksi aivan uudenlaisen tavan testaukseen, ns. [snapshot](https://facebook.github.io/jest/docs/en/snapshot-testing.html)-testauksen. Mielenkiintoista snapshot-testauksessa on se, että sovelluskehittäjän ei tarvitse itse määritellä ollenkaan testejä, snapshot-testauksen käyttöönotto riittää. -->
Jest offers a completely different alternative to "traditional" testing called [snapshot](https://facebook.github.io/jest/docs/en/snapshot-testing.html) testing. The interesting feature of snapshot testing is that developers do not need to define any tests themselves, it is simply enough to adopt snapshot testing. 

<!-- Periaatteena on verrata komponenttien määrittelemää HTML:ää aina koodin muutoksen jälkeen siihen, minkälaisen HTML:n komponentit määrittelivät ennen muutosta. -->
The fundamental principle is to compare the HTML code defined by the component after it has changed to the HTML code that existed before it was changed.

<!-- Jos snapshot-testi huomaa muutoksen komponenttien määrittelemässä HTML:ssä, voi kyseessä joko olla haluttu muutos tai vahingossa aiheutettu "bugi". Snapshot-testi huomauttaa sovelluskehittäjälle, jos komponentin määrittelemä HTML muuttuu. Sovelluskehittäjä kertoo muutosten yhteydessä, oliko muutos haluttu. Jos muutos tuli yllätyksenä, eli kyseessä oli bugi, sovelluskehittäjä huomaa sen snapshot-testauksen ansiosta nopeasti. -->
If the snapshot notices some change in the HTML defined by the component, then either it is new functionality or a "bug" caused by the accident. Snapshot tests notify the developer if the HTML code of the component changes. The developer has to tell Jest if the change was desired or undesired. If the change to the HTML code is unexpected it strongly implicates a bug, and developer can become aware of these potential issues easily thanks to snapshot testing.

<!-- ### End to end -testaus -->
### End to end tests

<!-- Olemme tehneet sekä backendille että frontendille hieman niitä kokonaisuutena testaavia integraatiotestejä. Eräs tärkeä testauksen kategoria on vielä käsittelemättä, [järjestelmää kokonaisuutena](https://en.wikipedia.org/wiki/System_testing) testaavat "end to end" (eli E2E) -testit. -->
We have written integration tests for testing the entire component for both the frontend and the backend. We have not yet taken a look at another important category of tests, that test [the entire system](https://en.wikipedia.org/wiki/System_testing) with "end to end" (E2E) tests.

<!-- Web-sovellusten E2E-testaus tapahtuu simuloidun selaimen avulla esimerkiksi [Selenium](http://www.seleniumhq.org)-kirjastoa käyttäen. Toinen vaihtoehto on käyttää ns. [headless browseria](https://en.wikipedia.org/wiki/Headless_browser) eli selainta, jolla ei ole ollenkaan graafista käyttöliittymää. Esim. Chromea on mahdollista suorittaa Headless-moodissa. -->
The E2E testing of web applications happens by simulating a browser with a library like [Selenium](http://www.seleniumhq.org). Another alternative is to use a so-called [headless browseria](https://en.wikipedia.org/wiki/Headless_browser), that is a browser without a graphical user interface. It's even possible to use Chrome in Headless mode.

<!-- E2E testit ovat potentiaalisesti kaikkein hyödyllisin testikategoria, sillä ne tutkivat järjestelmää saman rajapinnan kautta kuin todelliset käyttäjät. -->
End to end tests have the potential to be the most valuable category of tests, as they inspect the application through the same interface as real end users.

<!-- E2E-testeihin liittyy myös ikäviä puolia. Niiden konfigurointi on haastavampaa kuin yksikkö- ja integraatiotestien. E2E-testit ovat tyypillisesti myös melko hitaita ja isommassa ohjelmistossa niiden suoritusaika voi helposti nousta minuutteihin, tai jopa tunteihin. Tämä on ikävää sovelluskehityksen kannalta, sillä sovellusta koodatessa on erittäin hyödyllistä pystyä ajamaan testejä mahdollisimman usein koodin regressioiden varalta. -->
There are also challenging aspects related to E2E tests. Configuring them is a lot more challenging than configuring unit and integration tests. E2E tests also tend to be quite slow to run, and in a larger application their execution time can range from minutes up to hours. This is unfortunate for application development, as it is extremely useful to be able to run tests as often as possible in order to catch any possible regressions quickly.

<!-- Palaamme end to end -testeihin kurssin viimeisessä, eli seitsemännessä osassa. -->
We will return to the topic of end to end testing in the final part of the course material.

</div>
