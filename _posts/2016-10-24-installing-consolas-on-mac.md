---
title: Installing Consolas Font on macOS Sierra
categories:
- install
tags:
- Consolas
- Font
- macOS Sierra
- mac OS X
---

I'm a big fan of using [Consolas Font Family](https://www.microsoft.com/typography/fonts/family.aspx?FID=300) for programming purposes, the one from [Visual Studio](https://www.visualstudio.com/). Well, it turns out that I'm no longer developing under Windows(thanks to aliens 👽), but still I want to use the font on my mac for the same purpose.

If you  have my same taste, you're in luck. Just follow this steps to achieve it.

Open your preferred terminal manager app(mine os [iTerm2](https://www.iterm2.com/)) and type following

```
brew install cabextract
```

This instructs [Homebrew](http://brew.sh/) to install [cabextract](http://www.cabextract.org.uk/), which allow us to extract *Microsoft cabinet files*, _aka_ *.CAB files*

`cd` into `Downloads` directory, create a new one(lets call it `consolas`) and `cd` into it.

```
cd ~/Downloads
mkdir consolas
cd consolas
```

Now, proceed to download a MS Power Point viewer application, which contains the font we're looking for.

```
curl -O http://download.microsoft.com/download/f/5/a/f5a3df76-d856-4a61-a6bd-722f52a5be26/PowerPointViewer.exe
```

We're almost done. Now extract the `PowerPointViewer.exe` executable using `cabextract` file which contains a `ppviewer.cab`.
That's file you're looking for. Extract it and proceed to open  the `CONSOLA*.TTF` file

```
cabextract PowerPointViewer.exe
cabextract ppviewer.cab
open CONSOLA*.TTF
```

Voilà!. You successfully installed the font. Now you can delete the directory `consolas` and all its content
