<template>
    <video id="video" ref="videoPlayer" :src="props.src" controls></video>
</template>

<script setup lang="ts">
import { ref } from 'vue';

interface Props {
    src: string;
}

defineOptions({
    name: 'VideoPlayer'
});

const props = withDefaults(defineProps<Props>(), {
    src: ''
});

const videoPlayer = ref(null);
</script>
