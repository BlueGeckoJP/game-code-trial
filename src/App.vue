<script setup lang="ts">
import { onMounted, ref, watch } from "vue";
import { register } from "@tauri-apps/plugin-global-shortcut";
import { open, save } from "@tauri-apps/plugin-dialog";
import { readTextFile, writeTextFile } from "@tauri-apps/plugin-fs";
import hljs from "highlight.js";
import "highlight.js/styles/atom-one-dark.css";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import { faCoins, faM } from "@fortawesome/free-solid-svg-icons";

const editorTextarea = ref<HTMLTextAreaElement | null>(null);
const lineNumbersEl = ref<HTMLDivElement | null>(null);
const editorHighlight = ref<HTMLPreElement | null>(null);
const code = ref("");
const lineNumbers = ref<number[]>([1]);
const language = ref("");
const isManualLanguage = ref(false);
const coins = ref(0);

let isOpeningDialog = false;
let oldCode = "";

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
}

function keywordDetection(oldCode: string, newCode: string) {
  const oldDoc = new DOMParser().parseFromString(oldCode, "text/html");
  const newDoc = new DOMParser().parseFromString(newCode, "text/html");

  const newHighlightElements: string[] = [];
  const newKeywordElements = newDoc.getElementsByClassName("hljs-keyword");

  Array.from(newKeywordElements).forEach((newElement) => {
    const getElementPath = (element: Element): string => {
      const path: string[] = [];
      let current: Element | null = element;

      while (current && current.parentElement) {
        const index = Array.from(current.parentElement.children).indexOf(
          current
        );
        path.unshift(`${current.tagName}:nth-child(${index + 1})`);
        current = current.parentElement;
      }

      return path.join(" > ");
    };

    const elementPath = getElementPath(newElement);
    const oldElement = oldDoc.querySelector(elementPath);

    if (!oldElement || !oldElement.classList.contains("hljs-keyword")) {
      newHighlightElements.push(newElement.innerHTML);
    }
  });

  return newHighlightElements;
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
      const newKeyword = keywordDetection(oldCode, highlightedCode.value);
      if (newKeyword.length != 0) {
        coins.value += newKeyword.length * 10;
      }
      editorHighlight.value!!.innerHTML = highlightedCode.value;
      oldCode = highlightedCode.value;
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
        <FontAwesomeIcon v-if="isManualLanguage" :icon="faM" class="fa-m-manual-lang"></FontAwesomeIcon>
      </div>
      <div class="points-container">
        <FontAwesomeIcon :icon="faCoins"></FontAwesomeIcon>
        <span v-text="coins" class="span-coins"></span>
      </div>
    </div>
    <div class="notifications-container-fixed"></div>
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
}

.language-selector-container {
  width: 200px;
  display: flex;
  align-items: center;
  border-right: 1px solid #000;
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
  flex-grow: 1;
}

.fa-m-manual-lang {
  margin-right: 8px;
  width: 15px;
  height: 15px;
}

.points-container {
  width: 200px;
  border-left: 1px solid #000;
  margin-left: auto;
  padding-left: 8px;
  padding-right: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.span-coins {
  margin-left: 4px;
}

.notifications-container-fixed {
  position: fixed;
  width: 200px;
  height: 100px;
  bottom: calc(40px + var(--status-bar-height));
  right: 40px;
  border: 1px solid black;
}
</style>
