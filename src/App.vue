<template>
  <div id="app">
    <TouchID Side="Right" :Percent="48" />
  </div>
</template>

<script>
import TouchID from './components/TouchID.vue';

export default {
  name: 'App',
  components: {
    TouchID,
  },
  mounted() {
    let interval = setInterval(() => {
      if (window.LiveChatWidget) {
        setTimeout(() => {
          window.LiveChatWidget.call('hide');
        }, 500);
        clearInterval(interval);
      }
    }, 50);
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
#chat-widget-container {
  display: none;
}
</style>
