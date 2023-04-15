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
  margin-left: 10px;
  width: 150px;
  height: 80px;
  border-radius: 40px;
  background-color: #90fff0;
  color: #000000;
  border: #ffffff;
}
.bar-2{
  width: 150px;
  text-align: center;
  height: 120px;
  margin-left: 20px;
  border-radius: 40px;
  background-color: #ff00c8;
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
.progressbar {
  width: 200px;
  border: solid 1px #ffffff;
  border-radius: 6px;
}
.progressbar .inner {
  height: 15px;
  animation: progressbar-countdown;
  animation-duration: 40s;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
  animation-play-state: paused;
  animation-timing-function: linear;
  border-radius: 6px;
}
@keyframes progressbar-countdown {
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
  <blockquote id = "greet-text">First timer? Don't worry! You'll get the hang of it!</blockquote>
</div>
<br>
<div class="play-container">
  <button type="button" id="play-button">Play</button>
  <button type="button" id="close-game">Close</button>
  <div id="timer-container">
    <div id = "timer">
      <table id="bar">
        <tr>
          <th><button type="button" class="bar-1"><span id="match-count">Score</span></button></th>
          <th><button type="button" class="bar-2"></button></th>
          <th><div id='progressbar'></div></th>
        </tr>
      </table>
    </div>
  </div>
  <br>
  <div id="game-container">
      <section id="canvas" class="hidden">
      <div id='progressbar'></div>
      <div id="game">
      </div>
      <img id="popup-image" src="{{site.baseurl}}/images/m.png">
    </section>
  </div>
  <br>
</div>
<script>
const gameDiv = document.getElementById('game');

for (let i = 1; i <= 16; i++) {
  const flipCardDiv = document.createElement('div');
  flipCardDiv.id = 'flip-card-' + i;
  flipCardDiv.classList.add('flip-card');
  flipCardDiv.innerHTML = `
    <div class="flip-card-front"></div>
    <div class="flip-card-back"></div>
  `;
  gameDiv.appendChild(flipCardDiv);
}

var playbutton = document.getElementById("play-button");
var closegame = document.getElementById("close-game");
var playButton = document.querySelector("#play-button");
var canvas = document.querySelector("#canvas");
var flipCardElements = document.querySelectorAll(".flip-card");
var cardSides = document.querySelectorAll(".flip-card .flip-card-back");
var replay = document.querySelector("#close-game");
var matchCountDisplay = document.querySelector("#match-count");
var matchCounter = 0;
var totalCards = flipCardElements.length;
var flippedCards = [];
var matchedCards = [];
var locked = false;
var flipTimeout = 700;
var url = "{{site.baseurl}}/images/";
var possibleCardSides = [url + "bug.png", 
                        url + "c.png", 
                        url + "ch.png", 
                        url + "d.png", 
                        url + "e.png", 
                        url + "g.png", 
                        url + "s.png", 
                        url + "sc.png", 
                        url + "bug.png", 
                        url + "c.png", 
                        url + "ch.png", 
                        url + "d.png", 
                        url + "e.png", 
                        url + "g.png", 
                        url + "s.png", 
                        url + "sc.png"];

playbutton.onclick = function() {
  document.getElementById("game-container").style.display = "block";
  document.getElementById("timer-container").style.display = "block";
  document.getElementById("play-button").style.display = "none";
  document.getElementById("close-game").style.display = "block";
}
closegame.onclick = function() {
  document.getElementById("game-container").style.display = "none";
  document.getElementById("timer-container").style.display = "none";
  document.getElementById("play-button").style.display = "block";
  document.getElementById("close-game").style.display = "none";
}
                        
function getRandomIndex(length) {
  return Math.floor(Math.random() * length);
}

function getRandomSide(randomIndex) {
  var side;
  randomIndex = getRandomIndex(possibleCardSides.length);
  side = possibleCardSides[randomIndex];
  possibleCardSides.splice(randomIndex, 1);
  return side;
}

function assignCardSides(cardSides) {
  for (var i = 0; i < 16; i++) {
    cardSides[i].innerHTML = '<img src="' + getRandomSide() + '">';
  }
  possibleCardSides = [url + "bug.png", 
                      url + "c.png", 
                      url + "ch.png", 
                      url + "d.png", 
                      url + "e.png", 
                      url + "g.png", 
                      url + "s.png", 
                      url + "sc.png", 
                      url + "bug.png", 
                      url + "c.png", 
                      url + "ch.png", 
                      url + "d.png", 
                      url + "e.png", 
                      url + "g.png", 
                      url + "s.png", 
                      url + "sc.png"];
}

function unFlipped(card) {
  return !card.classList.contains("flipped");
}

function areMatching(flippedCards) {
  return (flippedCards[0].innerHTML === flippedCards[1].innerHTML);
}

function hideCards(flippedCards) {
  setTimeout(function() {
    flippedCards[0].classList.remove("flipped");
    flippedCards[1].classList.remove("flipped");
    locked = false;
  }, flipTimeout);
}

function reset(cardSides, flipCardElements) {
  assignCardSides(cardSides);
  matchedCards = [];
  flipCardElements.forEach(function(card) {
    card.classList.remove("flipped");
  });
}

assignCardSides(cardSides);

playButton.addEventListener("click", function() {
  canvas.classList.remove("hidden");
});

canvas.addEventListener("click", function(event) {
  if (event.target.classList.contains("flip-card-front") && !locked) {
    var card = event.target.closest(".flip-card");
    if (unFlipped(card)) {
      card.classList.add("flipped");
      flippedCards.push(card);
    }
    if (flippedCards.length === 2) {
      if (areMatching(flippedCards)) {
        matchCountDisplay.textContent = ++matchCounter;
        matchedCards.push(...flippedCards);
      } else {
        locked = true;
        hideCards(flippedCards);
      }
      flippedCards = [];
    }
  }
});

function createProgressbar(id, duration, callback) {
  const pb = document.getElementById(id);
  pb.className = 'progressbar';
  const pbi = document.createElement('div');
  pbi.className = 'inner';
  pbi.style.animationDuration = duration;
  if (typeof(callback) === 'function')
    pbi.addEventListener('animationend', callback);
  pb.appendChild(pbi);
  pbi.style.animationPlayState = 'running';
}

addEventListener('load', () => createProgressbar('progressbar', '15s', () => {
  const c = document.getElementById("game-container");
  c.classList.add("frozen");
  document.getElementById("popup-image").style.display = "block";
}));

replay.addEventListener("click", function() {
  const c = document.getElementById("game-container");
  reset(cardSides, flipCardElements);
  matchCounter = 0;
  matchCountDisplay.textContent = matchCounter;
  c.classList.remove("frozen");
  document.getElementById("popup-image").style.display = "none";
});
</script>