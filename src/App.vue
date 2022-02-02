<template>
  <div id="app"
  class="recorder">

    <div class="recorder__header">
      <h1 class="recorder__h1">screen recorder</h1>
    
      <div class="recorder__buttons">

        <button class="recorder__btn"
          v-if="!isRecording"
          @click="startRecording()">
          Start recorder
        </button>

        <button class="recorder__btn recorder__btn--red"
          v-if="isRecording"
          @click="stopRecording()">
          Stop
        </button>

      </div>
    </div>


    <div class="recorder__wrapper">
      <div class="recorder__footer" v-if="isShowFooter">

        <video 
          class="recorder__video"
          controls="controls"
          :src="videoSrc">
        </video>

        <div class="recorder__buttons">
          <a class="recorder__btn"
            @click="downloadVideo()"
            ref="downloadBtn">
            Download
          </a>
          
          <button class="recorder__btn recorder__btn--red"
            @click="clearData()">
            Clear
          </button>
        </div>

      </div>
    </div>

  </div>
</template>

<script>

export default {
  name: 'App',
  data() {
    return {
      isRecording: false,
      isShowFooter: false,
      options: { mimeType: 'video/webm; codecs=vp9' },
      displayMediaOptions: {
        video: {
          cursor: "always",
          logicalSurface: false
        },
        audio: {
          echoCancellation: false,
          noiseSuppression: false,
          sampleRate: 44100
        }
      },
      recordedChunks: [],
      stream: {},
      mediaRecorder: {},
      videoSrc: null
    }
  },
  methods: {
    async startRecording() {
      this.isRecording = true;
      
      this.stream = await navigator.mediaDevices.getDisplayMedia(this.displayMediaOptions);
      this.mediaRecorder = new MediaRecorder(this.stream, this.options);
      this.mediaRecorder.ondataavailable = this.handleDataAvailable;
      this.mediaRecorder.start();
    },
    handleDataAvailable(event) {
      if (event.data.size > 0) {
        this.recordedChunks.push(event.data);

        this.isRecording = false;

        this.videoSrc = this.makeVideoUrl();
      }
    },
    downloadVideo() {
      if (this.isRecording === false) {
        const btn = this.$refs.downloadBtn;

        btn.href = this.makeVideoUrl();
        btn.download = Date.now();
      }
    },
    makeVideoUrl() {
      let blob = new Blob(this.recordedChunks, {
        'type' : 'video/webm'
      });
      return window.URL.createObjectURL(blob);
    },
    stopRecording() {
      this.mediaRecorder.stop();
      this.stream.getTracks().forEach( track => track.stop() );

      this.isShowFooter = true;
    },
    clearData() {
      this.isShowFooter = false,
      this.recordedChunks = [],
      this.stream = {},
      this.mediaRecorder = {},
      this.videoSrc = null
    }
  }
}
</script>

<style lang="scss">
body {
  background: #1b2631;
  font-family: Arial, Helvetica, sans-serif;
}
.recorder {
  &__h1 {
    font-size: 45px;
    letter-spacing: 3px;
    color: #f1c40f;
    text-align: center;
  }

  &__buttons {
    display: flex;
    justify-content: center;
  }

  &__btn {
    display: inline-block;
    background: #1b2631;
    margin: 10px auto;
    padding: 10px 30px;
    border: 2px solid #f1c40f;
    color: #f1c40f;
    font-family: Arial, Helvetica, sans-serif;
    text-transform: uppercase;
    letter-spacing: 1px;
    font-weight: 600;
    cursor: pointer;
    transition: 0.8s;

    &:hover {
      background: #f1c40f;
      color: #1b2631;
    }

    &--red {
      border: 2px solid #b40808;
      color: #b40808;

        &:hover {
        background: #b40808;
        color: white;
      }
    }
  }

  &__video {
    display: block;
    width: 50%;
    margin: 30px auto;
  }
}
</style>
