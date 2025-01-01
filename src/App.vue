<script setup lang="ts">
import { onMounted, ref, watch } from "vue";
import Prism from "prismjs";
import "prismjs/themes/prism.css";

const editorTextarea = ref<HTMLTextAreaElement | null>(null);
const lineNumbersEl = ref<HTMLDivElement | null>(null);
const editorHighlight = ref<HTMLPreElement | null>(null);
const code = ref("");
const lineNumbers = ref<number[]>([1]);

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

onMounted(() => {
  if (editorTextarea.value && lineNumbersEl.value && editorHighlight.value) {
    editorTextarea.value.addEventListener("scroll", () => {
      lineNumbersEl.value!.scrollTop = editorTextarea.value!.scrollTop;
      editorHighlight.value!.scrollTop = editorTextarea.value!.scrollTop;
      editorHighlight.value!.scrollLeft = editorTextarea.value!.scrollLeft;
    });
  }
});

watch(code, () => {
  lineNumbers.value = getLineNumbers();
  setTimeout(() => Prism.highlightAll(), 0);
});
</script>

<template>
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

html body {
  margin: 0;
}

body {
  background-color: #d8c4b6;
}

.editor-container {
  display: flex;
  width: 100vw;
  height: 100vh;
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
  border: 1px solid black;
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

pre[class*="language-"] {
  padding: 0;
  margin: 0;
  font-size: 14px;
  line-height: 1.5;
}

code[class*="language-"] {
  font-size: 14px;
  font-family: "Fira Code", serif;
  font-optical-sizing: auto;
  font-weight: normal;
  font-style: normal;
}
</style>
