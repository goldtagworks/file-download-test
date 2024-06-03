<template>
    <q-page class="row items-center justify-evenly">
        <q-btn @click="onClickedFileRead"> 파일읽기 </q-btn>
        <q-btn @click="onClickedDownload"> 다운로드 </q-btn>
        <q-linear-progress size="25px" :value="progress1" color="accent">
            <div class="absolute-full flex flex-center">
                <q-badge color="white" text-color="accent" :label="progressLabel1" />
            </div>
        </q-linear-progress>
    </q-page>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { useQuasar } from 'quasar';
import { Filesystem, Directory, Encoding } from '@capacitor/filesystem';

import axios from 'axios';

defineOptions({
    name: 'IndexPage'
});

const $q = useQuasar();
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

            const downloadStream = await axios.post(
                'http://127.0.0.1/BigBuckBunny.mp4',
                {},
                {
                    responseType: 'stream',
                    cancelToken: axios.CancelToken.source().token
                }
            );

            if (downloadStream) {
                let transferred = 0;
                const total = parseInt(downloadStream?.headers['content-length'] ?? '0');

                downloadStream?.data.on('data', (chunk: any): void => {
                    transferred += chunk.length;

                    progress1.value = (transferred * 100) / total;
                });

                return (async (): Promise<any> => {
                    try {
                        const filename = 'test.mp4';
                        await Filesystem.writeFile({ path: filename, data: downloadStream?.data, directory: Directory.Data, encoding: Encoding.ASCII });
                        //console.log(`File downloaded to ${query?.filename}`);
                        return { status: '0' };
                    } catch (err) {
                        const e: any = err;
                        console.error(`Something went wrong. ${e.message}`);
                    }
                })();
            }
        })
        .onCancel(() => {
            // console.log('Cancel')
        })
        .onDismiss(() => {
            // console.log('I am triggered on both OK and Cancel')
        });
};
</script>
