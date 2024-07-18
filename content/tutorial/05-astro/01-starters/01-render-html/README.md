# Display HTML text in Astro

## Goal for this pattern

We want to craft Astro components that can render text content in mutliple scenarios.

Let's start with a basic example, a `WelcomeSection` that accepts a string `content` props and renders it.

```astro
/// file: WelcomeSection.astro
---
const { content } = Astro.props
---
<p className="welcome-section">{content}</p>
```

```astro
/// file: Homepage.astro
---
import WelcomeSection from "./WelcomeSection.astro"
const welcomeText = "Hello friends!"
---
<WelcomeSection
  content={welcomeText}
/>
```

But what if `welcomeText` contains HTML? It could be `"Hello <strong>best friends!</strong>"` if we want to emphasize on the "best friends" part.

![The rendered text is literally "Hello <strong>best friends!</strong>"](./public/img/hellobestfriends.png)

We see the `"<strong>"` part, literally! This is not the expected result, what we wanted is to see the "best friends!" part in bold.

It is common to meet HTML strings in real life. In the [State of JavaScript](https://stateofjs.com/) survey,
we have a complex translation management system. We call that "internationalization" or "i18n".  
Our translations are stored in a database and can be written in plain text or in HTML. 

We want to be able to render these strings containing HTML, for example:

```astro
/// file: TranslatedSection.astro
---
//  "Bonjour <strong>les meilleurs amis !</strong>"
const welcomeText = await getTextFromDb()
---
<TranslatedText 
  content={welcomeText}
/>
```

Text content can be even more complicated, sometimes we want to render element instead of just strings, we will also cover this advanced use case later on.

Let's start with HTML strings.

## Rendering HTML strings with the `set:html` directive


So we want to create a component that is able to display strings containing some HTML.

In Astro, we use the [`set:html` directive](https://docs.astro.build/en/reference/directives-reference/#sethtml) to achieve that.

```astro
/// file: TranslatedText.astro
---
// Hello "<strong>best friends!</strong>"
const { content } = Astro.props
---
<span 
 class="translated-text" 
 set:html={content} 
/>
```

> We often apply `set:html` to a [Fragment](https://docs.astro.build/en/basics/astro-syntax/#fragments),
> so that the parent component keeps full control on the rendered HTML.

⚠️ ️Use `set:html` with care: you should never pass untrusted user content there! 

You could allow a script injection attack, also known as [XSS](https://owasp.org/www-community/attacks/xss/) or more precisely [DOM based XSS](https://owasp.org/www-community/attacks/DOM_Based_XSS)).

```astro
/// file: XssInjection.astro
---
const userContent = await getUserContentFromDb()
---
{/** 
BAD!
if "userContent" is "<script src="./hack-your-website.js">",
you could create a severe security breach!
*/}
<p set:html={userContent} />
```

You should try avoiding sending user inputs,
or inputs obtained from a database here.

If you really need to, 
[sanitize](https://en.wikipedia.org/wiki/HTML_sanitization) the input so that valid HTML code is escaped.

> In React, the equivalent to `set:html` is [dangerouslySetInnerHTML](https://legacy.reactjs.org/docs/dom-elements.html#dangerouslysetinnerhtml). 
> The name is more frightening, but we use this pattern less often.

The `set:html` directive is all you need if your text is represented as a string value. But in some even more elaborate scenarios, you may want to render elements, and not just strings.


## Rendering elements with slots

### What is an element?

You can think of a string as a variable with quotes, like `"Hello <strong>best friends!</strong>". An element is the same thing, but without the quotes!

When you write some Astro code, you are creating elements.

```astro
/// file: WelcomeSection.astro
---
// This is a string
const welcomeText = "Hello <strong>best friends!</strong>"
---
<div>
  {/* This is an element */}
  <p>Happy to see you !</p>
</div>
---
```

Sometimes you want to pass elements to other components. 
This is particularly useful for creating reusable UI (user interface) components, 
for instance a "BigText" component that display a text with a very big font size.

In React, you can pass elements as props like shown below:


```jsx
/// file: ReactSection.jsx
/** 
 * This is React code, 
 * but it doesn't work in .astro files :(
 * We need to use "slots" instead 
 */
<ReactBigText content={<p>Some content</p>} />
```

However in Astro this doesn't work and things are a bit more complicated, we need "slots".

### Slots to pass elements to components

Astro has a "slot" mechanism that allows
passing rendered elements to another component.

```astro
/// file: BigText.astro
<slot class="big" name="content"></slot>
```

```astro
/// file: Jumbotron.astro
<BigText>
  <p slot="content">Make me <strong>BIG!</strong></p>
</BigText>
```

If you don't know slots yet,
I invite you to read [the official documentation](https://docs.astro.build/en/basics/astro-components/#slots).

Then, come back here to discover an advanced pattern : accepting both strings and elements in a component ! 

## Handling strings and elements at the same time

### Flexibility for low-level UI components

In a bigger companies, developers can dedicate a lot of time to crafting reusable low-level UI components, for example buttons, menu items, lists etc.

They are useful to guarantee brand consistency across a product and make other developers life easier.

These UI components need to be very flexible, because they can be used in different context. 

For text content, we may want to allow developer to pass either strings, or elements, depending on their needs. For instance a `BigText` component could be used in different manners as show below:

```astro
/// file: BigTextDemo.astro
{/** Basic scenario : just passing a string */}
<BigText content="Hello Big world!" />
{/** Mild scenario : the string has some HTML */}
<BigText content="Hello <strong>Big</string> world!" />
{/** Complex scenario : passing elements via a slot */}
<BigText>
    <p slot="content">A more complicated
    <strong>big world</strong></p>
</BigText>
```

### Astro.slots.render to turn elements into strings

The easiest way to support both strings and elements (`"<span>my text</span>"` or `<span>my text</span>`), is to turn elements into strings before you render them.

To achieve that, you can use the [`Astro.slots.render`](https://docs.astro.build/en/reference/api-reference/#astroslots) utility. It does exactly what it's name implies : it turns a slot element into a simple string.

The point is to remove the need to use a `<slot>` element and manipulate the slot content more freely.

It definitely sounds complicated, but let's see an exemple: 

```astro
/// file: BigText.astro
---
let { content } = Astro.props
if (!content) {
  content =  await Astro.slots.render("content")
}
---
<p set:html={content}></p>
```

THe `BigText` component can now work with either a content prop, or a content slot!

```astro
/// file: BigTextDemo.astro
{/* Will work! */}
<BigText content="hello friends!"/>
{/* Will work! */}
<BigText content="hello <strong>best friends!</strong>" />
{/* Will work! */}
<BigText>hello <strong>best friends!</strong></BigText>
{/* Will work! */}
<BigText>
  hello <AnotherAstroComponent>best friends!</AnotherAstroComponent>
</BigText>
```

### Picking the right pattern

Then, how to choose between using a prop or an element?
It depends on how the text content is generated in the first place.

- if your text is just a string without HTML, you will use a prop
- if you are getting HTML text as a string from a datasource, you will use a prop
- if you are coding a page directly in HTML or Astro code, you will use a slot

## Exercise

### First steps 

1) Implement a `DisplayString` component that can display a string passed as the "content" prop.
2) Implement a `DisplayHtmlString` element that can display a string containing HTML passed as the "content" prop. Use the `set:html` directive
3) Implement a `DisplayElement` component
that can display a slot named "content"

### Advanced

4) Implement a `DisplayHtmlStringOrSlot` component, that can display a string containing HTML passed as the "content" prop, or the "content" slot if this prop is empty. Use [`Astro.slots.render`](https://docs.astro.build/en/reference/api-reference/#astroslotsrender)
5) Implement a `DisplaySlotOrHtmlString` component, that can display the "content" slot, and use the "content" prop as fallback. It's the reverse of the previous component! You can use slots "fallbacks" to implement it.
