<template>
  <div class="dropdown" v-if="options">

    <!-- Dropdown Input -->
    <input class="dropdown-input" :name="name" @input="search()" @keydown="handleKeyDown" @blur="exit()"
           v-model="searchFilter" :placeholder="placeholder" @focus="inpuFocusFn" />

    <!-- Dropdown Menu -->
    <div class="dropdown-content" v-show="optionsShown">
      <div class="dropdown-item" @mousedown="selectOption(option)" @mouseover="updateSelectedIndex(index)"
           v-for="(option, index) in filteredOptions" :key="index" :class="{ 'selected': index === selectedIndex }">
        {{ option.name || option.id || '-' }}
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'Dropdown',
  template: 'Dropdown',
  props: {
    name: {
      type: String,
      required: false,
      default: 'dropdown',
      note: 'Input name'
    },

    placeholder: {
      type: String,
      required: false,
      default: 'search for cities...',
      note: 'Placeholder of dropdown'
    },
    maxItem: {
      type: Number,
      required: false,
      default: 5,
      note: 'Max items showing'
    },
    userInput: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
      selected: {},
      optionsShown: false,
      selectedIndex: 0,
      searchFilter: '',
      options: [],
      avoidShake:null
    }
  },
  created() {
  },
  methods: {
    selectOption(option) {
      this.selected = option;
      this.optionsShown = false;
      this.searchFilter = this.selected.name;
    },
    search() {
      this.selectedIndex = 0;
      this.optionsShown = true;
      this.getLocations();
    },
    exit() {
      if (!this.selected.id){
        this.selected = {};
        this.searchFilter = '';
      }else {
        this.searchFilter = this.selected.name;
      }
      this.optionsShown = false;
    },
    handleKeyDown: function (event) {
      if (event.key === 'ArrowUp') {
        if (this.selectedIndex > 0) {
          this.selectedIndex--
        } else {
          this.selectedIndex = this.filteredOptions.length - 1
        }
      }
      if (event.key === 'ArrowDown') {
        if (this.selectedIndex < this.filteredOptions.length - 1) {
          this.selectedIndex++;
        } else {
          this.selectedIndex = 0;
        }

      }
      if (event.key === 'Enter') this.selectOption(this.filteredOptions[this.selectedIndex]);

    },
    updateSelectedIndex: function (index) {
      this.selectedIndex = index
    },
    async getLocations() {
      try {
        if (this.searchFilter) {
          clearTimeout(this.avoidShake)
          this.avoidShake = setTimeout(async ()=>{
            let arr = []
            const res = await axios.get(`https://us1.locationiq.com/v1/search?key=pk.55ec4342dd7f0f5032b399375ad91704&q=${this.searchFilter}&format=json`);
            console.log('res', res)
            res.data.forEach((element) => {
              let temp = (element.display_name).split(",").filter(item => item.trim())
              arr.push(...new Set(temp))
            })
            const tempArr = []
            arr.forEach((item, index) => {
              let obj = {};
              obj.id = index;
              obj.name = item.trim();
              tempArr.push(obj);
            })
            this.options = [...tempArr]
            this.optionsShown = true;
          },100)


        } else {
          this.optionsShown = false;
        }
      } catch (error) {
        console.log(error);
      }
    },
    inpuFocusFn(){
      this.getLocations()
    }
  },
  mounted() {
    if (this.userInput.length) {
      this.searchFilter = this.userInput
    }
  },
  watch: {
    options: {
      handler(newVal, oldVal) {
        const filtered = [];
        const regOption = new RegExp(this.searchFilter, 'ig');
        for (const option of JSON.parse(JSON.stringify(newVal))) {
          if (this.searchFilter.length < 1 || option.name.match(regOption)) {
            if (filtered.length < this.maxItem) filtered.push(option);
          }
        }
        this.filteredOptions = [...filtered]
      },
      deep: true

    },

  }
};
</script>


<style lang="scss" scoped>
.dropdown {
  position: relative;
  display: block;
  margin: auto;

  .dropdown-input {
    background: #fff;
    cursor: pointer;
    border: 1px solid #e7ecf5;
    border-radius: 3px;
    color: #333;
    display: block;
    font-size: .8em;
    padding: 6px;
    min-width: 250px;
    max-width: 250px;

    &:hover {
      background: #f8f8fa;
    }
  }

  .dropdown-content {
    position: absolute;
    background-color: #fff;
    min-width: 248px;
    max-width: 248px;
    max-height: 248px;
    border: 1px solid #e7ecf5;
    box-shadow: 0px -8px 34px 0px rgba(0, 0, 0, 0.05);
    overflow: auto;
    z-index: 1;

    .dropdown-item {
      color: black;
      font-size: .7em;
      line-height: 1em;
      padding: 8px;
      text-decoration: none;
      display: block;
      cursor: pointer;

      &:hover {
        background-color: #e7ecf5;
      }
    }

    .selected {
      background-color: #e7ecf5;
    }
  }

  .dropdown:hover .dropdowncontent {
    display: block;
  }
}
</style>