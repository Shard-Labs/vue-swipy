# Vue-Swipy

Vue swipeable component for Tinder-like apps.

🚀 [See live demo](https://shard-labs.github.io/vue-swipy/)

## Installation

```bash
npm install vue-swipy
```

## Usage

```html
<template>
  <div id="app">
    <Swipeable
      v-for="card in cards"
      :key="card.id"
      v-on:swipe="onSwipe"
      :style="{
        position: 'absolute',
        height: '400px',
        width: '250px',
        background: card.color,
        borderRadius: '8px',
      }"
    />
  </div>
</template>

<script>
  import { Swipeable } from "vue-swipy";

  function getRandomColor() {
    var letters = "0123456789ABCDEF";
    var color = "#";
    for (var i = 0; i < 6; i++) {
      color += letters[Math.floor(Math.random() * 16)];
    }
    return color;
  }

  export default {
    name: "App",
    components: {
      Swipeable,
    },
    data() {
      return { cards: [] };
    },
    mounted() {
      this.cards.push({ id: Math.random(), color: getRandomColor() });
      this.cards.push({ id: Math.random(), color: getRandomColor() });
      this.cards.push({ id: Math.random(), color: getRandomColor() });
    },
    methods: {
      onSwipe(direction) {
        console.log(direction);
        setTimeout(() => {
          this.cards.pop();
          this.cards.unshift({ id: Math.random(), color: getRandomColor() });
        }, 300);
      },
    },
  };
</script>

<style>
  body {
    margin: 0;
  }
  #app {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    width: 100vw;
  }
</style>
```
