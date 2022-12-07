<script>
export default {
    data() {
        return {
            canvas: null,
            context: null,
            colorMap: {
                'P': 'pink',
                'C': 'cyan'
            },
            brickGap: 2,
            brickWidth: 25,
            brickHeight: 12,
            wallSize: 12,
            bricks: [],

            levels:
                [
                    [
                        [],
                        [],
                        [],
                        [],
                        ['C', 'C', 'C', 'P', 'C', 'C', 'C', 'P', 'C', 'C', 'P', 'C', 'C', 'C'],
                        ['C', 'P', 'C', 'C', 'C', 'C', 'C', 'P', 'C', 'C', 'C', 'C', 'P', 'C'],
                        ['C', 'C', 'C', 'C', 'C', 'C', 'C', 'C', 'C', 'C', 'C', 'C', 'C', 'C'],
                        ['C', 'C', 'C', 'C', 'C', 'C', 'C', 'C', 'C', 'C', 'C', 'C', 'C', 'C']
                    ]
                ],
            colorCode: '',

            paddle: {
                x: 0,
                y: 440,
                width: 0,
                height: 0,

                // paddle x velocity
                dx: 0,

                speed: 8,

                img: null,
            },

            ball: {
                x: 130,
                y: 260,
                width: 20,
                height: 20,
                multiplier: 1.05,

                // how fast the ball should go in either the x or y direction
                speed: 2,

                // ball velocity
                dx: 0,
                dy: 0,

                img: null,
            },

            brick: {},
            render: false,
        }
    },

    methods: {

        // check for collision between two objects using axis-aligned bounding box (AABB)
        collides: function (obj1, obj2) {
            return (
                obj1.x < obj2.x + obj2.width &&
                obj1.x + obj1.width > obj2.x &&
                obj1.y < obj2.y + obj2.height &&
                obj1.y + obj1.height > obj2.y
            )
        },
        loop: function () {
            requestAnimationFrame(this.loop)
            this.context.clearRect(0, 0, this.canvas.width, this.canvas.height)

            // move paddle by it's velocity
            this.paddle.x += this.paddle.dx

            // prevent paddle from going through walls
            if (this.paddle.x < this.wallSize) {
                this.paddle.x = this.wallSize
            } else if (this.paddle.x + (this.brickWidth * 4) > this.canvas.width - this.wallSize) {
                this.paddle.x = this.canvas.width - this.wallSize - (this.brickWidth * 4)
            }

            // move ball by it's velocity
            this.ball.x += this.ball.dx
            this.ball.y += this.ball.dy

            // prevent ball from going through walls by changing its velocity
            // left & right walls
            if (this.ball.x < this.wallSize) {
                this.ball.x = this.wallSize
                this.ball.dx *= -1
            } else if (this.ball.x + this.ball.width > this.canvas.width - this.wallSize) {
                this.ball.x = this.canvas.width - this.wallSize - this.ball.width
                this.ball.dx *= -1
            }
            // top wall
            if (this.ball.y < this.wallSize) {
                this.ball.y = this.wallSize
                this.ball.dy *= -1
            }

            // reset ball if it goes below the screen
            if (this.ball.y > this.canvas.height) {
                this.ball.x = 130
                this.ball.y = 260
                this.ball.dx = 0
                this.ball.dy = 0
            }

            // check to see if ball collides with paddle. if they do change y velocity
            if (this.collides(this.ball, this.paddle)) {
                this.ball.dy *= -this.ball.multiplier

                // move ball above the paddle otherwise the collision will happen again
                // in the next frame
                this.ball.y = this.paddle.y - this.ball.height
            }

            // check to see if ball collides with a brick. if it does, remove the brick
            // and change the ball velocity based on the side the brick was hit on
            for (let i = 0; i < this.bricks.length; i++) {
                this.brick = this.bricks[i]

                if (this.collides(this.ball, this.brick)) {

                    // increase the speed of the game
                    // this.paddle.speed *= 1.05

                    // remove brick from the bricks array
                    this.bricks.splice(i, 1)

                    // ball is above or below the brick, change y velocity
                    // account for the balls speed since it will be inside the brick when it
                    // collides
                    if (this.ball.y + this.ball.height - this.ball.speed <= this.brick.y ||
                        this.ball.y >= this.brick.y + this.brick.height - this.ball.speed) {
                        this.ball.dy *= -1
                    }
                    // ball is on either side of the brick, change x velocity
                    else {
                        this.ball.dx *= -1
                    }


                    console.log(this.bricks.length)
                    if (this.brick.color === 'pink') {
                        console.log('мысль')
                    }

                    if (this.bricks.length === 0) {
                        console.log('ended')
                        alert('Игра закончена!')
                    }

                    break
                }
            }

            // draw walls
            this.context.fillStyle = 'lightgrey'
            this.context.fillRect(0, 0, this.canvas.width, this.wallSize)
            this.context.fillRect(0, 0, this.wallSize, this.canvas.height)
            this.context.fillRect(this.canvas.width - this.wallSize, 0, this.wallSize, this.canvas.height)

            // draw ball if it's moving
            if (this.ball.dx || this.ball.dy) {
                // this.context.fillRect(this.ball.x, this.ball.y, this.ball.height, this.ball.width)
                this.context.drawImage(this.ball.img, this.ball.x, this.ball.y, this.ball.width, this.ball.height)
            }

            // draw bricks
            this.bricks.forEach((brick) => {
                let grd = this.context.
                createLinearGradient(
                    (brick.x + (brick.width / 2)), (brick.y),
                    (brick.x + (brick.width / 2)), (brick.y + brick.height))
                if (brick.color === 'cyan') {
                    // console.log(brick)
                    grd.addColorStop(0, "green")
                    grd.addColorStop(0.5, "cyan")
                    grd.addColorStop(1, "green")
                }
                if (brick.color === 'pink') {
                    // console.log(brick)
                    grd.addColorStop(0, "#a62276")
                    grd.addColorStop(0.1, "#ffd3e6")
                    grd.addColorStop(0.5, "#fc4697")
                    grd.addColorStop(1, "#fe87c2")
                }

                this.context.fillStyle = grd
                this.context.fillRect(brick.x, brick.y, brick.width, brick.height)
            })

            // draw paddle
            // this.context.fillRect(this.paddle.x, this.paddle.y, this.paddle.width, this.paddle.height)
            this.context.drawImage(this.paddle.img, this.paddle.x, this.paddle.y, this.paddle.width, this.paddle.height)
        }

    },

    mounted() {
        // set game-screen
        this.canvas = document.getElementById('game')
        this.context = this.canvas.getContext('2d')

        // set paddle
        // place the paddle horizontally in the middle of the screen
        this.paddle.x = this.canvas.width / 2 - this.brickWidth / 2
        this.paddle.width = this.brickWidth * 4
        this.paddle.height = this.brickHeight * 3


        // set bricks
        for (let row = 0; row < this.levels[0].length; row++) {
            for (let col = 0; col < this.levels[0][row].length; col++) {
                this.colorCode = this.levels[0][row][col]

                this.bricks.push({
                    x: this.wallSize + (this.brickWidth + this.brickGap) * col,
                    y: this.wallSize + (this.brickHeight + this.brickGap) * row,
                    color: this.colorMap[this.colorCode],
                    width: this.brickWidth,
                    height: this.brickHeight
                })
            }
        }

        // listen to keyboard events to move the paddle
        document.addEventListener('keydown', (e) => {
            // left arrow key
            if (e.key === 'ArrowLeft') {
                e.preventDefault()
                this.paddle.dx = -this.paddle.speed
            }
            // right arrow key
            else if (e.key === 'ArrowRight') {
                e.preventDefault()
                this.paddle.dx = this.paddle.speed
            }

            // space key
            // if they ball is not moving, we can launch the ball using the space key. ball
            // will move towards the bottom right to start
            if (this.ball.dx === 0 && this.ball.dy === 0 && e.code === 'Space') {
                this.ball.dx = this.ball.speed
                this.ball.dy = this.ball.speed
            }
        })

        document.addEventListener('keyup', (e) => {
            if (e.key === 'ArrowLeft' || e.key === 'ArrowRight') {
                this.paddle.dx = 0
            }
        })

        this.ball.img = new Image()
        this.ball.img.onload = () => this.render = true
        this.ball.img.src = './ball.svg'

        this.paddle.img = new Image()
        this.paddle.img.onload = () => this.render = true
        this.paddle.img.src = './orbit-paddle.png'

        this.loop()
    },


    watch: {
        'ball.dy': {
            deep: true,
            handler: function (payload) {
                console.log('ball is now: ' + payload)
            }
        }
    }
}
</script>

<template>
    <canvas width="400" height="500" id="game"></canvas>
</template>

<style scoped>
html {
    font-size: 4.2675vw;
}

@media (min-width: 1024px) {
    html {
        font-size: 0.833vw;
    }
}

html, body {
    height: 100%;
    width: 100%;
    margin: 0;
    padding: 0;
}

body {
    background: black;
    display: flex;
    align-items: center;
    justify-content: center;
}
</style>
