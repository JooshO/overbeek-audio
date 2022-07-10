<script lang="ts">
  import { onMount } from "svelte";

  // @ts-ignore
  import YoutubePlayer from "youtube-player";

  // necesary info - video id, nickname, delete callback
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
  let isPlaying: boolean = false;

  // this type is awful and does not add clarity so I'm breaking convention and leaving it
  // like this
  let player;

  onMount(() => createPlayer());

  function createPlayer() {
    player = YoutubePlayer(playerElem, {
      // make the video invisable
      height: "0",
      width: "0",
      videoId: id,
      playerVars: {
        autoplay: 0,
        loop: 1,
        playlist: id, // this is necesary for looping a single video properly
      },
      showInfo: false,
      origin: "https://",
    });

    player.on("stateChange", onPlayerStateChange);

    // Tear down player when done
    return () => player.destroy();
  }

  /**
   * Calls our callback for us
   */
  function onDeleteHelper() {
    onDelete(id, nickname);
  }

  /**
   * Sets whether we thing we are playing or not
   * @param event
   */
  function onPlayerStateChange(event) {
    switch (event.data) {
      case PlayerState.ENDED:
      case PlayerState.PAUSED:
      case PlayerState.CUED:
        isPlaying = false;
        break;
      case PlayerState.PLAYING:
        isPlaying = true;
        break;
      default:
    }
  }

  /**
   * Toggles whether we are paused or playing
   */
  function onPausePlay() {
    if (!isPlaying) {
      player.playVideo().then(function () {
        // Leaving space here in case I need to handle something
      });
    } else {
      player.pauseVideo().then(function () {
        // Leaving space here in case I need to handle something
      });
    }
  }

  /**
   * Seeks to the start of the video then pauses it
   */
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
    margin: 0.5em 0.5em 0.5em 0.5em;
  }
</style>
