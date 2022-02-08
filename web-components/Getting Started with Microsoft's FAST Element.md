---
title: Getting Started with Microsoft's FAST Element
published: false
description: A quick guide on getting up and running with Microsoft's FAST Element
tags: #webcomponents, #webdev, #javascript, #html
//cover_image: https://direct_url_to_image.jpg
---

If you haven't heard already, web components have started to take off in popularity. The features they bring to the table is very compelling. Among them are: framework-agnostic reusable components, strong style encapsulation, and blazingly fast performance.

A few of the more popular tools for building web component libraries include [lit](https://lit.dev) and [StencilJS](https://stenciljs.com/), but for the last few weeks I have had the opportunity to work with [Microsoft's FAST Element](https://fast.design) and have had a lot of fun working with it.

One thing that I struggled as I started out, was finding a standard way to easily stand up dev environments so I could experiment and ideate. The FAST team doesn't provide one (yet), so I went ahead and [built a quick one](https://www.npmjs.com/package/create-fast-element) for us to experiment with. I will be using it to generate the examples used in this article.

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

## Styling

