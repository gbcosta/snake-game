### Snake Game

Just a Snake Game made width JS and HTML ;)

<canvas></canvas>

<script>
  const canvasConfig = {
  width: 800,
  height: 600,
  backgroundColor: "#000",
};

const tileMapSize = 20;

const tileMapConfig = {
  maxX: canvasConfig.width / tileMapSize,
  maxY: canvasConfig.height / tileMapSize,
};

const snakeConfig = {
  headColor: "white",
  tailColor: "white",
};

const keys = {
  up: "w",
  down: "s",
  left: "a",
  right: "d",
};

const getContext = () => {
  const canvas = document.querySelector("canvas");
  const context = canvas.getContext("2d");
  const { width, height, backgroundColor } = canvasConfig;

  canvas.width = width;
  canvas.height = height;
  canvas.fillStyle = backgroundColor;

  context.fillRect(0, 0, width, height);

  return context;
};

const context = getContext();

const randomPosition = (max, min) => {
  return Math.floor(Math.random() * (max - min)) + min;
};

const last = (array) => {
  return array[array.length - 1];
};

class Snake {
  constructor() {
    this.head = new TailRect({ x: 10, y: 10 });
    this.head.previousPosition = { x: 9, y: 10 };
    this.tail = [];
    this.KeyNextMove = null;
    this.alive = true;
  }

  addTail() {
    let newTail = null;
    if (this.tail.length == 0) {
      newTail = new TailRect(this.head.previousPosition);
      newTail.nextTailRect = this.head;
      this.tail.push(newTail);
      return;
    }

    newTail = new TailRect(last(this.tail).previousPosition);
    newTail.nextTailRect = last(this.tail);
    this.tail.push(newTail);
  }

  drawn() {
    context.fillStyle = snakeConfig.headColor;

    context.fillRect(
      this.head.position.x * 20,
      this.head.position.y * 20,
      tileMapSize,
      tileMapSize
    );

    context.fillStyle = snakeConfig.tailColor;
    this.tail.map((tailRect) => {
      context.fillRect(
        tailRect.position.x * 20,
        tailRect.position.y * 20,
        tileMapSize,
        tileMapSize
      );
    });
  }

  setKeyNextMove(keyNextMove) {
    this.KeyNextMove = keyNextMove;
  }

  moveSnake(moveDirection) {
    this.setPreviousPosition();
    moveDirection(this.head);
    this.wallTeleport();
    this.moveTail();
  }

  move() {
    switch (this.KeyNextMove) {
      case keys.up:
        this.moveSnake(this.moveUp);
        break;
      case keys.down:
        this.moveSnake(this.moveDown);
        break;
      case keys.left:
        this.moveSnake(this.moveLeft);
        break;
      case keys.right:
        this.moveSnake(this.moveRight);
        break;

      default:
        break;
    }
  }

  setPreviousPosition() {
    this.head.previousPosition.x = this.head.position.x;
    this.head.previousPosition.y = this.head.position.y;
  }

  moveUp(head) {
    head.position.y -= 1;
  }

  moveDown(head) {
    head.position.y += 1;
  }

  moveLeft(head) {
    head.position.x -= 1;
  }

  moveRight(head) {
    head.position.x += 1;
  }

  wallTeleport() {
    if (this.head.position.x > tileMapConfig.maxX) this.head.position.x = 0;
    else if (this.head.position.x < 0)
      this.head.position.x = tileMapConfig.maxX;
    else if (this.head.position.y > tileMapConfig.maxY)
      this.head.position.y = 0;
    else if (this.head.position.y < 0)
      this.head.position.y = tileMapConfig.maxY;
  }

  moveTail() {
    this.tail.map((tailRect) => {
      tailRect.move();
    });
  }
}

class TailRect {
  constructor(position) {
    this.position = { x: -1, y: -1 };
    this.position.x = position.x;
    this.position.y = position.y;
    this.previousPosition = { x: null, y: null };
    this.nextTailRect = null;
  }

  move() {
    this.previousPosition.x = this.position.x;
    this.previousPosition.y = this.position.y;

    this.position.x = this.nextTailRect.previousPosition.x;
    this.position.y = this.nextTailRect.previousPosition.y;
  }
}

class Apple {
  constructor() {
    this.position = { x: -1, y: -1 };
    this.eaten = true;
  }

  spawnApple() {
    if (this.eaten) {
      this.position = {
        x: randomPosition(tileMapConfig.maxX, 0),
        y: randomPosition(tileMapConfig.maxY, 0),
      };
      this.eaten = false;
    }
  }

  snakeEaten() {
    this.eaten = true;
  }

  drawApple() {
    context.fillStyle = "red";
    context.fillRect(
      this.position.x * 20,
      this.position.y * 20,
      tileMapSize,
      tileMapSize
    );
  }
}

class Collisions {
  constructor() {}

  CollisionHeadTail() {
    snake.tail.map((tailRect) => {
      if (
        snake.head.position.x === tailRect.position.x &&
        snake.head.position.y === tailRect.position.y
      ) {
        snake.alive = false;
      }
    });
  }

  collisionHeadApple() {
    if (
      snake.head.position.x == apple.position.x &&
      snake.head.position.y == apple.position.y
    ) {
      apple.eaten = true;
      snake.addTail();
    }
  }
}

var snake = new Snake();
var apple = new Apple();
var collisions = new Collisions();

const game = () => {
  setInterval(loop, 1000 / 20);
};

const loop = () => {
  update();
  draw();
};

const draw = () => {
  context.fillStyle = canvasConfig.backgroundColor;
  context.fillRect(0, 0, canvasConfig.width, canvasConfig.height);

  snake.drawn();
  apple.drawApple();
};

const update = () => {
  collisions.collisionHeadApple();
  collisions.CollisionHeadTail();
  apple.spawnApple();
  snake.move();
  if (!snake.alive) {
    snake = new Snake();
    apple = new Apple();
  }
};

document.addEventListener(
  "keydown",
  (event) => {
    var key = event.key;
    snake.setKeyNextMove(key);
  },
  false
);

game();

</script>
