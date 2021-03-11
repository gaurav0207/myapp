<template>
  <div class="flex align-items-center" style="max-height: 250px">
    <div id="container">
      <video muted autoplay></video>

      <div id="webcamcontrols">
        <ion-button
          class="ion-margin-top"
          type="primary"
          id="record"
          @click="startRecording"
          >Start</ion-button
        >
        <ion-button
          class="ion-margin-top"
          type="primary"
          id="record"
          @click="stopRecording"
          >STOP</ion-button
        >
      </div>
      <div id="recordings"></div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";

import { IonButton } from "@ionic/vue";

interface State {
  webcamstream: any;

  streamRecorder: any;

  list: any;

  buttonText: string;
  mediaRecord: any;
}

export default defineComponent({
  name: "Video",

  data() {
    return {
      webcamstream: null,

      streamRecorder: {},

      list: "",
      mediaRecord: {},

      buttonText: "Record",
    } as State;
  },

  components: {
    IonButton,
  },

  methods: {
    initalizeVideo() {
      const mimeType = "audio/webm";

      const w: any = window;

      this.mediaRecord = new w.MediaRecorder(this.webcamstream, {
        type: mimeType,
      });

      let chunks: any = [];
      console.log(this.mediaRecord);
      this.mediaRecord.addEventListener("dataavailable", (event: any) => {
        if (typeof event.data === "undefined") return;

        if (event.data.size === 0) return;
        console.log(event.data);
        chunks.push(event.data);
      });
      this.mediaRecord.addEventListener("stop", () => {
        const recording = new Blob(chunks, {
          type: mimeType,
        });
        this.renderRecording(recording);
        chunks = [];
      });
    },
    async startRecording() {
      const constraints = {
        video: { width: { exact: 640 }, height: { exact: 480 } },

        audio: true,
      };

      const video: any = document.querySelector("video");
      await navigator.mediaDevices.getUserMedia(constraints).then((stream) => {
        video.srcObject = stream;
        this.webcamstream = stream;
        this.initalizeVideo();
      });
      this.mediaRecord.start();
    },
    stopRecording() {
      console.log(this.mediaRecord, "stop");
      this.mediaRecord.stop();
    },
    renderRecording(blob: any) {
      console.log(blob);

      this.list = blob;

      //   this.$emit("hide-video");

      const blobUrl = URL.createObjectURL(blob);
      window.open(blobUrl, "_blank");
      this.webcamstream;
      this.webcamstream.getTracks().forEach(function(track: any) {
        track.stop();
      });
    },
  },
});
</script>

<style scoped>
video {
  width: 100%;
  height: 200px;
}
</style>
