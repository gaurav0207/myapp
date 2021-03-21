<template>
  <div class="flex align-items-center" style="height: 85%">
    <div class="video">
      <video muted autoplay></video>
      <div class="video-cancel">
        <ion-icon @click="closeModal" :icon="closeOutline"></ion-icon>
      </div>
      <div class="video-controls">
        {{ mediaRecord }}
        <ion-button
          v-if="!recording"
          class="ion-margin-top"
          type="primary"
          id="record"
          @click="startRecording"
          >Start</ion-button
        >
        <ion-button
          v-if="recording"
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
import { IonButton, IonIcon } from "@ionic/vue";
import { closeOutline } from "ionicons/icons";
interface State {
  webcamstream: any;
  streamRecorder: any;
  list: any;
  buttonText: string;
  mediaRecord: any;
  recording: boolean;
}
export default defineComponent({
  name: "Video",
  setup() {
    return { closeOutline };
  },
  data() {
    return {
      webcamstream: null,
      streamRecorder: {},
      list: "",
      mediaRecord: {},
      buttonText: "Record",
      recording: false,
    } as State;
  },
  components: {
    IonButton,
    IonIcon,
  },
  mounted() {
    this.initalizeVideo();
  },
  beforeUnmount() {
    if (this.webcamstream) {
      this.webcamstream.getTracks().forEach(function(track: any) {
        track.stop();
      });
    }
    this.recording = false;
  },
  methods: {
    async initalizeVideo() {
      const mimeType = "audio/webm";
      const w: any = window;
      const constraints = {
        video: { width: { exact: 640 }, height: { exact: 480 } },
        audio: true,
      };
      const video: any = document.querySelector("video");
      await navigator.mediaDevices.getUserMedia(constraints).then((stream) => {
        video.srcObject = stream;
        this.webcamstream = stream;
      });
      this.mediaRecord = new w.MediaRecorder(this.webcamstream, {
        type: mimeType,
      });
      let chunks: any = [];
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
    startRecording() {
      this.mediaRecord.start();
      this.recording = true;
    },
    stopRecording() {
      this.mediaRecord.stop();
      this.recording = false;
    },
    renderRecording(blob: any) {
      this.$store.dispatch("Media/addVideos", blob);
      this.$emit("reset-component");
    },
    closeModal() {
      this.$emit("reset-component");
    },
  },
});
</script>

<style lang="scss" scoped>
.video {
  position: relative;
  height: 100%;
  & video {
    width: 100%;
  }
  &-cancel {
    position: absolute;
    top: 10px;
    right: 10px;
    font-size: 20px;
    padding: 4px 6px 1px 5px;
    cursor: pointer;
    background: white;
    border-radius: 50%;
    box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.25);
  }
  &-controls {
    display: flex;
    position: absolute;
    bottom: 0px;
    width: 50%;
    opacity: 1;
    left: 25%;
  }
}
</style>
