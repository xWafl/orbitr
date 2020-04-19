<template>
    <div class="title">
        <h1 class="header">Orbitr</h1>
        <div class="main">
            <div class="info">
                <p v-if="!levelup && !gameOver">
                    In this game, you are a spaceship orbiting around a star. You have to dodge the fireballs that are
                    being hurtled towards you. To speed up or slow your ship, hold the mouse button. When you dodge 30
                    bullets, you will advance to the next level.
                </p>
                <p v-if="gameOver && score >= scoreThresh">
                    Great job! You beat level {{ level + 1 }} in {{ time }} seconds.
                </p>
                <p v-if="gameOver && score < scoreThresh">
                    Oh no! You lost. You dodged {{ score }}/30 bullets. Try this level again!
                </p>
            </div>
            <div class="play" @click="play">
                {{ gameOver && (score &lt; scoreThresh) ? "Try again" : "Play" }}
            </div>
            <a href="https://github.com/xWafl/ludumdaregame" target="_blank" class="github">
                <div>
                    Github
                </div>
            </a>
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
            started: Boolean,
        },
        methods: {
            play () {
                this.$emit("play", this.started && this.gameOver);
            },
        },
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
        display: grid;
        grid-template-rows: repeat(20, 5%);
        grid-template-columns: repeat(20, 5%);
        height: 50%;
        width: 90%;
        margin: 5%;
    }

    .info {
        grid-row: 1 / 21;
        grid-column: 1 / 10;
        width: 100%;
        height: 100%;
        display: flex;
        font-size: 30px;
        background-color: #2c3e50;
        border-radius: 10px;
        align-items: center;
    }

    .play {
        grid-column: 11 / 21;
        grid-row: 1 / 9;
        width: 100%;
        height: 100%;
        font-size: 48px;
        font-weight: bold;
        background-color: #2c3e50;
        border-radius: 10px;
        line-height: 350%;
        cursor: pointer;
    }

    .github {
        grid-column: 11 / 21;
        grid-row: 12 / 21;
        height: 100%;
        width: 100%;
        font-size: 48px;
        font-weight: bold;
        background-color: #2c3e50;
        border-radius: 10px;
        line-height: 350%;
        color: mediumpurple;
    }
</style>
