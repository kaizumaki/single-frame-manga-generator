<template>
  <div>
    <div ref="result" class="manga-container">
      <img :src="`/manga/${imageFileName}`" alt="" />
      <div v-if="text" class="manga-text-outer">
        <p class="manga-text">
          <span class="first-letter">{{ text.substr(0, 1) }}</span>
          <span v-html="text.substr(1).replace('\n', '<br/>')"></span>
        </p>
      </div>
    </div>
    <div>
      <p>１コマにセリフをつけてみよう</p>
      <textarea v-model="text" rows="3" cols="50" />
    </div>
    <div>
      <button type="button" @click="uploadImage">画像を送信</button>
    </div>
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
      imageFileName: '',
      text: '',
      downloadURL: '',
    }
  },
  mounted() {
    this.getImage()
  },
  methods: {
    getImage() {
      const randomNumber = this.getRandomNumber(7, 1)
      this.imageFileName = `${this.getZeroPad(randomNumber, 2)}.png` || '01.png'
    },
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
    getRandomNumber(max: number, min: number = 0) {
      return Math.floor(Math.random() * (max - min) + min)
    },
    getZeroPad(value: number, num: number) {
      const _num = typeof num !== 'undefined' ? num : 2
      return value.toString().padStart(_num, '0')
    },
  },
})
</script>

<style lang="scss" scoped>
img {
  max-width: 100%;
  height: auto;
  vertical-align: bottom;
}
.manga-container {
  position: relative;
  display: inline-block;
}
.manga-text-outer {
  position: absolute;
  top: 8vw;
  right: 6.5vw;
  max-height: 70vw;
}
.manga-text {
  font-size: 5vw;
  letter-spacing: 5px;
  writing-mode: vertical-rl;
  .first-letter {
    display: inline-block;
    width: 10vw;
    height: 10vw;
    line-height: 10vw;
    border: 2px solid brown;
    border-radius: 50%;
    font-size: 10vw;
    font-weight: bold;
    color: brown;
    padding: 8px;
    text-align: center;
    letter-spacing: 0;
  }
}
</style>
