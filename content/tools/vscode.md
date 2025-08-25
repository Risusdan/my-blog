+++
date = '2025-08-23T12:16:10+08:00'
draft = false
title = 'Visual Studio Code Extensions'
+++

Here are the extensions that I use frequently:

- Material Icon Theme
- One Monokai Theme
- indent-rainbow
- Peacock
- C/C++
- Clang-Format
- Python
- Pylance
- Python Debugger
- Markdown All in One
- Markdown Preview Github Styling
- Git Graph
- Bookmarks
- CodeSnap

## Customized Settings

### Peacock

1. Create a JSON file at `./vscode/settings.json`.
2. Copy and paste the following format to the JSON file.

```json
{
    "peacock.favoriteColors": [
        { "name": "My Color - Blue", "value": "#375669" },
        { "name": "My Color - Green", "value": "#456525" },
        { "name": "My Color - Red", "value": "#82231c" },
        { "name": "My Color - Yellow", "value": "#9e7e38" },
        { "name": "My Color - Purple", "value": "#511536" }
    ]
}
```

3. Then you can swith the editor's color by `Peacock: Change to a Favorite Color` in the command palette.

### CodeSnap

1. Open *Settings* and search for CodeSnap settings.
2. Change the following settings.

| Item                 | Change to |
|----------------------|-----------|
| Container Padding    | 0em       |
| Rounded Corners      | uncheck   |
| Show Window Controls | uncheck   |
