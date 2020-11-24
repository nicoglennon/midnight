<template>
  <div
    class="templatewrapper"
    :style="{
      backgroundColor: currentPlaylistName
        ? playlistData[currentPlaylistName].color
        : 'unset',
    }"
  >
    <div class="topleftlogo">
      <i class="nes-icon heart is-medium"></i>
      <i class="nes-icon heart is-medium"></i>
      <i class="nes-icon heart is-medium is-half"></i>
    </div>
    <div class="madebytext">
      <div class="nes-balloon from-right nes-pointer" v-on:click="openModal">
        <p>?</p>
      </div>
    </div>
    <div class="playlistselectwrapper">
      <div class="nes-select playlistselectinner">
        <select
          required
          v-model="currentPlaylistName"
          class="dropdown"
          @change="playlistSelected"
        >
          <option value="null" disabled selected>vibe check...</option>
          <option v-for="opt in playlistDataKeys" :key="opt" :value="opt">{{
            playlistData[opt].label
          }}</option>
        </select>
      </div>
    </div>
    <div class="marqueewrapper">
      <div class="marqueeouter" v-if="firstPlay">
        <div class="marqueeinner">
          <div class="playerwrapper">
            <div v-on:click="onClickBack" class="buttonwrapper">
              <button type="button" class="nes-btn">&lt;</button>
            </div>
            <div
              v-on:click="onPause"
              v-if="state === 'playing'"
              class="buttonwrapper"
            >
              <button type="button" class="nes-btn playpause">Pause</button>
            </div>
            <div v-else v-on:click="onPlay" class="buttonwrapper">
              <button type="button" class="nes-btn playpause">Play</button>
            </div>
            <div v-on:click="onNext" class="buttonwrapper">
              <button type="button" class="nes-btn">&gt;</button>
            </div>
          </div>
          <div class="songlinkwrapper">
            <a
              :href="currentTrack.url"
              target="_blank"
              class="songlink"
              v-if="currentTrack"
            >
              <button type="button" class="nes-btn">
                {{ currentTrack.artist + " - " + currentTrack.title }}
              </button>
            </a>
          </div>
        </div>
      </div>
    </div>
    <div class="modalwrapper" v-if="modalOpen">
      <div class="modalcontent">
        <div class="nes-container with-title is-dark">
          <p class="title">Message</p>
          <p>nes.fm was made by <a href="https://nico.gl">Nico</a>.</p>
          <p>
            To submit a playlist, you must create it on soundcloud and email me
            the link at
            <a href="mailto:hi@nico.gl">hi@nico.gl</a>.
          </p>
          <p>Enjoy the rest of your day on the internet.</p>
          <button class="nes-btn" v-on:click="closeModal">
            Back
          </button>
          <a href="https://nico.gl">
            <button class="nes-btn is-primary">
              @nicoglennon
            </button>
          </a>
          <br />
          <br />
          <p class="copyright">© 2020 nes.fm</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
/*global SC*/
import playlistData from "../utils/playlistData";
const modulo = (n, m) => {
  var remain = n % m;
  return Math.floor(remain >= 0 ? remain : remain + m);
};

export default {
  name: "Main",
  props: {
    msg: String,
  },
  data: () => ({
    currentPlaylistName: null,
    currentPlaylistId: null,
    currentTrackId: null,
    trackIds: [],
    tracks: [],
    trackIndex: 0,
    currentTrack: null,
    loaded: false,
    state: "paused",
    firstPlay: false,
    playlistData,
    playlistDataKeys: Object.keys(playlistData),
    modalOpen: false,
  }),
  mounted: async function() {
    //
  },
  methods: {
    pullSong: async function() {
      try {
        this.player = await SC.stream(`/tracks/${this.currentTrackId}`);
        // console.log("pulled successfully!");
        this.player.on("finish", this.onNext);
      } catch (e) {
        // console.error(
        //   "Playback rejected. Try calling play() from a user interaction.",
        //   e
        // );
      }
    },
    onPlay: async function() {
      await this.player.play();
      this.state = "playing";
      this.currentTrack = this.tracks[this.trackIndex];
      if (!this.firstPlay) {
        this.firstPlay = true;
      }
    },
    onNext: async function() {
      await this.player.pause();
      this.trackIndex = (this.trackIndex + 1) % this.trackIds.length;
      this.currentTrackId = this.trackIds[this.trackIndex];
      await this.pullSong();
      await this.onPlay();
    },
    onBack: async function() {
      await this.player.pause();
      this.trackIndex = modulo(this.trackIndex - 1, this.trackIds.length);
      this.currentTrackId = this.trackIds[this.trackIndex];
      await this.pullSong();
      await this.onPlay();
    },
    onClickBack: async function() {
      const currentTime = await this.player.currentTime();
      if (currentTime > 2000) {
        await this.player.seek(0);
      } else {
        await this.onBack();
      }
    },
    onPause: async function() {
      this.player.pause();
      this.state = "paused";
    },
    playlistSelected: async function(e) {
      if (this.player) {
        await this.player.pause();
      }
      if (this.currentPlaylistId !== playlistData[e.target.value].id) {
        this.currentPlaylistId = playlistData[e.target.value].id;
        await this.mountPlaylist();
      }
    },
    mountPlaylist: async function() {
      const res = await SC.get(`/playlists/${this.currentPlaylistId}`);
      this.tracks = res.tracks.map((track) => ({
        id: track.id,
        title: track.title,
        artist: track.user.username,
        url: track.permalink_url,
        duration: track.duration,
      }));
      this.trackIds = res.tracks.map((track) => track.id);
      this.trackIndex = 0;
      this.currentTrackId = this.trackIds[this.trackIndex];
      await this.pullSong();
      await this.onPlay();
    },
    openModal: function() {
      this.modalOpen = true;
    },
    closeModal: function() {
      this.modalOpen = false;
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
p {
  background-color: inherit;
}
.madebytext {
  position: fixed;
  font-size: 16px;
  bottom: 0;
  right: 0;
  padding: 10px;
  z-index: 5;
}
.templatewrapper {
}
.marqueewrapper {
  position: fixed;
  width: max-content;
  bottom: 0;
  left: 0;
  right: 0;
  font-size: 14px;
  display: flex;
  flex-direction: column;
  z-index: 4;
}
.marqueeinner {
  padding: 2px;
  width: max-content;
  text-align: center;
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  flex-shrink: 1;
}
.marqueeouter {
  padding: 15px;
}
.tracktitle {
  margin: 0;
  margin-left: 10px;
}
.playerwrapper {
  padding: 0px;
  display: flex;
  width: max-content;
  justify-content: center;
}
.buttonwrapper {
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}
.songlink {
  display: flex;
  justify-content: center;
  align-items: center;
  width: max-content;
  transition: opacity 200ms ease;
}
.songlinkwrapper {
  flex: 1;
}
.playpause {
  min-width: 100px;
}
.playlistselectwrapper {
  position: relative;
  margin: auto;
  max-width: 480px;
  height: 100vh;
}
.playlistselectinner {
  margin: 0;
  position: absolute;
  top: 50%;
  -ms-transform: translateY(-50%);
  transform: translateY(-50%);
}
.dropdown {
  font-size: 20px;
}
.message-left {
  display: flex;
  text-align: left;
}
.message-right {
  display: flex;
  justify-content: flex-end;
  text-align: left;
}
.topleftlogo {
  position: absolute;
  top: 15px;
  left: 15px;
  z-index: 5;
}

.modalwrapper {
  position: fixed; /* Stay in place */
  z-index: 1; /* Sit on top */
  left: 0;
  top: 0;
  width: 100%; /* Full width */
  height: 100%; /* Full height */
  overflow: auto; /* Enable scroll if needed */
  background-color: #212529; /* Fallback color */
}

/* Modal Content/Box */
.modalcontent {
  margin: 15% auto; /* 15% from the top and centered */
  padding: 20px;
  max-width: 600px; /* Could be more or less, depending on screen size */
}
.copyright {
  text-align: right;
}
</style>
