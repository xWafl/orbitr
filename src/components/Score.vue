<template>
    <div class="score">
        Level: {{ level + 1 }} Score: {{ this.score }} Time: {{ this.time }}s
    </div>
</template>

<script>
    export default {
        name: "Score",
        props: {
            score: Number,
            time: Number,
            level: Number,
            gameOver: Boolean,
            msg: String
        }
    }
</script>

<style scoped>
    .score {
        text-align: center;
        color: white;
        font-size: 48px;
        user-select: none;
    }
</style>
