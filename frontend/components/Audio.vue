<template>
  <v-card style="text-align: center; box-shadow:none">
    <v-card-text>
      <v-btn
        icon
        class="ma-2"
        color="teal"
        @click.native="playing ? pause() : play()"
        :disabled="!loaded"
      >
        <v-icon v-if="!playing || paused" class="fas fa-play"></v-icon>
        <v-icon v-else class="fas fa-pause"></v-icon>
      </v-btn>
      <v-btn
        icon
        class="ma-2"
        color="teal"
        @click.native="stop()"
        :disabled="!loaded"
      >
        <v-icon class="fas fa-stop"></v-icon>
      </v-btn>
      <v-btn
        icon
        class="ma-2"
        color="teal"
        @click.native="mute()"
        :disabled="!loaded"
      >
        <v-icon v-if="!isMuted" class="fa fa-volume-up"></v-icon>
        <v-icon v-else class="fa fa-volume-mute"></v-icon>
      </v-btn>
      <v-btn
        icon
        class="ma-2"
        color="teal"
        @click.native="loaded ? download() : reload()"
      >
        <v-icon v-if="!loaded" class="fa fa-redo"></v-icon>
        <v-icon v-else class="fa fa-arrow-alt-circle-down"></v-icon>
      </v-btn>
      <v-progress-linear
        v-model="percentage"
        height="5"
        style="margin-top: 15px; margin-bottom: 15px;"
        @click.native="setPosition()"
        color="teal"
        :disabled="!loaded"
      ></v-progress-linear>
      <p>{{ currentTime }} / {{ duration }}</p>
    </v-card-text>
    <audio
      id="player"
      ref="player"
      v-on:ended="ended"
      v-on:canplay="canPlay"
      :src="file"
      preload="true"
    ></audio>
  </v-card>
</template>
<script>
const formatTime = second =>
  new Date(second * 1000).toISOString().substr(11, 8)
export default {
  name: 'vuetify-audio',
  props: {
    file: {
      type: String,
      default: null
    },
    autoPlay: {
      type: Boolean,
      default: false
    },
    ended: {
      type: Function,
      default: () => {}
    },
    canPlay: {
      type: Function,
      default: () => {}
    }
  },
  computed: {
    duration: function() {
      return this.audio ? formatTime(this.totalDuration) : ''
    }
  },
  data() {
    return {
      firstPlay: true,
      isMuted: false,
      loaded: false,
      playing: false,
      paused: false,
      percentage: 0,
      currentTime: '00:00:00',
      audio: undefined,
      totalDuration: 0
    }
  },
  methods: {
    setPosition() {
      this.audio.currentTime = parseInt(
        (this.audio.duration / 100) * this.percentage
      )
    },
    stop() {
      this.paused = this.playing = false
      this.audio.pause()
      this.audio.currentTime = 0
    },
    play() {
      if (this.playing) return
      this.paused = false
      this.audio.play().then(_ => (this.playing = true))
    },
    pause() {
      this.paused = !this.paused
      this.paused ? this.audio.pause() : this.audio.play()
    },
    download() {
      this.audio.pause()
      window.open(this.file, 'download')
    },
    mute() {
      this.isMuted = !this.isMuted
      this.audio.muted = this.isMuted
      this.volumeValue = this.isMuted ? 0 : 75
    },
    reload() {
      this.audio.load()
    },
    _handleLoaded: function() {
      if (this.audio.readyState >= 2) {
        if (this.audio.duration === Infinity) {
          // Fix duration for streamed audio source or blob based
          // https://stackoverflow.com/questions/38443084/how-can-i-add-predefined-length-to-audio-recorded-from-mediarecorder-in-chrome/39971175#39971175
          this.audio.currentTime = 1e101
          this.audio.ontimeupdate = () => {
            this.audio.ontimeupdate = () => {}
            this.audio.currentTime = 0
            this.totalDuration = parseInt(this.audio.duration)
            this.loaded = true
          }
        } else {
          this.totalDuration = parseInt(this.audio.duration)
          this.loaded = true
        }
        if (this.autoPlay) this.audio.play()
      } else {
        throw new Error('Failed to load sound file')
      }
    },
    _handlePlayingUI: function(e) {
      this.percentage = (this.audio.currentTime / this.audio.duration) * 100
      this.currentTime = formatTime(this.audio.currentTime)
    },
    _handlePlayPause: function(e) {
      if (e.type === 'play' && this.firstPlay) {
        // in some situations, audio.currentTime is the end one on chrome
        this.audio.currentTime = 0
        if (this.firstPlay) {
          this.firstPlay = false
        }
      }
      if (
        e.type === 'pause' &&
        this.paused === false &&
        this.playing === false
      ) {
        this.currentTime = '00:00:00'
      }
    },
    _handleEnded() {
      this.paused = this.playing = false
    },
    init: function() {
      this.audio.addEventListener('timeupdate', this._handlePlayingUI)
      this.audio.addEventListener('loadeddata', this._handleLoaded)
      this.audio.addEventListener('pause', this._handlePlayPause)
      this.audio.addEventListener('play', this._handlePlayPause)
      this.audio.addEventListener('ended', this._handleEnded)
    }
  },
  mounted() {
    this.audio = this.$refs.player
    this.init()
  },
  beforeDestroy() {
    this.audio.removeEventListener('timeupdate', this._handlePlayingUI)
    this.audio.removeEventListener('loadeddata', this._handleLoaded)
    this.audio.removeEventListener('pause', this._handlePlayPause)
    this.audio.removeEventListener('play', this._handlePlayPause)
    this.audio.removeEventListener('ended', this._handleEnded)
  }
}
</script>
