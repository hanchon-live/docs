---
title: "Copy from terminal output using xclip"
date: 2021-05-15T10:30:30+02:00
categories:
  - tips
tags:
  - terminal
  - debian
  - ubuntu
  - console
last_modified_at: 2021-05-15T10:30:30+02:00
---

In this post we are going to copy the terminal output using `xclip`, this is useful when you need to take the data outside the terminal.

# Install xclip on Ubuntu/Debian

The first step is to install the application in our system:

``` sh
sudo apt-get install xclip
```

## Create alias:

Let's crete alias so `xclip` works the same as `pbcopy`/`pbpaste` when using MacOS.

If you are using bash, edit your ~/.bashrc file adding these lines:
``` sh
alias pbcopy='xclip -selection c'
alias pbpaste='xclip -o -selection c'
```

# Example

Let's make a request to an API:

```sh
curl https://pokeapi.co/api/v2/pokemon/mew
```

If we didn't have `xclip`, we could select all the information using the mouse and then press `control+shift+c` to copy it.

But this data is very long (400k characters), so it's a waste of time to select it all and it's not conformable to scroll several pages of the terminal.

Let's use the alias to redirect the curl output to the clipboard

``` sh
curl https://pokeapi.co/api/v2/pokemon/mew | pbcopy
```

Now we can paste the content anywhere we want.
