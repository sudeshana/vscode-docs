---
Order: 8
Area: extensions
TOCTitle: Webview API
ContentId: adddd33e-2de6-4146-853b-34d0d7e6c1f1
PageTitle: Using the Webview API
DateApproved: 4/5/2018
MetaDescription: Using the Webview Api to create fully customizable views within VS Code
---
# VS Code Webview API

The webview API allows extensions to create fully customizable views within Visual Studio Code. The built-in Markdown extension for example uses webviews to render Markdown previews. Webviews can also be used to build complex user interfaces beyond what VS Code's native APIs supports.

Think of a webview as an `iframe` within VS Code that your extension controls. A webview can render almost any HTML content in this frame, and it communicate with extensions using message passing. This freedom makes webview incredibly powerful, and opens up a whole new range of extension possibilities.

## Should I use a Webview?

Webviews are pretty amazing, but they should also be used sparingly and only when VS Code's native API is inadequate. Webviews are resource heavy and run in a separate context from normal extensions. A poorly designed webview can also easily feel out of place within VS Code.

Before using a webview, please consider the following:

* Does this functionality really need to live within VS Code? Would it be better as a separate app or website?

* Is a webview the only way to accomplish this? Can I use the regular VS Code APIs instead?

* Will my webview add enough user value to justify its high resource cost?

Remember: Just because you can do something with webviews, doesn't mean you should. However, if you are confident that you need to use webviews, then this document is here to help. Let's get started.

## Webviews API basics

To explain the webview API, we are going to build a simple extension called **Cat Coding**. This extension will use a webview to show a gif of a cat writing some code (presumably in VS Code). As we work through the API, we'll continue adding functionality to the extension, including a counter that keeps track of how many lines of source code our cat has written and notifications that inform the user when the cat introduces a bug.

Here's the `package.json` for the first version of the **Cat Coding** extension. You can find the complete code for the example app [here](https://github.com/Microsoft/vscode-extension-samples/blob/master/webview-sample/README.md). The first version of our extension [contributes a command](/docs/extensionAPI/extension-points.md#contributescommands) called `catCoding.newCat`. When a user invokes this command, we will show a simple webview with our cat in it. Users will be able to invoke this command from the **Command Palette** or even setup a keybinding for it they are so inclined.

```json
{
  "name": "cat-coding",
  "description": "Cat Coding",
  "version": "0.0.1",
  "publisher": "bierner",
  "engines": {
    "vscode": "^1.23.0"
  },
  "activationEvents": [
    "onCommand:catCoding.newCat"
  ],
  "main": "./out/src/extension",
  "contributes": {
    "commands": [
      {
        "command": "catCoding.newCat",
        "title": "Start new cat count",
        "category": "Cat Coding"
      }
    ]
  },
  "scripts": {
    "vscode:prepublish": "tsc -p ./",
    "compile": "tsc -watch -p ./",
    "postinstall": "node ./node_modules/vscode/bin/install"
  },
  "dependencies": {
    "vscode": "*"
  },
  "devDependencies": {
    "@types/node": "^9.4.6",
    "typescript": "^2.8.3"
  }
}
```

Now let's implement the `catCoding.newCat` command. In our extension's main file, we register the `catCoding.newCat` command and use it to show a basic webview:

```ts
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
    context.subscriptions.push(vscode.commands.registerCommand('catCoding.newCat', () => {
        // Create and show a new webview
        const panel = vscode.window.createWebviewPanel(
            'catCoding', // Identifies the type of the webview. Used internally
            "Cat Coding", // Title of the panel displayed to the user
            vscode.ViewColumn.One, // Editor column to show the new webview panel in.
            { } // Webview options. More on these later.
        );
    }));
}
```

The `vscode.window.createWebviewPanel` function creates and shows a webview in the editor. Here is what you see if you try running the `catCoding.newCat` command in its current state:

![An empty webview](images/webview/basics-no_content.png)

Our command opens a new webview panel with the correct title, but with no content! To add our cat to new panel, we also need to set the HTML content of the webview using `webview.html`:

```ts
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
    context.subscriptions.push(vscode.commands.registerCommand('catCoding.newCat', () => {
        // Create and show panel
        const panel = vscode.window.createWebviewPanel('catCoding', "Cat Coding", vscode.ViewColumn.One, { });

        // And set its HTML content
        panel.webview.html = getWebviewContent();
    }));
}

function getWebviewContent() {
    return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cat Coding</title>
</head>
<body>
    <img src="https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif" width="300" />
</body>
</html>`;
}
```

If you run the command again, now the webview looks like this:

![A webview with some HTML](images/webview/basics-html.png)

Progress!

`webview.html` should always be a complete HTML document. HTML fragments or malformed HTML may cause unexpected behavior.

### Updating webview content

`webview.html` can also update a webview's content after it has been created. Let's use this to make **Cat Coding** more dynamic by introducing a rotation of cats:

```ts
import * as vscode from 'vscode';

const cats = {
    'Coding Cat': 'https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif',
    'Compiling Cat':'https://media.giphy.com/media/mlvseq9yvZhba/giphy.gif'
};

export function activate(context: vscode.ExtensionContext) {
    context.subscriptions.push(vscode.commands.registerCommand('catCoding.newCat', () => {
        const panel = vscode.window.createWebviewPanel('catCoding', "Cat Coding", vscode.ViewColumn.One, { });

        let iteration = 0;
        const updateWebview = () => {
            const cat = iteration++ % 2 ? 'Compiling Cat' : 'Coding Cat'
            panel.title = cat;
            panel.webview.html = getWebviewContent(cat);
        }

        // Set initial content
        updateWebview();

        // And schedule updates to the content every second
        setInterval(updateWebview, 1000);
    }));
}

function getWebviewContent(cat: keyof typeof cats) {
    return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cat Coding</title>
</head>
<body>
    <img src="${cats[cat]}" width="300" />
</body>
</html>`;
}
```

![Updating the webview content](images/webview/basics-update.gif)

Setting `webview.html` replaces the entire webview content, similar to reloading an iframe. This is important to remember once you start using scripts in a webview, since it means that setting `webview.html` also resets the script's state.

The example above also uses `webview.title` to change the title of the document displayed in the editor. Setting the title does not cause the webview to be reloaded.

### Lifecycle

Webview panels are owned by the extension that creates them. The extension must hold onto the webview returned from `createWebviewPanel` If your extension loses this reference, it cannot regain access to that webview again, even though the webview will continue to show in VS Code.

As with text editors, a user can also close a webview panel at any time. When a webview panel is closed by the user, the webview itself is destroyed. Attempting to use a destroyed webview throws an exception. This means that the example above using `setInterval` actually has an important bug: if the user closes the panel, `setInterval` will continue to fire, which will try to update `panel.webview.html`, which of course will throw an exception. Cats hate exceptions. Let's fix this!

The `onDidDispose` event is fired when a webview is destroyed. We can use this event to cancel further updates and clean up the webview's resources:

```ts
import * as vscode from 'vscode';

const cats = {
    'Coding Cat': 'https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif',
    'Compiling Cat':'https://media.giphy.com/media/mlvseq9yvZhba/giphy.gif'
};

export function activate(context: vscode.ExtensionContext) {
    context.subscriptions.push(vscode.commands.registerCommand('catCoding.newCat', () => {
        const panel = vscode.window.createWebviewPanel('catCoding', "Cat Coding", vscode.ViewColumn.One, {});

        let iteration = 0;
        const updateWebview = () => {
            const cat = iteration++ % 2 ? 'Compiling Cat' : 'Coding Cat'
            panel.title = cat;
            panel.webview.html = getWebviewContent(cat);
        }

        updateWebview();
        const interval = setInterval(updateWebview, 1000);

        panel.onDidDispose(() => {
            // When the panel is closed, cancel any future updates to the webview content
            clearInterval(interval);
        }, null, context.subscriptions)
    }));
}
```

Extension can also programmatically close webviews by calling `dispose()` on them. If, for example, we wanted to restrict our cat's workday to five seconds:

```ts
export function activate(context: vscode.ExtensionContext) {
    context.subscriptions.push(vscode.commands.registerCommand('catCoding.newCat', () => {

        const panel = vscode.window.createWebviewPanel('catCoding', "Cat Coding", vscode.ViewColumn.One, {});

        panel.webview.html = getWebviewContent('https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif');

        // After 5sec, pragmatically close the webview panel
        const timeout = setTimeout(() => panel.dispose(), 5000)

        panel.onDidDispose(() => {
            // Handle user closing panel before the 5sec have passed
            clearTimeout(timeout);
        }, null, context.subscriptions)
    }));
}
```

### Visibility and Moving

When a webview panel is moved into a background tab, it becomes hidden. It is not destroyed however. VS Code will automatically restore the webview's content from `webview.html` when the panel is brought to the foreground again:

![Webview content is automatically restored when the webview becomes visible again](images/webview/basics-restore.gif)

The `.visible` property tells you if the webview panel is currently visible or not.

Extensions can programmatically bring a webview panel to the foreground by calling `reveal()`. This method takes an optional target view column to show the panel in. A webview panel may only show in a single editor column at a time. Calling `reveal()` or dragging a webview panel to a new editor column moves the webview into that new column.

![Webviews are moved when you drag them between tabs](images/webview/basics-drag.gif)

Let's update our extension to only allow a single webview to exist at a time. If the panel is in the background, then the `catCoding.newCat` command will bring it to the foreground:

```ts
export function activate(context: vscode.ExtensionContext) {

    // Track currently webview panel
    let currentPanel: vscode.WebviewPanel | undefined = undefined;

    context.subscriptions.push(vscode.commands.registerCommand('catCoding.newCat', () => {
        const columnToShowIn = vscode.window.activeTextEditor ? vscode.window.activeTextEditor.viewColumn : undefined;

        if (currentPanel) {
            // If we already have a panel, show it in the target column
            currentPanel.reveal(columnToShowIn);
        } else {
            // Otherwise, create a new panel
            currentPanel = vscode.window.createWebviewPanel('catCoding', "Cat Coding", columnToShowIn, {});
            currentPanel.webview.html = getWebviewContent('https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif');

            // Reset when the current panel is closed
            currentPanel.onDidDispose(() => {
                currentPanel = undefined;
            }, null, context.subscriptions);
        }
    }));
}
```

Here's the new extension in action:

![Using a single panel and reveal](images/webview/basics-single_panel.gif)

Whenever a webview's visibility changes, or when a webview is moved into a new column, the `onDidChangeViewState` event is fired. Our extension can use this event to change cats based on which column the webview is showing in:

```ts
const cats = {
    'Coding Cat': 'https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif',
    'Compiling Cat':'https://media.giphy.com/media/mlvseq9yvZhba/giphy.gif',
    'Testing Cat': 'https://media.giphy.com/media/3oriO0OEd9QIDdllqo/giphy.gif'
};

export function activate(context: vscode.ExtensionContext) {
    context.subscriptions.push(vscode.commands.registerCommand('catCoding.newCat', () => {
        const panel = vscode.window.createWebviewPanel('catCoding', "Cat Coding", vscode.ViewColumn.One, {});
        panel.webview.html = getWebviewContent(cats['Coding Cat']);

        // Update contents based on view state changes
        panel.onDidChangeViewState(e => {
            const panel = e.webviewPanel;
            switch (panel.viewColumn) {
                case vscode.ViewColumn.One:
                    updateWebviewForCat(panel, 'Coding Cat');
                    return;

                case vscode.ViewColumn.Two:
                    updateWebviewForCat(panel, 'Compiling Cat');
                    return;

                case vscode.ViewColumn.Three:
                    updateWebviewForCat(panel, 'Testing Cat');
                    return;
            }
        }, null, context.subscriptions);
    }));
}

function updateWebviewForCat(panel: vscode.WebviewPanel, catName: keyof typeof cats) {
    panel.title = catName;
    panel.webview.html = getWebviewContent(cats[catName]);
}
```

![Responding to onDidChangeViewState events](images/webview/basics-ondidchangeviewstate.gif)

### Inspecting and debugging webviews

The **Developer: Open Webview Developer Tools** VS Code command lets you debug webviews. Running the command launches an instance of Developer Tools for any currently visible webviews:

![Webview developer tools](images/webview/basics-developer_tools.png)

The contents of the webview are within an iframe inside the webview document. You can use developer tools to inspect and modify the webview's DOM, and debug scripts running within the webview itself.

## Loading local content

Webviews run in isolated contexts that cannot directly access local resources. This is done for security reasons. This means that in order to load images, stylesheets, and other resources from your extension, or to load any content from the user's current workspace, you must use the `vscode-resource:` scheme inside webview.

The `vscode-resource:` scheme is similar to the `file:` scheme, but it only allows access to select local files. Like with `file:`, `vscode-resource` loads a resource at a given absolute path from the disk.

Say we want to start bundling the cat gifs for our extension rather pulling them from Giphy. To do this, we first create a uri to the file on disk and then update this uri to use the `vscode-resource` scheme:

```ts
import * as vscode from 'vscode';
import * as path from 'path';

export function activate(context: vscode.ExtensionContext) {
    context.subscriptions.push(vscode.commands.registerCommand('catCoding.newCat', () => {
        const panel = vscode.window.createWebviewPanel('catCoding', "Cat Coding", vscode.ViewColumn.One, { });

        // Get path to resource on disk
        const onDiskPath = vscode.Uri.file(path.join(extensionPath, 'media', 'cat.gif'));

        // And get the special uri to use with the webview
        const catGifSrc = onDiskPath.with({ scheme: 'vscode-resource' });

        panel.webview.html = getWebviewContent(catGifSrc);
    }));
}
```

The value for `catGifSrc` will be something like:

```bash
vscode-resource:/Users/toonces/projects/vscode-cat-coding/media/cat.gif
```

By default, `vscode-resource:` can only access resources in the following locations:

* Within your extension's install directory.
* Within the user's currently active workspace.

You can also always use data URIs to embed resources directly within the webview.

### Controlling access to local resources

Webviews can control which resources `vscode-resource:` can load using the `localResourceRoots` option. `localResourceRoots` defines a set of root URIs from which local content may be loaded.

We can use `localResourceRoots` to restrict *Cat Coding* webviews to only load resources from a `media` directory in our extension:

```ts
import * as vscode from 'vscode';
import * as path from 'path';

export function activate(context: vscode.ExtensionContext) {
    context.subscriptions.push(vscode.commands.registerCommand('catCoding.newCat', () => {
        const panel = vscode.window.createWebviewPanel('catCoding', "Cat Coding", vscode.ViewColumn.One, {
            // Only allow the webview to access resources in our extension's media directory
            localResourceRoots: [
                vscode.Uri.file(path.join(extensionPath, 'media'))
            ]
        });

        const onDiskPath = vscode.Uri.file(path.join(extensionPath, 'media', 'cat.gif'));
        const catGifSrc = onDiskPath.with({ scheme: 'vscode-resource' });

        panel.webview.html = getWebviewContent(catGifSrc);
    }));
}
```

To disallow all local resources, just set `localResourceRoots` to `[]`.

In general, webviews should be as restrictive as possible in loading local resources. However, keep in mind that `vscode-resource` and `localResourceRoots` do not offer complete security protection on their own. Make sure your webview also follows [security best practices](#security), and strongly consider adding a [content security policy](#content-security-policy) to further restrict the content that can be loaded.

## Scripts and message passing

Webviews are just like iframes, which means that they can also run scripts. JavaScript is disabled in webviews by default, but it can easily re-enable by passing in the `enableScripts: true` option.

Let's use a script to add a counter tracking the lines of code our cat has written. It's pretty simple:

```ts
import * as path from 'path';
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
    context.subscriptions.push(vscode.commands.registerCommand('catCoding.newCat', () => {
        const panel = vscode.window.createWebviewPanel('catCoding', "Cat Coding", vscode.ViewColumn.One, {
            // Enable scripts in the webview
            enableScripts: true
        });

        panel.webview.html = getWebviewContent();
    }));
}

function getWebviewContent() {
    return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cat Coding</title>
</head>
<body>
    <img src="https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif" width="300" />
    <h1 id="lines-of-code-counter">0</h1>

    <script>
        const counter = document.getElementById('lines-of-code-counter');

        let count = 0;
        setInterval(() => {
            counter.textContent = count++;
        }, 100);
    </script>
</body>
</html>`;
}
```

![A script running in a webview](images/webview/scripts-basic.gif)

Wow! that's one productive cat.

Webview scripts can do just about anything that a script on a normal webpage can. Keep in mind though that webviews exist in their own context, so scripts in a webview do not have access to the VS Code API. That's where message passing comes in!

### Passing messages from an extension to a webview

An extension can send data to its webviews using `webview.postMessage()`. This method sends any json serializable data to the webview. The message is received inside the webview through the standard `message` event.

To demonstrate this, let's add a new command to *Cat Coding* that instructs the currently coding cat to refactor their code (thereby reducing the total number of lines). The new `catCoding.doRefactor` command use `postMessage` to send the instruction to the current webview, and `window.addEventListener('message' event => { ... })` inside the webview itself to handle the message:

```ts
export function activate(context: vscode.ExtensionContext) {

    // Only allow a single Cat Coder
    let currentPanel: vscode.WebviewPanel | undefined = undefined;

    context.subscriptions.push(vscode.commands.registerCommand('catCoding.newCat', () => {
        if (currentPanel) {
            currentPanel.reveal(vscode.ViewColumn.One);
        } else {
            currentPanel = vscode.window.createWebviewPanel('catCoding', "Cat Coding", vscode.ViewColumn.One, {
                enableScripts: true
            });
            currentPanel.webview.html = getWebviewContent();
            currentPanel.onDidDispose(() => { currentPanel = undefined; }, undefined, context.subscriptions);
        }
    }));

    // Our new command
    context.subscriptions.push(vscode.commands.registerCommand('catCoding.doRefactor', () => {
        if (!currentPanel) {
            return;
        }

        // Send a message to our webview.
        // You can send any JSON serializable data.
        currentPanel.webview.postMessage({ command: 'refactor' });
    }));
}

function getWebviewContent() {
    return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cat Coding</title>
</head>
<body>
    <img src="https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif" width="300" />
    <h1 id="lines-of-code-counter">0</h1>

    <script>
        const counter = document.getElementById('lines-of-code-counter');

        let count = 0;
        setInterval(() => {
            counter.textContent = count++;
        }, 100);

        // Handle the message inside the webview
        window.addEventListener('message', event => {

            const message = event.data; // The json data our extension sent

            switch (message.command) {
                case 'refactor':
                    count = Math.ceil(count * 0.5);
                    counter.textContent = count;
                    break;
            }
        });
    </script>
</body>
</html>`;
```

![Passing messages to a webview](images/webview/scripts-extension_to_webview.gif)

### Passing messages from a webview to an extension

Webviews can also pass messages back to their extension. This is accomplished by calling `window.parent.postMessage()` in the webview with some json serializable data, and listening to the `webview.onDidReceiveMessage` event in the extension itself.

We can use `window.parent.postMessage` in our *Cat Coding* webview to alert the extension when our cat introduces a bug in their code:

```js
export function activate(context: vscode.ExtensionContext) {
    context.subscriptions.push(vscode.commands.registerCommand('catCoding.newCat', () => {
        const panel = vscode.window.createWebviewPanel('catCoding', "Cat Coding", vscode.ViewColumn.One, {
            enableScripts: true
        });

        panel.webview.html = getWebviewContent();

        // Handle messages from the webview
        panel.webview.onDidReceiveMessage(message => {
            switch (message.command) {
                case 'alert':
                    vscode.window.showErrorMessage(message.text);
                    return;
            }
        }, undefined, context.subscriptions);
    }));
}

function getWebviewContent() {
    return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cat Coding</title>
</head>
<body>
    <img src="https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif" width="300" />
    <h1 id="lines-of-code-counter">0</h1>

    <script>
        const counter = document.getElementById('lines-of-code-counter');

        let count = 0;
        setInterval(() => {
            counter.textContent = count++;

            // Alert the extension when our cat introduces a bug
            if (Math.random() < 0.001 * count) {
                window.parent.postMessage(
                    { command: 'alert', text: '🐛  on line ' + count },
                    '*')
            }
        }, 100);
    </script>
</body>
</html>`;
}
```

![Passing messages from the webview to the main extension](images/webview/scripts-webview_to_extension.gif)

## Security

As with any webpage, when creating a webview you must follow some basic security best practices.

### Limit capabilities

A webview should have the minimum set of capabilities that it needs. For example, if your webview does not need to run scripts, do not set the `enableScripts: true`. If your webview does not need to load resources from the user's workspace, set `localResourceRoots` to `[vscode.Uri.file(extensionContext.extensionPath)]` or even `[]` to disallow access to all local resources.

### Content security policy

[Content security policies](https://developers.google.com/web/fundamentals/security/csp/) further restrict the content that can be loaded and executed in webviews. For example, a content security policy can make sure that only a whitelist of scripts can be run in the webview, or even tell the webview to only load images over https.

To add a content security policy, put a `<meta http-equiv="Content-Security-Policy">` directive at the top of the webview's `<head>`

```ts
function getWebviewContent() {
    return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">

    <meta http-equiv="Content-Security-Policy" content="default-src 'none';">

    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Cat Coding</title>
</head>
<body>
    ...
</body>
</html>`;
}
```

The policy `default-src 'none';` disallows all content. We can then turn back on the minimal amount of content that our extension needs to function. Here's a content security policy that allows loading local scripts and stylesheets, and loading images over https:

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'none'; img-src vscode-resource: https:; script-src vscode-resource:; style-src vscode-resource:;">
```

This content security policy also implicitly disables inline scripts and styles. It best practice to extract all inline styles and scripts to external files so that they can be properly loaded without relaxing the content security policy.

### Only load content over https

If your webview allows loading external resources, it is strongly recommended that you only allow these resources to be loaded over https and not over http. The example content security policy above already does this by only allowing images to be loaded over `https:`.

### Sanitize all user input

Just as you would for a normal webpage, when constructing the html for a webview, you must sanitize all user input. Failing to properly sanitize input can allow content injections, which may open your users up to a security risk.

Example values that must be sanitized:

* File contents.
* File and folder paths.
* User and workspace settings.

Consider using a helper library to construct your html strings, or at least ensure that all content from the user's workspace is properly sanitized.

Never rely on sanitization alone for security. Make sure to follow the other security best practices, such as having a content security policy, to minimize the impact of any potential content injections.

### Do not trust messages received from webviews

Say your webview is compromised. That's bad, but it is probably not the end of the world. Webviews are sandboxes which theoretically limits the damage that an attacker can inflict. But it does mean that an attacker can now execute anything  that your `webview.onDidReceiveMessage` does. This may not be so bad, if all your handler does is show notifications, but it could be very bad if the handler runs `eval` or deletes a file at given path on disk.

Always operate under the assumption that your webview is compromised. Always validate all messages the webview sends to the extension, especially if these messages are used to perform potentially dangerous operations.

## Persistence

In the standard webview [lifecycle](#lifecycle), webviews are created by `createWebviewPanel` and destroyed when the user closes them or when `.dispose()` is called. The contents of webviews however are created when the webview becomes visible and destroyed when the webview is moved into the background. Any state inside the webview will be lost when the webview is moved to a background tab.

The best way to solve this is to make your webview stateless. Use [message passing](#passing-messages-from-a-webview-to-an-extension) to save off the webview's state and then restore the state when the webview becomes visible again.

### retainContextWhenHidden
For webviews with very complex UI or state that cannot be quickly saved and restored, you can instead use the `retainContextWhenHidden` option. This option makes a webview keep its content around—but in a hidden state—even when the webview itself is no longer in the foreground.

Although *Cat Coding* can hardly be said to have complex state, let's try enabling `retainContextWhenHidden` to see how  the option changes a webview's behavior:

```ts
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
    context.subscriptions.push(vscode.commands.registerCommand('catCoding.newCat', () => {
        const panel = vscode.window.createWebviewPanel('catCoding', "Cat Coding", vscode.ViewColumn.One, {
            enableScripts: true,
            retainContextWhenHidden: true
        });
        panel.webview.html = getWebviewContent();
    }));
}

function getWebviewContent() {
    return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cat Coding</title>
</head>
<body>
    <img src="https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif" width="300" />
    <h1 id="lines-of-code-counter">0</h1>

    <script>
        const counter = document.getElementById('lines-of-code-counter');

        let count = 0;
        setInterval(() => {
            counter.textContent = count++;
        }, 100);
    </script>
</body>
</html>`;
}
```

![](images/webview/persistence-retrain.gif)

Notice how the counter does not reset now when the webview is hidden and then restored. No extra code required!

Although `retainContextWhenHidden` may be appealing, this option has high memory overhead and should only be used when other persistence techniques will not work.