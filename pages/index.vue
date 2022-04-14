<template>
  <div>
    <div ref="result">
      <div class="manga-container">
        <img src="/test.png" alt="" />
        <p>{{ text }}</p>
      </div>
    </div>
    <input v-model="text" type="text" />
    <button type="button" @click="uploadImage">upload</button>
    <div v-if="downloadURL">
      <a :href="downloadURL" target="_blank">{{ downloadURL }}</a>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import html2canvas from 'html2canvas'

export default Vue.extend({
  data() {
    return {
      text: '',
      downloadURL: '',
    }
  },
  methods: {
    uploadImage() {
      const storageRef = this.$fire.storage.ref()
      const randomString = this.getRandomString(8)
      const now = Date.now()
      html2canvas(this.$refs.result as HTMLElement).then((canvas) => {
        canvas.toBlob((blob) => {
          storageRef
            .child(`${randomString}${now}.png`)
            .put(blob!)
            .then((snapshot) => {
              snapshot.ref.getDownloadURL().then((downloadURL) => {
                this.downloadURL = downloadURL
              })
            })
        }, 'image/png')
      })
    },
    getRandomString(length: number) {
      const source = 'abcdefghijklmnopqrstuvwxyz'
      let result = ''
      for (let i = 0; i < length; i++) {
        result += source[Math.floor(Math.random() * source.length)]
      }
      return result
    },
  },
})
</script>
