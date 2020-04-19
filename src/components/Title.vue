<template>
    <div class="title">
        <h1 class="header">Ludumdaregame</h1>
        <div class="main">
            <div class="info">
                <p v-if="!levelup && !gameOver">
                    In this game, you are a spaceship orbiting around a star. You have to dodge the fireballs that are being hurtled towards you. To speed up or slow your ship, hold the mouse button. When you dodge 30 fireballs, you will advance to the next level.
                </p>
                <p v-if="gameOver && score >= scoreThresh">
                    Great job! You beat level {{ level + 1 }} in {{ time }} seconds.
                </p>
                <p v-if="gameOver && score < scoreThresh">
                    Oh no! You lost. You dodged {{ score }} bullets. Try this level again!
                </p>
            </div>
            <div class="buttons">
                <div class="play" @click="play">
                    {{ gameOver && (score &lt; scoreThresh) ? "Try again" : "Play" }}
                </div>
                <a href="https://github.com/xWafl/ludumdaregame" target="_blank">
                    <div class="github">
                        Github
                    </div>
                </a>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: "Title",
        props: {
            levelup: Boolean,
            time: Number,
            score: Number,
            level: Number,
            scoreThresh: Number,
            gameOver: Boolean,
            started: Boolean
        },
        methods: {
            play () {
                this.$emit("play", this.started && this.gameOver);
            }
        }
    }
</script>

<style scoped>
    .title {
        color: white;
    }
    .header {
        font-size: 72px;
        background-color: #2c3e50;
        height: 20%;
        margin: 5%;
        border-radius: 10px;
        line-height: 200%;
    }
    .main {
        display: flex;
        height: 50%;
        align-items: center;
    }
    .info {
        width: 40%;
        margin: 5%;
        display: flex;
        font-size: 30px;
        background-color: #2c3e50;
        border-radius: 10px;
        height: 90%;
        align-items: center;
    }
    .buttons {
        width: 40%;
        margin: 5%;
        display: inline-block;
        font-size: 24px;
        border-radius: 10px;
        height: 100%;
        user-select: none;
    }
    .play {
        width: 100%;
        margin: 5%;
        display: inline-block;
        font-size: 48px;
        font-weight: bold;
        background-color: #2c3e50;
        border-radius: 10px;
        height: 40%;
        line-height: 300%;
        cursor: pointer;
    }
    .github {
        width: 100%;
        margin: 5%;
        display: inline-block;
        font-size: 48px;
        font-weight: bold;
        background-color: #2c3e50;
        border-radius: 10px;
        height: 40%;
        line-height: 300%;
        color: mediumpurple;
    }
</style>
