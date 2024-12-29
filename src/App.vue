<script setup lang="ts">
import { ref } from "vue";

const contents = ref([
  "Hello, world!",
  "And hello again!",
  "This is a test message!",
]);
</script>

<template>
  <div class="editor">
    <div class="line-numbers">
      <div class="line-number" v-for="num in contents.length">
        {{ num }}
      </div>
    </div>
    <div class="editor-contents">
      <div v-for="line in contents" class="editor-line">
        <span contenteditable="true">{{ line }}</span>
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
}

.editor-contents {
  flex-grow: 1;
  overflow: auto;
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
