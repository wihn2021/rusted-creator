<template>
  <div>
    <span>选择一个模板</span>
    <select v-model="tmplSelect">
      <option v-for="t in templates" :key="t" :value="t">{{ t.cn }}</option>
    </select>
  </div>
  <template v-if="tmplSelect">
    <div v-for="e in unit" :key="e.id">
      <button @click="e.show=!e.show" style="width: 50%;text-align: left" v-if="e.sectionName">{{
          e.sectionName
        }}
      </button>
      <div v-if="e.show">
        <template v-for="f in e" :key="f.key">
          <div v-if="f.isNecessary||f.isShow" class="unitdata">
            <span :title="f.des" @dblclick="f.isShow=false">{{ f.trans }}: </span>
            <input type="checkbox" v-if="f.type==='bool'" v-model="f.value">
            <input type="text" v-if="f.type==='string'" v-model="f.value">
            <input type="text" v-if="f.type==='int'" v-model.number="f.value">
            <input type="text" v-if="f.type==='float'" v-model.number="f.value">
            <input type="text" v-if="f.type==='time'" v-model="f.value">
            <div class="image file" v-if="f.type==='file (image)'">
              <img :src="f.value">
              <input type="file" accept="image/png" @change="base64encrypt(e, f)" :id="e.sectionKey + f.key">
            </div>
            <select v-if="f.type==='enum'" v-model="f.value">
              <option v-for="g in f.enum" :key="g">{{ g }}</option>
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
      output: null,
      outZip: new JSZip(),
    }
  },
  created() {
    this.$watch('tmplSelect', (newValue) => {
      fetch("/templates/" + newValue.en + ".json", {
        method: "GET",
      }).then(resp => resp.json()).then(res => {
        for (let i in res) {
          for (let j in res[i]) {
            console.log("i=" + i + "j=" + j)
            this.unit[i][j].value = res[i][j]
            this.unit[i][j].isShow = true
          }
        }
      })
    })
  },
  computed: {},
  methods: {
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
          result += "[" + raw[ee].sectionKey + "]\n"
          for (let ff in raw[ee]) {
            if (typeof (raw[ee][ff]) == 'object' && raw[ee][ff].isShow) {
              if (raw[ee][ff].type === "file (image)") {
                this.outZip.file(raw[ee].sectionKey + raw[ee][ff].key + '.png', raw[ee][ff].value.substring(raw[ee][ff].value.indexOf(',')+1), {base64: true})
                result += raw[ee][ff].key + ":" + raw[ee].sectionKey + raw[ee][ff].key + '.png\n'
              }
              else {
                result += raw[ee][ff].key + ":" + raw[ee][ff].value + "\n"
              }
            }
          }
        }
      }
      this.output = result
      this.outZip.file(this.unit.core.name.value + '.ini', result)
      this.outZip.generateAsync({type: "blob"})
      .then(function (content) {
        saveAs(content, 'mod.zip')
      })
    },
    base64encrypt(e, f) {
      //console.log(e.target.files[0])
      console.log(JSON.parse(JSON.stringify(e)))
      console.log(JSON.parse(JSON.stringify(f)))
      const reader = new FileReader()
      reader.readAsDataURL(document.getElementById(e.sectionKey + f.key).files[0])
      reader.onload = function () {
        f.value = this.result
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
      this.unit = res
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
