---
date: 2023-03-06
title: Generative art!
---

Today we're going to be doing something new! We'll be creating generative art, like so:

![Canvas](/images/generative-art/canvas1.png)

The cool thing about this is that you can download these as pictures, which means you can get them on a canvas and printed out... how cool is that? You can also customize it by adding your own colors (we use random colors!) and punctuation (yep, those are `*`s in the picture). Let's get started!

## Setting up

We'll be building this in Replit. The first thing you'll want to do is go to Replit, click on Create > Search for a template called `canvas-sketch` > Search community templates, and click on the first one you see. It'll take you to [https://replit.com/@jianmin-chen/canvas-sketch-template](https://replit.com/@jianmin-chen/canvas-sketch-template), where you can then click on Use Template.

Give it a name, and you're all good to go!

## Creating da art

When it loads up, you'll get a blank screen like this:

![Blank canvas](/images/generative-art/blank.png)

Tip: Open in a new tab to see your art in its glory!

Before we draw any art, let's take a good look at the code. There's two files with code, `index.js` and `sketch.js`. If you go into `index.js`, you'll see this:

```javascript
const canvasSketch = require("canvas-sketch");
const sketch = require("./sketch.js");
const settings = { dimensions: [2048, 2048] };

canvasSketch(sketch, settings);
```

Here, we're using the library `canvas-sketch`, a collection of code that's going to help us draw the art. We set up the dimensions of our imaginary canvas, which is going to be 2048 by 2048 pixels. This is a very basic template for us to get started with.

The code we're going to write is in `sketch.js`. If you open it, you'll see this:

```javascript
const { lerp } = require("canvas-sketch-util/math");
const random = require("canvas-sketch-util/random");
const palettes = require("nice-color-palettes");

const sketch = () => {};

module.exports = sketch;
```

All of our code is going to go inside the `sketch` function, which is what's used in `index.js` to draw stuff. Right now, it's empty, so we have a blank canvas.

Let's make it not empty. The first lines we're going to add are:

```javascript
random.setSeed(Math.random() * 1000);
const palette = random.pick(palettes);
```

Here, all we're doing is choosing a random color palette.

The next thing we'll do is create a function called `createGrid` to generate some random points using a little bit of math:

```javascript
const createGrid = () => {
    const currPalette = palette.slice(1);
    const symbols = ["*"];
    const points = [];
    const count = 40;
    for (let x = 0; x < count; x++) {
        for (let y = 0; y < count; y++) {
            const u = count <= 0 ? 0.5 : x / (count - 1);
            const v = count <= 0 ? 0.5 : y / (count - 1);
            const size = Math.abs(random.noise2D(u, v) * 0.2);
            points.push({
                color: random.pick(currPalette),
                size,
                rotation: random.noise2D(u, v),
                symbol: random.pick(symbols),
                position: [u, v]
            });
        }
    }

    return points;
};
```

Here, we start by getting the color palette. Then, we have an array of symbols (remember, arrays are just lists of things) - right now it only has `*`. Then, we create an empty array called `points`. We going to add 40 points to this array, as determined by `const count = 40`.

Then, we have a for loop with a for loop inside. This basically generate the x and y of every point. Remember that the top of the canvas is (0, 0) and kind of grows as we move further away to the right and down.

The point's X and Y coordinates are represented by the variables `u` and `v`. The code is a bit interesting:

```javascript
const u = count <= 0 ? 0.5 : x / (count - 1);
const v = count <= 0 ? 0.5 : y / (count - 1);
```

This is just saying: is `count` less than or equal to zero? If so, the least it can be is 0.5. This helps create a margin towards the top and left of the canvas.

Then, we use `random.noise2D` to create a random font size. Finally, we add it to our array of points. Each point will have a:

-   Color
-   Size
-   Rotation - how much to rotate it by
-   Symbol - if you have more than one symbol, it will choose a random one
-   And of course, the position

The last thing we do is `return points`.

Now let's run this function inside `sketch`:

```javascript
const points = createGrid();
```

Here's how your code should look like now:

```javascript
const sketch = () => {
    const palette = random.pick(palettes);
    const createGrid = () => {
        const currPalette = palette.slice(1);
        const symbols = ["="];
        const points = [];
        const count = 40;
        for (let x = 0; x < count; x++) {
            for (let y = 0; y < count; y++) {
                const u = count <= 0 ? 0.5 : x / (count - 1);
                const v = count <= 0 ? 0.5 : y / (count - 1);
                const size = Math.abs(random.noise2D(u, v) * 0.2);
                points.push({
                    color: random.pick(currPalette),
                    size,
                    rotation: random.noise2D(u, v),
                    symbol: random.pick(symbols),
                    position: [u, v]
                });
            }
        }

        return points;
    };

    const points = createGrid();
};
```

Now let's actually get these points on the canvas! Add this code to `sketch()`:

```javascript
return ({ context, width, height }) => {
    context.fillStyle = palette[0];
    context.fillRect(0, 0, width, height);
    points.forEach(({ color, size, rotation, symbol, position: [u, v] }) => {
        const x = lerp(0, width, u);
        const y = lerp(0, height, v);

        context.save();
        context.fillStyle = color;
        context.font = `${size * width}px 'Arial'`;
        context.translate(x, y);
        context.rotate(rotation);
        context.fillText(symbol, 0, 0);
        context.restore();
    });
};
```

This basically returns a function. It doesn't look like one, but you can notice the overarching outline of `() => {}`. Except, this time, we're actually passing in arguments! Specifically, we're passing in `{ context, width, height }`. These are going to be passed in by the `canvas-sketch` library.

Inside, we start working with the canvas. The first thing we do is fill in the background with the first color in our palette.

Then, we go through each point we created using `forEach`. `const x = lerp(0, width, u)` and `const y = lerp(0, height, v)` simply "map" the point to the actual canvas. (Imagine scaling up on a graph, like going from 0.75 to 75% of 2048 pixels. That's what `lerp` is used for.) 

Afterwards, we save the canvas before drawing in our point. To do this, we change the `fillStyle` color again, change the font size to the random size we generated, move to `x` and `y` with `translate`, rotate our canvas by `rotation` (imagine a painter actually rotating the screen), and then filling in the text on the screen. Finally, we do `restore()`, which resets the translation and rotation (painter going back to bird's eye view).

That's it! Your canvas should look something like this now:

![Canvas](/images/generative-art/canvas2.png)

If you right click on it, you can click on "Save as image". And boom, you have your own art! How cool is that!

## Adding finishing touches

There's a bunch of stuff you could do with this. Try adding some of your own flair!

* Add more symbols. You can also use whole words for this. What about emojis? Hehe.
* Add a color palette. There's lots to find online.

### Create a gallery

One cool thing you can do is create a gallery. In `index.js`, simply replace `canvasSketch(sketch, settings);` with:

```javascript
for (let i = 0; i < 10; i++) {
    canvasSketch(sketch, settings);
}
```

This draw 10 pictures. There's only one problem - these pictures all look the same! That's because we're using the same seed for `random()`.

> Sidenote: Computers don't actually create random numbers when you tell them to do so! They actually create pseudo-random numbers using a complex algorithm that takes an initial starting point called a seed.

Let's fix this real quick. Inside `sketch()`, add the following line to the top:

```javascript
random.setSeed(Math.random() * 1000);
```

That sets a random seed! Awesome.
