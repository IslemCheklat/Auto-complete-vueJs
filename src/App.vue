<template>
  <div id="app">
    <b-row align-h="center">
      <b-col cols="4">
        <auto-complete
            v-model="inputValue"
            :search="searchApi"
            @submit="submit"
            maxlength="100"
            type="text"
        ></auto-complete>
      </b-col>
    </b-row>
  </div>
</template>

<script>
import AutoComplete from "@/components/AutoComplete";
import {BRow, BCol} from "bootstrap-vue";

export default {
  name: 'App',
  components: {
    AutoComplete, BRow, BCol
  },
  data() {
    return {
      inputValue: ''
    }
  },
  methods: {
    submit(value) {
      console.log('submited value : ', value)
      this.inputValue = value;
    },

    async searchApi(value) {
      if (value.length >= 1) {
        return await fetch("https://api.npms.io/v2/search?q=vue")
            .then(response => response.json())
            .then(data => {
              let list = data.results.map(item => item.package.name)
              list = list.filter(item => item.toLowerCase().startsWith(value))
              return list
            })
      } else {
        return []
      }
    },

    async search(value) {
      const list = [{value: '86001', ville: 'Poitiers'},
        {value: '86700', ville: 'Poitiers'},
        {value: '868000', ville: 'Poitiers'},
        {value: '86701', ville: 'Poitiers'},
        {value: '867012', ville: 'Poitiers'},
        {value: '867180', ville: 'Poitiers'}]

        if (value.length >= 1)
          return list.filter((item) => item.value.toLowerCase().startsWith(value.toLowerCase()))
        return []

    },

    async searchWithSleep(value) {
      const list = ['86001', '86700', '868000', '86701', '867012', '867180']
      console.log("start sleep")
      return this.sleep().then(() => {
        console.log("after sleep")
        if (value.length >= 1)
          return list.filter((item) => item.toLowerCase().startsWith(value.toLowerCase()))
        return []
      });
    },
    async sleep() {
      let j = 20
      for (let i = 0; i < 900000000; i++) {
        j = j * j
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
