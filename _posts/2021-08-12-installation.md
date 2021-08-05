---
layout: post
title: Installing a few things
---

# Install Excalidraw

Follow [README](https://github.com/excalidraw/excalidraw#readme) to install it

To run production build

```
yarn build
npm install -g serve
```

```
start_excalidraw () {
	cd $HOME/excalidraw
	serve -s build
}
```