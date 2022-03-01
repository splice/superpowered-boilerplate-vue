<template>
  <img
    id="superpowered-logo"
    alt="Superpowered logo"
    src="./assets/superpowered-black.svg"
  />
  <img alt="Vue logo" src="./assets/logo.png" />
  <h3>
    The following example will setup Superpowered for you and play a test tone.
  </h3>
  <button v-if="$data.Superpowered && !$data.webAudioManager" @click="onBoot()">
    Start tone
  </button>

  <div v-if="$data.webAudioManager">
    <p>
      Sample Rate:
      {{ $data.webAudioManager.audioContext.sampleRate / 1000 }}
      kHz
    </p>
    <p>
      Estimated Latency:
      {{ $data.webAudioManager.audioContext.baseLatency.toFixed(3) }}s
    </p>
  </div>

</template>

<script>
import {
  SuperpoweredGlue,
  SuperpoweredWebAudio,
} from "./assets/superpowered/SuperpoweredWebAudio.js";

export default {
  name: "App",
  data() {
    return {
      Superpowered: null, // Reference to the Superpowered module.
      webAudioManager: null, // The SuperpoweredWebAudio helper class managing Web Audio for us.
      audioNode: null, // This example uses one audio node only.
    };
  },
  methods: {
    //Instantite and initialize the Superpowered WebAssembly
    async initialiseSuperpoweredWebAssembly() {
      this.Superpowered = await SuperpoweredGlue.fetch(
        "./superpowered/superpowered.wasm"
      );
      this.Superpowered.Initialize("ExampleLicenseKey-WillExpire-OnNextUpdate");
    },
    // Create a WebAudioManager
    createWebAudioManager() {
      this.webAudioManager = new SuperpoweredWebAudio(48000, this.Superpowered);
    },
    async onBoot() {
      this.createWebAudioManager();
      //use the WebAudioManager to create the audioNode for reverbProcessor
      this.audioNode = await this.webAudioManager.createAudioNodeAsync(
        "/toneProcessor.js",
        "SuperpoweredSingleGeneratorStageProcessor"
      );

      // connect the reverb node to end destination node
      this.audioNode.connect(this.webAudioManager.audioContext.destination);

    },
    onParamChange(id, e) {
      this.audioNode.sendMessageToAudioScope({ [id]: e.target.value });
    },
  },
  created() {
    this.initialiseSuperpoweredWebAssembly();

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
  margin-top: 50px;

  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 400px;
  margin-left: auto;
  margin-right: auto;
  padding: 10px;
}

#superpowered-logo {
  width: 100%;
  margin-bottom: 60px;
}
</style>
