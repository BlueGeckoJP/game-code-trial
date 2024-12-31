<script setup lang="ts">
import { ref } from "vue";

const contents = ref([
  "Hello, world!",
  "And hello again!",
  "This is a test message!",
]);

function getIndex(target: HTMLSpanElement) {
  return Array.from(
    (document.querySelector(".editor-contents") as HTMLDivElement).children
  ).indexOf(target.parentElement as HTMLDivElement);
}

function updateContents(target: HTMLSpanElement) {
  const index = getIndex(target);
  contents.value[index] = target.innerHTML;
}

function lineSpanKeyPressHandler(event: KeyboardEvent) {
  switch (event.key) {
    case "Enter":
      onLineSpanEnter(event);
      break;
    default:
      updateContents(event.target as HTMLSpanElement);
      break;
  }
}

function lineSpanKeyDownHandler(event: KeyboardEvent) {
  switch (event.key) {
    case "Backspace":
      onLineSpanBackspace(event);
      updateContents(event.target as HTMLSpanElement);
      break;
    case "ArrowUp":
      event.preventDefault();
      var caretPos = window.getSelection()?.getRangeAt(0).startOffset;
      const prevLine = (
        event.target as HTMLSpanElement
      ).parentElement?.previousElementSibling?.querySelector(
        "span"
      ) as HTMLSpanElement;
      prevLine.focus();
      if (caretPos) {
        const range = window.getSelection()?.getRangeAt(0);
        if (range && caretPos <= prevLine.innerHTML.length) {
          const child = prevLine.firstChild;
          if (child) {
            range.setStart(child, caretPos);
          }
        }
      }
      break;
    case "ArrowDown":
      event.preventDefault();
      var caretPos = window.getSelection()?.getRangeAt(0).startOffset;
      const nextLine = (
        event.target as HTMLSpanElement
      ).parentElement?.nextElementSibling?.querySelector(
        "span"
      ) as HTMLSpanElement;
      nextLine.focus();
      if (caretPos) {
        const range = window.getSelection()?.getRangeAt(0);
        if (range && caretPos <= nextLine.innerHTML.length) {
          const child = nextLine.firstChild;
          if (child) {
            range.setStart(child, caretPos);
          }
        }
      }
      break;
  }
}

function onLineSpanEnter(event: KeyboardEvent) {
  event.preventDefault();

  const target = event.target as HTMLSpanElement;
  const index = getIndex(target);
  const caretIndex = window.getSelection()?.getRangeAt(0).startOffset;

  const rightOfCaret = contents.value[index].slice(caretIndex);
  contents.value[index] = contents.value[index].slice(0, caretIndex);

  contents.value.splice(index + 1, 0, rightOfCaret);
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
  const index = getIndex(target);
  const caretIndex = window.getSelection()?.getRangeAt(0).startOffset;

  if (caretIndex === 0) {
    event.preventDefault();
    setTimeout(() => {
      const prevLine =
        target.parentElement?.previousElementSibling?.querySelector(
          "span"
        ) as HTMLSpanElement;
      prevLine.focus();
      contents.value[index - 1] += contents.value[index];
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
        <span
          contenteditable="true"
          @keypress="lineSpanKeyPressHandler($event)"
          @keydown="lineSpanKeyDownHandler($event)"
          >{{ line }}</span
        >
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
  white-space: pre-wrap;
}
</style>
