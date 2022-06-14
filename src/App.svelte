<script lang="ts">
  export const name: string = "Overbeek Audio";
  import { each } from "svelte/internal";

  import AudioBox from "./AudioBox.svelte";
  import { save, open } from "@tauri-apps/api/dialog";
  import { readTextFile, writeFile } from "@tauri-apps/api/fs";

  let currentID: string = "";
  let currentNickname: string = "";
  let audios = []; //d4eRXTWTkAU
  let targetedFile = "";

  function onAdd() {
    audios = [...audios, { id: currentID, nickname: currentNickname }];
    currentID = "";
    currentNickname = "";
  }

  function del(id: String, nickname: String) {
    audios = audios.filter((a) => a.id !== id && a.nickname != nickname);
  }

  function clear() {
    audios = [];
    currentID = "";
    currentNickname = "";
  }

  function saveDialog() {
    let pathToFile = save({
      title: "Select a file to save to",
      filters: [{ extensions: ["oacfg"], name: "Overbeek Audio Config File" }],
    });

    pathToFile.then((value: string) => {
      console.log(JSON.stringify(audios));
      writeFile({ path: value, contents: JSON.stringify(audios) });
    });
  }

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
    <input bind:value={currentID} placeholder="Input Video ID" />
    <input bind:value={currentNickname} placeholder="Input Video Nickname" />
    <button on:click={onAdd}>Add Video</button>
    <button on:click={clear}>Clear Videos</button>
  </div>
  <div class="audio-zone">
    {#each audios as audio}
      <AudioBox nickname={audio.nickname} id={audio.id} onDelete={del} />
    {:else}
      Add some music!
    {/each}
  </div>
  <div class="save-load-zone">
    <button on:click={saveDialog}> Save </button>
    <button on:click={loadDialog}> Load </button>
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
