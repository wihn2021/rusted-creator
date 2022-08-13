<template>
  <div>
    <span>选择一个模板</span>
    <select v-model="tmplSelect">
      <option v-for="t in templates" :key="t" :value="t">{{ t.cn }}</option>
    </select>
  </div>
  <template v-if="tmplSelect">
    <div v-for="ee in Object.keys(unit)" :key="ee.id">
      <button @click="template[ee].show=!template[ee].show" style="width: 50%;text-align: left" v-if="template[ee].sectionName">{{
          template[ee].sectionName
        }}
      </button>
      <div v-if="template[ee].show">
        <template v-for="ff in Object.keys(unit[ee])" :key="ff">
          <div v-if="template[ee][ff].isNecessary || template[ee][ff].isShow" class="unitdata">
            <span :title="template[ee][ff].des" @dblclick="template[ee][ff].isShow=false">{{ template[ee][ff].trans }}: </span>
            <input type="checkbox" v-if="template[ee][ff].type==='bool'" v-model="unit[ee][ff]">
            <input type="text" v-if="template[ee][ff].type==='string'" v-model="unit[ee][ff]">
            <input type="text" v-if="template[ee][ff].type==='int'" v-model.number="unit[ee][ff]">
            <input type="text" v-if="template[ee][ff].type==='float'" v-model.number="unit[ee][ff]">
            <input type="text" v-if="template[ee][ff].type==='time'" v-model="unit[ee][ff]">
            <div class="image file" v-if="template[ee][ff].type==='file (image)'">
              <img :src="unit[ee][ff]" alt="">
              <input type="file" accept="image/png" @change="base64encrypt(ee, ff)" :id="template[ee].sectionKey + ff">
            </div>
            <select v-if="template[ee][ff].type==='enum'" v-model="unit[ee][ff]">
              <option v-for="g in template[ee][ff].enum" :key="g">{{ g }}</option>
            </select>
          </div>
        </template>
      </div>
    </div>
  </template>
  <div id="output">
    <button @click="outputMethod">导出zip</button>
  </div>
</template>

<script>
import {toRaw} from "vue";
import JSZip from "jszip";
import {saveAs} from "file-saver";

export default {
  name: "UnitEditor",
  data() {
    return {
      templates: null,
      tmplSelect: null,
      unit: {},
      template: null,
      output: null,
      outZip: new JSZip(),
    }
  },
  created() {
    this.$watch('tmplSelect', (newValue) => {
      fetch("/templates/" + newValue.en + ".json", {
        method: "GET",
      }).then(resp => resp.json()).then(res => {
        this.unit = res
        for (let i in this.unit) {
          this.template[i].show = false
          for (let j in this.unit[i]) {
            this.template[i][j].isShow = true
          }
        }
      })
    })
  },
  computed: {},
  methods: {
    fetchHelpInfo() {

    },
    outputMethod() {
      if (this.tmplSelect === null) {
        alert("还没有数据！")
        return
      }
      this.outZip = new JSZip()
      let result = ""
      let raw = toRaw(this.unit)
      for (let ee in raw) {
        console.log(ee)
        if (ee in {"core":{}, "graphics":{}, "attack":{}, "movement":{}}) {
          result += "[" + this.template[ee].sectionKey + "]\n"
          for (let ff in raw[ee]) {
            if (this.template[ee][ff].isShow) {
              if (this.template[ee][ff].type === "file (image)") {
                this.outZip.file(this.template[ee].sectionKey + ff + '.png', raw[ee][ff].substring(raw[ee][ff].indexOf(',')+1), {base64: true})
                result += ff + ":" + this.template[ee].sectionKey + ff + '.png\n'
              }
              else {
                result += ff + ":" + raw[ee][ff] + "\n"
              }
            }
          }
        }
      }
      this.output = result
      this.outZip.file(this.unit.core.name + '.ini', result)
      this.outZip.generateAsync({type: "blob"})
      .then(function (content) {
        saveAs(content, 'mod.zip')
      })
    },
    base64encrypt(e, f) {
      const reader = new FileReader()
      reader.readAsDataURL(document.getElementById(this.template[e].sectionKey + f).files[0])
      const outThis = this;
      reader.onload = function() {
        outThis.unit[e][f] = this.result
      }

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
    }).then(resp => resp.json()).then(res => {
      this.template = res
    })
    this.outZip = new JSZip()
  },
}
</script>

<style scoped>
.unitdata {
  margin-left: 0;
}
</style>
