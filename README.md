# Stackbit Nextjs V2

The NextJs core starter for Stackbit. 

## Quickstart

```
npm install
```

```
npm run dev
```

### Add a new page

Create a new markdown file `content/pages/new-page.md` with the following frontmatter.

```yaml
---
title: "New Example Page"
layout: "AdvancedLayout"
sections:
  - type: "HeroSection"
    variant: "variant-a"
    width: "wide"
    height: "auto"
    alignHoriz: "left"
    badge: "Brand New"
    title: "My Heading"
    text: "a long description"
    actions:
      - type: "Button"
        url: "/"
        label: "Home"
        style: "primary"
    feature:
      type: "ImageBlock"
      imageUrl: "/images/hero.png"
      imageAltText: "Image alt text"
      imageCaption: "Image caption"
---
```

Visit http://localhost:3000/new-page/ to see the new page. 

> **Note:** The pages url will match the file path - `content/pages/new-page.md` will resolve to _/new-page_. A nested folder like `content/pages/blog/index.md` will resolve to _/blog/_

### Add a menu item

Menu items are configured in `content/data/config.json`. Edit the config file and add a new menu object to the `primaryLinks` array.

```js
// content/data/config.json
{
  "navBar": {
    // ...
    "primaryLinks": [
      // ...
      {
        "type": "Link",
        "label": "My New Page",
        "url": "/new-page",
        "altText": ""
      }
    ],
    // ...
  }
}
```

### Adding more sections to the page

Add a [🧩 CtaSection](https://components.stackbit.com/?path=/docs/components-contactsection--primary) to the page by adding the component to the the sections array.

```yaml
# content/pages/new-page.md
---
title: "New Example Page"
layout: "AdvancedLayout"
sections: # sections array
  - type: "HeroSection"
    # ...
  - type: "CtaSection"
    variant: "variant-a"
    width: "wide"
    height: "auto"
    alignHoriz: "center"
    title: "Let's do this"
    text: "The Stackbit theme is flexible and scalable to every need. It can manage any layout and any screen."
    actions:
      - type: "Button"
        url: "/contact"
        label: "Get Started"
        style: "primary"
---
```


**Component Library:** The [Stackbit Component Library](https://develop--stackbit-components.netlify.app/?path=/story/layouts-advancedlayout--primary) has full documentation on each component including the available props and frontmatter.


 **Component Examples**

  - [🧩 CtaSection](https://components.stackbit.com/?path=/docs/components-contactsection--primary)
  - [🧩 ContactSection](https://components.stackbit.com/?path=/docs/components-contentsection--primary)
  - [🧩 FeaturedPeopleSection](https://components.stackbit.com/?path=/docs/components-featuredpeoplesection--primary)
  - [🧩 FeaturedPostsSection](https://components.stackbit.com/?path=/docs/components-featuredpostssection--primary)

### Understanding Layouts

Explain how layouts work.

* How does the `layout` field work?
* How does the `layout` field effect which frontmatter fields can be used?
* The layout is dependant on a stackbit.yaml model. The model defines which components can be used in the sections array. Explain how the layout and model are used.

### Adding your own components

Explain how to add your own component

### Extending a Stackbit component

Explain how to extend an existing Stackbit component.


## Advanced Guide
### Layouts

Every page in `content/pages` must have a `layout` field. The layout is both a component and a content model.

The layout will determine what frontmatter fields can be used in the page  based on it's stackbit.yaml model. The AdvancedLayout uses the [AdvancedLayout.yaml model](https://github.com/stackbit/stackbit-components/blob/main/models/AdvancedLayout.yaml) shown below:

```yaml
type: page
name: AdvancedLayout
label: Advanced page
layout: AdvancedLayout
hideContent: true
fields:
  - type: string
    name: title
    label: Title
  - type: list
    name: sections
    label: Sections
    items:
      type: model
      models:
        - ContactSection
        - ContentSection
        - CtaSection
        - FeaturedPostsSection
        - HeroSection
        - TestimonialsSection
```

The AdvancedLayout is a **page model** with a `title` and `sections` field. The `sections` field is a list (an array of objects) with each object being it's own **object model**. The AdvancedLayout can use the following components from the Stackbit Components Library:

- ContactSection
- ContentSection
- CtaSection
- FeaturedPostsSection
- HeroSection
- TestimonialsSection

### Content

Content is sourced from the filesystem using [Sourcebit](https://github.com/stackbit/sourcebit). It loads markdown and json files stored in `content/pages` and `content/data` and tranforms them into page objects which are used by `getStaticProps()`. 

* This theme comes with a pre-configured sourcebit config `sourcebit.js`
* This theme comes pre-configured to use `sourcebit.fetch()` in the `next.config.js`


```js
// next.config.js

const sourcebit = require('sourcebit');
const sourcebitConfig = require('./sourcebit.js');

sourcebit.fetch(sourcebitConfig);

module.exports = {
  ...
}
```

Whenever you run `npm run dev` or `npm run build` Sourcebit fetches content and outputs the `.sourcebit-nextjs-cache.json` file which contains an array of page objects. 

Inside a Nextjs page route like `src/pages/[[...slug]].js` you can load page data by it's path using the `sourcebitDataClient.getStaticPropsForPageAtPath(path)` function inside of `getStaticProps`

```js
// src/pages/[[...slug]].js

export async function getStaticProps({ params }) {
  const props = await sourcebitDataClient.getStaticPropsForPageAtPath(params.slug);
  return { props };
}
```

## Components

This theme uses the [Stackbit component library](https://github.com/stackbit/stackbit-components) 

* This theme comes pre-configured to use `withStackbitComponents()` in the `next.config.js`

```js
// next.config.js

const withStackbitComponents = require('@stackbit/components/with-stackbit-components');

module.exports = withStackbitComponents({
  componentsMapPath: '.stackbit/components-map.json',
  ...
})
```

`withStackbitComponents` generates a dynamic import map for the Stackbit component library. This provides a framework to override existing Stackbit components and add your own new components. This approach reduces the bundle size by only importing components that are used.

* Generates `.stackbit/components-map.json` - Edit this file to override or add new components
* Generates `.stackbit/dynamic-components.js` - This file is dynamically generated from `components-map.json` and should not be edited or committed to git.

You can now use `getDynamicComponent(ComponentName)` in a Nextjs page. 

```
// src/pages/[[...slug]].js 
import { getDynamicComponent } from '@stackbit/components/components-registry';

function Page(props) {
  console.log(props);
  const { page, site } = props;
  const { layout } = page;

  const PageLayout = getDynamicComponent(layout);

  return <PageLayout page={page} site={site} />;
}
```


## Tailwind

You can edit the tailwind config in `tailwind.config.js`

The Stackbit component library includes a number of Tailwind preset configurations which will dramatically change the themes look and feel. 

* [@stackbit/components/themes/tailwind.bold.config.js](https://github.com/stackbit/stackbit-components/blob/main/themes/tailwind.bold.config.js)
* [@stackbit/components/themes/tailwind.eco.config.js](https://github.com/stackbit/stackbit-components/blob/main/themes/tailwind.eco.config.js)
* [@stackbit/components/themes/tailwind.modern.config.js](https://github.com/stackbit/stackbit-components/blob/main/themes/tailwind.modern.config.js)
* [@stackbit/components/themes/tailwind.retro.config.js](https://github.com/stackbit/stackbit-components/blob/main/themes/tailwind.retro.config.js)

Use the `presets` option to change the configuration.

```js
// tailwind.config.js
module.exports = {
  presets: [require('@stackbit/components/themes/tailwind.bold.config')],
  // ...
}
```
