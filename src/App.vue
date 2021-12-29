<template>
  <div id="app">
    <div style="margin: 20px;">
      H264:
        <input ref="fileH264" type="file" @change="openH264">
    </div>
    <div style="margin: 20px;">
      H265:
        <input ref="fileH265" type="file" @change="openH265">
    </div>
    <div style="margin-top: 10px">
      <canvas ref="playCanvas" width="800" height="480"></canvas>
    </div>

    <div style="margin: 20px;">
      <button @click="play">play</button>
      <button @click="stop">stop</button>
      <button @click="resume">resume</button>
    </div>
  </div>
</template>

<script>
import Decoder from 'video-decoder'

export default {
  name: 'App',
  components: {
  },
  created () {
    Decoder.setReadyCb(this.deReady)
    this.file = null
    this.typ = ''
    this.blockSize = 4096
  },
  methods: {
    deReady () {
      console.log('deReady')
      Decoder.setLogLevel('trace')
    },
    openH264 () {
      console.log('openH264')
      if (this.$refs.fileH264.files.length > 0) {
        this.typ = 'h264'
        this.file = this.$refs.fileH264.files[0]
        this.createDe()
      }
    },
    openH265 () {
      console.log('openH265')
      if (this.$refs.fileH265.files.length > 0) {
        this.typ = 'h265'
        this.file = this.$refs.fileH265.files[0]
        this.createDe()
      }
    },
    createDe () {
      if (this.de) {
        this.de.dispose()
      }
      this.de = new Decoder(this.typ)
      this.reader = new FileReader()
      this.reader.addEventListener('load', this.onLoad)
      this.fileOffset = 0
    },
    play () {
      this.playing = true
      this.fileOffset = 0
      this.fileOffset += this.readFile(this.reader, this.file, this.fileOffset, this.blockSize)
    },
    resume () {
      this.playing = true
      this.fileOffset += this.readFile(this.reader, this.file, this.fileOffset, this.blockSize)
    },
    stop () {
      this.playing = false
    },
    onLoad2 (e) {
      if (!this.playing) {
        return
      }
      const buf = new Uint8Array(e.target.result)
      // console.log('buf:', buf)
      this.de.put(buf)
      const self = this
      console.log('offset:', self.fileOffset)
      // console.log(self.fileOffset, buf.length, md5(buf))
      if (self.fileOffset < 1024 * 1024 * 2) {
        self.fileOffset += self.readFile(self.reader, self.file, self.fileOffset, self.blockSize)
        return
      }
      console.log('offset:', self.fileOffset)
      console.log('init buf end')

      const getFrame = async () => {
        // if (!this.playing) {
        //   return
        // }
        const f = self.de.get()
        if (f) {
          console.log('frame:', f)
          const canvas = this.$refs.playCanvas
          // console.log('canvas:', canvas)
          const oc = canvas.getContext('2d')
          const img = new ImageData(f.width, f.height)
          img.data.set(f.data, 0)
          const ib = await createImageBitmap(img)
          oc.drawImage(ib, 0, 0, f.width, f.height, 0, 0, canvas.clientWidth, canvas.clientHeight)
          // oc.drawImage(ib, 0, 0)
          setTimeout(getFrame, 100)
        }
      }
      setTimeout(getFrame, 0)
    },
    onLoad (e) {
      if (!this.playing) {
        return
      }
      const buf = new Uint8Array(e.target.result)
      // console.log('buf:', buf)
      this.de.put(buf)
      const self = this
      console.log('================================== offset:', self.fileOffset)
      // if (self.fileOffset > 1024 * 1024 * 3) {
      //   console.log('================ offset:', self.fileOffset)
      //   return
      // }
      const getFrame = async () => {
        // if (!this.playing) {
        //   return
        // }
        const f = self.de.get()
        if (f) {
          console.log('frame:', f)
          const canvas = this.$refs.playCanvas
          // console.log('canvas:', canvas)
          const oc = canvas.getContext('2d')
          const img = new ImageData(f.width, f.height)
          img.data.set(f.data, 0)
          const ib = await createImageBitmap(img)
          oc.drawImage(ib, 0, 0, f.width, f.height, 0, 0, canvas.clientWidth, canvas.clientHeight)
          // oc.drawImage(ib, 0, 0)
          setTimeout(getFrame, 100)
        } else {
          self.fileOffset += self.readFile(self.reader, self.file, self.fileOffset, self.blockSize)
        }
      }
      setTimeout(getFrame, 0)
    },
    readFile (reader, file, offset, chunkSize) {
      let end = offset + chunkSize
      const total = file.size
      if (end > total) {
        end = total
      }
      if (end <= offset) {
        return 0
      }
      let s
      if (file.slice) {
        s = file.slice(offset, end)
      } else if (file.mozSlice) {
        s = file.mozSlice(offset, end)
      } else if (file.webkitSlice) {
        s = file.webkitSlice(offset, end)
      } else {
        return 0
      }
      reader.readAsArrayBuffer(s)
      return end - offset
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
