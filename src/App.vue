<template>
  <div>
    <Todo />
    <Footer />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import Todo from "./views/Todo.vue";
import Footer from "./components/Footer.vue";

@Component({
  components: {
    Todo,
    Footer
  }
})
export default class App extends Vue {}
</script>

<style>
 html,
body {
  margin: 0;
  padding: 0;
}

a,
button {
  transition: color 0.3s ease;
}

input[type="checkbox"],
a,
button {
  cursor: pointer;
}

button {
  margin: 0;
  padding: 0;
  border: 0;
  background: none;
  font-size: 100%;
  vertical-align: baseline;
  font-family: inherit;
  font-weight: inherit;
  color: inherit;
  -webkit-appearance: none;
  appearance: none;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  font-family: "Noto Sans HK", sans-serif;
  font-size: 12px;
  line-height: 1.4em;
  background-color: #8f4df8;
  color: #000;
  min-width: 230px;
  max-width: 550px;
  margin: 0 auto;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-weight: 300;
}

:focus {
  outline: 0;
}

.hidden {
  display: none;
}
</style>
