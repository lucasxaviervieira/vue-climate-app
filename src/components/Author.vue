<template>
    <div class="author">
        <a href="#">
            <img :src="logoImg" alt="Logo" />
        </a>
    </div>
</template>

<script setup>
const logoImg = "assets/logo-white.png";
</script>

<style scoped>
.author {
    position: absolute;
    bottom: .2em;
    left: 0;
    width: 100%;
    text-align: center;
    font-size: 1em;
    color: #fff;
    padding: 1em;
    z-index: 2000;
}
</style>