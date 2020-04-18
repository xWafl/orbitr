<template>
    <div class="score">
        {{ displayedVal }}
    </div>
</template>

<script>
    export default {
        name: "Score",
        props: {
            score: Number,
            time: Number,
            gameOver: Boolean
        },
        computed: {
            displayedVal() {
                return this.gameOver === true ? `Game over! Your score was ${this.score} and you lasted ${this.time}s` : `Score: ${this.score} Time: ${this.time}s`;
            }
        }
    }
</script>

<style scoped>
    .score {
        text-align: center;
        color: white;
        font-size: 48px;
    }
</style>
