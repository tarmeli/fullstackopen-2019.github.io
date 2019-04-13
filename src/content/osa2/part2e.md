---
mainImage: ../../images/part-2.svg
part: 2
letter: e
---

<div class="content">

<!-- Sovelluksemme ulkoasu on tällä hetkellä hyvin vaatimaton. Osaan 0 liittyvässä [tehtävässä 0.2](/osa0/web_sovelluksen_toimintaperiaatteita#tehtavia) oli tarkoitus tutustua Mozillan [CSS-tutoriaaliin](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/CSS_basics). -->
The appearance of our current application is quite modest. In [exercise 0.2](/osa0/web_sovelluksen_toimintaperiaatteita#tehtavia), the assignment was to go through Mozilla's [CSS tutorial](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/CSS_basics).

<!-- Katsotaan vielä tämän osan lopussa nopeasti kahta tapaa liittää tyylejä React-sovellukseen. Tapoja on useita ja tulemme tarkastelemaan muita myöhemmin. Liitämme ensin CSS:n sovellukseemme vanhan kansan tapaan yksittäisenä, käsin eli ilman [esiprosessorien](https://developer.mozilla.org/en-US/docs/Glossary/CSS_preprocessor) apua kirjoitettuna tiedostona (tämä ei itseasiassa ole täysin totta, kuten myöhemmin tulemme huomaamaan). -->
Before we move onto the next part, let's take a look at how we can add styles to a React application. There are several different ways of doing this and we will take a look at the other methods later on. First we will add CSS to our application the old-school way, as a single file without using a [CSS preprocessor](https://developer.mozilla.org/en-US/docs/Glossary/CSS_preprocessor) (this is actually not entirely true as we will learn later on). 


<!-- Tehdään sovelluksen hakemistoon <i>src</i> tiedosto <i>index.css</i> ja liitetään se sovellukseen lisäämällä tiedostoon <i>index.js</i> seuraava import: -->
Let's add a new <i>index.css</i> file under the <i>src</i> directory, and then add it to the application by importing it in the <i>index.js</i> file:

```js
import './index.css'
```

<!-- Lisätään seuraava sääntö tiedostoon <i>index.css</i>: -->
Let's add the following CSS rule to the <i>index.css</i> file:

```css
h1 {
  color: green;
}
```

<!-- CSS-säännöt koostuvat valitsimesta, eli <i>selektorista</i> ja määrittelystä eli <i>deklaraatiosta</i>. Valitsin määrittelee, mihin elementteihin sääntö kohdistuu. Valitsimena on nyt <i>h1</i>, eli kaikki sovelluksessa käytetyt <i>h1</i>-otsikkotägit. -->
CSS rules are comprised of <i>selectors</i> and <i>declarations</i>. The selector defines which elements the rule should be applied to. The selector above is <i>h1</i>, which will match all of the <i>h1</i> header tags in our application.

<!-- Määrittelyosa asettaa ominaisuuden _color_, eli fontin värin arvoksi vihreän, eli <i>green</i>. -->
The declaration sets the _color_ property to the value <i>green</i>.

<!-- Sääntö voi sisältää mielivaltaisen määrän määrittelyjä. Muutetaan edellistä siten, että tekstistä tulee kursivoitua, eli fontin tyyliksi asetetaan <i>italics</i>: -->
One CSS rule can contain an arbitrary number of properties. Let's modify the previous rule by making the text cursive, by defining the font style as <i>italic</i>:

```css
h1 {
  color: green;
  font-style: italic;  // highlight-line
}
```

<!-- Erilaisia selektoreja eli tapoja valita tyylien kohde on [lukuisia](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors). -->
There are many ways of matching elements by using [different types of CSS selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors).

<!-- Jos haluamme kohdistaa tyylejä esim. jokaiseen muistiinpanoon, voisimme nyt käyttää selektoria <i>li</i>, sillä muistiinpanot ovat <i>li</i>-tagien sisällä: -->
If we want to target our styles e.g. to each note, we could use the selector <i>li</i>, as all of the notes are inside <i>li</i> tags:

```js
const Note = ({ note, toggleImportance }) => {
  const label = note.important 
    ? 'make not important' 
    : 'make important';

  return (
    <li>
      {note.content} 
      <button onClick={toggleImportance}>{label}</button>
    </li>
  )
}
```

<!-- lisätään tyylitiedostoon seuraava (koska osaamiseni tyylikkäiden web-sivujen tekemiseen on lähellä nollaa, nyt käytettävissä tyyleissä ei ole sinänsä mitään järkeä): -->
Let's add the following rule to our style sheet (since my knowledge of elegant web design is close to zero, the styles don't make much sense):

```css
li {
  color: grey;
  padding-top: 3px;
  font-size: 15px;
}
```

<!-- Tyylien kohdistaminen elementtityypin sijaan on kuitenkin hieman ongelmallista, jos sovelluksessa olisi myös muita  <i>li</i>-tageja, kaikki saisivat samat tyylit. -->
Using element types for defining CSS rules is slightly problematic. If our application contained other <i>li</i> tags, the same style rule would also be applied to them.

<!-- Jos haluamme kohdistaa tyylit nimenomaan muistiinpanoihin, on parempi käyttää [class selectoreja](https://developer.mozilla.org/en-US/docs/Web/CSS/Class_selectors). -->
If we want to apply our style specifically to notes, then it is better to use [class selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/Class_selectors).

<!-- Normaalissa HTML:ssä luokat määritellään elementtien attribuutin  <i>class</i> arvona: -->
In regular HTML, classes are defined as the value of the <i>class</i> attribute:

```html
<li class="note">tekstiä</li>
```

<!-- Reactissa tulee kuitenkin classin sijaan käyttää attribuuttia [className](https://reactjs.org/docs/dom-elements.html#classname), eli muutetaan komponenttia <i>Note</i> seuraavasti: -->
In React we have to use the [className](https://reactjs.org/docs/dom-elements.html#classname) attribute instead of the class attribute. With this in mind, let's make the following changes to our <i>Note</i> component:

```js
const Note = ({ note, toggleImportance }) => {
  const label = note.important 
    ? 'make not important' 
    : 'make important';

  return (
    <li className='note'> // highlight-line
      {note.content} 
      <button onClick={toggleImportance}>{label}</button>
    </li>
  )
}
```

<!-- Luokkaselektori määritellään syntaksilla _.classname_, eli: -->
Class selectors are defined with the _.classname_ syntax:

```css
.note {
  color: grey;
  padding-top: 5px;
  font-size: 15px;
}
```

<!-- Jos nyt lisäät sovellukseen muita li-elementtejä, ne eivät saa muistiinpanoille määriteltyjä tyylejä. -->
If you now add other <i>li</i> elements to the application, they will not be affected by the style rule above.

<!-- ### Parempi virheilmoitus -->
### Improved error message

<!-- Toteutimme äsken olemassaolemattoman muistiinpanon tärkeyden muutokseen liittyvän virheilmoituksen <em>alert</em>-metodilla. Toteutetaan se nyt Reactilla omana komponenttinaan. -->
We previously implemented the error message that was displayed when the user tried to toggle the importance of a deleted note with the <em>alert</em> method. Let's implement the error message as its own React component.

<!-- Komponentti on yksinkertainen: -->
The component is quite simple:

```js
const Notification = ({ message }) => {
  if (message === null) {
    return null
  }

  return (
    <div className="error">
      {message}
    </div>
  )
}
```

<!-- Jos propsin <em>message</em> arvo on <em>null</em> ei renderöidä mitään, muussa tapauksessa renderöidään viesti div-elementtiin. Elementille on liitetty tyylien lisäämistä varten luokka <i>error</i>. -->

If the value of the <em>message</em> prop is <em>null</em>, then nothing is rendered to the screen, and in other cases the message gets rendered inside of a div element.

<!-- Lisätään komponentin <i>App</i> tilaan kenttä <i>error</i> virheviestiä varten, laitetaan kentälle heti jotain sisältöä, jotta pääsemme heti testaamaan komponenttia: -->
Let's add a new piece of state called <i>errorMessage</i> to the <i>App</i> component. Let's initialize it with some error message so that we can immediately test our component:

```js
const App = () => {
  const [notes, setNotes] = useState([]) 
  const [newNote, setNewNote] = useState('')
  const [showAll, setShowAll] = useState(true)
  const [errorMessage, setErrorMessage] = useState('virhe...') // highlight-line

  // ...

  return (
    <div>
      <h1>Muistiinpanot</h1>

      <Notification message={errorMessage} /> // highlight-line
      
      <div>
        <button onClick={() => setShowAll(!showAll)}>
          näytä {showAll ? 'vain tärkeät' : 'kaikki'}
        </button>
      </div>
      <ul>
        {rows()}
      </ul>

      <form onSubmit={addNote}>
        <input
          value={newNote}
          onChange={handleNoteChange}
        />
        <button type="submit">tallenna</button>
      </form>      
    </div>
  )
}
```

<!-- Lisätään sitten virheviestille sopiva tyyli: -->
Then let's add a style rule that suits an error message:

```css
.error {
  color: red;
  background: lightgrey;
  font-size: 20px;
  border-style: solid;
  border-radius: 5px;
  padding: 10px;
  margin-bottom: 10px;
}
```

<!-- Nyt olemme valmiina lisäämään virheviestin logiikan. Muutetaan metodia <em>toggleImportanceOf</em> seuraavasti: -->
Now we are ready to add the logic for displaying the error message. Let's change the <em>toggleImportanceOf</em> function in the following way:

```js
  const toggleImportanceOf = id => {
    const note = notes.find(n => n.id === id)
    const changedNote = { ...note, important: !note.important }

    noteService
      .update(changedNote).then(returnedNote => {
        setNotes(notes.map(note => note.id !== id ? note : returnedNote))
      })
      .catch(error => {
        // highlight-start
        setErrorMessage(
          `muistiinpano '${note.content}' poistettu palvelimelta`
        )
        setTimeout(() => {
          setErrorMessage(null)
        }, 5000)
        // highlight-end
        setNotes(notes.filter(n => n.id !== id))
      })
  }
```

<!-- Eli virheen yhteydessä asetetaan tilaan <em>errorMessage</em> sopiva virheviesti. Samalla käynnistetään ajastin, joka asettaa 5 sekunnin kuluttua tilan <em>errorMessage</em>-kentän arvoksi <em>null</em>. -->
When the error occurs we add a descriptive error message to the <em>errorMessage</em> state. At the same time we start a timer, that will set the <em>errorMessage</em> state to <em>null</em> after five seconds.

<!-- Lopputulos näyttää seuraavalta -->
The result looks like this:

![](../images/2/26b.png)

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/FullStack-HY/part2-notes/tree/part2-7), branchissa <i>part2-7</i>. -->
The code for the current state of our application can be found in the  <i>part2-7</i> branch on [github](https://github.com/FullStack-HY/part2-notes/tree/part2-7).

<!-- ### Inline-tyylit -->
### Inline styles

<!-- React mahdollistaa myös tyylien kirjoittamisen suoraan komponenttien koodin joukkoon niin sanoittuina [inline-tyyleinä](https://react-cn.github.io/react/tips/inline-styles.html). -->
React also makes it possible to write styles directly in the code as so-called [inline styles](https://react-cn.github.io/react/tips/inline-styles.html).

<!-- Periaate inline-tyylien määrittelyssä on erittäin yksinkertainen. Mihin tahansa React-komponenttiin tai elementtiin voi liittää attribuutin [style](https://reactjs.org/docs/dom-elements.html#style), jolle annetaan arvoksi Javascript-oliona määritelty joukko CSS-sääntöjä. -->
The idea behind defining inline styles is extremely simple. Any React component or element can be provided with a set of CSS properties as a JavaScript object through the [style](https://reactjs.org/docs/dom-elements.html#style) attribute.

<!-- CSS-säännöt määritellään Javascriptin avulla hieman eri tavalla kuin normaaleissa CSS-tiedostoissa. Jos haluamme esimerkiksi asettaa jollekin elementille vihreän, kursivoidun ja 16 pikselin korkuisen fontin, eli CSS-syntaksilla ilmaistuna -->
CSS rules are defined slightly differently in JavaScript than in normal CSS files. Let's say that we'd want to give some element a green color, and an italic font with the size of 16 pixels. In CSS it would look like this:

```css
{
  color: green;
  font-style: italic;
  font-size: 16px;
}
```

<!-- tulee tämä muotoilla Reactin inline-tyylin määrittelevänä oliona seuraavasti -->
But as a React inline style object it would look like this:

```js
 {
  color: 'green',
  fontStyle: 'italic',
  fontSize: 16
}
```

<!-- Jokainen CSS-sääntö on olion kenttä, joten ne erotetaan Javascript-syntaksin mukaan pilkuilla. Pikseleinä ilmaistut numeroarvot voidaan määritellä kokonaislukuina. Merkittävin ero normaaliin CSS:ään on väliviivan sisältämien CSS-ominaisuuksien kirjoittaminen camelCase-muodossa. -->
Every CSS property is defined as a separate property of the JavaScript object. Numeric values for pixels can be simply defined as integers. One of the major differences compared to regular CSS, is that hyphenated (kebab case) CSS properties are written in camelCase.

<!-- Voisimme nyt lisätä sovelluksemme "alapalkin", muodostavan komponentin <i>Footer</i>, ja määritellä sille inline-tyylit seuraavasti: -->
Next, we could add a "bottom block" to our application by creating a <i>Footer</i> component and define the following inline styles for it:

```js
const Footer = () => {
  const footerStyle = {
    color: 'green',
    fontStyle: 'italic',
    fontSize: 16
  }

  return (
    <div style={footerStyle}>
      <br />
      <em>Note app, Department of Computer Science 2019</em>
    </div> 
  )
}

const App = () => {
  // ...

  return (
    <div>
      <h1>Muistiinpanot</h1>

      <Notification message={errorMessage} />

      // ...  

      <Footer /> // highlight-line
    </div>
  )
}
```

<!-- Inline-tyyleillä on tiettyjä rajoituksia, esim. ns. pseudo-selektoreja ei ole mahdollisuutta käyttää (ainakaan helposti). -->
Inline selectors come with certain limitations. E.g. so-called pseudo selectors can't be used in any straightforward manner.

<!-- Inline-tyylit ja muutamat myöhemmin kurssilla katsomamme tavat lisätä tyylejä Reactiin ovat periaatteessa täysin vastoin vanhoja hyviä periaatteita, joiden mukaan Web-sovellusten ulkoasujen määrittely eli CSS tulee erottaa sisällön (HTML) ja toiminnallisuuden (Javascript) määrittelystä. Vanha koulukunta pyrkiikin siihen että sovelluksen CSS, HTML ja Javascript on kaikki kirjoitettu omiin tiedostoihinsa. -->
Inline styles and some of the other ways of adding styles to React components go completely against the grain of old conventions. Traditionally it was considered a best practice to separate CSS entirely from content (HTML) and functionality (JavaScript). According to the old school of thought, the goal was to write CSS, HTML, and JavaScript into their own separate files.

<!-- Itseasiassa Reactin filosofia on täysin päinvastainen. Koska CSS:n, HTML:n ja Javascriptin erottelu eri tiedostoihin ei ole kuitenkaan osoittautunut erityisen skaalautuvaksi ratkaisuksi suurissa järjestelmissä, on Reactissa periaatteena tehdä erottelu (eli jakaa sovelluksen koodi eri tiedostoihin) noudattaen sovelluksen loogisia toiminnallisia kokonaisuuksia. -->
The philosophy of React is in fact the polar opposite. Because the separation of CSS, HTML, and JavaScript into separate files never appeared to scale well in larger applications, in React the fundamental principle is to divide the application along the lines of its logical and functional entities.

<!-- Toiminnallisen kokonaisuuden strukturointiyksikkö on React-komponentti, joka määrittelee niin sisällön rakenteen kuvaavan HTML:n, toiminnan määrittelevät Javascript-funktiot kuin komponentin tyylinkin yhdessä paikassa, siten että komponenteista tulee mahdollisimman riippumattomia ja yleiskäyttöisiä. -->
The unit of a functional entity is the React component, that defines both the structure expressed through HTML, its functionality defined as JavaScript functions, and also its style in one place, so that the individual components become as independent and reusable as possible.

</div>

<div class="tasks">

<!-- <h3>Tehtäviä</h3> -->
<h3>Exercises</h3>

<!-- <h4>2.19: puhelinluettelo step11</h4> -->
<h4>2.19: Phonebook step11</h4>

<!-- Toteuta osan 2 esimerkin [parempi virheilmoitus](/osa2/tyylien_lisaaminen_react_sovellukseen#parempi-virheilmoitus) tyyliin ruudulla muutaman sekunnin näkyvä ilmoitus, joka kertoo onnistuneista operaatioista (henkilön lisäys ja poisto, sekä numeron muutos): -->
Use the [parempi virheilmoitus](/osa2/tyylien_lisaaminen_react_sovellukseen#parempi-virheilmoitus) example from part 2 as a guide, for showing a notification that lasts for a few seconds when a successful operation is executed (a person is added or a number is changed): 

![](../images/2/27b.png)

<!-- <h4>2.20*: puhelinluettelo step12</h4> -->
<h4>2.20*: Phonebook step12</h4>

<!-- Avaa sovelluksesi kahteen selaimeen. **Jos poistat jonkun henkilön toisesta selaimesta 1** hieman ennen kun yrität <i>muuttaa henkilön numeroa</i> selaimesta 2, tapahtuu virhetilanne: -->
Open your application in two browsers. **If you delete some person from browser 1** a little bit before you try to <i>change the person's phone number</i> from browser 2, you will get the following error message:

![](../images/2/29b.png)

<!-- Korjaa ongelma osan 2 esimerkin [promise ja virheet](/osa2/palvelimella_olevan_datan_muokkaaminen#promise-ja-virheet) hengessä, mutta siten että  -->
 <!-- käyttäjälle ilmoitetaan operaation epäonnistumisesta. Onnistuneen ja epäonnistuneen operaation ilmoitusten tulee erota toisistaan:  -->
Fix the issue according to the example shown in [promise ja virheet](/osa2/palvelimella_olevan_datan_muokkaaminen#promise-ja-virheet) in part 2. Modify the example so that the user is shown a message when the operation does not succeed. The messages shown for successful and unsuccessful events should look different:

![](../images/2/28a.png)

<!-- Tämä oli osan viimeinen tehtävä ja on aika pushata koodi githubiin merkata tehdyt tehtävät [palautussovellukseen](https://studies.cs.helsinki.fi/fullstackopen2019/). -->
This was the last exercise of this part of the course. It's time to push your code to GitHub and mark all of your finished exercises to the [exercise submission system](https://studies.cs.helsinki.fi/fullstackopen2019).

</div>
