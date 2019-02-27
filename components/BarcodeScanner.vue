<template>
  <div>
    <span class="text-2xl">{{ barcode }}</span>
    <button
      class="bg-blue hover:bg-blue-dark text-white font-bold py-2 px-4 rounded"
      @click="stopScanning"
      v-if="scanning"
    >Stop</button>
    <button
      class="bg-blue hover:bg-blue-dark text-white font-bold py-2 px-4 rounded"
      @click="scanBarcode"
      v-else
    >Scan</button>
    <div ref="stream"></div>
  </div>
</template>

<script>
import Quagga from 'quagga'

export default {
  data() {
    return {
      scanning: false,
      barcode: null
    }
  },

  methods: {
    initCamera() {
      return Quagga.CameraAccess.enumerateVideoDevices()
    },

    scanBarcode() {
      this.initCamera()
      this.barcode = null
      Quagga.init(
        {
          inputStream: {
            name: 'Live',
            type: 'LiveStream',
            constraints: {
              width: { min: 640 },
              height: { min: 480 },
              facingMode: 'environment',
              aspectRatio: { min: 1, max: 2 }
            },
            target: this.$refs.stream
          },
          decoder: {
            readers: ['ean_reader']
          }
        },
        err => {
          if (err) {
            console.log(err)
            return
          }
          Quagga.start()
          this.scanning = true
        }
      )

      Quagga.onProcessed(result => {
        if (result) {
          if (result.codeResult && result.codeResult.code) {
            this.barcode = result.codeResult.code
            this.scanning = false
            Quagga.stop()
          }
        }
      })
    },

    stopScanning() {
      if (this.scanning) {
        Quagga.stop()
        this.scanning = false
      }
    }
  }
}
</script>

