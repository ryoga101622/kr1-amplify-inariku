<template>
  <div class="left">
    <textarea class="ioarea" v-model="inputText" placeholder="Type to translate."></textarea>
  </div>
  <div class="right">
    <textarea class="ioarea" v-model="translatedText" disabled> </textarea>
  </div>
  <form>
    <input v-model="translatedText" />
    <button type="button" @click="speech">Speech</button>
  </form>

</template>

<script>
import { Predictions } from 'aws-amplify';
import { ref, watch } from 'vue';

export default {
  setup() {
    const inputText = ref('');
    const translatedText = ref('翻訳する文字列');

    const translate = (text) => {
      if (!text.length) {
        translatedText.value = '';
        return;
      }

      // Text Translate の実装 ↓
      Predictions.convert({
        translateText: {
          source: {
            text: text,
          },
        },
      })
        .then((result) => {
          translatedText.value = result.text;
        })
        .catch((error) => {
          console.warn({ error });
        });
      // ↑↑↑↑↑↑
    }

    const speech = () => {
      // Text Speech の実装
      Predictions.convert({
        textToSpeech: {
          source: {
            text: translatedText.value,
          },
        },
      })
        .then((result) => {
          const AudioContext = window.AudioContext || window.webkitAudioContext;
          const audioCtx = new AudioContext();
          const source = audioCtx.createBufferSource();

          audioCtx.decodeAudioData(result.audioStream, (buffer) => {
           source.buffer = buffer;
           source.connect(audioCtx.destination);
           source.start(0);
         });
       })
       .catch((error) => console.warn(error));
       // ↑↑↑↑↑↑
      };
    

    watch(inputText, (inputText) => {
      translate(inputText);
    });

    return {
      inputText,
      translatedText,
      translate,
      speech,
    };
  },
};
</script>
