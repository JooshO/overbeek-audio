<script lang="ts">
  export const name: string = "Overbeek Audio";

  // import our Audio element and file io requirements
  import AudioBox from "./AudioBox.svelte";
  import { save, open } from "@tauri-apps/api/dialog";
  import { readTextFile, writeFile } from "@tauri-apps/api/fs";

  import Modal from "./Modal.svelte";

  let showVideoModal: boolean = false;
  let showHelpModal: boolean = false;
  let currentID: string = "";
  let currentNickname: string = "";
  let audios = [];

  /**
   * Add a new Audio element based on the current values of the id and nickname field
   * then clear those fields.
   */
  function onAdd() {
    // this syntax forces the app to reload what audio boxes are visable
    audios = [...audios, { id: currentID, nickname: currentNickname }];
    currentID = "";
    currentNickname = "";
  }

  /**
   * Predefined callback function for deleting AudioBoxes
   * @param id The video ID
   * @param nickname The nickname of the video
   */
  function del(id: String, nickname: String) {
    // filters the list to only contain the elements that don't match
    audios = audios.filter((a) => a.id !== id && a.nickname != nickname);
  }

  /**
   * Clears all audioboxes
   */
  function clear() {
    audios = [];
    currentID = "";
    currentNickname = "";
  }

  /**
   * Handles saving the file
   */
  function saveDialog() {
    let pathToFile = save({
      title: "Select a file to save to",
      filters: [{ extensions: ["oacfg"], name: "Overbeek Audio Config File" }],
    });

    pathToFile.then((value: string) => {
      writeFile({ path: value, contents: JSON.stringify(audios) });
    });
  }

  /**
   * Handles loading a file
   */
  function loadDialog() {
    let pathToFile = open({
      title: "Select a file to load",
      filters: [{ extensions: ["oacfg"], name: "Overbeek Audio Config File" }],
    });

    pathToFile.then((value: string) => {
      let data = readTextFile(value);

      data.then((value: string) => {
        audios = JSON.parse(value);
      });
    });
  }
</script>

<main>
  <div class="input-zone">
    <button on:click={saveDialog}> Save </button>
    <button on:click={loadDialog}> Load </button>
    <button on:click={() => (showVideoModal = true)}>Add Video</button>
    {#if showVideoModal}
      <Modal on:close={() => (showVideoModal = false)}>
        <h2 slot="header">Add Video</h2>

        <input bind:value={currentID} placeholder="Input Video ID" />
        <input
          bind:value={currentNickname}
          placeholder="Input Video Nickname"
        />
        <button on:click={onAdd}> Load </button>
      </Modal>
    {/if}
    <button on:click={() => (showHelpModal = true)}>?</button>
    {#if showHelpModal}
      <Modal on:close={() => (showHelpModal = false)}>
        <h2 slot="header">Help</h2>
        To use, first click add video to open the add pop-up. Then, simply click
        the share link under a YouTube video and copy the part of the URL highlighted
        here:
        <img src="../youtubeurl.png" alt="YouTube Share URL" />
        Then paste that into the "id" box on the app, add a nickname for the audio,
        and click "Submit"
      </Modal>
    {/if}

    <button on:click={clear}>Clear Videos</button>
  </div>
  <div class="audio-zone">
    <!-- Loads each audio in the audio list as an audio box. This is reloaded on assignment -->
    {#each audios as audio}
      <AudioBox nickname={audio.nickname} id={audio.id} onDelete={del} />
    {:else}
      Add some music!
    {/each}
  </div>
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
  }

  .audio-zone {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
