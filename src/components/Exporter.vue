<template>
  <q-btn label="Export" color="primary" @click="createFile" class="full-width"/>
</template>

<script>
function download(data, filename, type) {
    var file = new Blob([data], {type: type});

}

export default {
  props: {
    messages: {
      type: Array,
      default: () => []
    }
  },
  methods: {
    createFile() {
      const messages = this.messages.map(({sent, text}) => `${sent ? 'User' : 'Bot'}: ${text.join('\t')}`)
      const file = new Blob([JSON.stringify(messages)], { type: "text/plain;charset=utf-8" })
      if (window.navigator.msSaveOrOpenBlob) {
        window.navigator.msSaveOrOpenBlob(file, 'dump.txt')

      } else { // Others
        const a = document.createElement("a")
        const url = URL.createObjectURL(file)
        a.href = url
        a.download = 'dump.txt'
        document.body.appendChild(a)
        a.click()
        setTimeout(function() {
            document.body.removeChild(a)
            window.URL.revokeObjectURL(url)  
        }, 0) 
      }
    }
  }
}
</script>
