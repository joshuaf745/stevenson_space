<template>
  <div class="live">
    <div class="background-block" />

    <div class="main">
      <home-link class="home-link" :invert="false" />
      <div class="inner-background">
        <h1 class="title"><font-awesome-icon class="icon" :icon="icons.faRadio" /> Meet Jukebox</h1>
        <p class="subtitle">Discover original music from Stevenson students.</p>
        <div class="now-playing">
          <img
            class="album-art"
            :src="
              songs[currentSongIndex].album
                ? `https://music-backend.stevenson-space.workers.dev/${songs[currentSongIndex].album}`
                : defaultAlbum
            "
          />
          <div class="song-info">
            <p class="song-title">{{ songs[currentSongIndex].name }}</p>
            <p class="song-artist">
              {{ songs[currentSongIndex].artist ? songs[currentSongIndex].artist : "Anonymous" }},
              {{ songs[currentSongIndex].year }}
            </p>
          </div>
          <div class="controls">
            <button @click="previousSong()" class="control-button">
              <font-awesome-icon :icon="icons.faStepBackward" />
            </button>
            <button @click="playing ? pause() : play()" class="control-button play-button">
              <font-awesome-icon :icon="playing ? icons.faPause : icons.faPlay" />
            </button>
            <button
              @click="
                currentSongIndex + 1 < songs.length ? setSongOf(currentSongIndex + 1) : setSongOf(0)
              "
              class="control-button"
            >
              <font-awesome-icon :icon="icons.faStepForward" />
            </button>
          </div>
          <div class="progress">
            <p class="progress-text">{{ formatTime(currentTimePlaying) }}</p>
            <div class="progress-bar">
              <div class="progress-bar-fill" :style="`width: ${percentFinished}%;`" />
              <div class="progress-bar-handle" :style="`left: ${percentFinished}%;`" />
            </div>
            <p class="progress-text">{{ formatTime(totalSongTime) }}</p>
          </div>
        </div>
        <div class="song-list" v-for="(song, index) in songs" :key="index">
          <button @click="setSongOf(index)" class="play-button">
            <font-awesome-icon :icon="icons.faPlay" />
          </button>
          <p @click="setSongOf(index)" class="song-title">{{ song.name }}</p>
          <p class="song-bullet">&bull;</p>
          <p class="song-artist">{{ song.artist ? song.artist : "Anonymous" }}, {{ song.year }}</p>
        </div>
        <div class="footer-group">
          <p class="footer">
            Have a song you'd like to publish on Jukebox?
            <a href="https://docs.google.com/forms/d/e/1FAIpQLSd33UAW_OQjYCfsAJsw_DQPhMAv2Td42MOV9GGZTpEbVgCdig/viewform?usp=dialog9" target="_blank" rel="noreferrer"
              >Submit it here <font-awesome-icon :icon="icons.faArrowUpRightFromSquare"
            /></a>
          </p>
          <p class="footer">Music is student-created and not endorsed by Stevenson High School</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import HomeLink from "@/components/HomeLink.vue";
import songs from "@/data/music.json";
import {
  faRadio,
  faPlay,
  faStepBackward,
  faStepForward,
  faPause,
  faArrowUpRightFromSquare,
} from "@fortawesome/free-solid-svg-icons";
import { Howl } from "howler";
import defaultAlbum from "@/assets/default-music-album.png";

export default {
  components: { HomeLink },
  data() {
    return {
      icons: {
        faRadio,
        faPlay,
        faStepBackward,
        faStepForward,
        faPause,
        faArrowUpRightFromSquare,
      },
      currentSongIndex: 0,
      playing: false,
      percentFinished: 0,
      currentTimePlaying: 0,
      totalSongTime: 0,
      defaultAlbum,
      songs,
    };
  },
  mounted() {
    const fileFormatRegex = /\.[0-9a-z]+$/i;
    for (const song of songs) {
      song.howler = new Howl({
        src: [`https://music-backend.stevenson-space.workers.dev/${song.file}`], // Cloudflare R2 instance URL
        format: [".mp3"],
        html5: true,
        volume: 0.5,
        onend: () => {
          if (this.currentSongIndex + 1 <= songs.length) {
            this.setSongOf(this.currentSongIndex + 1);
            this.play();
          }
        },
      });
      song.id = 0;
    }
    document.addEventListener("keyup", (e) => {
      if (e.key === " ") {
        e.preventDefault();
        // eslint-disable-next-line no-unused-expressions
        this.playing ? this.pause() : this.play();
      }
    });
  },
  async beforeRouteLeave(to, from, next) {
    // stop current song when leaving page
    await this.songs[this.currentSongIndex].howler.stop();
    next();
  },
  methods: {
    previousSong() {
      // 1 backwards press sets song to beginning, 2 presses skips to previous song
      if (this.songs[this.currentSongIndex].howler.seek() >= 10) {
        this.songs[this.currentSongIndex].howler.seek(0);
      } else if (this.currentSongIndex > 0) {
        this.setSongOf(this.currentSongIndex - 1);
      }
    },
    setSongOf(index) {
      this.songs[this.currentSongIndex].howler.stop();
      this.currentSongIndex = index;
      this.updateProgressBar();
      if (this.playing) {
        this.play();
      } else {
        this.playing = false;
      }
    },
    play() {
      const currentSong = this.songs[this.currentSongIndex];
      if (currentSong.id === 0) {
        this.songs[this.currentSongIndex].id = currentSong.howler.play();
      } else currentSong.howler.play(currentSong.id);
      this.songs[this.currentSongIndex].progressBar = setInterval(
        () => this.updateProgressBar(),
        100
      );
      currentSong.howler.fade(0, 0.5, 1);
      this.playing = true;
    },
    async pause() {
      this.playing = false;
      clearInterval(this.songs[this.currentSongIndex].progressBar);
      // this.currentSong.howler.fade(0.5, 0, 200, this.songId);
      // await this.sleep(200);
      this.songs[this.currentSongIndex].howler.pause();
    },
    updateProgressBar() {
      const seek = this.songs[this.currentSongIndex].howler.seek();
      const duration = this.songs[this.currentSongIndex].howler.duration();
      this.percentFinished = (seek / duration) * 100;
      this.currentTimePlaying = seek;
      this.totalSongTime = duration;
    },
    formatTime(seconds) {
      const minutes = Math.floor(seconds / 60);
      const secondsLeft = (seconds - minutes * 60).toFixed(0); // ensure seconds is a whole number (0 decimal places)
      return `${minutes}:${secondsLeft < 10 ? `0${secondsLeft}` : secondsLeft}`;
    },
    sleep(ms) {
      // eslint-disable-next-line no-promise-executor-return
      return new Promise((resolve) => setTimeout(resolve, ms));
    },
  },
};
</script>

<style lang="sass" scoped>
@import '@/styles/style.sass'

$small: 300px
$medium: 900px

.live
  .background-block
    background-color: var(--headerBackgroundColor)
    height: 250px
    width: 100%
    left: 0
    position: absolute

  .main
    width: 80%
    max-width: $content-width + 80px // hardcoded value :( to make standard schedule periods all fit in one line (on my display)
    margin: auto
    position: relative
    +mobile
      width: 90%

    .inner-background
      +shadow
      background-color: var(--background)
      position: relative
      border-radius: 20px
      top: 100px
      overflow: hidden
      margin-bottom: 150px
      +mobile
        top: 75px

    .title
      text-align: left
      font-size: 2.5em
      margin: 15px 0 0 15px

    .icon
      color: var(--headerBackgroundColor)

    .subtitle
      text-align: left
      font-size: 1.5em
      margin: 10px 0 0 15px

    .home-link
      position: absolute
      right: 0
      top: 30px
      +mobile
        top: 20px

    .now-playing
      margin-top: 25px
      .progress
        display: flex
        .progress-text
          margin-left: auto
          margin-right: auto
          font-size: 0.8em
          font-weight: bold
          color: var(--textColor)
        .progress-bar
          width: 80%
          display: flex
          margin: 15px auto auto auto
          text-align: center
          background-color: #BCBCBC
          border-radius: 10px
        .progress-bar-fill
          height: 15px
          width: 20%
          background-color: var(--headerBackgroundColor)
          // transition: width 0.05s ease-in-out
          border-radius: 10px
        .progress-bar-handle
          background-color: var(--headerBackgroundColor)
          border-radius: 9999px

      .album-art
        display: block
        margin-top: 0px
        width: 300px
        height: 300px
        object-fit: cover
        margin-left: auto
        margin-right: auto
        border-radius: 10px

      .song-info
        // display: flex
        text-align: center
        margin: 0 0 0 0
        .song-title
          font-size: 1.5em
          font-weight: bold
          margin-bottom: 5px
        .song-artist
          font-size: 1.5em
          margin-top: 5px
          font-weight: bold
          color: var(--secondary)

      .controls
        display: flex
        justify-content: center

        .control-button
          background-color: transparent
          border: none
          margin: 0 10px 0 10px
          font-size: 2em
          color: var(--secondary)
          &:hover
            color: var(--headerBackgroundColor)
            cursor: pointer
        .play-button
          background-color: var(--headerBackgroundColor)
          border-radius: 50%
          padding: 10px 20px 9px 21px
          margin: 0
          transition: 0.25s
          font-size: 2.5em
          color: white
          &:hover
            color: white
            // font-size: 2.6em
            cursor: pointer
            box-shadow: 0 0 10px 0 var(--tertiary)

    .song-list
      // TODO: reactivity
      // @media screen and (max-width: $small) {
        // display: block
      // }
      // @media screen and (max-width: $medium) {
      display: flex
      justify-content: center
      // }
      .play-button
        border: 0
        background-color: transparent
        margin: 0
        transition: 0.15s
        font-size: 1em
        margin: 3px 5px 0 0
        color: var(--headerBackgroundColor)
        &:hover
          font-size: 1.2em
          cursor: pointer
      .song-title
        font-size: 1em
        font-weight: bold
        &:hover
          cursor: pointer
          text-decoration: underline
      .song-bullet
        font-weight: bold
        font-size: 1em
        margin: auto 5px auto 5px
        color: var(--secondary)
      .song-artist
        font-size: 1em
        font-weight: bold
        color: var(--secondary)

    .footer-group
      margin-left: 20px
      padding-right: 20px
    .footer
      text-align: center
      font-weight: bold
      font-size: 0.8em
      margin: 10px 0 20px 0
      color: var(--secondary)
      a
        color: var(--secondary)
        text-decoration: none
        &:hover
          text-decoration: underline
</style>
