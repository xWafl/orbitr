<template>
    <div class="game">
        <canvas
            id="gameCanvas"
            :width="windowWidth"
            :height="windowHeight"
            @mousedown="orbiterSpeed = 0.015"
            @mouseup="orbiterSpeed = 0.005"
        ></canvas>
        <Score
            v-if="started && !gameOver"
            class="score"
            :score="score"
            :level="level"
            :time="time"
            :gameOver="gameOver"
        ></Score>
        <Title
            v-if="!started || gameOver"
            :time="time"
            :gameOver="gameOver"
            :score="score"
            :scoreThresh="levelupScore"
            :started="started"
            :level="level"
            class="title"
            @play="this.levelup"
        ></Title>
    </div>
</template>

<script>
    import Vue from 'vue';
    import VueWindowSize from 'vue-window-size';
    import ColorHash from "color-hash";
    import Score from "./Score.vue";
    import Title from "./Title.vue";

    Vue.use(VueWindowSize);

    export default {
        name: "Game",
        components: {
            Score,
            Title
        },
        data () {
            return {
                timeSteps: {
                    0: 1000,
                    1: 500,
                    2: 333,
                    3: 250,
                    4: 150,
                    5: 100,
                },
                levelupSteps: [10, 20, 30, 40, 50],
                currentStep: 0,
                levelupScore: 30,
                width: 600,
                height: 600,
                centerRadius: 50,
                orbiterRadius: 100,
                orbiterWidth: 20,
                orbiterAngle: 0,
                orbiterSpeed: 0.005,
                orbiterColor: "steelblue",
                fireballWidth: 20,
                fireballMovement: 10,
                fireballs: [],
                fireballColor: "#F80",
                score: 0,
                time: 0,
                gameOver: false,
                flashingCenter: false,
                level: localStorage.getItem("level") ? JSON.parse(localStorage.getItem("level")) : 0,
                started: false,
                currentTimeouts: []
            }
        },
        methods: {
            distance(x1, y1, x2, y2) {
                const xDist = x2 - x1;
                const yDist = y2 - y1;
                return Math.sqrt(yDist ** 2 + xDist ** 2);
            },
            drawCenter () {
                this.flashingCenter = false;
                const canvas = document.getElementById("gameCanvas");
                const ctx = canvas.getContext("2d");
                const [centerX, centerY] = [canvas.width >> 1, canvas.height >> 1];
                ctx.beginPath();
                ctx.arc(centerX, centerY, this.centerRadius, 0, 2 * Math.PI, false);
                ctx.fillStyle = '#CCC';
                ctx.fill();
                ctx.lineWidth = 5;
                ctx.strokeStyle = '#DDD';
                ctx.stroke();
            },
            flashCenter() {
                this.flashingCenter = true;
                const canvas = document.getElementById("gameCanvas");
                const ctx = canvas.getContext("2d");
                const [centerX, centerY] = [canvas.width >> 1, canvas.height >> 1];
                ctx.beginPath();
                ctx.arc(centerX, centerY, this.centerRadius, 0, 2 * Math.PI, false);
                ctx.fillStyle = '#FFF';
                ctx.fill();
                ctx.lineWidth = 5;
                ctx.strokeStyle = '#FFF';
                ctx.stroke();
                setTimeout(this.drawCenter, 100);
            },
            drawOrbiter () {
                const canvas = document.getElementById("gameCanvas");
                const distanceY = Math.sin(this.orbiterAngle) * this.orbiterRadius + (canvas.height >> 1);
                const distanceX = Math.cos(this.orbiterAngle) * this.orbiterRadius + (canvas.width >> 1);
                this.drawCircle(distanceX, distanceY, this.orbiterWidth, this.orbiterColor);
            },
            drawCircle (x, y, radius, color) {
                const canvas = document.getElementById("gameCanvas");
                const ctx = canvas.getContext("2d", {alpha: false});
                ctx.beginPath();
                ctx.arc(x, y, radius, 0, 2 * Math.PI, false);
                ctx.fillStyle = color;
                ctx.fill();
                ctx.strokeStyle = color;
                ctx.stroke();
            },
            clearOrbiter () {
                const canvas = document.getElementById("gameCanvas");
                const distanceY = Math.sin(this.orbiterAngle) * this.orbiterRadius + (canvas.height >> 1);
                const distanceX = Math.cos(this.orbiterAngle) * this.orbiterRadius + (canvas.width >> 1);
                this.drawCircle(distanceX, distanceY, this.orbiterWidth + 1, "#000");
            },
            moveOrbiter () {
                this.clearOrbiter();
                this.orbiterAngle += this.orbiterSpeed;
                this.drawOrbiter();
            },
            doOrbit() {
                if (this.gameOver === false) {
                    if (!this.flashingCenter) {
                        this.drawCenter();
                    }
                    this.moveOrbiter();
                    this.drawFireballs();
                    this.currentTimeouts.push(setTimeout(this.doOrbit, 10));
                }
            },
            incTimer () {
                if (this.gameOver === false) {
                    this.time++;
                    if (this.time === this.levelupSteps[0]) {
                        this.levelupSteps.shift();
                        this.currentStep++;
                    }
                    this.currentTimeouts.push(setTimeout(this.incTimer, 1000));
                }
            },
            throwFireball () {
                this.newFireball();
                this.currentTimeouts.push(setTimeout(this.throwFireball, this.timeSteps[this.currentStep]));
            },
            fillBg () {
                const canvas = document.getElementById("gameCanvas");
                const ctx = canvas.getContext("2d", {alpha: false});
                ctx.fillStyle = "#000";
                ctx.fillRect(0, 0, canvas.width, canvas.height);
            },
            drawFireballs () {
                const canvas = document.getElementById("gameCanvas");
                const [canvasX, canvasY] = [canvas.width >> 1, canvas.height >> 1];
                const [orbiterX, orbiterY] = [Math.cos(this.orbiterAngle) * this.orbiterRadius + (canvas.width >> 1), Math.sin(this.orbiterAngle) * this.orbiterRadius + (canvas.height >> 1)];
                for (const i in this.fireballs) {
                    const fireball = this.fireballs[i];
                    if (this.distance(fireball.x, fireball.y, canvasX, canvasY) < this.centerRadius * 1.25) {
                        this.fireballs.splice(i, 1);
                        this.score++;
                        this.drawCircle(fireball.x, fireball.y, this.fireballWidth + 1, "#000");
                        this.flashCenter();
                    } else if (this.distance(fireball.x, fireball.y, orbiterX, orbiterY) < this.orbiterWidth * 2) {
                        this.gameOver = true;
                    } else {
                        this.drawCircle(fireball.x, fireball.y, this.fireballWidth + 1, "#000");
                        fireball.y -= Math.sin(fireball.angle) * this.fireballMovement * 0.1;
                        fireball.x -= Math.cos(fireball.angle) * this.fireballMovement * 0.1;
                        this.drawCircle(fireball.x, fireball.y, this.fireballWidth, this.fireballColor);
                    }
                }
                if (this.score >= this.levelupScore) {
                    this.gameOver = true;
                }
            },
            newFireball () {
                const canvas = document.getElementById("gameCanvas");
                const side = Math.floor(Math.random() * 4);
                const offset = Math.floor(Math.random() * (side % 2 === 0 ? canvas.height : canvas.width));
                if (side === 0) {
                    this.fireballs.push({
                        x: offset,
                        y: 0,
                        angle: Math.atan2((canvas.height >> 1) * -1, offset - (canvas.width >> 1))
                    });
                } else if (side === 1) {
                    this.fireballs.push({
                        x: canvas.width,
                        y: offset,
                        angle: Math.atan2(offset - (canvas.height >> 1), (canvas.width >> 1))
                    });
                } else if (side === 2) {
                    this.fireballs.push({
                        x: offset,
                        y: canvas.height,
                        angle: Math.atan2((canvas.height >> 1), offset - (canvas.width >> 1))
                    });
                } else if (side === 3) {
                    this.fireballs.push({
                        x: 0,
                        y: offset,
                        angle: Math.atan2(offset - (canvas.height >> 1), (canvas.width >> 1) * -1)
                    });
                }
            },
            levelup () {
                if (this.score >= this.levelupScore) {
                    this.level++;
                    this.fireballColor = new ColorHash().hex(`${this.level}fire`);
                    this.orbiterColor = new ColorHash().hex(`${this.level}orbit`);
                    localStorage.setItem("level", this.level);
                }
                this.currentTimeouts.map(l => {
                    clearTimeout(l);
                });
                this.currentTimeouts = [];
                this.time = 0;
                this.score = 0;
                this.fireballs = [];
                this.orbiterAngle = 0;
                this.gameOver = false;
                this.orbiterSpeed = 0.005;
                this.levelupSteps = [10, 20, 30, 40, 50];
                this.currentStep = 0;
                this.startGame();
            },
            startGame () {
                this.started = true;
                this.fillBg();
                this.drawCenter();
                this.drawOrbiter();
                this.doOrbit();
                this.incTimer();
                this.throwFireball();
            }
        },
        mounted () {
            if (this.level !== 0) {
                this.fireballColor = new ColorHash().hex(`${this.level}fire`);
                this.orbiterColor = new ColorHash().hex(`${this.level}orbit`);
            }
        }
    }
</script>

<style scoped>
    .game {
        background-color: #000;
    }
    .score {
        position: absolute;
        width: 100%;
        top: 5%;
    }
    .title {
        position: absolute;
        width: 100%;
        height: 100%;
        top: 0;
        left: 0;
    }
</style>
