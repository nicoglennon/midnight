<template>
  <div class="templatewrapper">
    <div v-if="loaded">
      <div class="playerwrapper"></div>
      <div class="marqueewrapper">
        <div class="marqueeouter">
          <div
            class="marqueeinner"
            @mouseenter="handleShowSongTitle"
            @mouseleave="handleHideSongTitle"
          >
            <div class="playerwrapper">
              <div
                v-if="firstPlay"
                v-on:click="onClickBack"
                class="buttonwrapper"
              >
                <PhSkipBack :size="40" color="#fff" />
              </div>
              <div
                v-on:click="onPause"
                v-if="state === 'playing'"
                class="buttonwrapper"
              >
                <PhPauseCircle :size="40" color="#fff" />
              </div>
              <div v-else v-on:click="onPlay" class="buttonwrapper">
                <PhPlayCircle :size="40" color="#fff" />
              </div>
              <div v-if="firstPlay" v-on:click="onNext" class="buttonwrapper">
                <PhSkipForward :size="40" color="#fff" />
              </div>
            </div>
            <div>
              <a
                :href="currentTrack.url"
                target="_blank"
                class="songlink"
                v-if="currentTrack && showSongTitle"
              >
                <!-- <PhMusicNotesSimple :size="30" /> -->
                <p class="tracktitle">{{ currentTrack.title }}</p>
              </a>
            </div>
          </div>
        </div>
        <div
          class="timetrackerdiv"
          :style="{ width: `${percentageSong}%` }"
        ></div>
      </div>
    </div>
  </div>
</template>

<script>
/*global SC*/
const modulo = (n, m) => {
  var remain = n % m;
  return Math.floor(remain >= 0 ? remain : remain + m);
};
import {
  PhPauseCircle,
  PhPlayCircle,
  PhSkipForward,
  PhSkipBack,
} from "phosphor-vue";
export default {
  name: "Main",
  components: {
    PhPauseCircle,
    PhPlayCircle,
    PhSkipForward,
    PhSkipBack,
  },
  props: {
    msg: String,
  },
  data: () => ({
    currentPlaylistId: "1160871130",
    currentTrackId: null,
    trackIds: [],
    tracks: [],
    trackIndex: 0,
    currentTrack: null,
    loaded: false,
    state: "paused",
    firstPlay: false,
    showSongTitle: false,
    percentageSong: 0,
  }),
  mounted: async function () {
    const res = await SC.get(`/playlists/${this.currentPlaylistId}`);
    this.tracks = res.tracks.map((track) => ({
      id: track.id,
      title: track.title,
      artist: track.user.username,
      url: track.permalink_url,
      duration: track.duration,
    }));
    this.trackIds = res.tracks.map((track) => track.id);
    this.currentTrackId = this.trackIds[this.trackIndex];
    await this.pullSong();
    this.loaded = true;
  },
  methods: {
    pullSong: async function () {
      try {
        this.player = await SC.stream(`/tracks/${this.currentTrackId}`);
        // console.log("pulled successfully!");
        this.player.on("finish", this.onNext);
        this.player.on("time", this.trackTime);
      } catch (e) {
        // console.error(
        //   "Playback rejected. Try calling play() from a user interaction.",
        //   e
        // );
      }
    },
    onPlay: async function () {
      await this.player.play();
      this.state = "playing";
      this.currentTrack = this.tracks[this.trackIndex];
      if (!this.firstPlay) {
        this.firstPlay = true;
      }
    },
    onNext: async function () {
      await this.player.pause();
      this.trackIndex = (this.trackIndex + 1) % this.trackIds.length;
      this.currentTrackId = this.trackIds[this.trackIndex];
      await this.pullSong();
      await this.onPlay();
    },
    onBack: async function () {
      await this.player.pause();
      this.trackIndex = modulo(this.trackIndex - 1, this.trackIds.length);
      this.currentTrackId = this.trackIds[this.trackIndex];
      await this.pullSong();
      await this.onPlay();
    },
    onClickBack: async function () {
      const currentTime = await this.player.currentTime();
      if (currentTime > 2000) {
        await this.player.seek(0);
      } else {
        await this.onBack();
      }
    },
    onPause: async function () {
      this.player.pause();
      this.state = "paused";
    },
    handleShowSongTitle: async function () {
      this.showSongTitle = true;
    },
    handleHideSongTitle: async function () {
      this.showSongTitle = false;
    },
    trackTime: async function () {
      this.percentageSong =
        100 * (this.player.currentTime() / this.currentTrack.duration);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: inherit;
  text-decoration: none;
}
.templatewrapper {
  padding: 20px;
}
.marqueewrapper {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  font-size: 22px;
  /* text-align: center; */
  display: flex;
  flex-direction: column;
}
.marqueeinner {
  border-radius: 15px;
  padding: 5px;
  width: max-content;
  /* margin: auto; */
  text-align: center;
  /* background-color: rgba(255, 255, 255, 0.05); */
  transition: background-color 200ms ease;
  display: flex;
  align-items: center;
}
.marqueeinner:hover {
  background-color: rgba(255, 255, 255, 0.07);
  color: white;
}
.marqueeouter {
  padding: 5px;
}
.tracktitle {
  margin: 0;
  margin-left: 10px;
}
.playerwrapper {
  padding: 5px;
  margin: auto;
  display: flex;
  max-width: 180px;
  width: max-content;
  justify-content: center;
}
.buttonwrapper {
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}
.buttonwrapper:hover {
  opacity: 0.7;
  transition: opacity 200ms ease;
}
.buttonwrapper:active {
  transform: scale(0.95);
}
.timetrackerdiv {
  height: 3px;
  background-color: #fff;
}
.songlink {
  display: flex;
  justify-content: center;
  align-items: center;
  width: max-content;
  transition: opacity 200ms ease;
  margin: 10px;
  margin-left: 0px;
  /* font-style: italic; */
}
.songlink:hover {
  opacity: 0.7;
  text-decoration: underline;
}
</style>
