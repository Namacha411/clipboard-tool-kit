<script lang="ts">
  import { core } from "@tauri-apps/api";
  import {
    clear,
    writeText,
    readText,
    readImage,
  } from "@tauri-apps/plugin-clipboard-manager";
  import type { SaveDialogOptions } from "@tauri-apps/plugin-dialog";
  import { save } from "@tauri-apps/plugin-dialog";
  import { writeFile } from "@tauri-apps/plugin-fs";

  let errorMsg = $state("");
  let charCount = $state(0);
  let noSpaceCharCount = $state(0);
  let lineCount = $state(0);
  let wordCount = $state(0);
  let genkouyoushi = $state(0);

  function writeCharClipboard(char: string) {
    errorMsg = "";
    writeText(char)
      .then(null)
      .catch((e) => (errorMsg = e));
  }

  function generateRandomString() {
    const length = 12;
    const lowerCase = "abcdefghijklmnopqrstuvwxyz";
    const upperCase = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    const numbers = "0123456789";
    const symbols = "!@#$%^&*()_+[]{}|;:,.<>?";
    const allChars = lowerCase + upperCase + numbers + symbols;

    let randomString = "";
    for (let i = 0; i < length; i++) {
      const randomIndex = Math.floor(Math.random() * allChars.length);
      randomString += allChars[randomIndex];
    }

    writeText(randomString)
      .then(null)
      .catch((e) => (errorMsg = e));
  }

  function clearClipboard() {
    errorMsg = "";
    clear()
      .then(null)
      .catch((e) => (errorMsg = e));
  }

  function saveClipboardImage() {
    errorMsg = "";
    const filePath = `clipboard_image_${Date.now()}.png`;
    const optionDialog: SaveDialogOptions = {
      title: "Save Clipboard Image",
      filters: [{ name: "Image", extensions: ["png", "jpg"] }],
      defaultPath: filePath,
    };
    save(optionDialog)
      .then((path) => {
        if (path === null) {
          errorMsg = "Save was cancelled";
          return;
        }
        readImage().then((image) => {
          image.rgba().then((u8arr) => {
            writeFile(path, u8arr)
              .then(null)
              .catch((e) => (errorMsg = e));
          });
        });
      })
      .catch((e) => (errorMsg = e));
  }

  function countClipboardText() {
    errorMsg = "";
    readText()
      .then((v) => {
        charCount = v.length;
        lineCount = v.split("\n").length;
        genkouyoushi = Math.ceil(v.length / 400);
        wordCount = (v.match(/\b\w+\b/g) || []).length;
        noSpaceCharCount = v.replaceAll(/\s+/g, "").length;
      })
      .catch((e) => (errorMsg = e));
  }
</script>

<main class="container">
  {#if errorMsg}
    <p class="error">{errorMsg}</p>
  {/if}

  <section class="clipboard-actions">
    <div class="action-group">
      <h2>Clipboard Actions</h2>
      <div class="action-buttons">
        <button onclick={clearClipboard} class="clear-btn"
          >Clear Clipboard</button
        >
      </div>
    </div>

    <div class="action-group">
      <h2>Special Characters</h2>
      <div class="action-buttons">
        <button onclick={() => writeCharClipboard("\u200B")} class="char-btn"
          >Copy ZWSP</button
        >
        <button onclick={() => writeCharClipboard("\u2014")} class="char-btn"
          >Copy EMDash</button
        >
        <button onclick={generateRandomString} class="char-btn"
          >Random String</button
        >
      </div>
    </div>

    <div class="action-group">
      <h2>Clipboard Image</h2>
      <div class="action-buttons">
        <button onclick={saveClipboardImage} class="image-btn"
          >Save Clipboard Image</button
        >
      </div>
    </div>

    <div class="action-group">
      <h2>Clipboard Text</h2>
      <div class="action-buttons">
        <button onclick={countClipboardText} class="text-btn"
          >Count Clipboard Text</button
        >
        <div class="text-results">
          <p>文字数: {charCount}</p>
          <p>空白なし文字数: {noSpaceCharCount}</p>
          <p>単語数: {wordCount}</p>
          <p>行数: {lineCount}</p>
          <p>原稿用紙: {genkouyoushi}</p>
        </div>
      </div>
    </div>
  </section>
</main>

<style>
  .container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    font-family: Arial, sans-serif;
  }

  .error {
    color: red;
    text-align: center;
    font-weight: bold;
    margin-bottom: 20px;
  }

  .clipboard-actions {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 20px;
  }

  .action-group {
    background-color: #f9f9f9;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  }

  h2 {
    margin-bottom: 10px;
    font-size: 1.2rem;
    color: #333;
  }

  .action-buttons {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  button {
    padding: 10px 15px;
    font-size: 1rem;
    color: white;
    background-color: #007bff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.2s;
  }

  button:hover {
    background-color: #0056b3;
  }

  .text-results p {
    margin: 5px 0;
  }

  /* 各ボタンのスタイルを調整 */
  .clear-btn {
    background-color: #dc3545;
  }

  .char-btn {
    background-color: #28a745;
  }

  .image-btn {
    background-color: #17a2b8;
  }

  .text-btn {
    background-color: #ffc107;
  }
</style>
