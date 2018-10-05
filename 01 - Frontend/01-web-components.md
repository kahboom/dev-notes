Everything web components.

## Intro/Basics

- **"Web components"**: Umbrella term for 4 different specifications:
    1. **custom elements**: method for enabling author to define and use new types of DOM elements in a document. used to use xml for this.
    2. **HTML imports**: a way to include and reuse HTML documents in other HTML documents.
    3. **templates**: method for declaring inert DOM subtrees in HTML and manipulating them to instantiate document fragments with identical contents.
    4. **shadow dom**: method of establishing & maintaining functional boundaries between DOM trees and how these trees interact with each other within a document, thus enabling better functional encapsulation within the DOM.
-

## Loading Resources

### HTML

**HTML Imports**

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


## Other Resources

- [WebComponents.org](https://www.webcomponents.org)
- [Using Web Components](https://www.safaribooksonline.com/videos/using-web-components/9781491957264)

