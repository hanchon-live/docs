---
title: "Useful Keyboard Shortcuts for VSCode"
date: 2021-04-24T22:10:30+01:00
categories:
  - tips
tags:
  - vscode
  - python
  - terminal
  - keybindings
last_modified_at: 2021-04-24T12:59:30+01:00
---
To improve your speed when working with `VSCode` you can set custom keybindings, this is a list with my custom keyboard shortcuts.

{% include toc icon="cog" title="Content" %}

# Open Keyboard Shortcuts configuration on VSCode
Press `F1` and type `Keyboard Shortcuts (JSON)`, this will open the `keybind.json` file where you can add any keybinding.

{% include figure image_path="/assets/posts/useful-keybindings/keyboard-shorcuts.png" alt="Keyboard Shortcuts - keybindings" caption="" %}

# Move between terminal and editor.
This keybinding will allow you to move between the editor and your terminal (back and forth) by just pressing `control`+ `` ` ``.

That way you can create a new terminal using `control`+`shift`+ `` ` ``, and then pressing the same combination without the `shift` key will move your cursor between windows.
``` json
[
    { "key": "ctrl+`", "command": "workbench.action.terminal.focus"},
    { "key": "ctrl+`", "command": "workbench.action.focusActiveEditorGroup", "when": "terminalFocus"}
]
```