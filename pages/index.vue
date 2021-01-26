<template>
  <div>
    <form class="mt-4">
      <b-from-group id="fileInput">
        <b-form-file v-model="file" accept="image/jpeg, image/png" placeholder="Drag&drop or select add Image"></b-form-file>
      </b-from-group>
    </form>
      <b-from-group label="Language:">
        <b-form-checkbox-group v-model="selectedLangs" :options="langs"></b-form-checkbox-group>
      </b-from-group>
      <p ref="progress" class="my-3 text-center" v-if="progress">
        <span>{{progress.status}}</span>
        <span class="ml-1">{{Math.round(progress.progress * 100)}}%</span>
      </p>
      <div ref="preview" class="preview my-3">
        <img ref="previewImg" :src="previewImg">
        <canvas id="input-overlay" ref="input_overlay"></canvas>
      </div>
      <div ref="text" class="border p-3" v-if="result">
        <pre>{{result.text}}</pre>
      </div>
      <div v-if="result">
        {{result}}
      </div>
  </div>
</template>

<script>
import { createWorker } from "tesseract.js";

const worker = createWorker();

export default {
  name: "App",
  components: {},
  data(){
    return {
      selectedLangs: ["jpn", "enm"],
      langs: [
        { text: "Japanese", value: "jpn" },
        { text: "English", value: "enm" },
        { text: "Chinese(S)", value: "chi_sim" },
        { text: "Chinese(T)", value: "chi_tra" }
      ],
      file: null,
      previewImg: null,
      progress: null,
      result: null
    };
  },
  watch: {
    selectedLangs(val) {
      if (this.progress || !this.file || !this.selectedLangs.length) {
        return;
      }

      this.tesseract();
    },
    file(val) {
      console.log('test')
      if (!val) return;
      this.result = null;
      this.progress = null;

      this.tesseract();

      const fileReader = new FileReader();
      fileReader.onload = e => {
        this.previewImg = e.target.result;
      };
      fileReader.readAsDataURL(this.file);
    }
  },
  methods: {
    async tesseract() {
      await worker.load();
      await worker.loadLanguage('eng');
      await worker.initialize('eng');
      const { data: { text } } = await worker.recognize(this.file);
      console.log(text);
      await worker.terminate();
      this.result = text;
      // worker
      //   .recognize(this.file, this.selectedLangs.join("+"), {
      //     tessedit_create_hocr: 0,
      //     tessedit_create_tsv: 0
      //   })
      //   .progress(p => {
      //     this.progress = p;
      //   })
      //   .then(result => {
      //     console.log("result is: ", result);
      //     this.progress = null;
      //     this.result = result;
      //     this.createOverlay(result.words);
      //     worker.terminate();
      //   });
    },
    createOverlay(words) {
      this.ioctx.clearRect(
        0,
        0,
        this.$refs.input_overlay.width,
        this.$refs.input_overlay.height
      );
      words.forEach(w => {
        var b = w.bbox;

        this.ioctx.strokeWidth = 2;

        this.ioctx.strokeStyle = "red";
        this.ioctx.strokeRect(b.x0, b.y0, b.x1 - b.x0, b.y1 - b.y0);
        this.ioctx.beginPath();
        this.ioctx.moveTo(w.baseline.x0, w.baseline.y0);
        this.ioctx.lineTo(w.baseline.x1, w.baseline.y1);
        this.ioctx.strokeStyle = "green";
        this.ioctx.stroke();
      });
    }
  },
  // mounted() {
  //   this.ioctx = this.$refs.input_overlay.getContext("2d");
  //   this.$refs.previewImg.onload = e => {
  //     this.$refs.input_overlay.width = this.$refs.previewImg.naturalWidth;
  //     this.$refs.input_overlay.height = this.$refs.previewImg.naturalHeight;
  //   };
  // }
};
</script>

<style>
#app {
  font-family: sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
}

.preview {
  position: relative;
}

.preview img {
  display: block;
}

#input-overlay {
  position: absolute;
  left: 0;
  top: 0;
  z-index: 1;
}
</style>
