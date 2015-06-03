# Mobile Portfolio Website Optimization

The goal is to optimize a provided website with a number of optimization- and performance-related issues so that it achieves a target PageSpeed score of 90 and runs at 60 frames per second.

My PageSpeed Insights Speed scores:

<table>
  <thead>
    <tr>
      <th>page</th>
      <th>mobile</th>
      <th>desktop</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>index.html</td>
      <td>95/100</td>
      <td>92/100</td>
    </tr>
    <tr>
      <td>project-2048.html</td>
      <td>93/100</td>
      <td>94/100</td>
    </tr>
    <tr>
      <td>project-mobile.html</td>
      <td>95/100</td>
      <td>97/100</td>
    </tr>
    <tr>
      <td>project-webperf.html</td>
      <td>95/100</td>
      <td>97/100</td>
    </tr>
    <tr>
      <td>pizza.html</td>
      <td>97/100</td>
      <td>100/100</td>
    </tr>
  </tbody>
</table>

<p>Find the optimized site <a href="http://fffplok.github.io/frontend-nanodegree-mobile-portfolio/">here</a>.</p>

## index.html

* eliminated link to google fonts so there is no download of font
* link to style.css replaced with style element in document head with declarations from style.css
* link to print.css added attribute media="print" so it will only load for printing
* scripts with src attribute given async to prevent render blocking

## pizza.html

I put all style information into the head of pizza.html. Any declarations from bootstrap-grid.css of anything not col-md-8 were eliminated as they were not used. Bootstrap declarations were minified and then the minified version of style.css was appended.

Images were optimized using loss-less compression. pizzeria.jpg was resized from unnecessarily large size.

<b>javascript, generally:</b>

* reduced the number of times the keyword var is used by placing declared variables in comma separated list.
* replaced querySelectorAll with getElementsByClassName
* replaced querySelector with getElementsById
* minified main.js to main.min.js


<b>javascript, specific examples:</b>
### updatePositions

This function moves the sliding background pizzas based on scroll position

#### original

* note use of querySelectorAll to obtain items
* phase is calculated every time in loop

```javascript
function updatePositions() {
  ...
  var items = document.querySelectorAll('.mover');
  for (var i = 0; i < items.length; i++) {
    var phase = Math.sin((document.body.scrollTop / 1250) + (i % 5));
    items[i].style.left = items[i].basicLeft + 100 * phase + 'px';
  }
  ...
}
```

#### refactored

* items now obtained via getElementsByClassName
* minimized number of caculations
* there are only 5 phases, calculate each just once and hold in phases array for quick access

```javascript
function updatePositions() {
  ...
  var i, phases = [],
      scrollExtent = document.body.scrollTop / 1250,
      items = document.getElementsByClassName('mover');

  for (i = 0; i < 5; i++) {
    phases.push(Math.sin(scrollExtent + (i % 5)));
  }
  for (i = 0; i < items.length; i++) {
    items[i].style.left = items[i].basicLeft + 100 * phases[i%5] + 'px';
  }
  ...
}
```
<p>&nbsp;</p>
### DOMContentLoaded event listener
Generates the sliding pizzas when the page loads.

#### original

* note 200 images will be used... more than needed to fill the viewport
* querySelector is invoked every iteration of the loop
* there is a fractional width specified for each image

```javascript
document.addEventListener('DOMContentLoaded', function() {
  var cols = 8;
  var s = 256;
  for (var i = 0; i < 200; i++) {
    var elem = document.createElement('img');
    elem.className = 'mover';
    elem.src = "images/pizza.png";
    elem.style.height = "100px";
    elem.style.width = "73.333px";
    elem.basicLeft = (i % cols) * s;
    elem.style.top = (Math.floor(i / cols) * s) + 'px';
    document.querySelector("#movingPizzas1").appendChild(elem);
  }
  updatePositions();
});
```

#### refactored

* 4 rows of 8 columns of pizzas are visible - use 32 instead of 200
* get container for the moving pizzas just once using getElementById
* whole integer width makes it easier for browser to render image

```javascript
document.addEventListener('DOMContentLoaded', function() {
  var i, elem, cols = 8, s = 256, numPizzas = 32,
      moverContainer =  document.getElementById("movingPizzas1");
  for (i = 0; i < numPizzas; i++) {
    elem = document.createElement('img');
    elem.className = 'mover';
    elem.src = "images/pizza.png";
    elem.style.height = "100px";
    elem.style.width = "73px";
    elem.basicLeft = (i % cols) * s;
    elem.style.top = (Math.floor(i / cols) * s) + 'px';
    moverContainer.appendChild(elem);
  }
  updatePositions();
});
```
<p>&nbsp;</p>
### resizePizzas
resizePizzas(size) is called when the slider in the "Our Pizzas" section of the website moves.

#### original
* note use of querySelector and querySelectorAll
* querySelectorAll invoked multiple times every iteration of loop in changePizzaSizes

```javascript
var resizePizzas = function(size) {
  ...
  function changeSliderLabel(size) {
    switch(size) {
      case "1":
        document.querySelector("#pizzaSize").innerHTML = "Small";
        return;
      case "2":
        document.querySelector("#pizzaSize").innerHTML = "Medium";
        return;
      case "3":
        document.querySelector("#pizzaSize").innerHTML = "Large";
        return;
      default:
        console.log("bug in changeSliderLabel");
    }
  }
  ...
  function determineDx (elem, size) {
    var oldwidth = elem.offsetWidth;
    var windowwidth = document.querySelector("#randomPizzas").offsetWidth;
    var oldsize = oldwidth / windowwidth;
    ...
    var newsize = sizeSwitcher(size);
    var dx = (newsize - oldsize) * windowwidth;

    return dx;
  }

  function changePizzaSizes(size) {
    for (var i = 0; i < document.querySelectorAll(".randomPizzaContainer").length; i++) {
      var dx = determineDx(document.querySelectorAll(".randomPizzaContainer")[i], size);
      var newwidth = (document.querySelectorAll(".randomPizzaContainer")[i].offsetWidth + dx) + 'px';
      document.querySelectorAll(".randomPizzaContainer")[i].style.width = newwidth;
    }
  }
  ...
};
```

#### refactored
* in helper function changeSliderLabel, now uses getElementById to get the slider label element
* in determineDx, minimized use of variables, replaced querySelector with getElementById
* in changePizzaSizes obtained an array of elements before looping named pizzaContainers using getElementsByClassName
* calculate new width just once
* access each element by looping through pizzaContainers to assign the new width

```javascript
var resizePizzas = function(size) {
  ...
  function changeSliderLabel(size) {
    var sliderLabel = document.getElementById("pizzaSize");
    switch(size) {
      case "1":
        sliderLabel.innerHTML = "Small";
        return;
      case "2":
        sliderLabel.innerHTML = "Medium";
        return;
      case "3":
        sliderLabel.innerHTML = "Large";
        return;
      default:
        console.log("bug in changeSliderLabel");
    }
  }
  ...
  function determineDx (elem, size) {
     var windowwidth = document.getElementById("randomPizzas").offsetWidth,
        oldsize = elem.offsetWidth / windowwidth;
    ...
    var newsize = sizeSwitcher(size),
        dx = (newsize - oldsize) * windowwidth;
    return dx;
  }

  function changePizzaSizes(size) {
    var i = 0,
        pizzaContainers =  document.getElementsByClassName("randomPizzaContainer"),
        newwidth = (pizzaContainers[i].offsetWidth + determineDx(pizzaContainers[i], size)) + 'px';
    for (i; i < pizzaContainers.length; i++) {
      pizzaContainers[i].style.width = newwidth;
    }
  }
  ...
};
```

<p>&nbsp;</p>
# Project Description

## Website Performance Optimization portfolio project

Your challenge, if you wish to accept it (and we sure hope you will), is to optimize this online portfolio for speed! In particular, optimize the critical rendering path and make this page render as quickly as possible by applying the techniques you've picked up in the [Critical Rendering Path course](https://www.udacity.com/course/ud884).

To get started, check out the repository, inspect the code,

### Getting started

####Part 1: Optimize PageSpeed Insights score for index.html

Some useful tips to help you get started:

1. Check out the repository
1. To inspect the site on your phone, you can run a local server

  ```bash
  $> cd /path/to/your-project-folder
  $> python -m SimpleHTTPServer 8080
  ```

1. Open a browser and visit localhost:8080
1. Download and install [ngrok](https://ngrok.com/) to make your local server accessible remotely.

  ``` bash
  $> cd /path/to/your-project-folder
  $> ngrok 8080
  ```

1. Copy the public URL ngrok gives you and try running it through PageSpeed Insights! Optional: [More on integrating ngrok, Grunt and PageSpeed.](http://www.jamescryer.com/2014/06/12/grunt-pagespeed-and-ngrok-locally-testing/)

Profile, optimize, measure... and then lather, rinse, and repeat. Good luck!

####Part 2: Optimize Frames per Second in pizza.html

To optimize views/pizza.html, you will need to modify views/js/main.js until your frames per second rate is 60 fps or higher. You will find instructive comments in main.js.

You might find the FPS Counter/HUD Display useful in Chrome developer tools described here: [Chrome Dev Tools tips-and-tricks](https://developer.chrome.com/devtools/docs/tips-and-tricks).

### Optimization Tips and Tricks
* [Optimizing Performance](https://developers.google.com/web/fundamentals/performance/ "web performance")
* [Analyzing the Critical Rendering Path](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/analyzing-crp.html "analyzing crp")
* [Optimizing the Critical Rendering Path](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/optimizing-critical-rendering-path.html "optimize the crp!")
* [Avoiding Rendering Blocking CSS](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-blocking-css.html "render blocking css")
* [Optimizing JavaScript](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/adding-interactivity-with-javascript.html "javascript")
* [Measuring with Navigation Timing](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/measure-crp.html "nav timing api"). We didn't cover the Navigation Timing API in the first two lessons but it's an incredibly useful tool for automated page profiling. I highly recommend reading.
* <a href="https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/eliminate-downloads.html">The fewer the downloads, the better</a>
* <a href="https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/optimize-encoding-and-transfer.html">Reduce the size of text</a>
* <a href="https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/image-optimization.html">Optimize images</a>
* <a href="https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching.html">HTTP caching</a>

### Customization with Bootstrap
The portfolio was built on Twitter's <a href="http://getbootstrap.com/">Bootstrap</a> framework. All custom styles are in `dist/css/portfolio.css` in the portfolio repo.

* <a href="http://getbootstrap.com/css/">Bootstrap's CSS Classes</a>
* <a href="http://getbootstrap.com/components/">Bootstrap's Components</a>

### Sample Portfolios

Feeling uninspired by the portfolio? Here's a list of cool portfolios I found after a few minutes of Googling.

* <a href="http://www.reddit.com/r/webdev/comments/280qkr/would_anybody_like_to_post_their_portfolio_site/">A great discussion about portfolios on reddit</a>
* <a href="http://ianlunn.co.uk/">http://ianlunn.co.uk/</a>
* <a href="http://www.adhamdannaway.com/portfolio">http://www.adhamdannaway.com/portfolio</a>
* <a href="http://www.timboelaars.nl/">http://www.timboelaars.nl/</a>
* <a href="http://futoryan.prosite.com/">http://futoryan.prosite.com/</a>
* <a href="http://playonpixels.prosite.com/21591/projects">http://playonpixels.prosite.com/21591/projects</a>
* <a href="http://colintrenter.prosite.com/">http://colintrenter.prosite.com/</a>
* <a href="http://calebmorris.prosite.com/">http://calebmorris.prosite.com/</a>
* <a href="http://www.cullywright.com/">http://www.cullywright.com/</a>
* <a href="http://yourjustlucky.com/">http://yourjustlucky.com/</a>
* <a href="http://nicoledominguez.com/portfolio/">http://nicoledominguez.com/portfolio/</a>
* <a href="http://www.roxannecook.com/">http://www.roxannecook.com/</a>
* <a href="http://www.84colors.com/portfolio.html">http://www.84colors.com/portfolio.html</a>
