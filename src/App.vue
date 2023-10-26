<template>
    <GaltonBoard />
</template>

<script lang="ts">
import { defineComponent } from "vue";
import GaltonBoard from "./components/GaltonBoard.vue";

export default defineComponent({
    name: "App",
    components: { GaltonBoard },
});
</script>

<style scoped></style>
