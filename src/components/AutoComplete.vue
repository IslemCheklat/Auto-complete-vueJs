<template>
  <section>
      <b-form-input
          class="autocomplete"
          v-model="inputValue"
          @input="updatePropositionListDebounce"
          @keyup.down="onkeydown"
          @keyup.up="onkeyup"
          @keyup.enter="submitSelectedProposition(selectedIndex)"
          @focus="updatePropositionListDebounce"
          @blur="handleBlur"
          :type="type"
          :maxlength="maxlength"
          />

      <div class="icon-container" v-if="isLoading">
        <b-spinner class="loader" variant="secondary" />
      </div>

    <b-list-group v-if="showProposition" @mouseover="mouseOverlistProposition=true"
                  @mouseleave="mouseOverlistProposition=false">
      <b-list-group-item v-for="(itemProposition, index) in listProposition" :key="index"
                         type="button" @click="submitSelectedProposition(index)"
                         :class="{'selected':selectedIndex==index}"
                         v-html="highlight(itemProposition)">
      </b-list-group-item>
    </b-list-group>
  </section>
</template>

<script>
import {BFormInput, BListGroup, BListGroupItem,BSpinner} from "bootstrap-vue";

export default {
  name: "AutoComplete",
  components: {BFormInput, BListGroup, BListGroupItem,BSpinner},
  data() {
    return {
      listProposition: [],
      selectedIndex: 0,
      mouseOverlistProposition: false,
      timeout: null,
      isLoading: false,
    }
  },
  props: {
    value: {
      type: String,
      required: false,
      default: ''
    },
    search: {
      type: Function,
      required: true
    },
    debounce: {
      type: Number,
      required: false,
      default: 300
    },
    type: {
      type: String,
      required: false,
      default: 'text'
    },
    maxlength: {
      type: String,
      required: false,
      default: '300'
    },
  },
  methods: {

    async updatePropositionListDebounce() {
      if (this.timeout) clearTimeout(this.timeout)
      this.timeout = setTimeout(() => {
        this.isLoading = true
        this.search(this.inputValue).then((list) => {
          this.listProposition = list;
        }).finally(() => {
          this.isLoading = false
        });
      }, this.debounce)
    },
    onkeydown() {
      if (this.selectedIndex < this.listProposition.length - 1)
        this.selectedIndex++;
    },
    onkeyup() {
      if (this.selectedIndex > 0)
        this.selectedIndex--;
    },
    submitSelectedProposition(index) {
      this.$emit('submit', this.listProposition[index])
      this.inputValue = this.listProposition[index]
      this.listProposition = []
      this.selectedIndex = 0
      this.mouseOverlistProposition = false
    },
    highlight(text, highlightPart = this.inputValue) {
      if (highlightPart.length === 0) {
        return text;
      }
      return text.replace(new RegExp(highlightPart, "gi"), match => {
        return '<b class="highlightText">' + match + '</b>';
      });
    },
    handleBlur() {
      if (!this.mouseOverlistProposition) {
        this.selectedIndex = 0
        this.listProposition = []
      }
    },
  },
  computed: {
    showProposition() {
      return this.listProposition.length !== 0;
    },
    inputValue: {
      get() {
        return this.value
      },
      set(value) {
        this.$emit('input', value)
      },
    },
  },
}
</script>

<style scoped>

section {
  position: relative;
}

.autocomplete{
  padding-right: 35px;
}

.list-group {
  margin-top: 0.25em;
  border: 1px solid rgba(0, 0, 0, .12);
  box-shadow: 0 2px 2px rgba(0, 0, 0, .16);
  position: absolute;
  width: 100%;
}

.list-group-item {
  border: none;
  text-align: start;
}

.list-group-item:hover {
  background-color: #F0F0F0;

}

.selected {
  background-color: #F0F0F0;
}

.icon-container {
  position: absolute;
  right: 10px;
  top: calc(50% - 10px);
}

.loader {
  position: relative;
  height: 20px;
  width: 20px;
  display: inline-block;
}


</style>