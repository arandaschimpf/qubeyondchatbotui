<template>
  <q-page class="flex flex-center">
    <q-card class="col-8">
      <q-card-title>
        <q-btn color="primary" class="full-width" :disabled="recording" @click="startRecording">Start recording</q-btn>
      </q-card-title>
      <q-card-main>
        <q-scroll-area style="width: 100%; height: 100%;" ref="scroll">
          <div v-for="(message, index) in messages" :key="index">
            <q-chat-message
              :text="message.text"
              :sent="message.sent"
            />
          </div>
        </q-scroll-area>
      </q-card-main>
      <q-card-separator />
      <q-card-actions class="row justify-between">
        <q-input v-model="text" class="col-10" ref="input" @keyup.enter="addMessage(text)"/>
        <q-btn color="primary" class="col-2" icon="send" :disabled="text.length < 3" @click="addMessage(text)"/>
      </q-card-actions>
    </q-card>
  </q-page>
</template>

<style scoped>
.q-card {
  min-width: 50%;
}

.q-card-main {
  height: 50vh;
  overflow: auto;
}
</style>

<script>
import { debounce } from 'quasar'
const subscriptionKey = "0cb2eb1b14bc4c48a8212a1292167425"
const serviceRegion = "eastus"

const baseURL = 'http://localhost:5005/conversations/current/respond'

export default {
  name: 'PageIndex',
  data: () => ({
    text: '',
    recording: false,
    messages: []
  }),
  mounted () {
    if (!window.SpeechSDK) {
      this.recording = true
    }
    this.addMessage('Hello')
  },
  methods: {
    goToBottom: debounce(function() {
      this.$refs.scroll.setScrollPosition(Number.MAX_SAFE_INTEGER, 100)
    }, 50),
    addMessage (text, sent = true) {
      if (text.length < 3) return
      this.text = ''
      this.messages.push({text: [text], sent})
      this.goToBottom()
      if (sent) {
        this.sendMessage(text)
      }
    },
    sendMessage (query) {
      if (!baseURL) return
      fetch(baseURL, {
        method: 'post',
        body: JSON.stringify({ query })
      }).then(r => r.json()).then(messages => {
        for (const { text } of messages) {
          this.addMessage(text, false)
        }
      }).catch(error => {
        this.$q.notify({
          message: String(error),
          type: 'negative'
        })
      })
    },
    startRecording () {
      this.recording = true
      let speechConfig = SpeechSDK.SpeechConfig.fromSubscription(subscriptionKey, serviceRegion)
      speechConfig.speechRecognitionLanguage = "en-US"
  
      let audioConfig = SpeechSDK.AudioConfig.fromDefaultMicrophoneInput();
      let recognizer = new SpeechSDK.SpeechRecognizer(speechConfig, audioConfig);
      
      recognizer.recognizeOnceAsync(
        result => {
          this.recording = false
          this.text += result.text || ''
          this.$refs.input.focus()
          recognizer.close()
        },
        error => {
          this.recording = false
          this.$q.notify({
            message: String(error),
            type: 'negative'
          })
          recognizer.close()
        }
      )
    }
  }
}
</script>
