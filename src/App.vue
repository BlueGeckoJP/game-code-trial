<script setup lang="ts">
import { onMounted, ref, watch } from "vue";
import { register } from "@tauri-apps/plugin-global-shortcut";
import { open, save } from "@tauri-apps/plugin-dialog";
import { readTextFile, writeTextFile } from "@tauri-apps/plugin-fs";
import hljs from "highlight.js";
import "highlight.js/styles/atom-one-dark.css";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import { faM } from "@fortawesome/free-solid-svg-icons";

const editorTextarea = ref<HTMLTextAreaElement | null>(null);
const lineNumbersEl = ref<HTMLDivElement | null>(null);
const editorHighlight = ref<HTMLPreElement | null>(null);
const code = ref("");
const lineNumbers = ref<number[]>([1]);
const language = ref("");
const isManualLanguage = ref(false);

let isOpeningDialog = false;

function getLineNumbers() {
  const lines = code.value.split("\n");
  if (lines.length == 0) return [1];
  return Array.from({ length: lines.length }, (_, i) => i + 1);
}

function keyHandler(event: KeyboardEvent) {
  switch (event.key) {
    case "Tab":
      event.preventDefault();
      const start = editorTextarea.value!.selectionStart;
      const end = editorTextarea.value!.selectionEnd;
      code.value =
        code.value.substring(0, start) + "  " + code.value.substring(end);
      editorTextarea.value!.selectionStart = start + 2;
      editorTextarea.value!.selectionEnd = start + 2;
      break;
  }
}

function onChangeLanguage() {
  isManualLanguage.value = true;
  console.log(isManualLanguage.value);
}

register("CommandOrControl+O", async () => {
  if (!isOpeningDialog) {
    isOpeningDialog = true;
    const filename = await open({ multiple: false, directory: false });
    if (filename) {
      const file = await readTextFile(filename);
      editorTextarea.value!!.value = file;
      code.value = file;
    }
    isOpeningDialog = false;
  }
});

register("CommandOrControl+S", async () => {
  if (!isOpeningDialog) {
    isOpeningDialog = true;
    const filename = await save();
    if (filename) {
      await writeTextFile(filename, code.value);
    }
    isOpeningDialog = false;
  }
});

onMounted(() => {
  if (editorTextarea.value && lineNumbersEl.value && editorHighlight.value) {
    editorTextarea.value.addEventListener("scroll", () => {
      lineNumbersEl.value!.scrollTop = editorTextarea.value!.scrollTop;
      editorHighlight.value!.scrollTop = editorTextarea.value!.scrollTop;
      editorHighlight.value!.scrollLeft = editorTextarea.value!.scrollLeft;
    });
  }
});

watch(code, (newValue) => {
  lineNumbers.value = getLineNumbers();
  setTimeout(() => {
    if (isManualLanguage.value) {
      const highlightedCode = hljs.highlightAuto(newValue, [language.value]);
      editorHighlight.value!!.innerHTML = highlightedCode.value;
    } else {
      const highlightedCode = hljs.highlightAuto(newValue);
      editorHighlight.value!!.innerHTML = highlightedCode.value;
      language.value = highlightedCode.language as string;
    }
  }, 0);
});
</script>

<template>
  <div class="top-container">
    <div class="editor-container">
      <div class="line-numbers" ref="lineNumbersEl">
        <div class="line-number" v-for="num in lineNumbers" :key="num">
          {{ num }}
        </div>
      </div>
      <div class="editor">
        <textarea
          class="editor-textarea"
          spellcheck="false"
          v-model="code"
          ref="editorTextarea"
          @keydown="keyHandler($event)"
        ></textarea>
        <pre
          ref="editorHighlight"
          class="editor-highlight"
        ><code class="language-javascript" v-html="code"></code></pre>
      </div>
    </div>
    <div class="status-bar">
      <div class="language-selector-container">
        <select
          v-model="language"
          @change="onChangeLanguage"
          class="language-selector"
        >
          <option v-for="lang in hljs.listLanguages()">
            {{ lang }}
          </option>
        </select>
        <FontAwesomeIcon v-if="isManualLanguage" :icon="faM"></FontAwesomeIcon>
      </div>
    </div>
  </div>
</template>

<style>
@import url("https://fonts.googleapis.com/css2?family=Fira+Code:wght@300..700");

* {
  box-sizing: border-box;
  margin: 0;
  overflow: hidden;

  font-family: "Fira Code", serif;
  font-optical-sizing: auto;
  font-weight: normal;
  font-style: normal;
}

:root {
  --status-bar-height: 30px;
}

html body {
  margin: 0;
}

body {
  background-color: #d8c4b6;
}

.top-container {
  width: 100vw;
  height: 100vh;
}

.editor-container {
  display: flex;
  width: 100%;
  height: calc(100% - var(--status-bar-height));
}

.editor {
  flex-grow: 1;
  position: relative;
}

.editor-textarea,
.editor-highlight {
  position: absolute;
  width: 100%;
  height: 100%;
  margin: 0;
  top: 0;
  left: 0;
  font-size: 14px;
  line-height: 1.5;
  white-space: pre;
  overflow: auto;
  border-left: 1px solid black;
}

.editor-textarea {
  outline: none;
  color: transparent;
  background: transparent;
  caret-color: #000;
  resize: none;
  z-index: 1;
}

.editor-highlight {
  pointer-events: none;
  background: #f5efe7;
  white-space: pre-wrap;
  word-wrap: break-word;
  z-index: 0;
}

.line-numbers {
  padding: 2px 0;
}

.line-number {
  margin-right: 8px;
  font-size: 14px;
  text-align: right;
  min-width: 40px;
  user-select: none;
  line-height: 1.5;
}

.status-bar {
  position: sticky;
  bottom: 0;
  width: 100%;
  height: var(--status-bar-height);
  background-color: #d8c4b6;
  border-top: 1px solid black;
  display: flex;
  align-items: center;
}

.language-selector-container {
  width: 200px;
  display: flex;
}

.language-selector {
  appearance: none;
  border: none;
  background-color: transparent;
  margin-left: 8px;
  margin-right: 8px;
  line-height: 1.5;
  padding: 0 4px;
  border-radius: 0;
  width: min-content;
}
</style>
