<template>
    <q-page padding>
        <p class="row items-center justify-evenly">
            <VideoPlayer :src="videoUrl"></VideoPlayer>
        </p>

        <p>
            <q-btn @click="onClickedFileRead"> 파일읽기 </q-btn>
            <q-btn @click="onClickedDownload"> 다운로드 </q-btn>
        </p>
        <p>
            <q-linear-progress size="25px" :value="progress1" color="accent">
                <div class="absolute-full flex flex-center">
                    <q-badge color="white" text-color="accent" :label="progressLabel1" />
                </div>
            </q-linear-progress>
        </p>
    </q-page>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { useQuasar } from 'quasar';
import { Filesystem, Directory, Encoding } from '@capacitor/filesystem';

import VideoPlayer from 'src/components/VideoPlayer.vue';

defineOptions({
    name: 'IndexPage'
});

const $q = useQuasar();
const downloadUrl = 'http://127.0.0.1/BigBuckBunny.mp4';
const videoUrl = ref('');
const progress1 = ref(0);
const progressLabel1 = computed(() => (progress1.value * 100).toFixed(2) + '%');

const onClickedFileRead = async () => {
    await Filesystem.writeFile({
        path: '1.txt',
        data: '하하하',
        directory: Directory.Documents,
        encoding: Encoding.UTF8
    });
    const s = await Filesystem.readFile({
        path: '1.txt',
        directory: Directory.Documents,
        encoding: Encoding.UTF8
    });
    $q.dialog({
        message: s.data as string,
        cancel: true,
        persistent: true
    });
};

const onClickedDownload = () => {
    $q.dialog({
        message: '파일을 다운로드 하시겠습니까?',
        cancel: true,
        persistent: true
    })
        .onOk(async () => {
            // console.log('OK')
            // c:\workspacePHP\D300.source\htdocs\ElephantsDream.mp4
            // c:\workspacePHP\D300.source\htdocs\BigBuckBunny.mp4

            fetch(downloadUrl)
                .then(async (response: any) => {
                    //
                    const reader = response.body.getReader();
                    for await (const chunk of readChunks(reader)) {
                        console.log(`received chunk of size: ${chunk.length}`);
                        console.log(chunk);
                    }
                })
                .then((result: any) => {
                    console.log('all done!', result);
                })
                .catch((err: any) => {
                    console.error(err);
                });
        })
        .onCancel(() => {
            // console.log('Cancel')
        })
        .onDismiss(() => {
            // console.log('I am triggered on both OK and Cancel')
        });
};

const readChunks = (reader: any) => {
    return {
        async *[Symbol.asyncIterator]() {
            let readResult = await reader.read();
            while (!readResult.done) {
                yield readResult.value;
                readResult = await reader.read();
            }
        }
    };
};
</script>
<style>
p {
    margin-bottom: 16px;
}
</style>
