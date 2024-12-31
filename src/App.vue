<script setup lang="ts">
import { onMounted, ref, watch } from "vue";

const editorContent = ref("");
const lineNumbers = ref<number[]>([1]);

function getLineNumbers() {
  const lines = editorContent.value.split("\n");
  if (lines.length == 0) return [1];
  return Array.from({ length: lines.length }, (_, i) => i + 1);
}

onMounted(() => {
  const editor = document.querySelector(".editor") as HTMLTextAreaElement;
  const lineNumbersEl = document.querySelector(".line-numbers") as HTMLElement;

  editor.addEventListener("scroll", () => {
    lineNumbersEl.scrollTop = editor.scrollTop;
  });
});

watch(editorContent, () => {
  lineNumbers.value = getLineNumbers();
});
</script>

<template>
  <div class="editor-container">
    <div class="line-numbers">
      <div class="line-number" v-for="num in lineNumbers" :key="num">
        {{ num }}
      </div>
    </div>
    <textarea
      class="editor"
      spellcheck="false"
      v-model="editorContent"
    ></textarea>
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
  outline: none;
  background-color: #f5efe7;
  font-size: 14px;
  line-height: 1.5;
  resize: none;
  white-space: pre;
  overflow: auto;
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
</style>
