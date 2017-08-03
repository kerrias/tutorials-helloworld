BITS "Hello, World" Tutorial
---
This tutorial shows how to create a BITS module for BITS.

- [Objective](#objective)
- [Before You Begin](#before-you-begin)
- [Create Module Directory](#create-module-directory)
- [Create `module.json`](#create-module-json)
- [Create `index.js`](#create-index-js)
- [Add `app/` Directory](#add-app-directory)
- [Add Base Navigation](#add-base-navigation)

# <a name="objective"></a> Objective
- Create a BITS module.
- Write "Hello, World" to the console.
- Write "Hello, World" to the UI.

# <a name="before-you-begin"></a> Before You Begin
You need to setup a Base, and be able to communicate with a running instance. If you do not already have a Base running, you can create one by downloading the source and using the development command-line.

# <a name="create-module-directory"></a> Create Module Directory
A BITS module's source code runs from a directory in the Base data directory. Create the necessary Base data directory and `hello-world` module directory structure.
``` bash
export BITS_DATA_DIR=/tmp/bits
mkdir -p "$BITS_DATA_DIR/base/modules/modules/hello-world"
```

# <a name="create-module-json"></a> Create `module.json`
A JSON file named `module.json` is used to describe a BITS module. This file tells the Base how to appropiately load and allow access to the module's API. The `module.json` file must contain valid JSON and at minimum define the `name` and `version` properties. A `dependencies` object can also be defined to ensure this module only loads when the Base version is >= `2.0.0` && < `3.0.0`. Other properties may be defined in the `module.json`, some of these properties will be explored later in this document. Create the `module.json` file in the `hello-world` module directory.
``` bash
vim "$BITS_DATA_DIR/base/modules/modules/hello-world/module.json"
```
And then, place the following JSON data in the file.
``` json
{
    "name": "hello-world",
    "version": "1.0.0",
    "dependencies": {
        "bits-base": "^2.0.0"
    }
}
```
The `module.json` file is all that is required for a BITS module to load. To verify the `hello-world` module loads, run the Base.
``` bash
cd <path_to_Base_installation>
HTTP_PORT=9000 HTTPS_PORT=9001 node app.js -v -d "$BITS_DATA_DIR"
```
You should see some output text as the server starts, and the following text should be present somewhere within the stream:
```bash
...
2017-05-23T15:47:08.488Z - debug: Loaded module hello-world
{ name: 'hello-world', duration: 343 }
...
```

# <a name="create-index-js"></a> Create `index.js`
A BITS module can define a file named `index.js` to be run by Base on load. If the file is present it must export an object with two functions; `load` and `unload`. Create the `index.js` file in the `hello-world` module directory.
``` bash
vim "$BITS_DATA_DIR/base/modules/modules/hello-world/index.js"
```
Place the following code within that file:

``` javascript
class HelloWorldApp {
  load(messageCenter) {
    console.log('Hello, World!');
  }

  unload() {
    console.log('Goodbye!');
  }
}

module.exports = new HelloWorldApp();
```
Now when the `hello-world` module loads, `'Hello, World!'` will be written to the console. Verify the `index.js` is being run on module load.  First, start the server:
``` bash
HTTP_PORT=9000 HTTPS_PORT=9001 node app.js -v -d "$BITS_DATA_DIR"
```
Now, observe that the same data stream will be written to the screen as before, but this time the words "Hello, World!" are printed right as the module is being loaded:
```bash
...
Hello, World!
2017-05-23T15:23:36.357Z - debug: Loaded module hello-world
{ name: 'hello-world', duration: 337 }
...
```

# <a name="add-app-directory"></a> Add `app/` Directory
The Base allows for modules to add pages to the UI. A page is defined by a `contentImport` and a `contentElement`. The `contentElement` is the name of the HTML tag for your custom element.  This tag will be added to the DOM in the UI when the user browses to the module's page. The `contentImport` is the url of the static file that defines the custom element defined by the `contentElement` tag. A module can define an `appDir` directory that the Base will serve static http resources from. The `contentElement`, `contentImport`, and `appDir` properties must be defined in the module's `module.json` before the Base will allow access to the module's page in the UI. Add these properties to the `hello-world` module's `module.json`.  First, open the file:
``` bash
vim "$BITS_DATA_DIR/base/modules/modules/hello-world/module.json"
```
Next, add the following properties to the bottom of the properties list, ensuring that they stay within the curly brackets.  Be sure to add a trailing comma to the bottom most pre-existing property first.
``` json
{
    ...
    "appDir": "app/",
    "contentElement": "hello-world-app",
    "contentImport": "/elements/hello-world-app/hello-world-app.html"
}
```

Once these new properties have been added, your `module.json` file should look like the following.  Notice that the trailing comma has been added after the dependencies object, which was not previously present.
```json
{
    "name": "hello-world",
    "version": "1.0.0",
    "dependencies": {
        "bits-base": "^2.0.0"
    },
    "appDir": "app/",
    "contentElement": "hello-world-app",
    "contentImport": "/elements/hello-world-app/hello-world-app.html",
}
```
Now create the necessary directories and files to display the module's page.
``` bash
mkdir -p "$BITS_DATA_DIR/base/modules/modules/hello-world/app/elements/hello-world-app"
vim "$BITS_DATA_DIR/base/modules/modules/hello-world/app/elements/hello-world-app/hello-world-app.html"
```
Place the following HTML code into your new file.  This code defines the new HTML tag named "hello-world-app", which could them be used elsewhere within the module.

``` html
<link rel="import" href="../../bower_components/polymer/polymer.html">

<dom-module id="hello-world-app">
  <template>
    <style>
      :host {
        display: block;
      }
    </style>

    <h1>Hello, World!</h1>

  </template>
  <script>
  (() => {
    'use strict';

    Polymer({
      is: 'hello-world-app'
    });
  })();
  </script>
</dom-module>
```
Restart the Base to reload the `hello-world` module.
``` bash
HTTP_PORT=9000 HTTPS_PORT=9001 node app.js -v -d "$BITS_DATA_DIR"
```
Open Google's Chrome browser and navigate to `https://127.0.0.1:9001/hello-world`. You should see the BITS Base load the `hello-world` custom element load and display `"Hello, World!"`.

# <a name="add-base-navigation"></a> Add Base Navigation
The `hello-world` module is visible if a user directly navigates to the module by entering the specific url, but the user is currently unable to get the module by navigating from the Base's dashboard page. To add a module to the navigation menu, a module must define the `scopes`, `displayName`, and `icon` properties in the module's `module.json`. This `scopes` property defines which users are allowed to see and access the module. The `displayName` property defines the string to display to the user in the navigation menu's entry. The `icon` property defines the icon to use on the navigation menu's entry. The `category` property groups like modules into groups in the navigation menu. This property is optional. Add these properties to the `hello-world` module's `module.json` file.  First, open the file:
``` bash
vim "$BITS_DATA_DIR/base/modules/modules/hello-world/module.json"
```
Then, add the properties to the bottom of the properties list in the same manner you did before.  Be sure to remember the trailing comma after the most recent property in the file.
``` json
{
    ...
    "scopes": [
      {"name": "public", "displayName": "Public"}
    ],
    "displayName": "Hello World",
    "icon": "icons:favorite",
    "category": "Tutorials"
}
```
Restart the Base to reload the `hello-world` module.
``` bash
HTTP_PORT=9000 HTTPS_PORT=9001 node app.js -v -d "$BITS_DATA_DIR"
```
You should be able to navigate to `https://127.0.0.1:9001`. After the Base's dashboard loads, open the navigation menu, click on the `"Hello World"` navigation menu entry. The `hello-world` module page should be loaded.