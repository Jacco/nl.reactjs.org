---
id: react-component
title: React.Component
layout: docs
category: Reference
permalink: docs/react-component.html
redirect_from:
  - "docs/component-api.html"
  - "docs/component-specs.html"
  - "docs/component-specs-ko-KR.html"
  - "docs/component-specs-zh-CN.html"
  - "tips/UNSAFE_componentWillReceiveProps-not-triggered-after-mounting.html"
  - "tips/dom-event-listeners.html"
  - "tips/initial-ajax.html"
  - "tips/use-react-with-other-libraries.html"
---

Deze pagina bevat een uitgebreide API reference van de React component class. Er wordt ervan uitgegaan dat je bekend bent met fundamentele React concepten, zoals [Componenten en Props](/docs/components-and-props.html), evenals [State en Lifecycle](/docs/state-and-lifecycle.html). Als je dat niet bent, lees er dan eerst over.

## Overzicht {#overview}

React laat je componenten definiëren als classes of functies. Componenten die als classes gedefinieerd zijn bieden momeenteel meer mogelijkheden en die worden op deze pagina in detail beschreven. Om een React component class te definiëren moet je `React.Component` uitbreiden:

```js
class Welcome extends React.Component {
  render() {
    return <h1>Hallo, {this.props.name}</h1>;
  }
}
```

De enige methode die gedefinieerd *moet* worden in een subclass van `React.Component` is [`render()`](#render). Alle andere beschreven methoden op deze pagina zijn optioneel.

**We raden sterk af om je eigen basis class te creëeren.** In React componenten, [wordt hergebruik van code hoofdzakelijk bereikt door compositie in plaats van inheritance](/docs/composition-vs-inheritance.html).

>Opmerking:
>
>React dwingt je niet om de ES6 class syntax te gebruiken. Als je die probeert te vermijden, kun je, in plaats daarvan, de `create-react-class` module gebruiken of een vergelijkbare aangepaste abstractie. Kijk maar eens naar [Gebruik React zonder ES6](/docs/react-without-es6.html) als je er meer van wilt weten.

### De Lifecycle Van Componenten {#the-component-lifecycle}

Iedere component heeft verschillende "lifecycle methods" die je kunt overriden om op specifieke momenten in het process code uit te voeren. **Je kunt [dit lifecycle diagram](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/) gebruiken als spiekbrief** In de onderstaande lijst zijn alledaags gebruikte methoden in **bold** weergegeven. De overigen bestaan voor relatief zeldzame gevallen.

#### Aankoppelen (Mounting) {#mounting}

Deze methoden worden in de volgorde hieronder aangeroepen wanneer er een instance van de component wordt aangemaakt en in de DOM gevoegd wordt:

- [**`constructor()`**](#constructor)
- [`static getDerivedStateFromProps()`](#static-getderivedstatefromprops)
- [**`render()`**](#render)
- [**`componentDidMount()`**](#componentdidmount)

>Opmerking:
>
>De volgende methoden worden als verouderd beschouwd en je moet [ze vermijden](/blog/2018/03/27/update-on-async-rendering.html) in nieuwe code:
>
>- [`UNSAFE_componentWillMount()`](#unsafe_componentwillmount)

#### Bijwerken (Updating) {#updating}

Bijwerken kan veroorzaakt worden door veranderingen in props of state. Deze methoden worden in de volgorde hieronder aangeroepen wanneer de component opnieuw gerenderd wordt:

- [`static getDerivedStateFromProps()`](#static-getderivedstatefromprops)
- [`shouldComponentUpdate()`](#shouldcomponentupdate)
- [**`render()`**](#render)
- [`getSnapshotBeforeUpdate()`](#getsnapshotbeforeupdate)
- [**`componentDidUpdate()`**](#componentdidupdate)

>Opmerking:
>
>De volgende methoden worden als verouderd beschouwd en je moet [ze vermijden](/blog/2018/03/27/update-on-async-rendering.html) in nieuwe code:
>
>- [`UNSAFE_componentWillUpdate()`](#unsafe_componentwillupdate)
>- [`UNSAFE_componentWillReceiveProps()`](#unsafe_componentwillreceiveprops)

#### Afkoppelen (Unmounting) {#unmounting}

Deze medthode wordt aangeroepen wanneer een component uit de DOM verwijderd wordt:

- [**`componentWillUnmount()`**](#componentwillunmount)

#### Fout Afhandeling {#error-handling}

Deze methoden worden aangeroepen wanneer er een fout optreedt tijdens het renderen, in een lifecycle methode, of in de constructor van een child component.

- [`static getDerivedStateFromError()`](#static-getderivedstatefromerror)
- [`componentDidCatch()`](#componentdidcatch)

### Andere APIs {#other-apis}

Iedere component heeft ook APIs anders dan voor de lifecylce:

  - [`setState()`](#setstate)
  - [`forceUpdate()`](#forceupdate)

### Class Properties {#class-properties}

  - [`defaultProps`](#defaultprops)
  - [`displayName`](#displayname)

### Instance Properties {#instance-properties}

  - [`props`](#props)
  - [`state`](#state)

* * *

## Reference {#reference}

### Vaak Gebruikte Lifecycle Methoden {#commonly-used-lifecycle-methods}

De methoden in dit hoofdstuk dekken de grote meerderheid van gevallen die je tegenkomt bij het maken van React componenten. **Bekijk voor een visuele referentie [dit lifecycle diagram](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/).**

### `render()` {#render}

```javascript
render()
```

De `render()` methode is de enige vereiste methode in een class component.

Wanneer hij wordt aangeroepen, zou hij `this.props` en `this.state` moeten bekijken en één van de volgende typen moeten teruggeven:

- **React elementen.** Gebruikelijk aangemaakt met [JSX](/docs/introducing-jsx.html). Bijvoorbeeld, `<div />` en `<MyComponent />` zijn React elementen, die geven React de opdracht geven om respektievelijk een DOM node, of een user-defined component, te renderen.
- **Arrays en fragments.** Laten je meerde elementen teruggeven vanuit render. Bekijk de documentatie over [fragments](/docs/fragments.html) voor meer details.
- **Portals**. Laten je children renderen in een andere DOM subtree. Bekijk de documentatie over [portals](/docs/portals.html) voor meer informatie.
- **String and numbers.** Deze worden gerenderd als tekst nodes in de DOM.
- **Booleans or `null`**. Renderen niets. (Bestaan voornamelijk om het `return test && <Child />` patroon te ondersteunen, waar `test` een boolean is.)

De `render()` functie moet puur zijn, daarmee wordt bedoelt dat hij de state niet aanpast, hij geeft steeds hetzelfde resulaat iedere keer als hij wordt aangeroepen, en heeft geen directe interactie met de browser.

Als je interactie met de browsert nodig hebt, voer je, in plaats daarvan, dat werk uit in `componentDidMount()` of de andere lifecycle methoden. Het puur houden van `render()` maakt componenten eenvoudiger om over na te denken.

> Opmerking
>
> `render()` wordt niet aangeroepen wanneer [`shouldComponentUpdate()`](#shouldcomponentupdate) false teruggeeft.

* * *

### `constructor()` {#constructor}

```javascript
constructor(props)
```

**Als je geen state initialiseert en geen methoden bind hoef je geen constructor te implementeren voor je React component.**

De constructor voor een React component wordt aangeroepen voordat het aangekoppeld (gemount) wordt. Wanneer je de constructor voor een `React.Component` subclass implementeert, moet je `super(props)` aanroepen voor ieder ander statement. Anders zal `this.props` ongedefineerd zijn in de constructor, en dat kan leiden to bugs.

Gewoonlijk worden constructors in React maar om twee redenen gebruikt:

* Het initialiseren van de [lokale state](/docs/state-and-lifecycle.html) door een object toe te wijzen aan `this.state`.
* Het binden van [event handler](/docs/handling-events.html) methoden aan de instance.

Je **moet `setState()` niet aanroepen** in de `constructor()`. In plaats daarvan wijs je, als je component lokale state gebruikt, **de initiële state aan `this.state` toe** meteen in de constructor:

```js
constructor(props) {
  super(props);
  // this.setState() niet aanroepen hier!
  this.state = { counter: 0 };
  this.handleClick = this.handleClick.bind(this);
}
```

De constructor is de enige plek waar je `this.state` direct mag toewijzen. In alle andere methoden moet je in plaats daarvan `this.setState()` gebruiken.

Voorkom het introduceren van neveneffecten of subscriptions in de constructor. Gebruik in plaats daarvan voor die zaken `componentDidMount()`.

>Opmerking
>
>**Voorkom het kopieren van props naar de state! Dit is een veelvoorkomende fout:**
>
>```js
>constructor(props) {
>  super(props);
>  // Niet doen!
>  this.state = { color: props.color };
>}
>```
>
>Het probleem is dat het zowel onnodig is (je kunt `this.props.color` direct gebruiken), als bugs creëert (aanpassingen aan `color` prop worden niet weerspiegeld in de state).
>
>**Gebruik die patroon alleen als je opzettelijk prop aanpassingen wilt negeren.** In dat geval is het zinvol de naam van de prop de wijzigen in `initialColor` of `defaultColor`. Je kunt dan de state van een component indien nodig "resetten" door [zijn `key`] te wijzigen (/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key).
>
>Lees onze [blogpost over het voorkomen van afgeleide state](/blog/2018/06/07/you-probably-dont-need-derived-state.html) om te leren over wat je moet doen als je denk dat je een state nodig hebt die afhangt van de props.


* * *

### `componentDidMount()` {#componentdidmount}

```javascript
componentDidMount()
```

`componentDidMount()` wordt aangeroepen onmiddelijk nadat een component is gemount (toegevoegd aan de boomstructuur). Initialisatie die DOM nodes nodig heeft moet hier komen, Als je data moet inladen van een extern endpoint, is dit een goede plek om het netwerkverzoek te initialiseren.

Deze methode is een goede plaats om subscriptions op te zetten. Als je dat doet, vergeet dan niet dat weer ongedaan te maken in `componentWillUnmount()`.

Je **kunt `setState()` onmiddelijk aanroepen** in `componentDidMount()`. Het zal een extra render starten, maar dat zal gebeuren voordat de browser het scherm bijwerkt. Dit garandeert dat, hoewel de `render()` in dit geval twee keer aangeroepen is, de gebruiker de tussentoestand niet zal zien. Gebruik dit patroon met voorzichtigheid omdat het vaak tot performance problemen kan leiden. In de meeste gevallen zou je in staat moeten zijn om de initiële state toe te wijzen in de `constructor()`. Het kan echter noodzakelijk zijn, voor gevallen zoals modals en tooltips, wanneer je eerst een DOM node moet opmeten, voordat je iets rendert dat afhangt van de grootte of positie ervan.

* * *

### `componentDidUpdate()` {#componentdidupdate}

```javascript
componentDidUpdate(prevProps, prevState, snapshot)
```

`componentDidUpdate()` wordt onmiddelijk aangeroepen nadat een aanpassing plaatsvindt. Deze methode wordt niet aangeroepen voor de initiële render.

Gebruik deze gelegenheid om de DOM te bewerken wanneer de component gewijzigd is. Dit is ook een goede plek om netwerkverzoeken te doen zo lang je de huidge props met de vorige vergelijkt (e.g. een netwerkverzoek is misschien onnodig als de props niet gewijzigd zijn).

```js
componentDidUpdate(prevProps) {
  // Normaal gebruik (vergeet niet de props te vergelijken):
  if (this.props.userID !== prevProps.userID) {
    this.fetchData(this.props.userID);
  }
}
```

Je **kunt `setState()` meteen aanroepen** in `componentDidUpdate()` maar merk op dat **binnen een conditie moet gebeuren** zoals in het voorbeeld hierboven, anders veroorzaak je een oneindige lus. Het zou ook een extra render tot gevolg hebben die, hoewel onzichtbaar voor de gebruiker, wel de performance van het component kan beïnvloeden. Als je probeert een state the spiegelen naar een prop die van boven komt, overweeg dan de prop direct te gebruiken. Lees meer over [waarom het kopiëren van props naar de state tot bugs kan leiden](/blog/2018/06/07/you-probably-dont-need-derived-state.html).

Als je component de `getSnapshotBeforeUpdate()` lifecycle methode implementeert (wat zeldzaam is), zal de waarde die het teruggeeft als derde "snapshot" parameter worden doorgegeven aan `componentDidUpdate()`. Zo niet zal deze parameter ongedefinieerd zijn.

> Opmerking
>
> `componentDidUpdate()` wordt niet aangeroepen als [`shouldComponentUpdate()`](#shouldcomponentupdate) false teruggeeft.

* * *

### `componentWillUnmount()` {#componentwillunmount}

```javascript
componentWillUnmount()
```

`componentWillUnmount()` is invoked immediately before a component is unmounted and destroyed. Perform any necessary cleanup in this method, such as invalidating timers, canceling network requests, or cleaning up any subscriptions that were created in `componentDidMount()`.

You **should not call `setState()`** in `componentWillUnmount()` because the component will never be re-rendered. Once a component instance is unmounted, it will never be mounted again.

* * *

### Rarely Used Lifecycle Methods {#rarely-used-lifecycle-methods}

The methods in this section correspond to uncommon use cases. They're handy once in a while, but most of your components probably don't need any of them. **You can see most of the methods below on [this lifecycle diagram](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/) if you click the "Show less common lifecycles" checkbox at the top of it.**


### `shouldComponentUpdate()` {#shouldcomponentupdate}

```javascript
shouldComponentUpdate(nextProps, nextState)
```

Use `shouldComponentUpdate()` to let React know if a component's output is not affected by the current change in state or props. The default behavior is to re-render on every state change, and in the vast majority of cases you should rely on the default behavior.

`shouldComponentUpdate()` is invoked before rendering when new props or state are being received. Defaults to `true`. This method is not called for the initial render or when `forceUpdate()` is used.

This method only exists as a **[performance optimization](/docs/optimizing-performance.html).** Do not rely on it to "prevent" a rendering, as this can lead to bugs. **Consider using the built-in [`PureComponent`](/docs/react-api.html#reactpurecomponent)** instead of writing `shouldComponentUpdate()` by hand. `PureComponent` performs a shallow comparison of props and state, and reduces the chance that you'll skip a necessary update.

If you are confident you want to write it by hand, you may compare `this.props` with `nextProps` and `this.state` with `nextState` and return `false` to tell React the update can be skipped. Note that returning `false` does not prevent child components from re-rendering when *their* state changes.

We do not recommend doing deep equality checks or using `JSON.stringify()` in `shouldComponentUpdate()`. It is very inefficient and will harm performance.

Currently, if `shouldComponentUpdate()` returns `false`, then [`UNSAFE_componentWillUpdate()`](#unsafe_componentwillupdate), [`render()`](#render), and [`componentDidUpdate()`](#componentdidupdate) will not be invoked. In the future React may treat `shouldComponentUpdate()` as a hint rather than a strict directive, and returning `false` may still result in a re-rendering of the component.

* * *

### `static getDerivedStateFromProps()` {#static-getderivedstatefromprops}

```js
static getDerivedStateFromProps(props, state)
```

`getDerivedStateFromProps` is invoked right before calling the render method, both on the initial mount and on subsequent updates. It should return an object to update the state, or null to update nothing.

This method exists for [rare use cases](/blog/2018/06/07/you-probably-dont-need-derived-state.html#when-to-use-derived-state) where the state depends on changes in props over time. For example, it might be handy for implementing a `<Transition>` component that compares its previous and next children to decide which of them to animate in and out.

Deriving state leads to verbose code and makes your components difficult to think about.  
[Make sure you're familiar with simpler alternatives:](/blog/2018/06/07/you-probably-dont-need-derived-state.html)

* If you need to **perform a side effect** (for example, data fetching or an animation) in response to a change in props, use [`componentDidUpdate`](#componentdidupdate) lifecycle instead.

* If you want to **re-compute some data only when a prop changes**, [use a memoization helper instead](/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization).

* If you want to **"reset" some state when a prop changes**, consider either making a component [fully controlled](/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-controlled-component) or [fully uncontrolled with a `key`](/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key) instead.

This method doesn't have access to the component instance. If you'd like, you can reuse some code between `getDerivedStateFromProps()` and the other class methods by extracting pure functions of the component props and state outside the class definition.

Note that this method is fired on *every* render, regardless of the cause. This is in contrast to `UNSAFE_componentWillReceiveProps`, which only fires when the parent causes a re-render and not as a result of a local `setState`.

* * *

### `getSnapshotBeforeUpdate()` {#getsnapshotbeforeupdate}

```javascript
getSnapshotBeforeUpdate(prevProps, prevState)
```

`getSnapshotBeforeUpdate()` is invoked right before the most recently rendered output is committed to e.g. the DOM. It enables your component to capture some information from the DOM (e.g. scroll position) before it is potentially changed. Any value returned by this lifecycle will be passed as a parameter to `componentDidUpdate()`.

This use case is not common, but it may occur in UIs like a chat thread that need to handle scroll position in a special way.

A snapshot value (or `null`) should be returned.

For example:

`embed:react-component-reference/get-snapshot-before-update.js`

In the above examples, it is important to read the `scrollHeight` property in `getSnapshotBeforeUpdate` because there may be delays between "render" phase lifecycles (like `render`) and "commit" phase lifecycles (like `getSnapshotBeforeUpdate` and `componentDidUpdate`).

* * *

### Error boundaries {#error-boundaries}

[Error boundaries](/docs/error-boundaries.html) are React components that catch JavaScript errors anywhere in their child component tree, log those errors, and display a fallback UI instead of the component tree that crashed. Error boundaries catch errors during rendering, in lifecycle methods, and in constructors of the whole tree below them.

A class component becomes an error boundary if it defines either (or both) of the lifecycle methods `static getDerivedStateFromError()` or `componentDidCatch()`. Updating state from these lifecycles lets you capture an unhandled JavaScript error in the below tree and display a fallback UI.

Only use error boundaries for recovering from unexpected exceptions; **don't try to use them for control flow.**

For more details, see [*Error Handling in React 16*](/blog/2017/07/26/error-handling-in-react-16.html).

> Opmerking
> 
> Error boundaries only catch errors in the components **below** them in the tree. An error boundary can’t catch an error within itself.

### `static getDerivedStateFromError()` {#static-getderivedstatefromerror}
```javascript
static getDerivedStateFromError(error)
```

This lifecycle is invoked after an error has been thrown by a descendant component.
It receives the error that was thrown as a parameter and should return a value to update state.

```js{7-10,13-16}
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}
```

> Opmerking
>
> `getDerivedStateFromError()` is called during the "render" phase, so side-effects are not permitted.
For those use cases, use `componentDidCatch()` instead.

* * *

### `componentDidCatch()` {#componentdidcatch}

```javascript
componentDidCatch(error, info)
```

This lifecycle is invoked after an error has been thrown by a descendant component.
It receives two parameters:

1. `error` - The error that was thrown.
2. `info` - An object with a `componentStack` key containing [information about which component threw the error](/docs/error-boundaries.html#component-stack-traces).


`componentDidCatch()` is called during the "commit" phase, so side-effects are permitted.
It should be used for things like logging errors:

```js{12-19}
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // Example "componentStack":
    //   in ComponentThatThrows (created by App)
    //   in ErrorBoundary (created by App)
    //   in div (created by App)
    //   in App
    logComponentStackToMyService(info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}
```

> Opmerking
> 
> In the event of an error, you can render a fallback UI with `componentDidCatch()` by calling `setState`, but this will be deprecated in a future release.
> Use `static getDerivedStateFromError()` to handle fallback rendering instead.

* * *

### Legacy Lifecycle Methods {#legacy-lifecycle-methods}

The lifecycle methods below are marked as "legacy". They still work, but we don't recommend using them in the new code. You can learn more about migrating away from legacy lifecycle methods in [this blog post](/blog/2018/03/27/update-on-async-rendering.html).

### `UNSAFE_componentWillMount()` {#unsafe_componentwillmount}

```javascript
UNSAFE_componentWillMount()
```

> Opmerking
>
> This lifecycle was previously named `componentWillMount`. That name will continue to work until version 17. Use the [`rename-unsafe-lifecycles` codemod](https://github.com/reactjs/react-codemod#rename-unsafe-lifecycles) to automatically update your components.

`UNSAFE_componentWillMount()` is invoked just before mounting occurs. It is called before `render()`, therefore calling `setState()` synchronously in this method will not trigger an extra rendering. Generally, we recommend using the `constructor()` instead for initializing state.

Avoid introducing any side-effects or subscriptions in this method. For those use cases, use `componentDidMount()` instead.

This is the only lifecycle method called on server rendering.

* * *

### `UNSAFE_componentWillReceiveProps()` {#unsafe_componentwillreceiveprops}

```javascript
UNSAFE_componentWillReceiveProps(nextProps)
```

> Opmerking
>
> This lifecycle was previously named `componentWillReceiveProps`. That name will continue to work until version 17. Use the [`rename-unsafe-lifecycles` codemod](https://github.com/reactjs/react-codemod#rename-unsafe-lifecycles) to automatically update your components.

> Note:
>
> Using this lifecycle method often leads to bugs and inconsistencies
>
> * If you need to **perform a side effect** (for example, data fetching or an animation) in response to a change in props, use [`componentDidUpdate`](#componentdidupdate) lifecycle instead.
> * If you used `componentWillReceiveProps` for **re-computing some data only when a prop changes**, [use a memoization helper instead](/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization).
> * If you used `componentWillReceiveProps` to **"reset" some state when a prop changes**, consider either making a component [fully controlled](/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-controlled-component) or [fully uncontrolled with a `key`](/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key) instead.
>
> For other use cases, [follow the recommendations in this blog post about derived state](/blog/2018/06/07/you-probably-dont-need-derived-state.html).

`UNSAFE_componentWillReceiveProps()` is invoked before a mounted component receives new props. If you need to update the state in response to prop changes (for example, to reset it), you may compare `this.props` and `nextProps` and perform state transitions using `this.setState()` in this method.

Note that if a parent component causes your component to re-render, this method will be called even if props have not changed. Make sure to compare the current and next values if you only want to handle changes.

React doesn't call `UNSAFE_componentWillReceiveProps()` with initial props during [mounting](#mounting). It only calls this method if some of component's props may update. Calling `this.setState()` generally doesn't trigger `UNSAFE_componentWillReceiveProps()`.

* * *

### `UNSAFE_componentWillUpdate()` {#unsafe_componentwillupdate}

```javascript
UNSAFE_componentWillUpdate(nextProps, nextState)
```

> Opmerking
>
> This lifecycle was previously named `componentWillUpdate`. That name will continue to work until version 17. Use the [`rename-unsafe-lifecycles` codemod](https://github.com/reactjs/react-codemod#rename-unsafe-lifecycles) to automatically update your components.

`UNSAFE_componentWillUpdate()` is invoked just before rendering when new props or state are being received. Use this as an opportunity to perform preparation before an update occurs. This method is not called for the initial render.

Note that you cannot call `this.setState()` here; nor should you do anything else (e.g. dispatch a Redux action) that would trigger an update to a React component before `UNSAFE_componentWillUpdate()` returns.

Typically, this method can be replaced by `componentDidUpdate()`. If you were reading from the DOM in this method (e.g. to save a scroll position), you can move that logic to `getSnapshotBeforeUpdate()`.

> Opmerking
>
> `UNSAFE_componentWillUpdate()` will not be invoked if [`shouldComponentUpdate()`](#shouldcomponentupdate) returns false.

* * *

## Other APIs {#other-apis-1}

Unlike the lifecycle methods above (which React calls for you), the methods below are the methods *you* can call from your components.

There are just two of them: `setState()` and `forceUpdate()`.

### `setState()` {#setstate}

```javascript
setState(updater[, callback])
```

`setState()` enqueues changes to the component state and tells React that this component and its children need to be re-rendered with the updated state. This is the primary method you use to update the user interface in response to event handlers and server responses.

Think of `setState()` as a *request* rather than an immediate command to update the component. For better perceived performance, React may delay it, and then update several components in a single pass. React does not guarantee that the state changes are applied immediately.

`setState()` does not always immediately update the component. It may batch or defer the update until later. This makes reading `this.state` right after calling `setState()` a potential pitfall. Instead, use `componentDidUpdate` or a `setState` callback (`setState(updater, callback)`), either of which are guaranteed to fire after the update has been applied. If you need to set the state based on the previous state, read about the `updater` argument below.

`setState()` will always lead to a re-render unless `shouldComponentUpdate()` returns `false`. If mutable objects are being used and conditional rendering logic cannot be implemented in `shouldComponentUpdate()`, calling `setState()` only when the new state differs from the previous state will avoid unnecessary re-renders.

The first argument is an `updater` function with the signature:

```javascript
(state, props) => stateChange
```

`state` is a reference to the component state at the time the change is being applied. It should not be directly mutated. Instead, changes should be represented by building a new object based on the input from `state` and `props`. For instance, suppose we wanted to increment a value in state by `props.step`:

```javascript
this.setState((state, props) => {
  return {counter: state.counter + props.step};
});
```

Both `state` and `props` received by the updater function are guaranteed to be up-to-date. The output of the updater is shallowly merged with `state`.

The second parameter to `setState()` is an optional callback function that will be executed once `setState` is completed and the component is re-rendered. Generally we recommend using `componentDidUpdate()` for such logic instead.

You may optionally pass an object as the first argument to `setState()` instead of a function:

```javascript
setState(stateChange[, callback])
```

This performs a shallow merge of `stateChange` into the new state, e.g., to adjust a shopping cart item quantity:

```javascript
this.setState({quantity: 2})
```

This form of `setState()` is also asynchronous, and multiple calls during the same cycle may be batched together. For example, if you attempt to increment an item quantity more than once in the same cycle, that will result in the equivalent of:

```javaScript
Object.assign(
  previousState,
  {quantity: state.quantity + 1},
  {quantity: state.quantity + 1},
  ...
)
```

Subsequent calls will override values from previous calls in the same cycle, so the quantity will only be incremented once. If the next state depends on the current state, we recommend using the updater function form, instead:

```js
this.setState((state) => {
  return {quantity: state.quantity + 1};
});
```

For more detail, see:

* [State and Lifecycle guide](/docs/state-and-lifecycle.html)
* [In depth: When and why are `setState()` calls batched?](https://stackoverflow.com/a/48610973/458193)
* [In depth: Why isn't `this.state` updated immediately?](https://github.com/facebook/react/issues/11527#issuecomment-360199710)

* * *

### `forceUpdate()` {#forceupdate}

```javascript
component.forceUpdate(callback)
```

By default, when your component's state or props change, your component will re-render. If your `render()` method depends on some other data, you can tell React that the component needs re-rendering by calling `forceUpdate()`.

Calling `forceUpdate()` will cause `render()` to be called on the component, skipping `shouldComponentUpdate()`. This will trigger the normal lifecycle methods for child components, including the `shouldComponentUpdate()` method of each child. React will still only update the DOM if the markup changes.

Normally you should try to avoid all uses of `forceUpdate()` and only read from `this.props` and `this.state` in `render()`.

* * *

## Class Properties {#class-properties-1}

### `defaultProps` {#defaultprops}

`defaultProps` kunnen gedefinieerd worden als een property van de component class zelf, om de standaard props waarden voor de class in te stellen. Die worden gebruikt voor undefined props, maar niet voor null props. Bijvoorbeeld:

```js
class CustomButton extends React.Component {
  // ...
}

CustomButton.defaultProps = {
  color: 'blue'
};
```

Als `props.color` niet is meegegeven, wordt hij op default waarde `'blue'` gezet:

```js
  render() {
    return <CustomButton /> ; // props.color will be set to blue
  }
```

Als `props.color` op null gezet wordt, zal hij null blijven:

```js
  render() {
    return <CustomButton color={null} /> ; // props.color zall null blijven
  }
```

* * *

### `displayName` {#displayname}

De string `displayName` wordt gebruikt in debug berichten. Normaal gesproken hoef je deze niet expliciet in te stellen, omdat hij afgeleid wordt van de naam van de functie of class die de component definieert. Mogelijk wil je hem expliciet instellen op een andere naam voor debug-doeleinden, of wanneer je een higher-order component maakt, zie [Wrap de Display Name om Makkelijk te Debuggen](/docs/higher-order-components.html#convention-wrap-the-display-name-for-easy-debugging) voor details.

* * *

## Instance Properties {#instance-properties-1}

### `props` {#props}

`this.props` bevat de props die bepaald zijn door de aanroeper (caller) van deze component. Bekijk [Componenten en Props](/docs/components-and-props.html) voor een introductie over props.

In het bijzonder is `this.props.children` een speciale prop, meestal gedefinieerd door de child tags in de JSX-expressie in plaats van in de tag zelf.

### `state` {#state}

De state bevat data specifiek voor deze component die over te tijd kan wijzigen. De state wordt door de gebruiker gedefinieerd, en het moet een gewoon JavaScript object zijn.

Als een bepaalde waarde niet wordt gebruikt voor het renderen of dataflow (bijvoorbeeld, een ID van een timer), hoef je die niet in de state op te nemen. Zulke waarden kun je definiëren als velden op de component instance.

Bekijke [State and Lifecycle](/docs/state-and-lifecycle.html) voor meer informatie over de state.

Pas `this.state` nooit direct aan, omdat een aaroep van `setState()` daarna je aanpassing zou kunnen vervangen. Behandel `this.state` alsof het immutable is.
