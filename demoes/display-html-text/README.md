# Display HTML text in Astro

## Goal for this pattern

We want to create a component that is able to display HTML content.

HTML text can appear in two different situations: rendering strings, or rendering elements.

## 1) You want to render a string 

In the State of JavaScript survey,
we have a complex translation management system. We call that "internationalization" or "i18n". 

Our translations are stored in a database. They are not always plain text, they can also be written in HTML or markdown.

We want to be able to render these strings containing HTML, for example:

```astro
<TranslatedText 
content="<strong>Bonjour les amis !</strong>"
/>
```

In Astro, we use the [`set:html` directive](https://docs.astro.build/en/reference/directives-reference/#sethtml) to achieve that.

```astro
---
// TranslatedText.astro
const { content } = Astro.props
---
<div set:html={content} />
```

## 2) You want to render either a prop or an element

Astro has a "slot" mechanism that allows
passing rendered elements to another component.

This is particularly useful for creating reusable UI (user interface) components, for instance a "BigText" component that display an text with a very big font size.

```astro
---
// BigText.astro
---
<slot name="content"></slot>
```

```astro
<BigText>
  <p slot="content">Make me <strong>BIG!</strong></p>
</BigText>
```

If you don't know slots yet,
I invite you to read [the official documentation](https://docs.astro.build/en/basics/astro-components/#slots).

The tricky part is that you may also want to allow simpler strings. 
So you'd want to make your reusable component work in two different scenarios, as shown below.

```astro
{/** Basic scenario : just passing a string */}
<BigText content="Hello Big world!" />
{/** Mild scenario : the string has some HTML */}
<BigText content="Hello <strong>Big</string> world!" />
{/** Or using a complex slot*/}
<BigText>
    <p slot="content">A more complicated
    <strong>big world</strong></p>
</BigText>
```

To achieve that, you can use [`Astro.slots.render`](https://docs.astro.build/en/reference/api-reference/#astroslots).

This function let's you generate the HTML for a given slot, without using the `<slot>` tag. 
This way you can control the slot more precisely using JavaScript.

```astro
---
// BigText.astro
let { content } = Astro.props
if (!content) {
  content =  await Astro.slots.render("content")
}
---
<p set:html={content}></p>
```


## ⚠️ Be careful with `set:html`

Use `set:html` with care: you should never pass untrusted user content there! 
You could allow a script injection attack, also known as [XSS](https://owasp.org/www-community/attacks/xss/) or more precisely [DOM based XSS](https://owasp.org/www-community/attacks/DOM_Based_XSS)).

```jsx
cont userContent = await getUserContentFromDb()
{/** 
BAD!
if "userContent" is "<script src="./hack-your-website.js">",
you could create a severe security breach!
*/}
<p set:html={userContent} />
```

You should try avoiding sending user inputs,
or inputs obtained from a database here.

If you really need too, 
[sanitize](https://en.wikipedia.org/wiki/HTML_sanitization) the input so that valid HTML code is escaped.

## Exercise

### First steps 

1) Implement a `DisplayString` component that can display a string passed as the "content" prop.
2) Implement a `DisplayHtmlString` element that can display a string containing HTML passed as the "content" prop. Use the `set:html` directive
3) Implement a `DisplayElement` component
that can display a slot named "content"

### Advanced

4) Implement a `DisplayHtmlStringOrSlot` component, that can display a string containing HTML passed as the "content" prop, or the "content" slot if this prop is empty. Use [`Astro.slots.render`](https://docs.astro.build/en/reference/api-reference/#astroslotsrender)
5) Implement a `DisplaySlotOrHtmlString` component, that can display the "content" slot, and use the "content" prop as fallback. It's the reverse of the previous component! You can use slots "fallbacks" to implement it.
