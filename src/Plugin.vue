<template>
  <div class="classlist">
      <div class="add">
        <select v-model="currentType">
          <option :value="type" v-for="(type, index) in types" :key="type + index" :checked="currentType == type">{{type}}</option>
        </select>
        <span>-</span>
        <select v-model="currentWidth">
          <option v-if="showEmptyWidth()"></option>
          <option :value="width" v-for="(width, index) in widths" :key="width + index" :checked="currentWidth == width">{{width}}</option>
        </select>
        <div v-if="showVisiblity()">
          <span>-</span>
          <select v-model="currentVisibility">
            <option :value="visible" v-for="(visible, index) in visibilities" :checked="currentVisibility == visible" :key="visible + index">{{visible}}</option>
          </select>
        </div>
        <button class="uk-button uk-button-primary" @click="add()">+</button>
      </div>
      <span class="list__header">Classnames</span>
      <ul class="list">
        <li class="list__item" v-for="(classname, index) in model.classnames" :key="'classname' + index"><span>{{classname}}</span><button class="uk-button uk-button-secondary uk-button-small" @click="remove(index)"><i class="uk-icon-trash"></i></button></li>
      </ul>
  </div>
</template>

<script>
const TYPE = {
  show: 'show',
  hide: 'hide'
}
export default {
  mixins: [window.Storyblok.plugin],
  data: () => ({
    currentColumn: null,
    currentWidth: null,
    currentType: TYPE.show,
    currentVisibility: null,
  }),
  methods: {
    initWith() {
      return {
        plugin: 'sassflexboxgrid-row',
        classnames: []
      }
    },
    showEmptyWidth() {
      return [TYPE.show, TYPE.hide].includes(this.currentType)
    },
    showVisiblity() {
      if([TYPE.show, TYPE.hide].includes(this.currentType) && this.currentWidth) {
        return true
      }
      this.currentVisibility = null
      return false
    },
    pluginCreated() {
      this.types = [TYPE.show, TYPE.hide]
      this.widths = ['xs', 'sm', 'md', 'lg', 'xl']
      this.visibilities = [null, 'only']
    },
    remove(index) {
      this.model.classnames.splice(index, 1)
    },
    search(classname) {
      const splittedClassname = classname.split('-')
      let searchTerm = classname
      let position = -1

      if(splittedClassname.length == 3) {
        searchTerm = splittedClassname.slice(0, splittedClassname.length - 1).join('-')
      }

      this.model.classnames.some((item, index) => {
        const foundMatch = item.includes(searchTerm)
        if(foundMatch) {
          position = index
          return true
        }
        return false
      })

      return position

    },
    add() {
      let currentClassName = this.currentType;
      
      if(this.currentWidth) {
          currentClassName += `-${this.currentWidth}`
      }
      if(this.currentColumn) {
        currentClassName += `-${this.currentColumn}`
      }
      if(this.currentVisibility) {
        currentClassName += `-${this.currentVisibility}`
      }

      const searchIndex = this.search(currentClassName)

      if(searchIndex != -1) {
        this.model.classnames.splice(searchIndex, 1, currentClassName)
      } else {
        this.model.classnames.push(currentClassName)
      }
    }
  },
  watch: {
    'model': {
      handler: function (value) {
        this.$emit('changed-model', value);
      },
      deep: true
    },
    currentType() {
      this.currentWidth = this.showEmptyWidth() ? null : 'xs';
      this.currentVisibility = null;
    }
  }
}
</script>

<style>
  .add {
    display: flex;
    align-items: center;
    flex: 1;
    margin-bottom: 10px;
  }
  
  .add span {
    display: inline-block;
    margin: 0 5px;
  }

  .add div {
    display: flex;
    align-items: center;
  }

  .uk-form .add select {
    padding: 0 25px 0 15px !important;
  }

  .uk-form .add button {
    margin-left: 5px;
  }

  .uk-form .list {
    padding: 0;
    margin: 0;
  }

  .uk-form .list .list__item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 8px 10px;
  }
</style>
