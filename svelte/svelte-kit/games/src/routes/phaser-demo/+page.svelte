<script>
  import Phaser from "phaser"
  var config = {
    type: Phaser.AUTO,
    width: 800,
    height: 600,
    physics: {
      default: "arcade",
      arcade: {
        gravity: { y: 300 },
        debug: false,
      },
    },
    scene: {
      preload: preload,
      create: create,
      update: update,
    },
  }
  /** @type {Phaser.Types.Physics.Arcade.SpriteWithDynamicBody}*/
  var player
  /** @type {Phaser.Physics.Arcade.Group} */
  var stars
  /** @type {Phaser.Physics.Arcade.Group} */
  var bombs
  /** @type {Phaser.Physics.Arcade.StaticGroup} */
  var platforms
  /** @type {Phaser.Types.Input.Keyboard.CursorKeys} */
  var cursors
  var score = 0
  var gameOver = false
  /** @type {Phaser.GameObjects.Text} */
  var scoreText
  var game = new Phaser.Game(config)
  /**
   * @this {Phaser.Scene}
   */
  function preload() {
    this.load.image("sky", "assets/sky.png")
    this.load.image("ground", "assets/platform.png")
    this.load.image("star", "assets/star.png")
    this.load.image("bomb", "assets/bomb.png")
    this.load.spritesheet("dude", "assets/dude.png", {
      frameWidth: 32,
      frameHeight: 48,
    })
  }
  /**
   * @this {Phaser.Scene}
   */
  function create() {
    //  A simple background for our game
    this.add.image(400, 300, "sky")
    //  The platforms group contains the ground and the 2 ledges we can jump on
    platforms = this.physics.add.staticGroup()
    //  Here we create the ground.
    //  Scale it to fit the width of the game (the original sprite is 400x32 in size)
    platforms.create(400, 568, "ground").setScale(2).refreshBody()
    //  Now let's create some ledges
    platforms.create(600, 400, "ground")
    platforms.create(50, 250, "ground")
    platforms.create(750, 220, "ground")
    // The player and its settings
    player = this.physics.add.sprite(100, 450, "dude")
    //  Player physics properties. Give the little guy a slight bounce.
    player.setBounce(0.2)
    player.setCollideWorldBounds(true)
    //  Our player animations, turning, walking left and walking right.
    this.anims.create({
      key: "left",
      frames: this.anims.generateFrameNumbers("dude", { start: 0, end: 3 }),
      frameRate: 10,
      repeat: -1,
    })
    this.anims.create({
      key: "turn",
      frames: [{ key: "dude", frame: 4 }],
      frameRate: 20,
    })
    this.anims.create({
      key: "right",
      frames: this.anims.generateFrameNumbers("dude", { start: 5, end: 8 }),
      frameRate: 10,
      repeat: -1,
    })
    //  Input Events
    cursors = this.input.keyboard.createCursorKeys()
    //  Some stars to collect, 12 in total, evenly spaced 70 pixels apart along the x axis
    stars = this.physics.add.group({
      key: "star",
      repeat: 11,
      setXY: { x: 12, y: 0, stepX: 70 },
    })
    stars.children.iterate(function (child) {
      //  Give each star a slightly different bounce
      /** @type {Phaser.Physics.Arcade.Sprite} */
      (child).setBounceY(Phaser.Math.FloatBetween(0.4, 0.8))
    })

    bombs = this.physics.add.group()

    //  The score
    scoreText = this.add.text(16, 16, "score: 0", {
      fontSize: "32px",
      //      fill: "#000",
    })

    //  Collide the player and the stars with the platforms
    this.physics.add.collider(player, platforms)
    this.physics.add.collider(stars, platforms)
    this.physics.add.collider(bombs, platforms)

    //  Checks to see if the player overlaps with any of the stars, if he does call the collectStar function

    this.physics.add.overlap(player, stars, collectStar, undefined, this)
    this.physics.add.collider(player, bombs, hitBomb, undefined, this)
  }
  /**
   * @this {Phaser.Scene}
   */
  function update() {
    if (gameOver) {
      return
    }

    if (cursors.left.isDown) {
      player.setVelocityX(-160)

      player.anims.play("left", true)
    } else if (cursors.right.isDown) {
      player.setVelocityX(160)

      player.anims.play("right", true)
    } else {
      player.setVelocityX(0)

      player.anims.play("turn")
    }

    if (cursors.up.isDown && player.body.touching.down) {
      player.setVelocityY(-330)
    }
  }
  /**
   * @param {Phaser.Types.Physics.Arcade.GameObjectWithBody} player
   * @param {Phaser.Types.Physics.Arcade.GameObjectWithBody} star
   */
  function collectStar(player, star) {
    /** @type {Phaser.Physics.Arcade.Sprite}*/
    (star).disableBody(true, true)
    //  Add and update the score
    score += 10
    scoreText.setText("Score: " + score)
    if (stars.countActive(true) === 0) {
      //  A new batch of stars to collect
      stars.children.iterate(function (child) {
        let c = /** @type {Phaser.Physics.Arcade.Sprite}*/ (child)
        c.enableBody(true, c.x, 0, true, true)
      })
      var x =
        /** @type {Phaser.Physics.Arcade.Sprite}*/
        (player).x < 400
          ? Phaser.Math.Between(400, 800)
          : Phaser.Math.Between(0, 400)

      var bomb = bombs.create(x, 16, "bomb")
      bomb.setBounce(1)
      bomb.setCollideWorldBounds(true)
      bomb.setVelocity(Phaser.Math.Between(-200, 200), 20)
      bomb.allowGravity = false
    }
  }
  /**
   * @this {Phaser.Scene}
   * @param {Phaser.Types.Physics.Arcade.GameObjectWithBody} player
   * @param {Phaser.Types.Physics.Arcade.GameObjectWithBody} bomb
   */
  function hitBomb(player, bomb) {
    this.physics.pause()
    let p = /** @type {Phaser.Physics.Arcade.Sprite}*/ (player)
    p.setTint(0xff0000)
    p.anims.play("turn")
    gameOver = true
  }
</script>

<h1>Welcome to SvelteKit Phaser</h1>
