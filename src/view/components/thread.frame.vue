<style lang="sass">

</style>

<template lang="html">
  <ul class="thread timeline">
      <component is="tweetComponent" v-for="tweet in pretext" :tweet="tweet" :username="username" :now="now" track-by="id"></component>
      <component is="fatTweet" :tweet="base" :username="username" :now="now"></component>
      <component is="tweetComponent" v-for="tweet in replies" :tweet="tweet" :username="username" :now="now" track-by="id"></component>
    </ul>
</template>

<script>
'use strict';

var ipc = require('electron').ipcRenderer;
var Vue = require('vue');
var detectViewport = require('../directives/detectViewport');
Vue.use(detectViewport);
require('./tweet.fat.single.vue');
require('./tweet.single.vue');

var Thread = Vue.extend({
  props: ['base', 'pretext', 'replies', 'username', 'now', 'view'],
  events: {
    showThread: function (tweet) {
      if (tweet.id !== this.base.id) {
        this.pretext = [];
        this.replies = [];
        this.base = tweet;
        this.findContext();
      }
      return false;
    },
    newPretext: function (tweets) {
      if (tweets.length && tweets[tweets.length - 1].id === this.base.inReplyTo) {
        this.pretext = tweets;
        this.$nextTick(this.scrollToFat);
      }
      return false;
    },
    newReplies: function (tweets) {
      if (tweets.length && tweets[0].inReplyTo === this.base.id) {
        this.replies = tweets;
      }
      return false;
    }
  },
  methods: {
    findContext: function () {
      ipc.send('findContext', this.username, this.base.id);
    },
    scrollToFat: function () {
      var el = document.getElementsByClassName('fattweet')[0];
      if (el) {
        el.scrollIntoViewIfNeeded();
      }
    }
  },
  compiled: function () {
    this.findContext();
  },
  attached: function () {
    this.scrollToFat();
  }
});

Vue.component('thread', Thread);

module.exports = Thread;
</script>
