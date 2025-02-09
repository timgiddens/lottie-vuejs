<template>
  <div v-if="style" :style="style" ref="lavContainer"/>
</template>

<script>
import lottie from "lottie-web";
import axios from "axios";

export default {
  props: {
    path: {
      required: true
    },
    speed: {
      type: Number,
      required: false,
      default: 1
    },
    width: {
      type: Number,
      required: false,
      default: -1
    },
    height: {
      type: Number,
      required: false,
      default: -1
    },
    loop: {
      type:Boolean,
      required: false,
      default: true
    },
    autoPlay: {
      type:Boolean,
      required: false,
      default: true
    },
    loopDelayMin: {
      type: Number,
      required: false,
      default: 0
    },
    loopDelayMax: {
      type: Number,
      required: false,
      default: 0
    },
    preserveAspectRatio: {
      type: String,
      required: false,
      default: 'xMinYMin meet'
    }
  },
  data: () => ({
    name: 'lottie-animation',
    rendererSettings: {
      scaleMode: "centerCrop",
      clearCanvas: true,
      progressiveLoad: false,
      hideOnTransparent: true,
    },
    anim: null,
    style: null
  }),
  mounted() {
    this.init();
  },
  methods: {
    async loadJsonData(path) {
      let p;
      if(this.isURL(path)) p = path;
      else p = "/"+path;

      return await axios.get(p).then(response => {
        return response.data;
      });
    },
    isURL(path){
      // just something I stole from stack
      // https://stackoverflow.com/questions/5717093/check-if-a-javascript-string-is-a-url

      let urlpattern = new RegExp(
        '^(https?:\\/\\/)?'+ // protocol
        '((([a-z\\d]([a-z\\d-]*[a-z\\d])*)\\.)+[a-z]{2,}|'+ // domain name
        '((\\d{1,3}\\.){3}\\d{1,3}))'+ // OR ip (v4) address
        '(\\:\\d+)?(\\/[-a-z\\d%_.~+]*)*'+ // port and path
        '(\\?[;&a-z\\d%_.~+=-]*)?' // query string
      ,'i'); // fragment locator

      return !!urlpattern.test(path);
    },
    async init() {
      this.style = {
        width: (this.width != -1 )? `${this.width}px` : '100%',
        height: (this.height != -1 )? `${this.height}px` : '100%',
        overflow: "hidden",
        margin: "0 auto"
      };

      let jsonData = await this.loadJsonData(this.path);

      if(this.anim) {
        this.anim.destroy(); // Releases resources. The DOM element will be emptied.
      }

      this.anim = lottie.loadAnimation({
        container: this.$refs.lavContainer,
        renderer: "svg",
        loop: this.loop,
        autoplay: this.autoPlay,
        animationData: jsonData,
        rendererSettings: Object.assign(this.rendererSettings, { preserveAspectRatio: this.preserveAspectRatio })
      });

      this.$emit("AnimControl", this.anim);

      this.anim.setSpeed(this.speed);
      if (this.loopDelayMin > 0) {
        this.anim.loop = false;
        this.anim.autoplay = false;
        this.executeLoop();
      }
    },
    getRandomInt(min, max) {
      min = Math.ceil(min);
      max = Math.floor(max);
      return Math.floor(Math.random() * (max - min)) + min; //The maximum is exclusive and the minimum is inclusive
    },
    executeLoop() {
      this.anim.play();
      setTimeout(() => {
        this.anim.stop();
        this.executeLoop();
      }, this.getRandomInt(this.loopDelayMin, this.loopDelayMax == 0? this.loopDelayMin : this.loopDelayMax));
    },


  },
  watch: {
    path: function(newVal, oldVal) {
      this.init();
    }
  }
};
</script>