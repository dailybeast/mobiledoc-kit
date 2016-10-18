# Mobiledoc Kit

[![Sauce Test Status](https://saucelabs.com/browser-matrix/mobiledoc-kit.svg)](https://saucelabs.com/u/mobiledoc-kit)

[![Dependency Status](https://david-dm.org/bustlelabs/mobiledoc-kit/master.svg)](https://david-dm.org/bustlelabs/mobiledoc-kit/master)
[![devDependency Status](https://david-dm.org/bustlelabs/mobiledoc-kit/master/dev-status.svg)](https://david-dm.org/bustlelabs/mobiledoc-kit/master#info=devDependencies)

![Mobiledoc Logo](https://raw.githubusercontent.com/bustlelabs/mobiledoc-kit/master/demo/public/images/mobiledoc-logo-color-small.png)

[![Join the chat at https://gitter.im/bustlelabs/mobiledoc-kit](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/bustlelabs/mobiledoc-kit?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Mobiledoc Kit (warning: beta) is a library for building WYSIWYG editors
supporting rich content via cards.

## Libraries

This repository hosts the core Mobiledoc-Kit library. If you want to use Mobiledoc-Kit to *create a WYSIWYG editor* you have the following options:

| Environment | Library |
| ----------- | ------- |
| Plain JavaScript | [mobiledoc-kit](https://github.com/bustlelabs/mobiledoc-kit) (this repo) |
| Ember | [ember-mobiledoc-editor](https://github.com/bustlelabs/ember-mobiledoc-editor) |
| React | [react-mobiledoc-editor](https://github.com/upworthy/react-mobiledoc-editor) |

If you only want to use the Mobiledoc-Kit runtime, for *rendering mobiledoc posts only* (not editing or creating them), you can use:

| Output Format/Environment | Library |
| ----------  | ------- |
| Plain JavaScript In-Browser (DOM) | [mobiledoc-dom-renderer](https://github.com/bustlelabs/mobiledoc-dom-renderer) |
| Server-Side Rendering (HTML) | see [mobiledoc-dom-renderer's Rendering HTML section](https://github.com/bustlelabs/mobiledoc-dom-renderer#rendering-html) |
| Server-Side Rendering (Text-only, e.g. SEO) | [mobiledoc-text-renderer](https://github.com/bustlelabs/mobiledoc-text-renderer) |
| In-Browser (DOM) Rendering, with Ember | [ember-mobiledoc-dom-renderer](https://github.com/bustlelabs/ember-mobiledoc-dom-renderer) |

Mobiledoc is a deliberately simple and terse format, and you are encouraged to write your own renderer if you have other target output formats (e.g., a PDF renderer, an iOS Native Views Renderer, etc.).

## Demo

Try a demo at [bustlelabs.github.io/mobiledoc-kit/demo](https://bustlelabs.github.io/mobiledoc-kit/demo/).

## API Documentation

API Documentation is [available online](http://bustlelabs.github.io/mobiledoc-kit/demo/docs/).

## Intro to Mobiledoc Kit

* Posts are serialized to a JSON format called **Mobiledoc** instead of to
  HTML. Mobiledoc can be rendered for the web, mobile web, or in theory on any
  platform. Mobiledoc is portable and fast.
* The editor makes limited use of Content Editable, the siren-song of doomed
  web editor technologies.
* Mobiledoc is designed for *rich* content. We call rich sections of an
  article "cards" and rich inline elements "atoms" and implementing a new one doesn't require an understanding
  of Mobiledoc editor internals. Adding a new atom or card takes an afternoon, not several
  days. To learn more, see the docs for
  **[Atoms](https://github.com/bustlelabs/mobiledoc-kit/blob/master/ATOMS.md)**,
  **[Cards](https://github.com/bustlelabs/mobiledoc-kit/blob/master/CARDS.md)**
  and
  **[Mobiledoc Renderers](https://github.com/bustlelabs/mobiledoc-kit/blob/master/RENDERERS.md)**

To learn more about the ideas behind Mobiledoc and the editor (note that the
editor used to be named Content-Kit), see these blog posts:

* [The Content-Kit announcement post](http://madhatted.com/2015/7/31/announcing-content-kit-and-mobiledoc).
* [Building the Content-Kit Editor on Content Editable](https://medium.com/@bantic/building-content-kit-editor-on-contenteditable-99a94871c951)
* [Content-Kit: Programmatic Editing](http://madhatted.com/2015/8/25/content-kit-programmatic-editing)

The Mobiledoc kit saves posts in
**[Mobiledoc format](https://github.com/bustlelabs/mobiledoc-kit/blob/master/MOBILEDOC.md)**.

### Usage

The `Mobiledoc.Editor` class is invoked with an element to render into and
optionally a Mobiledoc to load. For example:

```js
var simpleMobiledoc = {
  version: "0.3.0",
  markups: [],
  atoms: [],
  cards: [],
  sections: [
    [1, "p", [
      [0, [], 0, "Welcome to Mobiledoc"]
    ]]
  ]
};
var element = document.querySelector('#editor');
var options = { mobiledoc: simpleMobiledoc };
var editor = new Mobiledoc.Editor(options);
editor.render(element);
```

`options` is an object which may include the following properties:

* `modiledoc` - [object] A mobiledoc object to load and edit.
* `placeholder` - [string] default text to show before a user starts typing.
* `spellcheck` - [boolean] whether to enable spellcheck. Defaults to true.
* `autofocus` - [boolean] When true, focuses on the editor when it is rendered.
* `undoDepth` - [number] How many undo levels should be available. Default
  value is five. Set this to zero to disable undo/redo.
* `cards` - [array] The list of cards that the editor may render
* `atoms` - [array] The list of atoms that the editor may render
* `cardOptions` - [object] Options passed to cards and atoms
* `unknownCardHandler` - [function] This will be invoked by the editor-renderer
  whenever it encounters an unknown card
* `unknownAtomHandler` - [function] This will be invoked by the editor-renderer
  whenever it encounters an unknown atom
* `parserPlugins` - [array] See [DOM Parsing Hooks](https://github.com/bustlelabs/mobiledoc-kit#dom-parsing-hooks)

### Editor API

* `editor.serialize(version="0.3.0")` - serialize the current post for persistence. Returns
  Mobiledoc.
* `editor.destroy()` - teardown the editor event listeners, free memory etc.
* `editor.disableEditing()` - stop the user from being able to edit the
  current post with their cursor. Programmatic edits are still allowed.
* `editor.enableEditing()` - allow the user to make direct edits directly
  to a post's text.
* `editor.editCard(cardSection)` - change the card to its edit mode (will change
  immediately if the card is already rendered, or will ensure that when the card
  does get rendered it will be rendered in the "edit" state initially)
* `editor.displayCard(cardSection)` - same as `editCard` except in display mode.

### Editor Lifecycle Hooks

API consumers may want to react to given interaction by the user (or by
a programmatic edit of the post). Lifecycle hooks provide notification
of change and opportunity to edit the post where appropriate.

Register a lifecycle hook by calling the hook name on the editor with a
callback function. For example:

```js
editor.didUpdatePost(postEditor => {
  let { range } = editor;
  let cursorSection = range.head.section;

  if (cursorSection.text === 'add-section-when-i-type-this') {
    let section = editor.builder.createMarkupSection('p');
    postEditor.insertSectionBefore(section, cursorSection.next);
    postEditor.setRange(new Mobiledoc.Range(section.headPosition));
  }
});
```

The available lifecycle hooks are:

* `editor.didUpdatePost(postEditor => {})` - An opportunity to use the
  `postEditor` and possibly change the post before rendering begins.
* `editor.willRender()` - After all post mutation has finished, but before
   the DOM is updated.
* `editor.didRender()` - After the DOM has been updated to match the
  edited post.
* `editor.willDelete((range, direction, unit))` - Provides `range`, `direction` and `unit` to identify the coming deletion.
* `editor.didDelete((range, direction, unit))` - Provides `range`, `direction` and `unit` to identify the completed deletion.
* `editor.cursorDidChange()` - When the cursor (or selection) changes as a result of arrow-key
  movement or clicking in the document.
* `editor.onTextInput()` - When the user adds text to the document (see [example](https://github.com/bustlelabs/mobiledoc-kit#responding-to-text-input))

### Programmatic Post Editing

A major goal of the Mobiledoc kit is to allow complete customization of user
interfaces using the editing surface. The programmatic editing API allows
the creation of completely custom interfaces for buttons, hot-keys, and
other interactions.

To change the post in code, use the `editor.run` API. For example, the
following usage would mark currently selected text as "strong":

```js
editor.run(postEditor => {
  postEditor.toggleMarkup('strong');
});
```

It is important that you make changes to posts, sections, and markers through
the `run` and `postEditor` API. This API allows the Mobiledoc editor to conserve
and better understand changes being made to the post.

```js
editor.run(postEditor => {
  const mention = postEditor.builder.createAtom("mention", "John Doe", { id: 42 });
  // insert at current cursor position:
  // or should the user have to grab the current position from the editor first?
  postEditor.insertMarkers(editor.range.head, [mention]);
});
```

For more details on the API of `postEditor`, see the [API documentation](https://github.com/bustlelabs/mobiledoc-kit/blob/master/src/js/editor/post.js).

For more details on the API for the builder, required to create new sections
atoms, and markers, see the [builder API](https://github.com/bustlelabs/mobiledoc-kit/blob/master/src/js/models/post-node-builder.js).

### Configuring hot keys

The Mobiledoc editor allows the configuration of hot keys and text expansions.
For instance, the hot-key command-B to make selected text bold, is registered
internally as:

```javascript
const boldKeyCommand = {
  str: 'META+B',
  run(editor) {
    editor.run(postEditor => postEditor.toggleMarkup('strong'));
  }
};
editor.registerKeyCommand(boldKeyCommand);
```

All key commands must have `str` and `run` properties as shown above.

`str` describes the key combination to use and may be a single key, or modifier(s) and a key separated by `+`, e.g.: `META+K` (cmd-K), `META+SHIFT+K` (cmd-shift-K)

Modifiers can be any of `CTRL`, `META`, `SHIFT`, or `ALT`.

The key can be any of the alphanumeric characters on the keyboard, or one of the following special keys:

`BACKSPACE`, `TAB`, `ENTER`, `ESC`, `SPACE`, `PAGEUP`, `PAGEDOWN`, `END`, `HOME`, `LEFT`, `UP`, `RIGHT`, `DOWN`, `INS`, `DEL`

#### Overriding built-in keys

You can override built-in behavior by simply registering a hot key with the same name.
For example, to submit a form instead of entering a new line when `enter` is pressed you could do the following:

```javascript
const enterKeyCommand = {
  str: 'enter',
  run(editor) {
    // submit the form
  }
};
editor.registerKeyCommand(enterKeyCommand);
```

To fall-back to the default behavior, return `false` from `run`.

### Responding to text input

The editor exposes a hook `onTextInput` that can be used to programmatically react
to text that the user enters. Specify a handler object with `text` or `match`
properties and a `run` callback function, and the editor will invoke the callback
when the text before the cursor ends with `text` or matches `match`.
The callback is called after the matching text has been inserted. It is passed
the `editor` instance and an array of matches (either the result of `match.exec`
on the matching user-entered text, or an array containing only the `text`).

```javascript
editor.onTextInput({
  text: 'X',
  run(editor) {
    // This callback is called after user types 'X'
  }
});

editor.onTextInput({
  match: /\d\dX$/,  // Note the "$" end anchor
  run(editor) {
    // This callback is called after user types number-number-X
  }
});
```
The editor has several default text input handlers that are defined in
`src/js/editor/text-input-handlers.js`.

To remove default text input handlers, simply call the unregister function.
```javascript
editor.unregisterAllTextInputHandlers();
```

### DOM Parsing hooks

A developer can override the default parsing behavior for leaf DOM nodes in
pasted HTML.

For example, when an `img` tag is pasted it may be appropriate to
fetch that image, upload it to an authoritative source, and create a specific
kind of image card with the new URL in its payload.

A demonstration of this:

```js
function imageToCardParser(node, builder, {addSection, addMarkerable, nodeFinished}) {
  if (node.nodeType !== 1 || node.tagName !== 'IMG') {
    return;
  }
  var payload = { src: node.src };
  var cardSection = builder.createCardSection('my-image', payload);
  addSection(cardSection);
  nodeFinished();
}
var options = {
  parserPlugins: [imageToCardParser]
};
var editor = new Mobiledoc.Editor(options);
var element = document.querySelector('#editor');
editor.render(element);
```

Parser hooks are called with three arguments:

* `node` - The node of DOM being parsed. This may be a text node or an element.
* `builder` - The abstract model builder.
* `env` - An object containing three callbacks to modify the abstract
  * `addSection` - Close the current section and add a new one
  * `addMarkerable` - Add a markerable (marker or atom) to the current section
  * `nodeFinished` - Bypass all remaining parse steps for this node

Note that you *must* call `nodeFinished` to stop a DOM node from being
parsed by the next plugin or the default parser.

## Caveats

### Mobiledoc-kit and the Grammarly extension
`mobiledoc-kit` and the [Grammarly extension](https://www.grammarly.com/) do not play well together (see [issue 422](https://github.com/bustlelabs/mobiledoc-kit/issues/422)). Until this is resolved, you can avoid any such problems by disabling Grammarly for the `mobiledoc-kit` instances on your page. To do this, add the `data-gramm="false"` attribute to the `mobiledoc-kit` main DOM element.

## Contributing

Fork the repo, write a test, make a change, open a PR.

### Tests

Install npm and bower:

  * [Node.js](http://nodejs.org/) is required
  * `npm install -g npm && npm install -g bower`
  * `broccoli`, via `npm install -g broccoli-cli`
  * `bower install`
  * `npm install`

Run tests via the built-in broccoli server:

  * `broccoli serve`
  * `open http://localhost:4200/tests`

Or run headless tests via testem:

  * `npm test`

Tests in CI are run at Travis via Saucelabs (see the `test:ci` npm script).

### Demo

There is a demo app that uses the Mobiledoc kit via the [ember-mobiledoc-editor](https://github.com/bustlelabs/ember-mobiledoc-editor)
in `demo/`. To run the demo:

 * `cd demo/ && npm install && bower install`
 * `ember serve` (shut down your broccoli server if it is already running on port 4200)
 * visit http://localhost:4200/


### Getting Help

If you notice a bug or have a feature request please [open an issue on github](https://github.com/bustlelabs/mobiledoc-kit/issues).
If you have a question about usage you can post in the [gitter channel](https://gitter.im/bustlelabs/mobiledoc-kit) or on StackOverflow using the [`mobiledoc-kit` tag](http://stackoverflow.com/questions/tagged/mobiledoc-kit).

### Releasing

* Use `np` (`npm install -g np`)
* `np <version>` (e.g. `np 0.12.0`)
* `git push <origin> --tags`

### Deploy the demo

The demo website is hosted at github pages. To publish a new version:

  * `npm run build-website` - This builds the website into `website/` and commits it
  * `npm run deploy-website` - Pushes the `website/` subtree to the `gh-pages`
     branch of your `origin` at github

Visit [bustlelabs.github.io/mobiledoc-kit/demo](https://bustlelabs.github.io/mobiledoc-kit/demo).

*Development of Mobiledoc and the supporting libraries was generously funded by [Bustle Labs](http://www.bustle.com/labs). Bustle Labs is the tech team behind the editorial staff at [Bustle](http://www.bustle.com), a fantastic and successful feminist and women’s interest site based in NYC.*
