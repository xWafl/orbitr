<template>
    <div class="game">
        <canvas
            id="gameCanvas"
            :width="windowWidth"
            :height="windowHeight"
            @mousedown="orbiterSpeed = 0.01"
            @mouseup="orbiterSpeed = 0.005"
        ></canvas>
        <Score
            class="score"
            :score="score"
            :time="time"
            :gameOver="gameOver"
        ></Score>
    </div>
</template>

<script>
    import Vue from 'vue';
    import VueWindowSize from 'vue-window-size';
    import Score from "./Score.vue";

    Vue.use(VueWindowSize);

    export default {
        name: "Game",
        components: {
            Score
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
                width: 600,
                height: 600,
                centerRadius: 50,
                orbiterRadius: 100,
                orbiterWidth: 20,
                orbiterAngle: 0,
                orbiterSpeed: 0.005,
                fireballWidth: 20,
                fireballMovement: 10,
                fireballs: [],
                score: 0,
                time: 0,
                gameOver: false,
            }
        },
        methods: {
            distance(x1, y1, x2, y2) {
                const xDist = x2 - x1;
                const yDist = y2 - y1;
                return Math.sqrt(yDist ** 2 + xDist ** 2);
            },
            drawCenter () {
                const canvas = document.getElementById("gameCanvas");
                const ctx = canvas.getContext("2d");
                const [centerX, centerY] = [canvas.width >> 1, canvas.height >> 1];
                ctx.beginPath();
                ctx.arc(centerX, centerY, this.centerRadius, 0, 2 * Math.PI, false);
                ctx.fillStyle = '#FFF';
                ctx.fill();
                ctx.lineWidth = 0;
                ctx.strokeStyle = '#EEE';
                ctx.stroke();
            },
            drawOrbiter () {
                const canvas = document.getElementById("gameCanvas");
                const distanceY = Math.sin(this.orbiterAngle) * this.orbiterRadius + (canvas.height >> 1);
                const distanceX = Math.cos(this.orbiterAngle) * this.orbiterRadius + (canvas.width >> 1);
                this.drawCircle(distanceX, distanceY, this.orbiterWidth, "steelblue");
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
                    this.drawCenter();
                    this.moveOrbiter();
                    this.drawFireballs();
                    setTimeout(this.doOrbit, 10);
                }
            },
            incTimer () {
                if (this.gameOver === false) {
                    this.time++;
                    if (this.time === this.levelupSteps[0]) {
                        this.levelupSteps.pop();
                        this.currentStep++;
                    }
                    setTimeout(this.incTimer, 1000);
                }
            },
            throwFireball () {
                this.newFireball();
                setTimeout(this.throwFireball, this.timeSteps[this.currentStep]);
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
                    } else if (this.distance(fireball.x, fireball.y, orbiterX, orbiterY) < this.orbiterWidth * 2) {
                        this.gameOver = true;
                    } else {
                        this.drawCircle(fireball.x, fireball.y, this.fireballWidth + 1, "#000");
                        fireball.y -= Math.sin(fireball.angle) * this.fireballMovement * 0.1;
                        fireball.x -= Math.cos(fireball.angle) * this.fireballMovement * 0.1;
                        this.drawCircle(fireball.x, fireball.y, this.fireballWidth, "#F80");
                    }
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
            startGame () {
                this.fillBg();
                this.drawCenter();
                this.drawOrbiter();
                this.doOrbit();
                this.incTimer();
                this.throwFireball();
            }
        },
        mounted () {
            this.startGame();
        }
    }
</script>

<style scoped>
    .score {
        position: absolute;
        width: 100%;
        top: 5%;
    }
</style>
