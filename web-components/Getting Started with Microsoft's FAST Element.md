---
title: Getting Started with Microsoft's FAST Element
published: false
description: A quick guide on getting up and running with Microsoft's FAST Element
tags: #webcomponents, #webdev, #javascript, #html
//cover_image: https://direct_url_to_image.jpg
---

If you haven't heard already, web components have started to take off in popularity. The features they bring to the table is very compelling. Among them are: framework-agnostic reusable components, strong style encapsulation, and blazingly fast performance.

A few of the more popular tools for building web component libraries include [lit](https://lit.dev), [StencilJS](https://stenciljs.com/), and even popular JavaScript frameworks (you can play with some of them at [webcomponents.dev](https://webcomponents.dev/)), but for the last few weeks I have had the opportunity to work with [Microsoft's FAST Element](https://fast.design) and have had fun working with it.

One thing that I struggled with as I started out, was finding a standard way to easily stand up dev environments so I could experiment and ideate with FAST components. The FAST team doesn't provide a way (yet), so I went ahead and [built a quick one](https://www.npmjs.com/package/create-fast-element) for us to experiment with. I will be using it to generate the examples used in this article.

In your terminal or command environment run the following command and follow the instructions to set up your local environment:

```bash
npm init fast-element my-fast-components
```

## Creating a New Component

In the `/src` directory, let's create a new folder and file called `/my-search/index.ts` and add our component definition to it.

```ts
import { FASTElement, customElement } from '@microsoft/fast-element';

/**
 * @tag my-search
 * 
 * @summary This is a search component
 *
 */
@customElement('my-search')
export class MySearch extends FASTElement {

}
```

Here we are using FAST's library to define and create our custom element - `<my-search></my-search>` - but we will also be using [jsDoc](https://jsdoc.app/) to help document our custom element, integrate it with the [Storybook](https://storybook.js.org/), and generate our [custom element manifest](https://custom-elements-manifest.open-wc.org/analyzer/getting-started/).

Now we can export our component from the `./src/index.ts` to include it with the rest of our components in our library.

```ts
export * from './my-counter';
export * from './my-search'; // added for our new component
```

### Adding Stories

Storybook provides us with a great workspace for us to build and experiment with our components. Once we set up the initial file, our custom elements manifest will handle a lot of the heavy lifting for us.

To get started, create a file called `/my-search/my-search.stories.ts` and add the following contents:

```ts
import "./index";

export default {
    title: 'Components/My Search',
    component: 'my-search'
};

const Template = (args: any) => `
    <my-search></my-search>
`;

export const Default: any = Template.bind({});
Default.args = {};
```

Now, we can start Storybook with the following command and we should see a new section on in the left column - `Components > My Search > Default`.

```bash
npm run dev
```

If you see a blank page when you click on the `Default` page, don't worry. We haven't added anything for our component to render yet. Let's do that now.

### Adding HTML

To add HTML to our component, let's update out component's class decorator with the following code:

```ts
@customElement({
    name: 'my-search',
    template: html`
        <label>
            My Search
            <input type="search" />
        </label>
        <button>Search</button>
    `
})
```

If your editor didn't already, you will need to make sure you update your import statement to include the `html` string template decorator.

```ts
import { FASTElement, customElement, html } from '@microsoft/fast-element';
```

You should now see the label, input field, and search button for our component rendered in the `Default` Storybook page.

## Attributes or Properties

Regular HTML element have attributes (sometimes called properties) that you can pass values to in order to create a specific behavior. For example, the `input` element has attributes like `type`, `name`, `value`, and `disabled`. Depending on the values that are provided to them, the element will behave in a certain way. When we create custom elements or web components, we can define our own attributes and map them to a behavior.

Let's start with making it possible to change the label for the input field. FAST uses the `@attr` decorator to identify these fields. We can add it to our component class along with the type and default value.

```ts
export class MySearch extends FASTElement {
    @attr label: string = 'My Search';
}
```

Again, you will need to update import statement to include the new `attr` decorator.

```ts
import { FASTElement, customElement, html, attr } from '@microsoft/fast-element';
```

Also, make sure to update the jsDoc comment above the class so that the values can be added to the custom element manifest and synced up with Storybook.

```ts
/**
 * @tag my-search
 * 
 * @summary This is a search component
 *
 * @attr {string} label - the label associated with the search field
 * 
 */
```

### Binding Attributes to Templates

To help provide some autocomplete functionality, we can add our component's class as a type on our template string.

```ts
template: html<MySearch>`
```

Now, let's replace the "My Search" text with the value provided in the attribute field. We can do this with some template string interpolation and an arrow function (this helps with efficient template updates).

```ts
template: html<MySearch>`
    <label>
        ${x => x.label}
        <input type="search" />
    </label>
    <button>Search</button>
`
```

### Adding Attributes in Storybook

If we update the template in our `my-search.stories.ts` with a `label` attribute and value, we should see it reflected in our Storybook page.

```ts
const Template = (args: any) => `
    <my-search label="Site Search"></my-search>
`;
```

Now let's make this a little more useful by wiring it up to the Storybook page so we can adjust the values without having to make any code changes.

```ts
const Template = (args: any) => `
    <my-search label="${args.label}"></my-search>
`;
```

***NOTE:*** I'm not sure if this is a bug or if this feature is coming soon, but for some reason the default value is not automatically defined, so we will need to add it to the `args` section of our default export.

```ts
export default {
    title: 'Components/My Search',
    component: 'my-search',
    args: {
       label: 'My Search' 
    }
};
```

If you haven't noticed already, In the "Addons" panel under the "Controls" tab you should see a section called "Properties" with an input for the `label` attribute.

If you don't see the "Addons" panel at the right or bottom of where your component is rendered, click the menu button in the upper-right side of the page and select "Show addons" (or press `A` on your keyboard). We will be using that a lot.

## Slots

Attributes are a great way to pass data like `strings`, `numbers`, `objects`, and `arrays` into components, but sometimes you need to be able to pass markup or HTML into a component. That's exactly what [slots](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/slot) are for. It is basically a placeholder for your content to go.

In our component, we will use a `slot` so we can pass content into our search `button`. Let's start by updating our template with a slot inside of the button. We can also define default content, if nothing is supplied, by adding between the opening and closing `<slot>` tags.

```ts
template: html<MySearch>`
    <label>
        ${x => x.label}
        <input type="search" />
    </label>
    <button>
        <slot>Search</slot>
    </button>
`
```

Let's also update our jsDoc comments above our component to include it in our custom elements manifest.

```ts
/**
 * @tag my-search
 * 
 * @summary This is a search component
 *
 * @attr {string} label - the label associated with the search field
 * 
 * @slot default - content displayed in the search button
 * 
 */
```

You should now see a new section in your Storybook controls called "Slots". Now we can wire that attribute up in our template in the `my-search.stories.ts` file as well as a default value for the argument.

```ts
export default {
    title: 'Components/My Search',
    component: 'my-search',
    args: {
       label: 'My Search',
       default: 'Search'
    }
};

const Template = (args: any) => `
    <my-search label="${args.label}">${args.default}</my-search>
`;
```

New we can pass in any value we want like "Submit" or an emoji ("😉"). We can even create a new template and pull in an icon library.

```ts
// using Bootstrap icons - https://icons.getbootstrap.com/
const IconTemplate = (args: any) => `
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.8.0/font/bootstrap-icons.css" />
    <my-search label="${args.label}">${args.default}</my-search>
`;

export const Icon: any = IconTemplate.bind({});
Icon.args = {
    default: "<i class=\"bi bi-search\"></i>"
};
```

## Events

We can sue `attributes` and `slots` to pass data into our components, but sometimes we need to get data out of our components. We can do this through emitting events.

We interact with native HTMl elements all of the time - `onClick`, `onInput`, `onBlur`, etc. FAST makes this pretty easy for us using the `$emit()` method provided in the `FASTElement` class our component inherits from.

### Listening for Events

In our component, we want to emit an event any time a user triggers the [search](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/search_event) event on our input or clicks on our search button. To do this, let's add two event handler methods to our component class that will emit our own `find` event.

```ts
export class MySearch extends FASTElement {
    @attr label: string = 'My Search';

    private searchHandler() {
        this.$emit('find');
    }

    private buttonClickHandler() {
        this.$emit('find');
    }
}
```

Now we can wire them up to our template.

```ts
template: html<MySearch>`
    <label>
        ${x => x.label}
        <input type="search" @search="${x => x.searchHandler()}" />
    </label>
    <button @click="${x => x.buttonClickHandler()}">
        <slot>Search</slot>
    </button>
`,
```

As you can see, FAST uses a different syntax for [listening for events](https://www.fast.design/docs/fast-element/declaring-templates#events) on elements. Rather than the normal `onSearch` or `onClick`, they use `@search` and `@click`. If you have ever used [Vue.js](https://vuejs.org/), then FAST's attribute and event binding syntax should look familiar.

Now we should be able to listen for an `onFind` event on our `<my-search>` element. You can do this by adding an `onFind` attribute to your element or using JavaScript to select our element and adding an event listener.

```ts
// select the element from the DOM
const mySearch = document.querySelector('my-element');

// add an event listenr for our custom event and log it to the console
mySearch.addEventListener('find', e => console.log(e));
```

### Capturing Events in Storybook

Rather than having to constantly add event listeners to our code or to the console any time we want to test our events, we can actually wire them up in Storybook and it will capture it for us. In our story's default export, we add a new `parameters` property to the object where we can define our custom events.

```ts
export default {
    title: 'Components/My Search',
    component: 'my-search',
    args: {
       label: 'My Search',
       default: 'Search',
       ['--font-size']: '1rem',
       ['--padding']: '0.25rem'
    },
    parameters: {
        actions: {
            handles: ['find'],
        },
    },
};
```

Now can we can see the event logged in the "Actions" tab along with the event information whenever our `filter` event is triggered.

### Using the `ref()` Directive

The last thing we need to do is to add our input value to the emitted event so we can use it. We can select an element within our custom element using a variation of `querySelector`.

```ts
const input = this.shadowRoot.querySelector('input');
```

There's nothing wrong with this approach, but FAST provides us with a number of [directives](https://www.fast.design/docs/fast-element/using-directives) that make common tasks simpler. In this case we can use the [`ref()`](https://www.fast.design/docs/fast-element/using-directives#the-ref-directive) directive to add a reference to our element in the component's context (`this`).

First, let's add `ref(;searchInput')` to our input element (make sure you import `ref` from `@microsoft/fast-element`).

```ts
template: html<MySearch>`
    <label>
        ${x => x.label}
        <input type="search" ${ref('searchInput')} @search="${x => x.searchHandler()}" />
    </label>
    <button @click="${x => x.buttonClickHandler()}">
        <slot>Search</slot>
    </button>
`,
```

Next, we can add a property to our class that matches the string in our ref and give it a type of `HTML InputElement`.

```ts
export class MySearch extends FASTElement {
    @attr label: string = 'My Search';
    searchInput: HTMLInputElement;
```

Finally, let's emit the input value of the input box with the search event.

```ts
private searchHandler() {
    this.$emit('find', this.searchInput.value);
}

private buttonClickHandler() {
    this.$emit('find', this.searchInput.value);
}
```

When we go back to Storybook, input some values, and click the search button, we should now see the input's value under the `detail` property of the event data.

```ts
{
    bubbles: true,
    cancelBubble: false,
    cancelable: true,
    composed: true,
    currentTarget: HTMLDivElement,
    defaultPrevented: false,
--> detail: "ergferf",
    eventPhase: 3,
    isTrusted: false,
    returnValue: true,
    srcElement: MySearch,
    target: undefined,
    timeStamp: 22556.699999928474,
    type: "find"
}
```

## Styling

I am planning on creating a separate post dedicated to styling web components, so I will keep this section simple.

We add styles by adding a `styles` property to our component definition class decorator and prefixing our template string with `css`.

```ts
@customElement({
    name: 'my-search',
    template: html<MySearch>`
        <label>
            ${x => x.label}
            <input type="search" />
        </label>
        <button>
            <slot>Search</slot>
        </button>
    `,
    styles: css``
})
```

We will also need to make sure we import `css` from FAST.

```ts
import { FASTElement, customElement, html, attr, css } from '@microsoft/fast-element';
```

Let's add some basic styling and then we can break it down.

```ts
styles: css`
    :host {
        --font-size: 1rem;
        --padding: 0.25rem;

        font-size: var(--font-size);
        display: block;
    }

    input {
        font-size: var(--font-size);
        padding: var(--padding);
    }

    button {
        cursor: pointer;
        font-size: var(--font-size);
        padding: var(--padding);
    }
`
```

### `:host`

The first thing you may have noticed is the strange `:host` selector. This targets our custom element's tag - `<my-search>`. This allows us to apply styles to the tag as well as define global styles for the element.

Custom elements apply the `display: inline;` style by default, so in our case we added the `display: block;` to ensure this would render the full width of the parent.

### Generic Selectors

You may have also noticed that the HTML elements are selected. Don't freak out, this was intentional. One of the nice things about the [Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM) is that it provides a layer of encapsulation. This means the component isn't affected by the styles outside of the component and the component's styles don't affect the rest of the application. I don't have to worry about these styles affecting any of the other inputs or buttons in my application.

***Note:*** I don't recommend you build your components this way. I did it merely to prove a point.

### CSS Custom Properties or CSS Variables

With that being said, there are still some ways we can control styles in our component. One fo them is with [CSS Custom Properties (aka - CSS Variables)](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties). With these defined, we can apply custom styles to our components.

```css
my-search {
    --font-size: 1.5rem;
    --padding: 1rem;
}
```

Or even with inline styles in our markup:

```html
<my-search style="--font-size: 1.5rem;--padding: 1rem;"></my-search>
```

### Adding Custom Properties to Storybook

First, we will need to update our jsDoc with our new custom properties to include them in our custom elements manifest.

```ts
/**
 * @tag my-search
 * 
 * @summary This is a search component
 *
 * @attr {string} label - the label associated with the search field
 * 
 * @slot default - content displayed in the search button
 * 
 * @cssprop [--font-size=1rem] - Controls the font size for all elements in the component
 * @cssprop [--padding=0.25rem] - Controls the padding for the `input` and `button` elements
 * 
 */
```

You should now see a new section in controls tab called "CSS Custom Properties" with our properties listed. Let's add some default values.

```ts
export default {
    title: 'Components/My Search',
    component: 'my-search',
    args: {
       label: 'My Search',
       default: 'Search',
       ['--font-size']: '1rem',
       ['--padding']: '0.25rem'
    }
};
```

Now, let's wire them up to our Storybook template.

```ts
const Template = (args: any) => `
    <style>
        my-search {
            --font-size: ${args['--font-size']};
            --padding: ${args['--padding']};
        }
    </style>
    <my-search label="${args.label}">${args.default}</my-search>
`;
```
