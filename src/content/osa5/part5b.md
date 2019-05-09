---
mainImage: ../../images/part-5.svg
part: 5
letter: b
---

<div class="content">

<!-- ### Kirjautumislomakkeen näyttäminen vain tarvittaessa -->
### Displaying the login form only when appropriate

<!-- Muutetaan sovellusta siten, että kirjautumislomaketta ei oletusarvoisesti näytetä: -->
Let's modify the application so that the login form is not displayed by default:

![](../images/5/10.png)

<!-- Lomake aukeaa, jos käyttäjä painaa nappia <i>login</i>: -->
The login form appears when the user presses the <i>login</i> button:

![](../images/5/11a.png)

<!-- Napilla <i>cancel</i> käyttäjä saa tarvittaessa suljettua lomakkeen. -->
The user can close the login form by clicking the <i>cancel</i> button.

<!-- Aloitetaan eristämällä kirjautumislomake omaksi komponentikseen: -->
Let's start by extracting the login form into its own component:

```js
import React from 'react'

const LoginForm = ({
   handleSubmit,
   handleUsernameChange,
   handlePasswordChange,
   username,
   password
  }) => {
  return (
    <div>
      <h2>Kirjaudu</h2>

      <form onSubmit={handleSubmit}>
        <div>
          käyttäjätunnus
          <input
            value={username}
            onChange={handleUsernameChange}
          />
        </div>
        <div>
          salasana
          <input
            type="password"
            value={password}
            onChange={handlePasswordChange}
          />
      </div>
        <button type="submit">kirjaudu</button>
      </form>
    </div>
  )
}

export default LoginForm
```

<!-- Tila ja tilaa käsittelevät funktiot on kaikki määritelty komponentin ulkopuolella ja välitetään komponentille propseina. -->
The state and all the functions related to it are defined outside of the component and are passed to the component as props.

<!-- Huomaa, että propsit otetaan vastaan <i>destrukturoimalla</i>, eli sen sijaan että määriteltäisiin -->
Notice that the props are assigned to variables through <i>destructuring</i>, which means that instead of writing:

```js
const LoginForm = (props) => {
  return (
    <div>
      <h2>Kirjaudu</h2>
      <form onSubmit={props.handleSubmit}>
        <div>
          käyttäjätunnus
          <input
            value={props.username}
            onChange={props.handleChange}
            name="username"
          />
        </div>
        // ...
        <button type="submit">kirjaudu</button>
      </form>
    </div>
  )
}
```

<!-- jolloin muuttujan _props_ kenttiin on viitattava muuttujan kautta esim. _props.handleSubmit_, otetaan kentät suoraan vastaan omiin muuttujiinsa. -->
where the properties of the _props_ object are accessed through e.g. _props.handleSubmit_, the properties are assigned directly to their own variables.

<!-- Nopea tapa toiminnallisuuden toteuttamiseen on muuttaa komponentin <i>App</i> käyttämä funktio _loginForm_ seuraavaan muotoon: -->
One fast way of implementing the functionality is to change the _loginForm_ function of the <i>App</i> component into the following form:

```js
const App = () => {
  const [loginVisible, setLoginVisible] = useState(false) // highlight-line

  // ...

  const loginForm = () => {
    const hideWhenVisible = { display: loginVisible ? 'none' : '' }
    const showWhenVisible = { display: loginVisible ? '' : 'none' }

    return (
      <div>
        <div style={hideWhenVisible}>
          <button onClick={() => setLoginVisible(true)}>log in</button>
        </div>
        <div style={showWhenVisible}>
          <LoginForm
            username={username}
            password={password}
            handleUsernameChange={({ target }) => setUsername(target.value)}
            handlePasswordChange={({ target }) => setPassword(target.value)}
            handleSubmit={handleLogin}
          />
          <button onClick={() => setLoginVisible(false)}>cancel</button>
        </div>
      </div>
    )
  }

  // ...
}
```

<!-- Komponentin <i>App</i> tilaan on nyt lisätty totuusarvo <i>loginVisible</i> joka määrittelee sen, näytetäänkö kirjautumislomake. -->
The <i>App</i> component's state now contains the boolean <i>loginVisible</i>, that defines if the login form should be shown to the user or not.

<!-- Näkyvyyttä säätelevää tilaa vaihdellaan kahden napin avulla, molempiin on kirjoitettu tapahtumankäsittelijän koodi suoraan: -->
The state that defines the visibility is toggled with two buttons. Both buttons have the event handler defined directly in the component:

```js
<button onClick={() => setLoginVisible(true)}>log in</button>

<button onClick={() => setLoginVisible(false)}>cancel</button>
```

<!-- Komponenttien näkyvyys on määritelty asettamalla komponentille [inline](/osa2/tyylien_lisaaminen_react_sovellukseen#inline-tyylit)-tyyleinä CSS-määrittely, jossa [display](https://developer.mozilla.org/en-US/docs/Web/CSS/display)-propertyn arvoksi asetetaan <i>none</i> jos komponentin ei haluta näkyvän: -->
The visibility of the component is defined by giving the component an [inline](/osa2/tyylien_lisaaminen_react_sovellukseen#inline-tyylit) style rule, where the value of the [display](https://developer.mozilla.org/en-US/docs/Web/CSS/display) property is <i>none</i> if we do not want the component to be displayed:

```js
    const hideWhenVisible = { display: loginVisible ? 'none' : '' }
    const showWhenVisible = { display: loginVisible ? '' : 'none' }

<div style="{hideWhenVisible}">
  // nappi
</div>

<div style="{showWhenVisible}">
  // lomake
</div>
```

<!-- Käytössä on taas kysymysmerkkioperaattori, eli jos _loginVisible_ on <i>true</i>, tulee napin CSS-määrittelyksi -->
We are once again using the "question mark" ternary operator. If _loginVisible_ is <i>true</i>, then the CSS rule of the component will be:

```css
display: 'none';
```

<!-- jos _loginVisible_ on <i>false</i>, ei <i>display</i> saa mitään napin näkyvyyteen liittyvää arvoa. -->
If _loginVisible_ is <i>false</i>, then <i>display</i>  will not receive any value related to the visibility of the component.

<!-- ### Komponentin lapset, eli props.children -->
### The component's children, aka. props.children

<!-- Kirjautumislomakkeen näkyvyyttä ympäröivän koodin voi ajatella olevan oma looginen kokonaisuutensa ja se onkin hyvä eristää pois komponentista <i>App</i> omaksi komponentikseen. -->
The code related to managing the visibility of the login form could be considered to be its own logical entity, and for this reason it is good to extract from the <i>App</i> component into its own separate component.

<!-- Tavoitteena on luoda komponentti <i>Togglable</i>, jota käytetään seuraavalla tavalla: -->
Our goal is to implement a new <i>Togglable</i> component that can be used in the following way:

```js
<Togglable buttonLabel='login'>
  <LoginForm
    username={username}
    password={password}
    handleUsernameChange={({ target }) => setUsername(target.value)}
    handlePasswordChange={({ target }) => setPassword(target.value)}
    handleSubmit={handleLogin}
  />
</Togglable>
```

<!-- Komponentin käyttö poikkeaa aiemmin näkemistämme siinä, että käytössä on nyt avaava ja sulkeva tagi, joiden sisällä määritellään toinen komponentti eli <i>LoginForm</i>. Reactin terminologiassa <i>LoginForm</i> on nyt komponentin <i>Togglable</i> lapsi. -->
The way that the component is used is slightly different from our previous components. The component has both an opening and a closing tag that surround another <i>LoginForm</i> component. In React terminology <i>LoginForm</i> is a child component of <i>Togglable</i>.

<!-- <i>Togglablen</i> avaavan ja sulkevan tagin sisälle voi sijoittaa lapsiksi mitä tahansa React-elementtejä, esim.: -->
We can add any React elements we want between the opening and closing tags of <i>Togglable</i>, like this for example:

```js
<Togglable buttonLabel="paljasta">
  <p>tämä on aluksi piilossa</p>
  <p>toinen salainen rivi</p>
</Togglable>
```

<!-- Komponentin koodi on seuraavassa: -->
The code for the <i>Togglable</i> component is shown below:

```js
import React, { useState } from 'react'

const Togglable = (props) => {
  const [visible, setVisible] = useState(false)

  const hideWhenVisible = { display: visible ? 'none' : '' }
  const showWhenVisible = { display: visible ? '' : 'none' }

  const toggleVisibility = () => {
    setVisible(!visible)
  }

  return (
    <div>
      <div style={hideWhenVisible}>
        <button onClick={toggleVisibility}>{props.buttonLabel}</button>
      </div>
      <div style={showWhenVisible}>
        {props.children}
        <button onClick={toggleVisibility}>cancel</button>
      </div>
    </div>
  )
}

export default Togglable
```

<!-- Mielenkiintoista ja meille uutta on [props.children](https://reactjs.org/docs/glossary.html#propschildren), jonka avulla koodi viittaa komponentin lapsiin, eli avaavan ja sulkevan tagin sisällä määriteltyihin React-elementteihin. -->
The new and interesting part of the code is [props.children](https://reactjs.org/docs/glossary.html#propschildren), that is used for referencing the child components of the component. The child components are the React elements that we define between the opening and closing tags of the component.

<!-- Tällä kertaa lapset ainoastaan renderöidään komponentin oman renderöivän koodin seassa: -->
This time the children are rendered in the code that is used for rendering the component itself:

```js
<div style={showWhenVisible}>
  {props.children}
  <button onClick={toggleVisibility}>cancel</button>
</div>
```

<!-- Toisin kuin "normaalit" propsit, <i>children</i> on Reactin automaattisesti määrittelemä, aina olemassa oleva propsi. Jos komponentti määritellään automaattisesti suljettavalla eli _/>_ loppuvalla tagilla, esim. -->
Unlike the "normal" props we've seen before, <i>children</i> is automatically added by React and it always exists. If a component is defined with an automatically closing _/>_ tag, like this:

```js
<Note
  key={note.id}
  note={note}
  toggleImportance={() => toggleImportanceOf(note.id)}
/>
```

<!-- on <i>props.children</i> tyhjä taulukko. -->
Then <i>props.children</i> is an empty array.

<!-- Komponentti <i>Togglable</i> on uusiokäytettävä ja voimme käyttää sitä tekemään myös uuden muistiinpanon luomisesta huolehtivan formin vastaavalla tavalla tarpeen mukaan näytettäväksi. -->
The <i>Togglable</i> component is reusable and we can use it to add similar visibility toggling functionality to the form that is used for creating new notes.

<!-- Eristetään ensin muistiinpanojen luominen omaksi komponentiksi -->
Before we do that, let's extract the form for creating notes into its own component:

```js
const NoteForm = ({ onSubmit, handleChange, value}) => {
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

<!-- ja määritellään lomakkeen näyttävä koodi komponentin <i>Togglable</i> sisällä -->
Next let's define the form component inside of a <i>Togglable</i> component:

```js
<Togglable buttonLabel="new note">
  <NoteForm
    onSubmit={addNote}
    value={newNote}
    handleChange={handleNoteChange}
  />
</Togglable>
```

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part2-notes/tree/part5-4), branchissa <i>part5-4</i>. -->
You can find the code for our current application in its entirety in the <i>part5-4</i> branch of [this github repository](https://github.com/fullstack-hy2019/part2-notes/tree/part5-4).

<!-- ### ref eli viite komponenttiin -->
### References to components with ref

<!-- Ratkaisu on melko hyvä, haluaisimme kuitenkin parantaa sitä erään seikan osalta. -->
Our current implementation is quite good but there is one aspect that could be improved.

<!-- Kun uusi muistiinpano luodaan, olisi loogista jos luomislomake menisi piiloon. Nyt lomake pysyy näkyvillä. Lomakkeen piilottamiseen sisältyy kuitenkin pieni ongelma, sillä näkyvyyttä kontrolloidaan <i>Togglable</i>-komponentin tilassa olevalla muuttujalla <i>visible</i>. Miten pääsemme tilaan käsiksi komponentin ulkopuolelta? -->
When a new note is created, it would make sense to hide the creation form. Currently the form stays visible. There is a slight problem with hiding the form, as the visibility is controlled with the <i>visible</i> variable inside of the <i>Togglable</i> component. How can we access the state outside of the component?

<!-- Reactin [ref](https://reactjs.org/docs/refs-and-the-dom.html)-mekanismi tarjoaa eräänlaisen viitteen komponenttiin. -->
The [ref](https://reactjs.org/docs/refs-and-the-dom.html) mechanism of React offers a certain kind of reference to the component.

<!-- Tehdään komponenttiin <i>App</i> seuraavat muutokset -->
Let's make the following changes to the <i>App</i> component:

```js
const App = () => {
  // ...
  const noteFormRef = React.createRef() // highlight-line

  const noteForm = () => (
    <Togglable buttonLabel="new note" ref={noteFormRef}> // highlight-line
      <NoteForm
        onSubmit={addNote}
        value={newNote}
        handleChange={handleNoteChange}
      />
    </Togglable>
  )

  // ...
}
```

<!-- Metodilla [createRef](https://reactjs.org/docs/react-api.html#reactcreateref) luodaan ref <i>noteFormRef</i>, joka kiinnitetään muistiinpanojen luomislomakkeen sisältävälle <i>Togglable</i>-komponentille. Nyt siis muuttuja <i>noteFormRef</i> toimii viitteenä komponenttiin. -->
The [createRef](https://reactjs.org/docs/react-api.html#reactcreateref) method is used to create a <i>noteFormRef</i> ref, that is assigned to the <i>Togglable</i> component that contains the creation form. The <i>noteFormRef</i> variable functions as a reference to the component.

<!-- Komponenttia <i>Togglable</i> laajennetaan seuraavasti -->
We also make the following changes to the <i>Togglable</i> component:

```js
import React, { useState, useImperativeHandle } from 'react' // highlight-line

const Togglable = React.forwardRef((props, ref) => { // highlight-line
  const [visible, setVisible] = useState(false)

  const hideWhenVisible = { display: visible ? 'none' : '' }
  const showWhenVisible = { display: visible ? '' : 'none' }

  const toggleVisibility = () => {
    setVisible(!visible)
  }

// highlight-start
  useImperativeHandle(ref, () => {
    return {
      toggleVisibility
    }
  })
// highlight-end

  return (
    <div>
      <div style={hideWhenVisible}>
        <button onClick={toggleVisibility}>{props.buttonLabel}</button>
      </div>
      <div style={showWhenVisible}>
        {props.children}
        <button onClick={toggleVisibility}>cancel</button>
      </div>
    </div>
  )
})

export default Togglable
```

<!-- Komponentin luova funktio on kääritty funktiokutsun [forwardRef](https://reactjs.org/docs/react-api.html#reactforwardref) sisälle, näin komponentti pääsee käsiksi sille määriteltyyn refiin. -->
The function that creates the component is wrapped inside of a [forwardRef](https://reactjs.org/docs/react-api.html#reactforwardref) function call. This way the component can access the ref that is assigned to it.

<!-- Komponentti tarjoaa [useImperativeHandle](https://reactjs.org/docs/hooks-reference.html#useimperativehandle)-hookin avulla sisäisesti määritellyn funktionsa <i>toggleVisibility</i> ulkopuolelta kutsuttavaksi. -->
The component uses the [useImperativeHandle](https://reactjs.org/docs/hooks-reference.html#useimperativehandle) hook to make its <i>toggleVisibility</i> function available outside of the component.

<!-- **HUOM** hookin _useImperativeHandle_ vanha nimi on _useImperativeMethod_. Jos käytät Reactin alpha-versiota, on hook siellä vielä vanhalla nimellä! -->
**NB** the old name for _useImperativeHandle_ is _useImperativeMethod_. If you are still using an alpha version of react, then the hook still uses the old name.

<!-- Voimme nyt piilottaa lomakkeen kutsumalla <i>noteFormRef.current.toggleVisibility()</i> samalla kun uuden muistiinpanon luominen tapahtuu: -->
We can now hide the form by calling <i>noteFormRef.current.toggleVisibility()</i> after a new note has been created:

```js
const App = () => {
  // ...
  const addNote = (event) => {
    event.preventDefault()
    noteFormRef.current.toggleVisibility() // highlight-line
    const noteObject = {
      content: newNote,
      date: new Date().toISOString(),
      important: Math.random() > 0.5
    }

    noteService
      .create(noteObject).then(returnedNote => {
        setNotes(notes.concat(returnedNote))
        setNewNote('')
      })
  }
  // ...
}
```

<!-- Käyttämämme [useImperativeHandle](https://reactjs.org/docs/hooks-reference.html#useimperativehandle) on siis React hook, jonka avulla funktiona määritellylle komponentille voidaan määrittää funktioita, joita on mahdollista kutsua sen ulkopuolelta. -->
To recap, the [useImperativeHandle](https://reactjs.org/docs/hooks-reference.html#useimperativehandle) function is a React hook, that is used for defining functions for the component that can be called and invoked from outside of the component.

<!-- Käyttämämme kikka komponentin tilan muuttamikseksi toimii, mutta se vaikuttaa hieman ikävältä. Saman olisi saanut aavistuksen siistimmin toteutettua "vanhan Reactin" class-perustaisilla komponenteilla, joihin tutustumme tämän osan lopussa. Tämä on toistaiseksi ainoa tapaus, jossa Reactin hook-syntaksiin nojaava ratkaisu on aavistuksen likaisemman oloinen kuin class-komponenttien tarjoama ratkaisu. -->
This trick for changing the state of component works but it looks a bit unpleasant. We could have accomplished the same functionality with slightly cleaner code with "old React" class-based components. We will take a look at these class components at the end of this part of the course material. So far this is the only situations where using React hooks leads to code that is not cleaner than with class components.

<!-- Refeille on myös [muita käyttötarkoituksia](https://reactjs.org/docs/refs-and-the-dom.html) kuin React-komponentteihin käsiksi pääseminen. -->
There are also [other use cases](https://reactjs.org/docs/refs-and-the-dom.html) for refs than access React components.

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part2-notes/tree/part5-5), branchissa <i>part5-5</i>. -->
You can find the code for our current application in its entirety in the <i>part5-5</i> branch of [this github repository](https://github.com/fullstack-hy2019/part2-notes/tree/part5-5).

<!-- ### Huomio komponenteista -->
### One point about components

<!-- Kun Reactissa määritellään komponentti -->
When we define a component in React:

```js
const Togglable = () => ...
  // ...
}
```

<!-- ja otetaan se käyttöön seuraavasti, -->
And use it like this:

```js
<div>
  <Togglable buttonLabel="1" ref={togglable1}>
    ensimmäinen
  </Togglable>

  <Togglable buttonLabel="2" ref={togglable2}>
    toinen
  </Togglable>

  <Togglable buttonLabel="3" ref={togglable3}>
    kolmas
  </Togglable>
</div>
```

<!-- syntyy <i>kolme erillistä komponenttiolioa</i>, joilla on kaikilla oma tilansa: -->
We create <i>three separate instances of the component</i> that all have their own separate state:

![](../images/5/12.png)

<!-- <i>ref</i>-attribuutin avulla on talletettu viite jokaiseen komponentin muuttujaan <i>togglable1</i>, <i>togglable2</i> ja <i>togglable3</i>. -->
The <i>ref</i> attribute is used for assigning a reference to each of the components in the variables <i>togglable1</i>, <i>togglable2</i> and <i>togglable3</i>.

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- #### 5.5 blogilistan frontend, step5 -->
#### 5.5 Blog list frontend, step5

<!-- Tee blogin luomiseen käytettävästä lomakkeesta ainoastaan tarvittaessa näytettävä osan 5 luvun [Kirjautumislomakkeen näyttäminen vain tarvittaessa](/osa5#kirjautumislomakkeen-näyttäminen-vain-tarvittaessa) tapaan. Voit halutessasi hyödyntää osassa 5 määriteltyä komponenttia <i>Togglable</i>. -->
Change the form for creating blog posts so that it is only displayed when it is appropriate, with functionality that is similar to what was shown [earlier in this part of the course material](/osa5#kirjautumislomakkeen-näyttäminen-vain-tarvittaessa). If you wish to do so, you can use the <i>Togglable</i> component defined in part 5.

<!-- **HUOM** hookin _useImperativeHandle_ vanha nimi on _useImperativeMethod_. Jos käytät Reactin alpha-versiota, on hook siellä vielä vanhalla nimellä! -->
**NB** the old name for _useImperativeHandle_ is _useImperativeMethod_. If you are still using an alpha version of react, then the hook still uses the old name.

<!-- #### 5.6* blogilistan frontend, step6 -->
#### 5.6* Blog list frontend, step6

<!-- Laajenna blogien listausta siten, että klikkaamalla blogin nimeä, sen täydelliset tiedot aukeavat -->
Modify the blog list so that all of the information about the blog post are displayed when its name is clicked in the list:

![](../images/5/13.png)

<!-- Uusi klikkaus blogin nimeen pienentää näkymän. -->
Clicking the name of an expanded blog post should hide the additional information.

<!-- Napin <i>like</i> ei tässä vaiheessa tarvitse tehdä mitään. -->
At this point the <i>like</i> button does not need to do anything.

<!-- Kuvassa on myös käytetty hieman CSS:ää parantamaan sovelluksen ulkoasua. -->
The application shown in the picture has a bit of additional CSS to improve the appearance of the application.

<!-- Tyylejä voidaan määritellä osan 2 tapaan helposti [inline](/osa2/tyylien_lisaaminen_react_sovellukseen#inline-tyylit)-tyyleinä seuraavasti: -->
It is easy to add styles to the application as shown in part 2 with [inline](/osa2/tyylien_lisaaminen_react_sovellukseen#inline-tyylit) styles as shown below:

```js
const Blog = ({ blog }) => {
  const blogStyle = {
    paddingTop: 10,
    paddingLeft: 2,
    border: 'solid',
    borderWidth: 1,
    marginBottom: 5
  }

  return (
    <div style={blogStyle}>
      <div onClick={() => console.log('clicked')}>  // highlight-line
        {blog.title} {blog.author}
      </div>
      // ...
  </div>
)}
```

<!-- **Huom1:** voit tehdä blogin nimestä klikattavan korostetun koodirivin tapaan. -->

**NB1:** you can make the name of a blog post click-able as shown in the part of the code that is highlighted.

<!-- **Huom2:** vaikka tämän tehtävän toiminnallisuus on melkein samanlainen kuin komponentin <i>Togglable</i> tarjoama toiminnallisuus, ei Togglable kuitenkaan sovi tarkoitukseen sellaisenaan. Helpoin ratkaisu lienee lisätä blogille tila, joka kontrolloi sitä missä muodossa blogi näytetään. -->
**NB2:** even though the functionality implemented in this part is almost identical to the functionality provided by the <i>Togglable</i> component, the component can not be used directly to achieve the desired behavior. The easiest solution will be to add state to the blog post that controls the displayed form of the blog post.

<!-- #### 5.7*: blogilistan frontend, step7 -->
#### 5.7*: Blog list frontend, step7

<!-- Toteuta like-painikkeen toiminnallisuus. Like lisätään backendiin blogin yksilöivään urliin tapahtuvalla _PUT_-pyynnöllä. -->
Implement the functionality for the like button. Likes are increased by making an HTTP _PUT_ request to the unique address of the blog post in the backend.

<!-- Koska backendin operaatio korvaa aina koko blogin, joudut lähettämään operaation mukana blogin kaikki kentät, eli jos seuraavaa blogia liketetään, -->
Since the backend operation replaces the entire blog post, you will have to send all of the fields of the blog post in the request body. IF you wanted to add a like to the following blog post:

```js
{
  _id: "5a43fde2cbd20b12a2c34e91",
  user: {
    _id: "5a43e6b6c37f3d065eaaa581",
    username: "mluukkai",
    name: "Matti Luukkainen"
  },
  likes: 0,
  author: "Joel Spolsky",
  title: "The Joel Test: 12 Steps to Better Code",
  url: "https://www.joelonsoftware.com/2000/08/09/the-joel-test-12-steps-to-better-code/"
},
```

<!-- tulee palvelimelle tehdä PUT-pyyntö osoitteeseen <i>/api/blogs/5a43fde2cbd20b12a2c34e91</i> ja sisällyttää pyynnön mukaan seuraava data: -->
You would have to make an HTTP PUT request to the address <i>/api/blogs/5a43fde2cbd20b12a2c34e91</i> with the following request data:

```js
{
  user: "5a43e6b6c37f3d065eaaa581",
  likes: 1,
  author: "Joel Spolsky",
  title: "The Joel Test: 12 Steps to Better Code",
  url: "https://www.joelonsoftware.com/2000/08/09/the-joel-test-12-steps-to-better-code/"
}
```

<!-- **Varoitus vielä kerran:** jos huomaat kirjoittavasi sekaisin async/awaitia ja _then_-kutsuja, on 99.9% varmaa, että teet jotain väärin. Käytä siis jompaa kumpaa tapaa, älä missään tapauksessa "varalta" molempia. -->
**One last warning:** if you notice that you are using async/await and the _then_-method in the same code, it is almost certain that you are doing something wrong. Stick to using one or the other, and never use both at the same "just in case". 

<!-- #### 5.8*: blogilistan frontend, step8 -->
#### 5.8*: Blog list frontend, step8

<!-- Järjestä sovellus näyttämään blogit <i>likejen</i> mukaisessa suuruusjärjestyksessä. Järjestäminen onnistuu taulukon metodilla [sort](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort). -->
Modify the application to display the blog posts according to the number of <i>likes</i>. Sorting the blog posts can be done with the array [sort](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) method.

<!-- #### 5.9*: blogilistan frontend, step9 -->
#### 5.9*: Blog list frontend, step9

<!-- Lisää nappi blogin poistamiselle. Toteuta myös poiston tekevä logiikka. -->
Add a new button for deleting blog posts. Also implement the logic for deleting blog posts in the backend.

<!-- Ohjelmasi voi näyttää esim. seuraavalta: -->
Your application could look something like this:

![](../images/5/14.png)

<!-- Kuvassa näkyvä poiston varmistus on helppo toteuttaa funktiolla[window.confirm](https://developer.mozilla.org/en-US/docs/Web/API/Window/confirm). -->
The confirmation dialog for deleting a blog post is easy to implement with the [window.confirm](https://developer.mozilla.org/en-US/docs/Web/API/Window/confirm) function.

<!-- #### 5.10*: blogilistan frontend, step10 -->
#### 5.10*: Blog list frontend, step10

<!-- Näytä poistonappi ainoastaan jos kyseessä on kirjautuneen käyttäjän lisäämä blogi. -->
Show the button for deleting a blog post only if the blog post was added by the user.

</div>

<div class="content">

### PropTypes

<!-- Komponentti <i>Togglable</i> olettaa, että sille määritellään propsina <i>buttonLabel</i> napin teksti. Jos määrittely unohtuu, -->
The <i>Togglable</i> component assumes that it is given the text for the button via the <i>buttonLabel</i> prop. If we forget to define it to the component:

```js
<Togglable> buttonLabel unohtui... </Togglable>
```

<!-- sovellus kyllä toimii, mutta selaimeen renderöityy hämäävästi nappi, jolla ei ole mitään tekstiä. -->
The application works, but the browser renders a button that that has no label text.

<!-- Haluaisimmekin varmistaa että jos <i>Togglable</i>-komponenttia käytetään, on propsille "pakko" antaa arvo. -->
We would like to enforce that when the <i>Togglable</i> component is used, the button label text prop must be given a value.

<!-- Komponentin olettamat ja edellyttämät propsit ja niiden tyypit voidaan määritellä kirjaston [prop-types](https://github.com/facebook/prop-types) avulla. Asennetaan kirjasto -->
The expected and required props of a component can be defined and expressed with the [prop-types](https://github.com/facebook/prop-types) package. Let's install the package:

```js
npm install --save prop-types
```

<!-- <i>buttonLabel</i> voidaan määritellä <i>pakolliseksi</i> string-tyyppiseksi propsiksi seuraavasti: -->
We can define the <i>buttonLabel</i> prop as a mandatory or <i>required</i> string-type prop as shown below:

```js
import PropTypes from 'prop-types'

const Togglable = React.forwardRef((props, ref) => {
  // ..
}

Togglable.propTypes = {
  buttonLabel: PropTypes.string.isRequired
}
```

<!-- Jos propsia ei määritellä, seurauksena on konsoliin tulostuva virheilmoitus -->
The console will display the following error message if the prop is left undefined:

![](../images/5/15.png)

<!-- Koodi kuitenkin toimii edelleen, eli mikään ei pakota määrittelemään propseja PropTypes-määrittelyistä huolimatta. On kuitenkin erittäin epäprofessionaalia jättää konsoliin <i>mitään</i> punaisia tulosteita. -->
The application still works and nothing forces us to define props despite the PropTypes definitions. Mind you, it is extremely unprofessional to leave <i>any</i> red output to the browser console.

<!-- Määritellään Proptypet myös <i>LoginForm</i>-komponentille: -->
Let's also defined PropTypes to the <i>LoginForm</i> component:

```js
import PropTypes from 'prop-types'

const LoginForm = ({
   handleSubmit,
   handleUsernameChange,
   handlePasswordChange,
   username,
   password
  }) => {
    // ...
  }

LoginForm.propTypes = {
  handleSubmit: PropTypes.func.isRequired,
  handleUsernameChange: PropTypes.func.isRequired,
  handlePasswordChange: PropTypes.func.isRequired,
  username: PropTypes.string.isRequired,
  password: PropTypes.string.isRequired
}
```

<!-- Funktionaalisen komponentin proptypejen määrittely tapahtuu samalla tavalla kuin luokkaperustaisten. -->
The prop types for functional components are defined the same was as for class components:

<!-- Jos propsin tyyppi on väärä, esim. yritetään määritellä propsiksi <i>handleSubmit</i> merkkijono, seurauksena on varoitus: -->
If the type of a passed prop is wrong, e.g. if we try to define the <i>handleSubmit</i> prop as a string, then this will result in the following warning:

![](../images/5/16.png)

### ESlint

<!-- Konfiguroimme osassa 3 koodin tyylistä huolehtivan [ESlintin](/osa3/validointi_ja_es_lint) backendiin. Otetaan nyt ESlint käyttöön myös frontendissa. -->
In part 3 we configured the [ESlint](/osa3/validointi_ja_es_lint) code style tool to the backend. Let's take ESlint into use in the frontend as well.

<!-- Create-react-app on asentanut projektille eslintin valmiiksi, joten ei tarvita muuta kun sopiva konfiguraatio tiedoston <i>.eslintrc.js</i>. -->
Create-react-app has installed ESlint to the project by default, so all that's left for us to do is to define our desired configuration in the <i>.eslintrc.js</i> file. 

<!-- **HUOM:** älä suorita komentoa _npm init_. Se asentaa uuden version eslintistä joka on epäsopiva create-react-app:in konfiguraatioiden kanssa! -->
**NB:** do not run the _npm init_ command. It will install the latest version of ESlint that is not compatible with the configuration file created by create-react-app!

<!-- Aloitamme seuraavaksi testaamisen, ja jotta pääsemme eroon testeissä olevista turhista huomautuksista asennetaan plugin [eslint-jest-plugin](https://www.npmjs.com/package/eslint-plugin-jest) -->
Next, we will start testing the frontend and in order to avoid undesired and irrelevant linter errors we will install the [eslint-jest-plugin](https://www.npmjs.com/package/eslint-plugin-jest) package:

```js
npm add --save-dev eslint-plugin-jest
```

<!-- Luodaan tiedosto <i>.eslintrc.js</i> ja kopioidaan sinne seuraava sisältö: -->
Let's create a <i>.eslintrc.js</i> file with the following contents:

```js
module.exports = {
    "env": {
        "browser": true,
        "es6": true,
        "jest/globals": true // highlight-line
    },
    // highlight-start
    "extends": [ 
        "eslint:recommended",
        "plugin:react/recommended"
    ],
    // highlight-end
    "parserOptions": {
        "ecmaFeatures": {
            "jsx": true
        },
        "ecmaVersion": 2018,
        "sourceType": "module"
    },
    "plugins": [
        "react", "jest" // highlight-line
    ],
    "rules": {
        "indent": [
            "error",
            2  // highlight-line
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
        ],
        // highlight-start
        "eqeqeq": "error",
        "no-trailing-spaces": "error",
        "object-curly-spacing": [
            "error", "always"
        ],
        "arrow-spacing": [
            "error", { "before": true, "after": true }
        ],
        "no-console": 0,
        "react/prop-types": 0
        // highlight-end
    }
};
```

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/part2-notes/tree/part5-6, branchissa <i>part5-6</i>. -->
You can find the code for our current application in its entirety in the <i>part5-6</i> branch of [this github repository](https://github.com/fullstack-hy2019/part2-notes/tree/part5-6).

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- #### 5.11: blogilistan frontend, step11 -->
#### 5.11: Blog list frontend, step11

<!-- Määrittele joillekin sovelluksesi komponenteille PropTypet. -->
Define PropTypes for one of the components of your application.

<!-- #### 5.12: blogilistan frontend, step12 -->
#### 5.12: Blog list frontend, step12

<!-- Ota projektiin käyttöön ESlint. Määrittele haluamasi kaltainen konfiguraatio. Korjaa kaikki lint-virheet. -->
Add ESlint to the project. Define the configuration according to your liking. Fix all of the linter errors.

</div>
