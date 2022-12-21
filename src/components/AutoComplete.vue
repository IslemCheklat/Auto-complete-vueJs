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
        autocomplete="off"
        :disabled="disabled"
    />

    <div class="icon-container" v-if="isLoading">
      <b-spinner class="loader" variant="secondary"/>
    </div>

    <b-list-group v-if="showProposition"
                  class="proposition-group"
                  ref="propositionGroup"
                  @mouseover="mouseOverlistProposition=true"
                  @mouseleave="mouseOverlistProposition=false">
      <b-list-group-item v-for="(propositionItem, index) in propositionList"
                         :key="index"
                         :class="[{'selected':selectedIndex==index}, 'proposition-item']"
                         ref="propositionItem"
                         type="button"
                         @click="submitSelectedProposition(index)"
                         v-html="highlight(propositionItem)">
      </b-list-group-item>
    </b-list-group>
  </section>
</template>

<script>
import {BFormInput, BListGroup, BListGroupItem, BSpinner} from "bootstrap-vue";

export default {
  name: "AutoComplete",
  components: {BFormInput, BListGroup, BListGroupItem, BSpinner},
  data() {
    return {
      propositionList: [],
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
    serializer: {
      type: Function,
      required: false,
      default: (value) => {
        return value
      }
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
    disabled: {
      type: Boolean,
      required: false,
      default: false
    }
  },
  methods: {
    async updatePropositionListDebounce() {
      if (this.timeout) clearTimeout(this.timeout)
      this.timeout = setTimeout(() => {
        this.isLoading = true
        this.search(this.inputValue).then((list) => {
          this.propositionList = list;
        }).finally(() => {
          this.isLoading = false
        });
      }, this.debounce)
    },
    submitSelectedProposition(index) {
      if (this.showProposition) {
        this.$emit('submit', this.propositionList[index])
        this.inputValue = this.serializer(this.propositionList[index])
        this.propositionList = []
        this.selectedIndex = 0
        this.mouseOverlistProposition = false
      }
    },
    highlight(item, highlightPart = this.inputValue) {
      const text = this.serializer(item)
      if (highlightPart.length === 0) {
        return text;
      }
      return text.replace(new RegExp(highlightPart, "gi"), match => {
        return '<b class="highlightText">' + match + '</b>';
      });
    },
    onkeydown() {
      if (this.selectedIndex < this.propositionList.length - 1) {
        this.selectedIndex++;
        const liHeight =this.$refs.propositionItem[this.selectedIndex].clientHeight
        if(liHeight*(this.selectedIndex) >= this.$refs.propositionGroup.clientHeight)
        this.$refs.propositionGroup.scrollTop +=liHeight
      }else{
        this.selectedIndex=0;
        this.$refs.propositionGroup.scrollTop=0;
      }
    },
    onkeyup() {
      if (this.selectedIndex > 0) {
        this.selectedIndex--;
        const liHeight =this.$refs.propositionItem[this.selectedIndex].clientHeight
        if(liHeight*(this.selectedIndex+1) >= this.$refs.propositionGroup.clientHeight)
          this.$refs.propositionGroup.scrollTop -=liHeight
      }
    },
    handleBlur() {
      if (!this.mouseOverlistProposition) {
        this.selectedIndex = 0
        this.propositionList = []
      }
    },
  },
  computed: {
    showProposition() {
      return this.propositionList.length !== 0;
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

.autocomplete {
  padding-right: 35px;
}

.proposition-group {
  margin-top: 0.25em;
  border: 1px solid rgba(0, 0, 0, .12);
  box-shadow: 0 2px 2px rgba(0, 0, 0, .16);
  position: absolute;
  width: 100%;
  height: auto;
  max-height: 280px;
  overflow-y: auto;
}

.proposition-item {
  border: none;
  text-align: start;

}

.proposition-item:hover {
  /*background-color: #F0F0F0;*/
  background-color: var(--bs-primary);
  color: white;
}

.selected {
  /*background-color: #F0F0F0;*/
  background-color: var(--bs-primary);
  color: white;
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