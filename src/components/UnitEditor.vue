<template>
  <div>
    <span>选择一个模板</span>
    <select v-model="tmplSelect">
      <option v-for="t in templates" :key="t" :value="t">{{ t.cn }}</option>
    </select>
  </div>
  <template v-if="tmplSelect">
    <div v-for="ee in Object.keys(unit)" :key="ee.id">
      <button @click="template[ee].show=!template[ee].show" style="width: 50%;text-align: left"
              v-if="template[ee].sectionName">{{
          template[ee].sectionName
        }}
      </button>
      <div v-if="template[ee].show">
        <template v-if="ee==='turret'&&unit[ee]">
          <div v-for="t in unit.turret" :key="unit.turret.indexOf(t)">
            <h2>turret{{ unit.turret.indexOf(t) }}</h2>
            <button @click="unit.turret.splice(unit.turret.indexOf(t),1)">删除炮塔</button>
            <!---
            <div style="position: static; text-align: center; display: block">
              <div style="text-align: center;position: center"><img :src="this.unit.graphics.image" style="margin: 0 auto"></div>
              <div style="text-align: center;position: center"><img :src="t.image" :style="turretStyle"></div>
            </div>
            --->
            <div v-for="ff in Object.keys(t)" :key="ff">
              <span :title="template.turret[ff].des">{{ template.turret[ff].trans }}</span>
              <input type="checkbox" v-if="template[ee][ff].type==='bool'" v-model="t[ff]">
              <input type="checkbox" v-if="template[ee][ff].type==='LogicBoolean'" v-model="t[ff]">
              <input type="text" v-if="template[ee][ff].type==='string'" v-model="t[ff]">
              <input type="text" v-if="template[ee][ff].type==='int'" v-model.number="t[ff]">
              <input type="text" v-if="template[ee][ff].type==='float'" v-model.number="t[ff]">
              <input type="text" v-if="template[ee][ff].type==='time'" v-model="t[ff]">
              <div class="image file" v-if="template[ee][ff].type==='file (image)'">
                <img :src="t[ff]" alt="">
                <input type="file" accept="image/png" @change="base64encrypt(ee, ff)"
                       :id="template[ee].sectionKey+ unit.turret.indexOf(t) + ff">
              </div>
              <select v-if="template[ee][ff].type==='enum'" v-model="t[ff]">
                <option v-for="g in template[ee][ff].enum" :key="g">{{ g }}</option>
              </select>
            </div>
          </div>
          <button @click="newTurret">新建炮塔</button>
        </template>
        <template v-else>
          <template v-for="ff in Object.keys(unit[ee])" :key="ff">
            <div v-if="template[ee][ff].isNecessary || template[ee][ff].isShow" class="unitdata">
              <span :title="template[ee][ff].des" @dblclick="template[ee][ff].isShow=false">{{
                  template[ee][ff].trans
                }}: </span>
              <input type="checkbox" v-if="template[ee][ff].type==='bool'" v-model="unit[ee][ff]">
              <input type="checkbox" v-if="template[ee][ff].type==='LogicBoolean'" v-model="unit[ee][ff]">
              <input type="text" v-if="template[ee][ff].type==='string'" v-model="unit[ee][ff]">
              <input type="text" v-if="template[ee][ff].type==='int'" v-model.number="unit[ee][ff]">
              <input type="text" v-if="template[ee][ff].type==='float'" v-model.number="unit[ee][ff]">
              <input type="text" v-if="template[ee][ff].type==='time'" v-model="unit[ee][ff]">
              <div class="image file" v-if="template[ee][ff].type==='file (image)'">
                <img :src="unit[ee][ff]" alt="">
                <input type="file" accept="image/png" @change="base64encrypt(ee, ff)"
                       :id="template[ee].sectionKey + ff">
              </div>
              <select v-if="template[ee][ff].type==='enum'" v-model="unit[ee][ff]">
                <option v-for="g in template[ee][ff].enum" :key="g">{{ g }}</option>
              </select>
            </div>
          </template>
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
import {TurretTmpl} from "@/components/TurretTmpl";

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
  computed: {
    turretStyle(t) {
      return {
        left: t.x + 'px',
        top: t.y + 'px',
      }
    }
  },
  created() {
    this.$watch('tmplSelect', (newValue) => {
      fetch("/templates/" + newValue.en + ".json", {
        method: "GET",
      }).then(resp => resp.json()).then(res => {
        this.unit = res
        for (let i in this.unit) {
          // stop using typical methods to deal with turrets here
          if (i === 'turret') continue
          this.template[i].show = false
          for (let j in this.unit[i]) {
            this.template[i][j].isShow = true
          }
        }
      })
    })
  },
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
        if (ee in {"core": {}, "graphics": {}, "attack": {}, "movement": {}}) {
          result += "[" + this.template[ee].sectionKey + "]\n"
          for (let ff in raw[ee]) {
            if (this.template[ee][ff].isShow) {
              if (this.template[ee][ff].type === "file (image)") {
                this.outZip.file(this.template[ee].sectionKey + ff + '.png', raw[ee][ff].substring(raw[ee][ff].indexOf(',') + 1), {base64: true})
                result += ff + ":" + this.template[ee].sectionKey + ff + '.png\n'
              } else {
                result += ff + ":" + raw[ee][ff] + "\n"
              }
            }
          }
        }
        else if(ee in {"turret":{}}) {
          for (let i=0; i<raw[ee].length; i++) {
            result += "[turret_" + i + "]\n"
            for (let ent in raw[ee][i]) {
              if (this.template.turret[ent].type === "file (image)") {
                this.outZip.file("turret" + i + '.png', raw[ee][i][ent].substring(raw[ee][i][ent].indexOf(',') + 1), {base64: true})
                result += ent + ":" + "turret" + i + '.png\n'
              } else {
                result += ent + ":" + raw[ee][i][ent] + "\n"
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
      reader.onload = function () {
        outThis.unit[e][f] = this.result
      }
    },
    newTurret() {
      this.unit.turret.push(TurretTmpl)
    }
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
