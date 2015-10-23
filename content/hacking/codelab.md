+++
Categories = ["Hacking"]
Description = ""
Tags = ["Google Devfest 2015", "HTML5", "Canvas", "Game", "Codelab"]
date = "2015-10-07T13:35:54+05:30"
menu = "main"
title = "Codelab - HTML5 Canvas and Game development"
+++

## Objective

This codelab will demonstrate how to build a game using HTML5 canvas. As an example, we will build a simple snake game.

We will walk you through the process of building the in a series of incremental steps. After each step you should have a working (not necessarily useful) version of the game. Each step is described in the form of a tag in the git repository.

## Requirements

The requirements for this codelab are:

* Basic familiarity with HTML.
* Basic familiarity with JavaScript.
* Basic knowledge in git will be helpful.

## Why HTML5?

The most obvious advantage for creating a HTML5 game is that it is cross platform. The same codebase can be packaged to run on browsers, Andorid, iOS, Windows Phone or any other modern device.

Another advantage the pervasiveness of javascript. It is everywhere and very easy to get started with. Most people are already familiar with Javascript and that all that is needed to develop a HTML5 game. The development process is fairly easy. No waiting for compilation, updates and debugging in real-time, and once the game is done, you can push out the update immediately.

## Step 0: Getting the Code

You can get the code from https://github.com/shivanshuag/codelab-snake
Clone the repository using `git clone https://github.com/shivanshuag/codelab-snake`.

You can check out each tag using `git checkout [tagname]`

To run the code, simply open the `index.html` file in the browser of your choice.

## Preliminaries

### Step 1: The Canvas Tag


`git checkout step1`

{{< highlight html >}}
<body>
<canvas id="myCanvas" width="578" height="200"></canvas>
<script>
  var canvas = document.getElementById('myCanvas');
  var context = canvas.getContext('2d');
  // do stuff here
</script>
</body>
{{< /highlight >}}

The canvas tag is similar to the `<div>`, `<a>`, or `<table>` tag, with the exception that its contents are rendered with JavaScript. The `width` and `height` attributes are used to specify the dimensions of the canvas.

`document.getElementById('myCanvas')` gives the reference to the canvas DOM object in the html document.

`canvas.getContext('2d')` gives the canvas context which is an object with properties and methods that you can use to render graphics inside the canvas element.  The context can be 2d or webgl (3d).

### Step 2: Drawing A Line

`git checkout step2`

{{< highlight html >}}
<body>
<canvas id="myCanvas" width="578" height="200"></canvas>
<script>
  var canvas = document.getElementById('myCanvas');
  var context = canvas.getContext('2d');

  context.beginPath();
  context.moveTo(100, 150);
  context.lineTo(450, 50);
  context.lineWidth = 15;
  context.strokeStyle = '#ff0000';
  context.lineCap = 'round'
  context.stroke();
</script>
</body>
{{< /highlight >}}

* `beginPath()` - to declare that we are about to draw a new path

* `moveTo()` - to position(coordinates) the context point (i.e. Starting point)

* `lineTo()` -  to draw a straight line from the starting position to a new position

* `lineWidth` - Sets the width of the line

* `strokeStyle` - color of the line

* `linecap` - Can be round, square or butt

* `stroke()` - to make the line visible i.e fill color in the line

All the properties of the line should be set before calling the stroke function.

### Step 3: Arc

`git checkout step3`

{{< highlight html >}}
<body>
<canvas id="myCanvas" width="578" height="200"></canvas>
<script>
  var canvas = document.getElementById('myCanvas');
  var context = canvas.getContext('2d');

  var x = canvas.width / 2;
  var y = canvas.height / 2;
  var radius = 75;
  var startAngle = 1.1 * Math.PI;
  var endAngle = 1.9 * Math.PI;
  var counterClockwise = false;
  context.beginPath();
  context.arc(x, y, radius, startAngle, endAngle, counterClockwise);
  context.lineWidth = 15;
  context.strokeStyle = 'black';
  context.stroke();
</script>
</body>
{{< /highlight >}}

* Arcs are defined by a center point, a radius, a starting angle, an ending angle, and the drawing direction (either clockwise or anticlockwise).
* Angle should be in radians.
* Arcs can be styled with the lineWidth, strokeStyle, and lineCap properties

#### Bonus Points
* Explore how to make Quadratic Curve and Bezier Curves and paths in the canvas.

### Step 4: Rectangle

`git checkout step4`

{{< highlight html >}}
<body>
<canvas id="myCanvas" width="578" height="200"></canvas>
<script>
  var canvas = document.getElementById('myCanvas');
  var context = canvas.getContext('2d');

  context.beginPath();
  context.rect(188, 50, 200, 100);
  context.fillStyle = 'yellow';
  context.fill();
  context.lineWidth = 7;
  context.strokeStyle = 'black';
  context.stroke();
</script>
</body>
{{< /highlight >}}

* `context.fillStyle` - Specifies the color to be filled inside the rectangle.
* `context.fill()` - Draws a solid shape by filling the path's content area.
* `context.stroke()` - Draws the outline of the shape.

When setting both the fill and stroke for a shape, make sure that you use fill() before stroke().Otherwise, the fill will overlap half of the stroke.

### Step 5: Images

`git checkout step5`

{{< highlight html >}}
<body>
<canvas id="myCanvas" width="578" height="200"></canvas>
<script>
  var canvas = document.getElementById('myCanvas');
  var context = canvas.getContext('2d');

  var imageObj = new Image();
  imageObj.onload = function() {
  	context.drawImage(imageObj, 69, 50);
  };
  imageObj.src = 'http://www.html5canvastutorials.com/demos/assets/darth-vader.jpg';
</script>
</body>
{{< /highlight >}}

* `image.onload` is a function which is called when the image has been fetched from its source.
* `drwaImage` function draws the image at the given coordinates.
* To change the size of the image, add two additional arguments to the drawImage() method, width and height. `context.drawImage(imageObj, x, y, width, height);`

#### Bonus Point
* Explore how to crop an image and then draw it on canvas.

### Step 6: Text

`git checkout step6`

{{< highlight html >}}
<body>
<canvas id="myCanvas" width="578" height="200"></canvas>
<script>
  var canvas = document.getElementById('myCanvas');
  var context = canvas.getContext('2d');

  context.font = 'italic 40pt Calibri';
  context.fillText('Hello World!', 150, 100);
</script>
</body>
{{< /highlight >}}

* To change set the `fillStyle` abd `strokeStyle` in same ways as for shapes.
* To align text, use `context.textAlign = "center"`. It can also be set to start, end, left, center, or right.
* When it's set to start and the document direction is ltr (left to right), or when it's set to end and the document direction is rtl (right to left).

## Snake Game

Now we will create a game using the tools we just learnt.

### Step 7: Creating Arena for the Snake

`git checkout step7`

{{< highlight html >}}
<body>
<canvas id="myCanvas" width="450" height="450"></canvas>
<script>
  var canvas = document.getElementById('myCanvas');
  var context = canvas.getContext('2d');
  var w = canvas.width
  var h = canvas.height

  context.fillStyle= "white";
  context.fillRect(0 , 0, w, h);
  context.strokeStyle = "black";
  context.strokeRect(0, 0, w, h);
</script>
</body>
{{< /highlight >}}

This will create a 450x450 canvas with a balck boundary.

### Step 8: Creating the Snake

`git checkout step8`

{{< highlight js >}}
function create_snake() {
var length = 5; //Length of the snake
snake_array = []; //Empty array to start with
for(var i = length-1; i>=0; i--) {
  //This will create a horizontal snake starting from the top left
  snake_array.push({x: i, y:0});
}
}
create_snake();   // calling the create_snake function
{{< /highlight >}}

Here, each element in the array represents a cell of the snake. Initially, the snake is made up of 5 cells. In a later step, we will write logic to increase the length of the snake when it eats food.

### Step 9: Painting the Snake

`git checkout step9`

{{< highlight js >}}
var cw = 10;//cell width
function paint() {
  for(var i = 0; i < snake_array.length; i++) {
    var c = snake_array[i];
    context.fillStyle  = "blue";
    //Lets paint 10px wide cells
    context.fillRect(c.x*cw, c.y*cw, cw, cw);
    context.strokeStyle  = "white";
    context.strokeRect(c.x*cw, c.y*cw, cw, cw);
  }
}
paint();
{{< /highlight >}}

In the earlier step, we just created a array containing the coordinates of all the cells of the snake. In this step, we are painting all the cells on the canvas.

`context.fillRect` fills a rectangle and `context.strokeRect` creates its boundary.

### Step 10: Moving the snake

`git checkout step10`

In this step, we will create the Game Loop. Game Loop contains the statements which will be called in every frame of the game. These instructions will be responsible for updating the state of the game (like the position of the snake, handling input).

{{< highlight js >}}
game_loop = setInterval(paint, 60);

//Put the following lines inside the paint function
  //Pop out the tail cell and place it infront of the head cell
  var nx = snake_array[0].x;
  var ny = snake_array[0].y;
  //These were the position of the head cell.
  //We will increment it to get the new head position
  nx++;
  var tail = snake_array.pop(); //pops out the last cell
  tail.x = nx;
  tail.y = ny;
  snake_array.unshift(tail); //puts back the tail as the first cell
{{< /highlight >}}

Game loop is a javascript setInterval timer. It calls the `paint` function every 60 milliseconds.

We modify the paint function to include logic for moving the snake forward in the right direction by popping out its tail and pushing it ahead of the head in the `snake_array`.

After every 60 ms, paint() function will create new rectangle but will not delete the previous rectangles. This will leave a trail of the movement of the snake. So, we will first paint the background of the canvas white inside the paint() function.

{{< highlight js >}}
// Put the following lines at the start of the paint function
context.fillStyle= "white";
context.fillRect(0 , 0, w, h);
context.strokeStyle = "white";
context.strokeRect(0, 0, w, h);
{{< /highlight >}}

### Step 11: Handling User Input

`git checkout step11`

First, we will add direction based movement to the snake. Instead of the line `nx++` in the code, add the following snippet.

{{< highlight js >}}
var d = "right"; //default direction. Add this line outside the paint function

//add the following lines in the paint function replacing 'nx++'
if(d == "right") nx++;
else if(d == "left") nx--;
else if(d == "up") ny--;
else if(d == "down") ny++;

{{< /highlight >}}

Add a key listener and change the direction of the snake accordingly.

{{< highlight js >}}
document.onkeydown = function(e) {
  var key = e.which;
  if(key == "37" && d != "right") d = "left";
  else if(key == "38" && d != "down") d= "up";
  else if(key == "39" && d != "left") d= "right";
  else if(key == "40" && d != "up") d= "down";
};
{{< /highlight >}}

`onkeydown` function is called when the user presses a key and `e.which` contains the keycode of the key which the user has pressed. Refer to [https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/keyCode](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/keyCode) for a list of the keycodes.

### Step 12: Creating Food

`git checkout step12`

First, we will make a function to paint cells. This function can be used while painting the cells of the snake too.

{{< highlight js >}}
function paint_cell(x,y) {
  context.fillStyle = "blue";
  context.fillRect(*cw, *cw, cw, cw);
  context.strokeStyle = "white";
  context.strokeRect(*cw, *cw, cw, cw);
}
{{< /highlight >}}

Then, we create a function which gives the coordinates of a random cell which is to be used as food.

{{< highlight js >}}
var food;  // global variable which contains food
function create_food() {
  food = {
    x: Math.floor(Math.random()*(w-cw)/cw),
    y: Math.floor(Math.random()*(h-cw)/cw),
  };
  //This will create a cell with x and y between 0-44
  //Because there are 45(450/10) positions accross the rows and columns
}
create_food();

//Inside the paint function
paint_cell(food.x, food.y);
{{< /highlight >}}

`Math.random()` gives a random value between 0 and 1.

### Step 13: Eating the Food

`git checkout step13`

{{< highlight js >}}
//Inside the paint function
//If the new head position matches with that of the food,
//Create a new head instead of moving the tail
if(nx == food.x && ny == food.y) {
  var tail = {x: nx, y: ny};
  //Create new food
  create_food();
}
else {
  var tail = snake_array.pop(); //pops out the last cell
  tail.x = nx; tail.y = ny;
}
{{< /highlight >}}

### Step 14: Game Over Condition

`git checkout step14`

First create an init function which will restore the game state to the initial state. Move the code used to set the initial state of the game to the init function and call it when the page loads.

{{< highlight js >}}
var game_loop; // Make gameloop a global variable similar to d, food and snake_array.

function init() {
  d = "right"; //default direction
  create_snake();
  create_food();
  game_loop = setInterval(paint, 60);
}

init(); // call init when the script loads.
{{< /highlight >}}

Add the following condition inside the paint function to detect collision with the boundaries.

{{< highlight js >}}
if(nx==-1 || nx==w/cw || ny==-1 || ny==h/cw) {
  // clear the game loop
  clearInterval(game_loop);
  init();
  return;
}
{{< /highlight >}}

`clearInterval` will clear the previous game_loop and the `init` function will create a new one.

### Step 15: Checking Body Collision

`git checkout step15`

Create the `check_collision` function as follows.

{{< highlight js >}}
function check_collision(x, y, array) {
  //This function will check if the provided x/y coordinates exist
  //in an array of cells or not, excluding the head
  for(var i = 1; i < array.length; i++) {
    if(array[i].x == x && array[i].y == y)
      return true;
  }
  return false;
}

{{< /highlight >}}

Modify the collision condition as follows

{{< highlight js >}}
if(nx==-1 || nx==w/cw || ny==-1 || ny==h/cw || check_collision(nx, ny, snake_array)) {
  // clear the game loop
  clearInterval(game_loop);
  init();
  return;
}
{{< /highlight >}}

### Step 16: Implement Scoring

This is a DIY step!

## Where to go From Here?

* Explore some awesome [Game Development libraries for Javascript canvas](https://html5gameengine.com/).
* Try making a more sophisticated game involving sprites, animations and sounds.
* Look at `three.js` for making 3d games.
* Learn how to package your HTML5 game for [Android](http://phonegap.com/) and [Windows Phone](https://www.youtube.com/watch?v=aNY4oZByTko).%
