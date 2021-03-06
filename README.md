# :beetle: Your Inner Insect :beetle:

Your Inner Insect was born as a reflection on the theme of metamorphosis through the eyes of Franz Kafka. In his book _The Metamorphosis_ (1916) the author links complex psychological conditions to an insect (a cockroach).

The website consists of a guided experience where users can explore their social insecurities by answering some questions through different types of interactions. Each question is associated with a different insect’s body part.
By answering each question the user will be assigned different types of insect’s body parts according to the answer selected . At the end of the test, all the body parts combined will give shape to a new kind of insect, your inner insect.
All the inner insects created by previous users can be viewed in the Archive, where they can be sorted by various parameters.

## How it works

### Assets

Because the focus of the website is the creation of new types of insects by combining different body parts, it was fundamental to gather pieces that could all fit together well. Starting from a great resource, [Beetles of Russia and Western Europe](https://www.zin.ru/ANIMALIA/Coleoptera/rus/jactab0.htm) (1915) by G. G. Jacobson, we selected a bunch of insect illustrations and then separated each one into pieces. To be sure that they would always fit well together, no matter the combination, we manually tweaked some junction points.

![image](/assets/pieces.png)

### The test

The test section is achieved with 4 different p5 instances: one handles the whole test and the insect construction, one for the mouse shake question, one for the webcam question and the last one for the audio question.

```
let s1 = new p5(sketch_1);
let s2 = new p5(sketch_Mouse);
let s3 = new p5(sketch_Webcam);
let s4 = new p5(sketch_Audio);
```

Each question is associated to an insect’s body part:

- 1st question - Chest
- 2nd question - Abdomen
- 3rd question - Legs
- 4th question - Head
- 5th question - Antennas

#### Mouse shake question

To answer the third question the user has to shake the mouse until a cursor reach the desired point, inside a range with three sections, one for each answer.

The cursor moves based on the average speed of the mouse by using this code:

```
let difX = p.abs(p.mouseX - p.pmouseX);
let difY = p.abs(p.mouseY - p.pmouseY);
let vel = difX + difY;

if (click == 0) {
  contenitore.push(vel);
  somma = somma + contenitore[indice];
  indice++;
  media = p.floor(somma / contenitore.length);
}
```

#### Webcam question

In the fourth question, the user answers using the brightness of the webcam image. The latter is rendered as a grid of little black squares whose size is relative to the brightness of that pixel area (the darker it is, the bigger it gets).

```
for (let y = 0; y < video.height; y += gridSize) {
  for (let x = 0; x < video.width; x += gridSize) {
    let index = (y _ video.width + x) _ 4;

    let r = video.pixels[index + 0];
    let g = video.pixels[index + 1];
    let b = video.pixels[index + 2];

    bright = (r + g + b) / 3;
    let size = p.map(bright, 0, 255, gridSize, 1);

    p.fill(0);
    p.noStroke();
    p.rect(x, y, size);
  }
}
```

#### Audio question

In the fifth question, the user answers through the use of the microphone. The louder the sound, the bigger the circle diameter `d` will be, allowing to choose the desired answer.

The sound management is achieved with the use of the p5.sound library.

```
if (mic) {
  const micLevel = mic.getLevel();
  d = p.map(micLevel, 0, 1, 1, 1000);

  p.push();
  p.stroke(173, 149, 127);
  p.noFill();
  p.circle(200, 200, d * 3);
  p.pop();
}

function startAudio() {
  p.userStartAudio();
  mic = new p5.AudioIn();
  mic.start();
  p.select("#stop-audio").removeClass("hide");
}
```

#### Insect creation

Each time the user answers a question, a certain body part is assigned, based on the answer given. These body parts are selected within 3 sub-groups: big, medium and small.

For example, if the user chooses the first answer for the first question he will be assigned a small chest, randomly picked from the chest-S array, which contains all the small chests. If he chooses the second answer he’ll be given a random medium chest and if he chooses the third, a random big chest.

```
setChestS.mousePressed(chest_S);
setChestM.mousePressed(chest_M);
setChestB.mousePressed(chest_B);

function chest_S() {
  let chestS_List = ["S-1", "S-2", "S-3", "S-4", "S-5", "S-6", "S-7", "S-8"];
  chestX = p.random(chestS_List);
}
function chest_M() {
  let chestM_List = ["M-1", "M-2", "M-3", "M-4", "M-5", "M-6", "M-7", "M-8", "M-9", "M-10"];
  chestX = p.random(chestM_List);
}
function chest_B() {
  let chestB_List = ["B-1", "B-2", "B-3", "B-4", "B-5", "B-6", "B-7", "B-8", "B-9", "B-10", "B-11", "B-12", "B-13"];
  chestX = p.random(chestB_List);
}
```

In the end all the body parts assigned that will form the new insect are stored inside an object and sent to the Firebase database.

```
const newUser = {
  name: nameX,
  chest: chestX,
  butt: buttX,
  leg: legX,
  head: headX,
  ant: antX,
  date: today,
  time: time,
};
addUser(newUser);
```

### The archive

To better manage the database, the users data on Firebase are stored inside an array called `InsectsArray`.

```
let giveMeData = ref(myDatabase, "insects");

onValue(giveMeData, function (snapshot) {
  const Insects = snapshot.val();
  InsectsArray = Object.values(Insects);
});
```

To display all the insects generated by the users in the archive section, a for loop is executed, displaying all the images corresponding to each body part, the name and the date.

```
for (let i = InsectsArray.length - 1; i >= 0; i--) {
  let box = createDiv().id("box-" + i).addClass("insect-box").parent("archive-container");

  noLoop();

  let Chest = InsectsArray[i].chest;
  let Butt = InsectsArray[i].butt;
  let Leg = InsectsArray[i].leg;
  let Head = InsectsArray[i].head;
  let Ant = InsectsArray[i].ant;

  let partsBox = createDiv().parent(box).addClass("partsBox");

  let legBox = createImg("assets/partsNew/leg/leg-" + Leg + ".png")
    .parent(partsBox)
    .addClass("part-img");
  let antBox = createImg("assets/partsNew/ant/ant-" + Ant + ".png")
    .parent(partsBox)
    .addClass("part-img");
  let headBox = createImg("assets/partsNew/head/head-" + Head + ".png")
    .parent(partsBox)
    .addClass("part-img");
  let buttBox = createImg("assets/partsNew/butt/butt-" + Butt + ".png")
    .parent(partsBox)
    .addClass("part-img");
  let chestBox = createImg("assets/partsNew/chest/chest-" + Chest + ".png")
    .parent(partsBox)
    .addClass("part-img");

  let name = createElement("p", InsectsArray[i].name).parent(box).addClass("name");
  let date = createElement("p", InsectsArray[i].date).parent(box).addClass("date");
}
```

## :busts_in_silhouette: Team

- Anel Alzhanova
- Lorenzo Bernini
- Chiara Poma
- Ludovica Rossi
- Enzo Taboada Fung

## :mortar_board: Faculty

- Andrea Benedetti
- Tommaso Elli
- Michele Mauri
