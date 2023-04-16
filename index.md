<style>
#greet-text{
    text-align: center;
}

.play-container{
    text-align: center;
}

.greet-container{
    text-align: center;
}

#play-button{
    display: block;
    margin: auto;
}

#close-game{
    display: none;
    margin: auto;
    background-color: rgb(223, 109, 109);
}

#game-container{
    position: relative !important;
    --bg-color: #90fff0 !important;
    --bg-color-light: #ff00c8; 
    background: linear-gradient(-45deg, var(--bg-color), var(--bg-color-light), var(--bg-color), var(--bg-color-light));
    background-size: 1200% 1200% !important;
    animation: gradient 7s ease infinite !important;
    text-align: center;
    width: 480px;
    height: 480px;
    border-radius: 20px;
    margin: auto;
    display: none;
}
#timer-container{
  display: none;
}
#bar{
  margin-top: 40px;
  font-family: 'Fira Mono', monospace !important;
  border-collapse: collapse;
  width: 100%;
  border-radius: 0.75em;
  box-shadow: 0 0 0.5em #175178;
  padding: 10px 10px;
}
.bar-1 {
  margin-left: 100px;
  width: 150px;
  height: 80px;
  border-radius: 40px;
  background-color: #90fff0;
  color: #000000;
  border: #ffffff;
}
#game {
  justify-self: center;
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: repeat(4, 1fr);
  width: 450px;
  height: 450px;  
}
.flip-card {
  background-color: transparent;
  width: 100px;
  height: 100px;
  perspective: 1000px;
  margin-top: 0px;
  position: relative;
  text-align: center;
  transition: transform 0.6s;
  transform-style: preserve-3d;
}
.flip-card div {
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
}
.flip-card .flip-card-front {
  width: 100px;
  height: 100px;
  background-color: #ff0000;
  border-radius: 20px;
}
.flip-card .flip-card-back {
  width: 100px;
  height: 100px;
  background-color: #f1dd00;
  transform: rotateY(180deg);
  border-radius: 20px;
}
.flip-card.flipped {
  transform: rotateY(180deg);
}
#canvas{
  position: relative;
  display: block;
  padding-top: 22px;
  margin: 21px
}
img {
  width: 50px;
  height: 50px;
}    
.frozen {
  pointer-events: none;
  opacity: 1;
}
.frozen-text {
  display: none;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 36px;
  font-weight: bold;
  font-family: "Lucida Console", "Monaco", monospace;
  color: #ff9304;
  text-align: center;
}
.frozen .frozen-text {
  display: block;
}
#popup-image {
  position: absolute;
  display: none;
  top: 45%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 400px;
  height: 300px;
}
.timer {
  width: 200px;
  border: solid 1px #ffffff;
  border-radius: 6px;
}
.timer .inner {
  height: 15px;
  animation: timer-start;
  animation-duration: 40s;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
  animation-play-state: paused;
  animation-timing-function: linear;
  border-radius: 6px;
}
@keyframes timer-start {
  0% {
    width: 100%;
    background: #1aff00;
  }
  100% {
    width: 0%;
    background: #F00;
  }
}
#highscores{
  font-family: 'Fira Mono', monospace !important;
  border-collapse: collapse;
  width: 100%;
  border-radius: 0.75em;
  box-shadow: 0 0 0.5em #175178;
  padding: 10px 10px;
  display: table;
}         
</style>

<div class="greet-container">
  <h2>Welcome To Code-Crunch!</h2>
  <blockquote id = "greet-text">First timer? Don't worry! You'll get the hang of it. Click the Play button, and try to match as many cards as possible in under 25 seconds. Each matched pair is 1 point. Your score is displayed on the blue box. Be careful, the timer isn't really fair!</blockquote>
</div>
<br>
<div class="play-container">
  <button type="button" id="play-button">Play</button>
  <button type="button" id="close-game">Close</button>
  <div id="timer-container">
    <table id="bar">
      <tr>
        <th><button type="button" class="bar-1"><span id="match-count">Score</span></button></th>
        <th><div id='timer'></div></th>
      </tr>
    </table>
  </div>
  <br>
  <div id="game-container">
      <section id="canvas" class="hidden">
      <div id="game">
      </div>
      <img id="popup-image" src="{{site.baseurl}}/images/m.png">
    </section>
  </div>
  <br>
</div>
<script>

// for loop: creating 16 cards each with a unique id
var gameDiv = document.getElementById('game');
for (let i = 1; i <= 16; i++) {
  var flipCardDiv = document.createElement('div');
  flipCardDiv.id = 'flip-card-' + i;
  flipCardDiv.classList.add('flip-card');
  flipCardDiv.innerHTML = `
    <div class="flip-card-front"></div>
    <div class="flip-card-back"></div>
  `;
  gameDiv.appendChild(flipCardDiv);
}

// onevent click listeners for play button and close button (closing and starting the game restarts the game including score and timer)
var playbutton = document.getElementById("play-button");
playbutton.onclick = function() {
  document.getElementById("game-container").style.display = "block";
  document.getElementById("timer-container").style.display = "block";
  document.getElementById("play-button").style.display = "none";
  document.getElementById("close-game").style.display = "block";
}

// by using the randSide function it is assured that the items in the list are assigned to each card randomly with no repeats
var sides = document.querySelectorAll(".flip-card .flip-card-back");
function assignSides(sides) {
  // each image is listed twice since there are 16 cards
  var url = "{{site.baseurl}}/images/";   // for max efficiency
  // CITATION: all of the following images are from the website Font Awesome
  var possibleSides = [
                        url + "bug.png",
                        url + "bug.png", 
                        url + "c.png",
                        url + "c.png",  
                        url + "ch.png", 
                        url + "ch.png", 
                        url + "d.png", 
                        url + "d.png",
                        url + "e.png", 
                        url + "e.png",
                        url + "g.png",
                        url + "g.png",  
                        url + "s.png", 
                        url + "s.png", 
                        url + "sc.png",   
                        url + "sc.png"
                      ];

  var sidesPostReplay = possibleSides.slice();
  
  for (var i = 0; i < 16; i++) {
    var randomIndex = Math.floor(Math.random() * possibleSides.length);
    var side = possibleSides[randomIndex];
    // each image is randomly selected and returned while also being deleted from the list 
    possibleSides.splice(randomIndex, 1);
    sides[i].innerHTML = '<img src="' + side + '">';
  }

  if (typeof assignSides.initialized == 'undefined') {
    assignSides.initialized = true;
    // since the reset function relies on possibleSides being repeated, it is more efficient to make a copy of the list instead
    possibleSides = sidesPostReplay;  // sidesPostReplay = copy of possibleSides
  }
}

function unflipped(card) {
  // usage of NOT operator: items with the flipped class result in False, which is inverted by the NOT operator, resulting in True
  return !card.classList.contains("flipped");
}

// checking if the html property (image) of two selected cards match
var flippedCards = [];
function matched(flippedCards) {
  return (flippedCards[0].innerHTML == flippedCards[1].innerHTML);
}
// if cards don't match, they will stay flipped for 700ms and then the "flipped" class is removed from both cards
var hold = 700;
function hideCards(flippedCards) {
  setTimeout(function() {
    flippedCards[0].classList.remove("flipped");
    flippedCards[1].classList.remove("flipped");
  }, 
  hold);
}

// Once the game is restarted, images are assigned randomly and all cards will be face-down
var flipCardElements = document.querySelectorAll(".flip-card");
function reset(sides, flipCardElements) {
  assignSides(sides);
  flipCardElements.forEach(function(card) {
    card.classList.remove("flipped");
  });
}

var matchCountDisplay = document.querySelector("#match-count");
var matchCounter = 0;     // 0 initial score
// assignSides function called
assignSides(sides);
var canvas = document.querySelector("#canvas");   
canvas.addEventListener("click", function(event) {     // interactive area with click event listener
  if (event.target.classList.contains("flip-card-front")) {   
    // closest element with the flip-card class
    var card = event.target.closest(".flip-card");
    if (unflipped(card)) {
      // flipped cards assigned the class "flipped" and added to the flippedCards array
      card.classList.add("flipped");
      flippedCards.push(card);
    }
    // if two cards have been flipped
    if (flippedCards.length == 2) {
      if (matched(flippedCards)) {
        // update score
        matchCountDisplay.textContent = ++matchCounter;
      } else {
        // cards hidden if don't match
        hideCards(flippedCards);
      }
      // array emptied
      flippedCards = [];
    }
  }
});

// timer function with parameters
function initTimer(id, duration, callback) {
  var t = document.getElementById(id);
  t.className = 'timer';
  var tInner = document.createElement('div');
  tInner.className = 'inner';
  tInner.style.animationDuration = duration;
  if (typeof(callback) === 'function')
    tInner.addEventListener('animationend', callback);
    t.appendChild(tInner);
    // start animation
    tInner.style.animationPlayState = 'running';
}

addEventListener('load', () => initTimer('timer', '25s', () => {    // duration set to 25s
  var scrnfreeze = document.getElementById("game-container");     
  scrnfreeze.classList.add("frozen");                             // game is frozen once timer reaches zero
  document.getElementById("popup-image").style.display = "block";        // time's up message pop up
}));

// reseting: close button acts as a replay button
var closegame = document.getElementById("close-game");
closegame.onclick = function() {
  var scrnfreeze = document.getElementById("game-container");
  reset(sides, flipCardElements);   // reset function called
  matchCounter = 0;   // score reset
  matchCountDisplay.textContent = matchCounter;    // score display reset
  scrnfreeze.classList.remove("frozen");            // frozen effect removed
  document.getElementById("popup-image").style.display = "none";    // pop up removed
  document.getElementById("game-container").style.display = "none";
  document.getElementById("timer-container").style.display = "none";
  document.getElementById("play-button").style.display = "block";
  document.getElementById("close-game").style.display = "none";
}
</script>