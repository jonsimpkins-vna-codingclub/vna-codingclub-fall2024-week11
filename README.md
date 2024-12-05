# Week 11: Recursion & Fractals

**What are we building:** We're going to build an animated fractal using p5.js, with the help of recursion

## What's Recursion?

Recursion is when you have a method that invokes itself, and then in that invocation it might invoke itself yet again, and so on.

Example:

```js
function factorial(n) {
    if (n == 1) {
        return 1;
    }

    return n * factorial(n-1);
}
```

So in the above example `factorial(3)` would end up returning `3 * factorial(2)`, meaning that the method depends on itself.

**IMPORTANTLY:** every method with recursion needs what we call a "terminal condition" - a way for it to know to __stop__ calling itself. In the above example, this happens in the if statement about `n==1`. If you don't do this, the recursion goes on infinitely and your computer crashes:

```js
function badRecursion(n) {
    // This will call itself forever, because it doesn't have
    // a terminal condition
    return n + badRecursion(n-1);
}
```


## Step 0

Ok, so to start off, let's open a new p5 project:

> https://editor.p5js.org/

We'll update the `sketch.js` code to set up a canvas that's 400 px square, and let's draw a circle in the center of it:

```js
function setup() {
  createCanvas(400, 400);
}

function draw() {
  // Black background
  background(0);
  
  // White lines, not filled in
  stroke(255);
  noFill();
  
  // Draw a circle in the center
  circle(200, 200, 100);
}
```

Alright, not a very exciting design yet, let's make it interesting...

## Step 1

So initially, we're drawing just one circle in the center of the screen. Let's try adding four smaller circles, where each one is smaller, and they're all positioned around the edge of the first circle. Go ahead and add:

```js
  // 4 smaller circles
  circle(150, 200, 50);
  circle(250, 200, 50);
  circle(200, 150, 50);
  circle(200, 250, 50);
```

to the end of your `draw()` method.

This is good and fine for now to just hard-code these 5 circles, but let's look at how the big one is related to the smaller ones:

* The smaller circles are half as big as the large one (diameter of `50` vs diameter of `100`)
* The smaller circles are located at the same center point (`(200, 200)`), plus or minus 50 (the small radius) on each dimension

Let's try changing our code so we get the same result, but without as many hard-coded numbers:

## Step 2

We're going to make a new method that draws the big circle **and** the smaller circles:

```js
function drawConstellation(xCenter, yCenter, diameter) {
  // Draw the larger circle
  circle(xCenter, yCenter, diameter);
  
  // Smaller circles are half as big
  let smallerDiameter = diameter * 0.5;
  
  // Draw 4 smaller circles
  circle(xCenter - smallerDiameter, yCenter, smallerDiameter);
  circle(xCenter + smallerDiameter, yCenter, smallerDiameter);
  circle(xCenter, yCenter - smallerDiameter, smallerDiameter);
  circle(xCenter, yCenter + smallerDiameter, smallerDiameter);
}
```

If you update your `draw()` method to call this new method, your full code should now look sort of like:

```js
function setup() {
  createCanvas(400, 400);
}

function draw() {
  // Black background
  background(0);
  
  // White lines, not filled in
  stroke(255);
  noFill();
  
  // Draw a constellation of circles
  drawConstellation(200, 200, 100);
}

function drawConstellation(xCenter, yCenter, diameter) {
  // Draw the larger circle
  circle(xCenter, yCenter, diameter);
  
  // Smaller circles are half as big
  let smallerDiameter = diameter * 0.5;
  
  // Draw 4 smaller circles
  circle(xCenter - smallerDiameter, yCenter, smallerDiameter);
  circle(xCenter + smallerDiameter, yCenter, smallerDiameter);
  circle(xCenter, yCenter - smallerDiameter, smallerDiameter);
  circle(xCenter, yCenter + smallerDiameter, smallerDiameter);
}
```

Ok, this is a bit better, fewer `100` / `200` values running around, but the picture looks exactly like before. So why change things up?

What about if we wanted to draw **even smaller** circles around those small circles? And **still even smaller** circles around those really small circles? That sounds a lot like a recursion problem!

## Step 3

Remember that the trick for recursion is to make sure that:

> the recursion method has a terminal state

In our case, let's stop the recursion when the circle diameter gets too small. To do that, let's update `drawConstellation` to start with this early exit:

```js
function drawConstellation(xCenter, yCenter, diameter) {
  if (diameter < 16) {
    // Circle's too small, don't draw or do recursion
    return;
  }

  // Draw the larger circle
  circle(xCenter, yCenter, diameter);

  // ... rest of method stays the same ...
```

and now here's the interesting bit: instead of calling `circle()` when we draw the smaller circles, we're going to instead call `drawConstellation()`.

After these changes, your method should look like:

```js
function drawConstellation(xCenter, yCenter, diameter) {
  if (diameter < 16) {
    // Circle's too small, don't draw or do recursion
    return;
  }
  
  // Draw the larger circle
  circle(xCenter, yCenter, diameter);
  
  // Smaller circles are half as big
  let smallerDiameter = diameter * 0.5;
  
  // Draw 4 smaller constellations
  drawConstellation(xCenter - smallerDiameter, yCenter, smallerDiameter);
  drawConstellation(xCenter + smallerDiameter, yCenter, smallerDiameter);
  drawConstellation(xCenter, yCenter - smallerDiameter, smallerDiameter);
  drawConstellation(xCenter, yCenter + smallerDiameter, smallerDiameter);
}
```

So you should now see:

* 1 big circle
* 4 smaller circles around that
* 4 tiny circles around each of those (16 total)

So technically, you could have done this by just hard-coding 21 `circle()` commands, why bother with recursion?

Welp, let's try changing the terminal condition:

```js
if (diameter < 2) { // Used to be 16, now it's 2
```

Now, you have a design with hundreds of intricate circles, perfectly arranged, and all you had to do was switch `16` to be `2`.

What else can we do?

## Step 4

Let's try drawing the circles in a different color as they get smaller (that might make it easier to see the pattern as we add more detail).

To do that, let's update our method to take in a new parameter for color:

```js
function drawConstellation(xCenter, yCenter, diameter, lineColor /* new */)
{

  // Use new lineColor
  stroke(lineColor);

  // ... rest of method ...

```

and then as we do recursion, let's have the smaller constellations get a value that's 80% of the value of the big circle (that will make the smaller ones darker, and the really tiny ones even darker):

```js
  // Smaller circles are half as big, and darker
  let smallerDiameter = diameter * 0.5;
  let smallerColor = 0.8 * lineColor;
  
  // Draw 4 smaller constellations
  drawConstellation(xCenter - smallerDiameter, yCenter, smallerDiameter, smallerColor);
  drawConstellation(xCenter + smallerDiameter, yCenter, smallerDiameter, smallerColor);
  drawConstellation(xCenter, yCenter - smallerDiameter, smallerDiameter, smallerColor);
  drawConstellation(xCenter, yCenter + smallerDiameter, smallerDiameter, smallerColor);
```

Ok, this is looking better, but it's still pretty cluttered.

: facepalm :

You know why it's cluttered? Because we're drawing all the small circles **on top of** the big circles. Let's try moving the "big circle" bit to the bottom of the method, so we draw those last:

```js
  // ... method above this ...
  
  // Use new lineColor
  stroke(lineColor);
  
  // Draw the larger circle
  circle(xCenter, yCenter, diameter);
} // end of method
```

Aha! Much better. Here's the full "current" version, just for reference:

```js
function setup() {
  createCanvas(400, 400);
}

function draw() {
  // Black background
  background(0);
  
  // Lines, not filled in
  noFill();
  
  // Draw a constellation of circles
  drawConstellation(200, 200, 100, 255);
}

function drawConstellation(xCenter, yCenter, diameter, lineColor) {
  if (diameter < 4) {
    // Circle's too small, don't draw or do recursion
    return;
  }

  // Smaller circles are half as big, and darker
  let smallerDiameter = diameter * 0.5;
  let smallerColor = 0.8 * lineColor;
  
  // Draw 4 smaller constellations
  drawConstellation(xCenter - smallerDiameter, yCenter, smallerDiameter, smallerColor);
  drawConstellation(xCenter + smallerDiameter, yCenter, smallerDiameter, smallerColor);
  drawConstellation(xCenter, yCenter - smallerDiameter, smallerDiameter, smallerColor);
  drawConstellation(xCenter, yCenter + smallerDiameter, smallerDiameter, smallerColor);

  // Use new lineColor
  stroke(lineColor);
  
  // Draw the larger circle
  circle(xCenter, yCenter, diameter);
}
```

But it's still kind of... static. Maybe we add some animation?

## Step 5

To animate our design, we'll leverage the `frameCount` value from p5.js, and the `cos()` method (cosine, which generates a number between -1 and 1).

For starters, let's animate the size of the circles, to be between 100 and 300 in diameter. The cool thing is, we **only** have to change the main `draw()` method to do this, all the recursion should react appropriately in response:

```js
function draw() {
  // Black background
  background(0);
  
  // Lines, not filled in
  noFill();
  
  let bigDiameter = 200 + 100 * cos(frameCount * 0.05);
  
  // Draw a constellation of circles
  drawConstellation(200, 200, bigDiameter, 255);
}
```

Now we kind of zoom in and out of the design, and you might actually notice that as we zoom in, that your computer is getting slower. The reason why is that we're actually **recursing deeper** when that happens, since the terminal condition is based on how tiny the tiniest circles are: when you start with a larger initial circle, you actually end up drawing more total circles, and the animation takes longer to draw.

Let's try also adding some animation to the placement of the circles. We're going to leverage some trigonometry to do this, but basically we're going to choose the "up, right, down, left" positions of the smaller constellations in a way that rotates around over time:

```js
function drawConstellation(xCenter, yCenter, diameter, lineColor) {
  if (diameter < 10) { /** CHANGE THIS VALUE TO 10 **/
    // Circle's too small, don't draw or do recursion
    return;
  }

  // Smaller circles are half as big, and darker
  let smallerDiameter = diameter * 0.5;
  let smallerColor = 0.8 * lineColor;

  /** THIS PART IS NEW **/
  let numChildren = 4;
  
  // Let's draw numChildren smaller constellations,
  // spaced out evenly around this circle
  for (let i = 0; i < numChildren; i++) {
    // Animate the starting rotation over time...
    let startingRotation = frameCount / 100;
    
    // Trigonometry is magic
    let rot = startingRotation + (2 * i / numChildren * PI);
    drawConstellation(
      xCenter + (smallerDiameter * cos(rot)),
      yCenter + (smallerDiameter * sin(rot)),
      smallerDiameter,
      smallerColor
    );
  }

  /** END OF NEW STUFF **/

  // Use new lineColor
  stroke(lineColor);
  
  // Draw the larger circle
  circle(xCenter, yCenter, diameter);
}
```

Right on, so now it rotates **and** zooms in.

> Just out of curiosity, what happens if you change `numChildren` to be a different number like `3` or `5`?

Alright, one more variation: what if we change `startingRotation` to depend on the `diameter` value? That'll mean that the farther down in the recursion we go, the faster or slower the rotation will occur, kind of like small gears spinning around big gears:

```js
let startingRotation = 0.25 * (frameCount / diameter);
```

## Step 6

Where could you go from here?

* Try changing the `numChildren` value based on how deep in the recursion you are (draw fewer children deeper down, or maybe draw more)
* Try changing the color to be more... colorful, not just black and white
* Try drawing different shapes besides circles. Remember the [reference](https://p5js.org/reference/#Shape) section for how to draws squares, ellipses, triangles, lines.
* Try using something like [mouseX](https://p5js.org/reference/p5/mouseX/) to change the drawing based on where your mouse is (maybe use `mouseX` to determine the starting diameter, or the starting rotation)
