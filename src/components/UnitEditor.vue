<template>
  <div>
    <span>选择一个模板</span>
    <select v-model="tmplSelect">
      <option v-for="t in templates" :key="t" :value="t">{{ t.cn }}</option>
    </select>
  </div>
  <template v-if="tmplSelect">
    <div v-for="e in unit" :key="e.id">
      <button @click="e.show=!e.show">{{ e.sectionName }}</button>
      <div v-if="e.show">
        <template v-for="f in e" :key="f.key">
          <div v-if="f.isNecessary||f.isShow" class="unitdata">
            <span :title="f.des" @dblclick="f.isShow=false">{{ f.trans }}: </span>
            <input type="checkbox" v-if="f.type==='bool'" v-model="f.value">
            <input type="text" v-if="f.type==='string'" v-model="f.value">
            <input type="text" v-if="f.type==='int'" v-model.number="f.value">
            <input type="text" v-if="f.type==='float'" v-model.number="f.value">
            <input type="text" v-if="f.type==='time'" v-model="f.value">
            <select v-if="f.type==='enum'" v-model="f.value">
              <option v-for="g in f.enum" :key="g">{{ g }}</option>
            </select>
          </div>
        </template>
      </div>
    </div>
  </template>
  <div id="output">
    <h2>
      导出窗口
    </h2>
    <button @click="outputMethod">导出</button>
    <textarea v-model="output" style="height: 30%;width: 40%;min-height: 100px">
    </textarea>
  </div>
</template>

<script>
import {toRaw} from "vue";

export default {
  name: "UnitEditor",
  data() {
    return {
      templates: null,
      tmplSelect: null,
      unit: {
      },
      output: null,
    }
  },
  created() {
    this.$watch('tmplSelect', (newValue) => {
      fetch("/templates/" + newValue.en + ".json", {
        method: "GET",
      }).then(resp => resp.json()).then(res => {
        for (let i in res) {
          for (let j in res[i]) {
            this.unit[i][j].value = res[i][j]
            this.unit[i][j].isShow = true
          }
        }
      }).catch(() => {
        alert("出错啦")
      })
    })
  },
  computed: {
  },
  methods: {
    outputMethod() {
      let result = ""
      let raw = toRaw(this.unit)
      for (let ee in raw) {
        result += "["+raw[ee].sectionKey+"]\n"
        for (let ff in raw[ee]) {
          if(typeof(raw[ee][ff]) == 'object' &&raw[ee][ff].isShow) {
            result += raw[ee][ff].key + ":" + raw[ee][ff].value + "\n"
          }
        }
      }
      this.output = result
    },
    pr() {
      console.log(toRaw(this.unit))
    },
    hide(e) {
      e.isShow = false
    },
  },
  mounted() {
    fetch("/templates/templates.txt", {
      method: "GET",
    }).then(resp => resp.json()).then(res => this.templates = res)
    fetch("/templates/tmpl.json", {
      method: "GET",
    }).then(resp =>resp.json()).then(res => {
      this.unit = res
    })
  },
}
</script>

<style scoped>
.unitdata {
  margin-left: 0;
}
</style>
