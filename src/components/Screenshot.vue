<template>
  <div id="screenshot">
    <video ref="video" autoplay></video>
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script>
export default {
  data: function() {
    return {
      stream: null
    };
  },
  methods: {
    async capture({ x = 0, y = 0, width = 0, height = 0 }) {
      const canvas = this.$refs.canvas;
      const video = this.$refs.video;
      // start capturing
      if (!this.stream || !this.stream.active) {
        alert(
          "Please select to stream the current browser tab to get the appropriate screenshots"
        );
        try {
          this.stream = await navigator.mediaDevices.getDisplayMedia({
            video: {
              // frameRate: 5,
              cursor: "never"
            },
            audio: false
          });
        } catch (err) {
          this.$store.commit("updateScreenshot", false);
        }
      }
      // get screenshot
      if (this.stream && this.stream.active) {
        video.srcObject = this.stream;
        video.play();
        setTimeout(() => {
          const context = canvas.getContext("2d");
          canvas.setAttribute("width", width || video.videoWidth);
          canvas.setAttribute("height", height || video.videoHeight);
          context.drawImage(
            video,
            x || 0,
            y || 0,
            width || video.videoWidth,
            height || video.videoHeight,
            0,
            0,
            width || video.videoWidth,
            height || video.videoHeight
          );
          canvas.toBlob(blob => {
            try {
              // eslint-disable-next-line no-undef
              const item = new ClipboardItem({ "image/png": blob });
              navigator.clipboard.write([item]);
              this.$store.commit("updateScreenshot", true);
            } catch (err) {
              this.$store.commit("updateScreenshot", false);
            }
          });
        }, 100);
      }
    }
  }
};
</script>

<style scoped>
video {
  width: 100%;
  height: 100%;
  display: none;
}
canvas {
  width: 100%;
  height: 100%;
  display: none;
}
</style>
