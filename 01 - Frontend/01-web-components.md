Everything web components. I first list notes for what web components are and general information about each of the specifications they entail, then I go over how to use them with specific frameworks, potential things to watch out for, and, finally, some additional external resources.

## Intro/Basics

**"Web components"**. Umbrella term for 4 different specifications:

   1. **Custom Elements**: method for enabling author to define and use new types of DOM elements in a document. We used to use XML for this.
   2. **HTML Imports**: a way to include and reuse HTML documents in other HTML documents.
   3. **Templates**: method for declaring inert DOM subtrees in HTML and manipulating them to instantiate document fragments with identical contents.
   4. **Shadow DOM**: method of establishing & maintaining functional boundaries between DOM trees and how these trees interact with each other within a document, thus enabling better functional encapsulation within the DOM. Private JS and CSS.

Around since 2009 with HTML5, custom not possible until much later.

**Relationship with ES6.** With ES6, we fix the issue where JS used to be pathologically global. We have lexically scoped variables like `let`, classes, and modules, etc., making global JS now encapsulated and private. HTML can provide the same thing with web components, going from being pathologically global (HTML & CSS) to encapsulated and private (e.g. jQuery and selectors).

**Polyfills**. A way to take holes in functionality and fill it in with a JS library, teach the browser to support it natively. Necessary for web components. Not necessary for Chrome or Opera (both use Blink, HTML render kit).

**Show CSS, JS, and HTML Structure for Shadow DOM in Google Chrome**. View > Developer > JavaScript Console. Click on the 3 vertical dots on the top right-hand corner of the console, a dropdown menu will appear. Click Settings. Under Elements, make sure "Show user agent shadow DOM" is selected/checked.

### Custom Elements

### HTML Imports

Example:
```
<head>
  <link rel="import" href="/path/to/imports/stuff.html">
</head>
```

```
<link rel="import" href="http://example.com/elements.html">
```

- Import location = URL of an import.
- To load content from another domain, the import location needs to be CORS-enabled.
- **De-duping**. The browser's network stack automatically de-dupes all requests from the same URL. This means that imports that reference the same URL are only retrieved once. Downside: only works on exact URL matches.

[The problem with using HTML imports for dependency management](https://github.com/tjvantoll/www.tjvantoll.com/blob/master/_posts/2014-08-12-the-problem-with-using-html-imports-for-dependency-management.markdown)

[HTML5 Rocks Web Components Tutorial: Imports](https://www.html5rocks.com/en/tutorials/webcomponents/imports/)

### Templates

### Shadow DOM

Rather than injecting styles, they should be loaded in the `<head>` inside Shadow DOM, so that they are truly encapsulated.

[MDN: Using Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM)

## Loading Resources

### NPM Dependencies

## Consuming Dependency Apps as Web Components
If you have a **host app** that needs to consume an app as a **dependency app**, you may run into the issue that the dependency was built using a different framework from that of the host app. You can turn the dependency app into a framework-agnostic web component, which is essentially a file you would include in your host app.

This is how it works:

1. **Set up the dependency app so that it is "web component"-friendly.** This is based on the framework with which the dependency app was built. See below for framework-specific solutions.
2. **Build the dependency app.** This is also based on the framework, however, it is required that `runtime.js` and `polyfills.js` are generated, as they contain the proper polyfills and dependencies we need to make them truly self-contained web components. See below for framework-specific solutions.
3. **Concatenate the build files generated into a single file.** We typically recommend creating a generic Node.js script for this, which can later be run using an npm script.
4. **Import the resulting concatenated file into the host app**, and load it as a web component.


### Exporting an Angular App as a Web Component (WIP)
Please note that dependency apps need to be on Angular core 6.1.x, as this is when support for the ShadowDOM V1 spec was fulfilled. Angular's `ViewEncapsulation.Native` method was deprecated in lieu of `ShadowDom` (which uses the browser API's Shadow DOM, see more [here](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM)).

**1. Set up the app to be "web component"-friendly.**
To set up an Angular app to be exported and consumed as a web component, we recommend using [Angular Elements](https://angular.io/guide/elements).

- Create a new Angular app: `$ ng new cheeseApp`

- Install Angular elements, which allows us to leverage web components with Angular: `$ npm i @angular/elements --save`

- Install custom elements: `$ npm i @webcomponents/custom-elements --save`

- In your `polyfills.ts` file, include the following:

```typescript
import '@webcomponents/custom-elements/src/native-shim';
import '@webcomponents/custom-elements/custom-elements.min';
```

- Create your Angular component: `$ ng g component cheese`

- In `app.module.ts`:

```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule, Injector } from '@angular/core';
import { createCustomElement } from '@angular/elements';

import { CheeseComponent } from './cheese/cheese.component';

@NgModule({
  declarations: [
    CheeseComponent
  ],
  imports: [
    BrowserModule
  ]
})

export class AppModule {
  constructor(private injector: Injector) { }

  ngDoBootstrap() {
    const appElement = createCustomElement(CheeseComponent, {injector: this.injector});
    customElements.define('cheese', appElement);
  }
}
```


- In `index.html`, you can pretty much replace all of the text with your selector: `<cheese></cheese>`

If you load the app in your browser, you should see "cheese works!".


**2. Build the dependency app.**
This depends on the setup you have for Angular, but we will assume you are using Angular's built-in CLI (as of v4.x). See step 3's npm script entry. In this case, the build command we are using is `ng build --prod --output-hashing=none`.

**3. Concatenate the build files generated.**
Install developer dependencies to concatenate the files.

`$ npm i concat fs-extra --save-dev`

Be sure that they get added as a `devDependency` in your `package.json` file, otherwise you'll get an error about it needing to be whitelisted.

For this step, it is advisable to create a JS file to do the concatenating for you. Here is an example we call `build-script.js`, where we reference three files generated as a result of the Angular build for production, and then concatenate them into a single file:

```javascript
const fs = require('fs-extra');
const concat = require('concat');

(async function build() {
  const files = [
    './dist/runtime.bundle.js',
    './dist/polyfills.bundle.js',
    './dist/main.bundle.js'
  ]

  await fs.ensureDir('wc')
  await concat(files, 'wc/cheese.js')
})();

```

[Here](https://gist.github.com/kahboom/a608c783520cd92bf4ea8bcd5f1aa5cc) is a gist you can download.

Make sure you include them in that order. Then it creates a new directory called `wc`, then you save the final output to a new file in that directory called `cheese.js`.

We can add an npm script entry in `package.json` to be able to run it from the command line. This takes care of both steps 2 and 3:
```json
"scripts": {
  "build:wc": "ng build --prod --output-hashing=none && node build-script.js",
}
```

Feel free to separate them into two entries.

**4. Import into the host app.**
Go back to your host app, and include the final file like so:

```html
<cheese></cheese>
<script src="../wc/cheese.js"></script>
```


If you would like to add the host app with Yarn, you can `$ yarn add <path to relative file>` instead.

### Exporting a React App as a Web Component (WIP)

## Resources
Several code snippets on here can be attributed to Scott Davis's [Using Web Components](https://www.safaribooksonline.com/videos/using-web-components/9781491957264) video, available on Safari Books Online.

- [CanIUse](https://caniuse.com/)
- [WebComponents.org](https://www.webcomponents.org)
- [Using Web Components](https://www.safaribooksonline.com/videos/using-web-components/9781491957264)

