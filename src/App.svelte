<script>
  import { onMount } from "svelte";
  import { createWorker } from "tesseract.js";

  let worker;
  let files;
  let preview;
  let loading;
  let result;
  let label;
  onMount(async () => {
    worker = await createWorker("ind", null, {});
  });

  function convert(files) {
    if (!files) return;
    const file = files[0];
    if(!file)return;
    clear();
    loading.style.display = "flex";
    preview.style.display = "block";
    preview.src = URL.createObjectURL(file);
    label.textContent = `Change file (${file.name})`;
    const reader = new FileReader();
    reader.onloadend = () => {
      const imageDataUri = reader.result;
      worker.recognize(imageDataUri).then(({ data: { text } }) => {
        result = text;
        loading.style.display = "none";
      });
    };
    reader.readAsDataURL(file);
  }

  function clear() {
    result = "";
    preview.src = "";
  }

  $: convert(files);
</script>

<div id="loading" bind:this={loading}>
  <p>loading...</p>
</div>
<main>
  <div id="input">
    <input id="pic" type="file" bind:files={files} accept=".jpg,.jpeg,.png" />
    <label for="pic" bind:this={label}>Select Picture ...</label>
    <img bind:this={preview} alt="preview">
  </div>
  <div id="result">
    <button type="button" on:click={()=>{
      navigator.clipboard.writeText(result);
    }}>Copy</button>
    <textarea disabled placeholder="Result..." bind:value={result}></textarea>
  </div>
</main>

<style>
  main {
    width: 90vw;
    min-height: 100%;
    margin: 2rem;
    display: flex;
    align-items: stretch;
  }
  div#loading {
    display: none;
    width: 100vw;
    height: 100vh;
    position: fixed;
    top: 0;
    left: 0;
    /* display: flex; */
    justify-content: center;
    align-items: center;
    background-color: #00000080;
  }
  div#input {
    padding: 1rem;
    margin-right: 1rem;
    display: flex;
    flex-direction: column;
    width: 50%;
  }
  div#input > img {
    margin-top: 1rem;
    max-width: 100%;
    object-fit: contain;
    display: none;
  }
  div#result {
    width: 50%;
    padding: 1rem;
    display: flex;
    flex-direction: column;
    gap: 1rem;
    border: #aaa 1px solid;
  }
  div#result > button {
    width: 100%;
    padding: 0.5rem;
    cursor: copy;
  }
  div#result > textarea {
    width: 100%;
    resize: none;
    border: none;
    background-color: #1b1b1b;
    flex-grow: 1;
  }

  input[type="file"] {
    display: none;
  }
  input[type="file"] + label {
    background-color: #aaa;
    color: #1b1b1b;
    padding: 1rem;
    cursor: pointer;
    text-align: center;
  }
</style>
