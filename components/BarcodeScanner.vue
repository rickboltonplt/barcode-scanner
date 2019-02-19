<template>
  <div>
    <label>
      <span>Barcode-Type</span>
      <select v-model="decoderType">
        <option value="code_128">Code 128</option>
        <option value="code_39">Code 39</option>
        <option value="code_39_vin">Code 39 VIN</option>
        <option value="ean">EAN</option>
        <option value="ean_extended">EAN-extended</option>
        <option value="ean_8">EAN-8</option>
        <option value="upc">UPC</option>
        <option value="upc_e">UPC-E</option>
        <option value="codabar">Codabar</option>
        <option value="i2of5">Interleaved 2 of 5</option>
        <option value="2of5">Standard 2 of 5</option>
        <option value="code_93">Code 93</option>
      </select>
    </label>

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
      decoderType: 'code_128',
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
            readers: [`${this.decoderType}_reader`]
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
  },

  watch: {
    decoderType() {
      this.scanBarcode()
    }
  }
}
</script>

