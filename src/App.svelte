<script lang="ts">
  export const name: string = "Overbeek Audio";

  // import our Audio element and file io requirements
  import AudioBox from "./AudioBox.svelte";
  import { save, open } from "@tauri-apps/api/dialog";
  import { readTextFile, writeFile } from "@tauri-apps/api/fs";

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
    <input bind:value={currentID} placeholder="Input Video ID" />
    <input bind:value={currentNickname} placeholder="Input Video Nickname" />
    <button on:click={onAdd}>Add Video</button>
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
