<script setup lang="ts">
import { ref } from "vue";

const contents = ref([
  "Hello, world!",
  "And hello again!",
  "This is a test message!",
]);

function lineSpanKeyHandler(event: KeyboardEvent) {
  switch (event.key) {
    case "Enter":
      onLineSpanEnter(event);
      break;
    case "Backspace":
      onLineSpanBackspace(event);
      break;
  }
}

function onLineSpanEnter(event: KeyboardEvent) {
  event.preventDefault();

  const target = event.target as HTMLSpanElement;
  const editorContents = document.querySelector(
    ".editor-contents"
  ) as HTMLDivElement;
  const index = Array.from(editorContents.children).indexOf(
    target.parentElement as HTMLDivElement
  );

  contents.value.splice(index + 1, 0, "");
  setTimeout(() => {
    const nextLine = target.parentElement?.nextElementSibling?.querySelector(
      "span"
    ) as HTMLSpanElement;
    nextLine.focus();
    target.parentElement?.querySelectorAll("br").forEach((br) => br.remove());
  });
}

function onLineSpanBackspace(event: KeyboardEvent) {
  const target = event.target as HTMLSpanElement;
  const editorContents = document.querySelector(
    ".editor-contents"
  ) as HTMLDivElement;
  const index = Array.from(editorContents.children).indexOf(
    target.parentElement as HTMLDivElement
  );

  if (target.textContent === "") {
    event.preventDefault();
    setTimeout(() => {
      const prevLine =
        target.parentElement?.previousElementSibling?.querySelector(
          "span"
        ) as HTMLSpanElement;
      prevLine.focus();
      contents.value.splice(index, 1);
    });
  }
}
</script>

<template>
  <div class="editor">
    <div class="line-numbers">
      <div class="line-number" v-for="num in contents.length">
        {{ String(num).padStart(3, "&nbsp;") }}
      </div>
    </div>
    <div class="editor-contents">
      <div v-for="line in contents" class="editor-line">
        <span contenteditable="true" @keypress="lineSpanKeyHandler($event)">{{
          line
        }}</span>
      </div>
    </div>
  </div>
</template>

<style>
@import url("https://fonts.googleapis.com/css2?family=Fira+Code:wght@300..700&family=Nunito:ital@0;1&display=swap");

* {
  box-sizing: border-box;
  margin: 0;

  font-family: "Fira Code", serif;
  font-optical-sizing: auto;
  font-weight: normal;
  font-style: normal;
}

html body {
  margin: 0;
}

body {
  background-color: #f5efe7;
}

.editor {
  display: flex;
  width: 100vw;
  height: 100vh;
  overflow: auto;
}

.editor-contents {
  flex-grow: 1;
}

.editor-line {
  display: flex;
}

.editor-line > span {
  flex-grow: 1;
}

.line-number {
  margin-right: 8px;
}

[contenteditable="true"]:focus {
  background-color: #d8c4b6;
  outline: none;
}
</style>
