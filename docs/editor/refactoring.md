---
Order: 6
Area: editor
TOCTitle: Refactoring
ContentId: 481dfd3a-d847-4ed3-b37b-7fc8d234a4c2
PageTitle: Refactoring source code in Visual Studio Code
DateApproved: 3/1/2018
MetaDescription: Refactoring source code in Visual Studio Code.
---
# Refactoring

[Source code refactoring](https://en.wikipedia.org/wiki/Code_refactoring) can improve the quality and maintainability of your project by restructuring your code while not modifying the runtime behavior. Being able to perform refactoring operations quickly and accurately is great for your productivity. Visual Studio Code supports refactoring operations such as **Extract method** and **Extract variable** to improve your code base from within your editor.

![refactoring hero image](images/refactoring/refactoring-hero.png)

For example, a common refactoring used to avoid duplicating code (a maintenance headache) is **Extract method** where you select source code that you'd like to reuse elsewhere and pull it out its own shared method.

Refactorings are provided by a language service and VS Code has built-in support for TypeScript and JavaScript refactoring through the [TypeScript](https://www.typescriptlang.org/) language service. Refactoring support for other programming languages is provided through VS Code [extensions](/docs/editor/extension-gallery.md) which contribute language services. The UI and commands for refactoring are the same across languages, and in this topic we'll demonstrate refactoring support with the TypeScript language service.

## Code Actions = Quick Fixes and Refactoring

In VS Code, Code Actions can provide both refactorings and Quick Fixes for detected issues (highlighted with green squigglies). An available Code Action is announced by a lightbulb near the source code when the cursor is on a squigglie or selected text region. Clicking on the Code Action lightbulb or using the **Quick Fix** command `kb(editor.action.quickFix)` will display Quick Fixes and refactorings.

If you'd just like to see refactorings without Quick Fixes, you can use the **Refactor** command (`kb(editor.action.refactor)`).

>**Note:** If you prefer to not see the Code Action lightbulb in your editor, you can disable lightbulbs with the `editor.lightbulb.enable` [setting](/docs/getstarted/settings.md). You can still open Quick Fixes through **Quick Fix** command and `kb(editor.action.quickFix)` keyboard shortcut.

## Refactoring

### Extract method or function

Simply select some code, then click on the lightbulb in the gutter or press (`kb(editor.action.quickFix)`) to see available refactorings. Source code fragments can be extracted into a new method, or into a new function at various different scopes. During the extract refactoring you will be prompted to provide a rename.

### Extract variable

TypeScript language service provides **Extract to const**  refactoring to create a new local variable for the currently selected expression:

![Extract local](images/refactoring/ts-extract-local.gif)

When working with classes, you can also extract a value to a new property.

## Rename symbol

Renaming is common operation related to refactoring source code and VS Code has a separate **Rename Symbol** command (`kb(editor.action.rename)`). Some languages support rename symbol across files. Press `kb(editor.action.rename)` and then type the new desired name and press `kbstyle(Enter)`. All usages of the symbol will be renamed, across files.

![Rename](images/refactoring/rename.png)

## Keybindings for Code Actions

The `editor.action.codeAction` command lets you configure keybindings for specific Code Actions. This keybinding for example triggers the Extract function refactoring Code Actions:

```json
{
  "key": "ctrl+shift+r ctrl+e",
  "command": "editor.action.codeAction",
  "args": {
    "kind": "refactor.extract.function"
  }
}
```

Code Action kinds are specified by extensions using the enhanced `CodeActionProvided` API. Kinds are hierarchical, so `"kind": "refactor"` will show all refactoring Code Actions, whereas `"kind": "refactor.extract.function"` will only show Extract function refactorings.

Using the above keybinding, if only a single `"refactor.extract.function"` Code Action is available, it will be automatically applied. If multiple Extract function Code Actions are available, we bring up a context menu to select them:

![Select Code Action context menu](images/refactoring/code-action-context-menu.png)

You can also control how/when Code Actions are automatically applied using the `apply` argument:

```json
{
  "key": "ctrl+shift+r ctrl+e",
  "command": "editor.action.codeAction",
  "args": {
    "kind": "refactor.extract.function",
    "apply": "first"
  }
}
```

Valid values for `"apply"`:

* `"first"` - Always automatically apply the first available Code Action.
* `"ifSingle"` - Default. Automatically apply the Code Action if only one is available. Otherwise, show the context menu.
* `"never"` — Always show the Code Action context menu, even if only a single Code Action is available.

## Extensions with refactorings

You can find extensions that support refactoring by looking in the VS Code [Marketplace](https://marketplace.visualstudio.com/vscode). You can go to the Extensions view (`kb(workbench.view.extensions)`) and typing 'refactor' in the search box. You can then sort by install count or ratings to see which extensions are popular.

<div class="marketplace-extensions-refactor"></div>

> Tip: The extensions shown above are dynamically queried. Click on an extension tile above to read the description and reviews to decide which extension is best for you.

## Next Steps

* [Intro Video - Code Editing](/docs/introvideos/codeediting.md) - Watch an introductory video on code editing features.
* [Code Navigation](/docs/editor/editingevolved) - VS Code lets you move quickly through your source code.
* [Debugging](/docs/editor/debugging.md) - Learn about debugging with VS Code.

## Common Questions

**Q. Why don't I see any lightbulbs when there are errors in my code?**

**A:** Lightbulbs (Code Actions) are only shown when your cursor is on the text showing the error. Hover can show the error description but you need to move the cursor or select text to see lightbulbs for Quick Fixes and refactorings.