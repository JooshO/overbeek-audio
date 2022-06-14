<script lang="ts">
  import { onMount } from "svelte";

  // @ts-ignore
  import YoutubePlayer from "youtube-player";

  export let id: string;
  export let nickname: string;
  export let onDelete: (id: String, nickname: String) => void;

  const PlayerState = {
    UNSTARTED: -1,
    ENDED: 0,
    PLAYING: 1,
    PAUSED: 2,
    BUFFERING: 3,
    CUED: 5,
  };

  let playerElem: HTMLDivElement;
  let className;
  let isPlaying: boolean = false;
  let player;

  onMount(() => createPlayer());

  function createPlayer() {
    player = YoutubePlayer(playerElem, {
      height: "0",
      width: "0",
      videoId: id,
      playerVars: {
        autoplay: 0,
        loop: 1,
        playlist: id, // this is necesary for looping a single video properly
      },
      // host: "https://www.youtube-nocookie.com",
      showInfo: false,
      origin: "https://",
    });

    player.on("stateChange", onPlayerStateChange);

    // Tear down player when done
    return () => player.destroy();
  }

  function onDeleteHelper() {
    onDelete(id, nickname);
  }

  function onPlayerStateChange(event) {
    console.log("Current state: " + event.data);
    switch (event.data) {
      case PlayerState.ENDED:
        isPlaying = false;
        break;
      case PlayerState.PLAYING:
        isPlaying = true;
        break;
      case PlayerState.PAUSED:
        isPlaying = false;
        break;
      case PlayerState.CUED:
        isPlaying = false;
        break;
      default:
    }
  }

  function onPausePlay() {
    console.log("Am I playing? " + isPlaying);
    if (!isPlaying) {
      player.playVideo().then(function () {
        // isPlaying = !isPlaying;
      });
    } else {
      player.pauseVideo().then(function () {
        // isPlaying = !isPlaying;
      });
    }
  }

  function resetVideo() {
    console.log("RESET");
    player.seekTo(0, true);
    player.pauseVideo().then(function () {});
  }
</script>

<div class="box">
  <h1>{nickname}</h1>
  <div id="{nickname}-player" bind:this={playerElem} />
  <button on:click={onPausePlay}>
    {isPlaying ? "Pause" : "Play"}
  </button>
  <button on:click={resetVideo}> Reset </button>
  <button on:click={onDeleteHelper}> Delete </button>
</div>

<style>
  .box {
    width: 300px;
    border: 1px solid #aaa;
    border-radius: 2px;
    box-shadow: 2px 2px 8px rgba(0, 0, 0, 0.1);
    padding: 1em;
    margin: 0 0 1em 0;
  }
</style>
