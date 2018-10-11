Everything web components. I first list notes for what web components are and general information about each of the specifications they entail, then I go over how to use them with specific frameworks, potential things to watch out for, and, finally, some additional external resources.

## Intro/Basics

**"Web components"**. Umbrella term for 4 different specifications:

   1. **Custom Elements**: method for enabling author to define and use new types of DOM elements in a document. used to use xml for this.
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

## Loading Resources

### NPM Dependencies

## Consuming Dependency Apps as Web Components
If you have a **host app** that wants to consume an app as a **dependency app**, you may run into the issue that the dependency was built using a different framework from that of the host app. You can turn the dependency app into a framework-agnostic web component, which is essentially a file you would include in your host app.

This is how it works:

1. **Set up the app so that it is "web component"-friendly.** This is based on the framework with which the dependency app was built. See below for framework-specific solutions.
2. **Build the dependency app.** This is also based on the framework, however, it is required that `runtime.js` and `polyfills.js` are generated, as they contain the proper polyfills and dependencies we need to make them truly self-contained web components. See below for framework-specific solutions.
3. **Concatenate the build files generated into a single file.** We typically recommend creating a generic Node.js script for this, which can later be run using an npm script.
4. **Import the resulting concatenated file into the host app**, and load it as a web component.


### Exporting an Angular App as a Web Component

**1. Set up the app to be "web component"-friendly.**
To set up an Angular app to be exported and consumed as a web component, we recommend using [Angular Elements](https://angular.io/guide/elements).



**2. Build the dependency app.**
N/A

**3. Concatenate the build files generated.**
N/A

**4. Import into the host app.**
N/A

For the second step, we recommend using [Angular Elements](https://angular.io/guide/elements).

### Exporting a React App as a Web Component

## Other Resources

- [CanIUse](https://caniuse.com/)
- [WebComponents.org](https://www.webcomponents.org)
- [Using Web Components](https://www.safaribooksonline.com/videos/using-web-components/9781491957264)

