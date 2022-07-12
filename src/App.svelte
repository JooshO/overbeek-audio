<script lang="ts">
  export const name: string = "Overbeek Audio";

  // import our Audio element and file io requirements
  import AudioBox from "./AudioBox.svelte";
  import { save, open } from "@tauri-apps/api/dialog";
  import { readTextFile, writeFile } from "@tauri-apps/api/fs";

  import Modal from "./Modal.svelte";

  // using https://github.com/harshkhandeparkar/tauri-settings
  // to use this I need to set "scope": ["**"], in the fs allowList which is kinda rough, I need to see if I can limit that scope
  import { set, get, has } from "tauri-settings";

  // settings schema
  type Schema = {
    isDarkTheme: boolean;
  };

  // show variables for modals
  let showVideoModal: boolean = false;
  let showHelpModal: boolean = false;
  let showSettingsModal: boolean = false;

  // input nickname and ID for adding a video
  let currentID: string = "";
  let currentNickname: string = "";

  // list of audio data
  let audios = [];

  // root style
  var root = document.querySelector(":root");

  // local dark theme tracker
  let darkTheme: boolean = true;
  let darkString: "isDarkTheme";

  // if the settings file does not have dark theme, then add it and set to true by default
  // this is basically upgrade code
  has<Schema>(darkString).then((present) => {
    if (!present) {
      set<Schema>(darkString, true);
    }
  });

  // get current dark theme setting
  get<Schema>(darkString)
    .then((theme) => {
      // this is kinda hacky and will cause the window to flash dark when loading in light theme
      if (!theme) toggleDark();
      darkTheme = theme;
    })
    .catch((reason) => {
      console.log(reason);
    });

  /**
   * Toggles the current theme between light and dark
   */
  function toggleDark() {
    if (darkTheme) {
      // this errors but it works exactly right so idk why this pretends not to work (it's bc TS)
      //@ts-ignore
      root.style.setProperty("--theme-color", "#000000"); //@ts-ignore
      root.style.setProperty("--theme-bg-color", "#FFFFFF"); //@ts-ignore
      root.style.setProperty("--theme-alt-bg", "#f4f4f4"); //@ts-ignore
      root.style.setProperty("--theme-input", "#ffffff"); //@ts-ignore
      root.style.setProperty("--theme-input-text", "#798290"); //@ts-ignore
      root.style.setProperty("--button-hover", "#b4b4b4"); //@ts-ignore
    } else {
      //@ts-ignore
      root.style.setProperty("--theme-color", "#e9ebf0"); //@ts-ignore
      root.style.setProperty("--theme-bg-color", "#303443"); //@ts-ignore
      root.style.setProperty("--theme-alt-bg", "#2a344a"); //@ts-ignore
      root.style.setProperty("--theme-input", "#4e6496"); //@ts-ignore
      root.style.setProperty("--theme-input-text", "#FFFFFF"); //@ts-ignore
      root.style.setProperty("--button-hover", "#1c2539"); //@ts-ignore
    }
    darkTheme = !darkTheme;
    set<Schema>(darkString, darkTheme).then(() => {});
  }

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

        <input
          class="text-input"
          bind:value={currentID}
          placeholder="Input Video ID"
        />
        <input
          class="text-input"
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

    <button on:click={() => (showSettingsModal = true)}>Settings</button>
    {#if showSettingsModal}
      <Modal on:close={() => (showSettingsModal = false)}>
        <h2 slot="header">Settings</h2>
        <button on:click={toggleDark}
          >{darkTheme ? "Light Mode" : "Dark Mode"}</button
        >
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
  :root {
    --theme-color: #e9ebf0;
    --theme-bg-color: #303443;
    --theme-alt-bg: #2a344a;
    --theme-input: #4e6496;
    --theme-input-text: #ffffff;
    --button-hover: #1c2539;
  }

  :global(body) {
    background-color: var(--theme-bg-color);
    color: var(--theme-color);
  }

  main {
    background-color: inherit;
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
  }

  button {
    color: inherit;
    background-color: var(--theme-alt-bg);
  }

  button:hover {
    background-color: var(--button-hover);
  }

  .text-input {
    color: inherit;
    background-color: var(--theme-input);
  }

  .audio-zone {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
  }

  ::placeholder {
    color: var(--theme-input-text);
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
