<template>
  <v-container grid-list-md>
    <v-row>
      <v-col cols="12">
        <v-btn color="green darken-1" text @click="toggleDebug(true)"
          >Start</v-btn
        >
        <v-btn color="red darken-1" text @click="toggleDebug(false)"
          >Stop</v-btn
        >
        <v-btn color="blue darken-1" text @click="debug = []">Clear</v-btn>
      </v-col>

      <v-col cols="12">
        <div
          id="debug_window"
          style="height:800px;width:100%;overflow-y:scroll;border: 1px solid grey; padding: 10px;"
          class="body-1"
          v-html="debug.join('')"
        ></div>
      </v-col>
    </v-row>
  </v-container>
</template>
<script>
import { socketEvents } from '@/plugins/socket'

import AnsiUp from 'ansi_up'
import { mapMutations } from 'vuex'

const ansiUp = new AnsiUp()

const MAX_DEBUG_LINES = 300

export default {
  name: 'Debug',
  props: {
    socket: Object
  },
  watch: {},
  computed: {},
  data () {
    return {
      debug: [],
      debugActive: false
    }
  },
  methods: {
    ...mapMutations(['showSnackbar']),
    toggleDebug (v) {
      this.debugActive = v
      this.showSnackbar('Debug ' + (v ? 'activated' : 'disabled'))
    }
  },
  mounted () {
    // init socket events
    const self = this
    this.socket.on(socketEvents.debug, data => {
      if (self.debugActive) {
        data = ansiUp.ansi_to_html(data)
        data = data.replace(/\n/g, '</br>')
        // \b[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}.[0-9]{3}Z\b
        self.debug.push(data)

        if (self.debug.length > MAX_DEBUG_LINES) self.debug.shift()

        const textarea = document.getElementById('debug_window')
        if (textarea) {
          // textarea could be hidden
          textarea.scrollTop = textarea.scrollHeight
        }
      }
    })
  },
  beforeDestroy () {
    if (this.socket) {
      // unbind events
      this.socket.off(socketEvents.debug)
    }
  }
}
</script>
