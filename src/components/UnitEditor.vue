<template>
  <div id="navi">
    <div>
      <span>选择一个模板</span>
      <select v-model="tmplSelect">
        <option v-for="t in templates" :key="t" :value="t">{{ t.cn }}</option>
      </select>
      <button @click="newUnit">新建单位</button>
    </div>
    <span>选择一个单位来进行编辑：</span>
    <select v-model="unitSelect">
      <option v-for="e in Object.keys(units)" :value="e" :key="'unit'+e">
        {{ e }}
      </option>
    </select>
    <button @click="saveUnit">保存单位</button>
    <button @click="deleteUnit">删除单位</button>
    <button @click="allModOutput">导出zip</button>
  </div>
  <div id="uniteditor" v-if="unit">
    <template v-if="tmplSelect">
      <div v-for="ee in Object.keys(unit)" :key="ee.id">
        <button @click="template[ee].show=!template[ee].show" style="width: 50%;text-align: left"
                v-if="template[ee].sectionName">{{
            template[ee].sectionName
          }}
        </button>
        <div v-if="template[ee].show">
          <template v-if="ee==='turret'">
            <div v-for="t in rangeArray(unit[ee])" :key="t">
              <h2>{{ ee }}{{ t }}</h2>
              <button @click="unit[ee].splice(t,1)">删除{{ template[ee].sectionName }}</button>
              <!---
              <div style="position: static; text-align: center; display: block">
                <div style="text-align: center;position: center"><img :src="this.unit.graphics.image" style="margin: 0 auto"></div>
                <div style="text-align: center;position: center"><img :src="t.image" :style="turretStyle"></div>
              </div>
              --->
              <div v-for="ff in Object.keys(unit.turret[t])" :key="ff">
                <span :title="template.turret[ff].des">{{ template.turret[ff].trans }}</span>
                <input type="checkbox" v-if="template[ee][ff].type==='bool'" v-model="unit.turret[t][ff]">
                <input type="checkbox" v-if="template[ee][ff].type==='LogicBoolean'" v-model="unit.turret[t][ff]">
                <input type="text" v-if="template[ee][ff].type==='string'" v-model="unit.turret[t][ff]">
                <input type="text" v-if="template[ee][ff].type==='int'" v-model.number="unit.turret[t][ff]">
                <input type="text" v-if="template[ee][ff].type==='float'" v-model.number="unit.turret[t][ff]">
                <input type="text" v-if="template[ee][ff].type==='time'" v-model="unit.turret[t][ff]">
                <div class="image file" v-if="template[ee][ff].type==='file (image)'">
                  <img :src="unit.turret[t][ff]" alt="">
                  <input type="file" accept="image/png" @change="base64encrypt(ee, ff)"
                         :id="template[ee].sectionKey+ t + ff">
                </div>
                <select v-if="template[ee][ff].type==='enum'" v-model="unit.turret[t][ff]">
                  <option v-for="g in template[ee][ff].enum" :key="g">{{ g }}</option>
                </select>
                <select v-if="template[ee][ff].type==='projectile'" v-model="unit.turret[t][ff]">
                  <option v-for="g in rangeArray(unit.projectile)" :key="'pro' + g">{{ g }}</option>
                </select>
              </div>
            </div>
            <button @click="newTurret">新建炮塔</button>
          </template>
          <template v-else-if="ee==='projectile'&&unit[ee]">
            <div v-for="t in rangeArray(unit.projectile)" :key="t">
              <h2>projectile{{ t }}</h2>
              <button @click="unit.projectile.splice(t,1)">删除炮弹</button>
              <!---
              <div style="position: static; text-align: center; display: block">
                <div style="text-align: center;position: center"><img :src="this.unit.graphics.image" style="margin: 0 auto"></div>
                <div style="text-align: center;position: center"><img :src="t.image" :style="turretStyle"></div>
              </div>
              --->
              <div v-for="ff in Object.keys(unit.projectile[t])" :key="ff">
                <span :title="template.projectile[ff].des">{{ template.projectile[ff].trans }}</span>
                <input type="checkbox" v-if="template[ee][ff].type==='bool'" v-model="unit.projectile[t][ff]">
                <input type="checkbox" v-if="template[ee][ff].type==='LogicBoolean'" v-model="unit.projectile[t][ff]">
                <input type="text" v-if="template[ee][ff].type==='string'" v-model="unit.projectile[t][ff]">
                <input type="text" v-if="template[ee][ff].type==='int'" v-model.number="unit.projectile[t][ff]">
                <input type="text" v-if="template[ee][ff].type==='float'" v-model.number="unit.projectile[t][ff]">
                <input type="text" v-if="template[ee][ff].type==='time'" v-model="unit.projectile[t][ff]">
                <div class="image file" v-if="template[ee][ff].type==='file (image)'">
                  <img :src="unit.projectile[t][ff]" alt="">
                  <input type="file" accept="image/png" @change="base64encrypt(ee, ff)"
                         :id="template[ee].sectionKey+ t + ff">
                </div>
                <select v-if="template[ee][ff].type==='enum'" v-model="unit.projectile[t][ff]">
                  <option v-for="g in template[ee][ff].enum" :key="g">{{ g }}</option>
                </select>
                <select v-if="template[ee][ff].type==='effect'" v-model="unit[ee][ff]">
                  <option v-for="g in rangeArray(unit.effect)" :key="'eff' + g">{{ g }}</option>
                </select>
              </div>
            </div>
            <button @click="newProjectile">新建炮弹</button>
          </template>
          <template v-else-if="ee==='leg'">
            <div v-for="t in rangeArray(unit[ee])" :key="t">
              <h2>{{ ee }}{{ t }}</h2>
              <button @click="unit[ee].splice(t,1)">删除腿脚</button>
              <!---
              <div style="position: static; text-align: center; display: block">
                <div style="text-align: center;position: center"><img :src="this.unit.graphics.image" style="margin: 0 auto"></div>
                <div style="text-align: center;position: center"><img :src="t.image" :style="turretStyle"></div>
              </div>
              --->
              <div v-for="ff in Object.keys(unit[ee][t])" :key="ff">
                <span :title="template[ee][ff].des">{{ template[ee][ff].trans }}</span>
                <input type="checkbox" v-if="template[ee][ff].type==='bool'" v-model="unit[ee][t][ff]">
                <input type="checkbox" v-if="template[ee][ff].type==='LogicBoolean'" v-model="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='string'" v-model="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='int'" v-model.number="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='float'" v-model.number="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='time'" v-model="unit[ee][t][ff]">
                <div class="image file" v-if="template[ee][ff].type==='file (image)'">
                  <img :src="unit[ee][t][ff]" alt="">
                  <input type="file" accept="image/png" @change="base64encrypt(ee, ff)"
                         :id="template[ee].sectionKey+ t + ff">
                </div>
                <select v-if="template[ee][ff].type==='enum'" v-model="unit[ee][t][ff]">
                  <option v-for="g in template[ee][ff].enum" :key="g">{{ g }}</option>
                </select>
              </div>
            </div>
            <button @click="newLeg">新建腿脚</button>
          </template>
          <template v-else-if="ee==='effect'">
            <div v-for="t in rangeArray(unit[ee])" :key="t">
              <h2>{{ ee }}{{ t }}</h2>
              <button @click="unit[ee].splice(t,1)">删除效果</button>
              <!---
              <div style="position: static; text-align: center; display: block">
                <div style="text-align: center;position: center"><img :src="this.unit.graphics.image" style="margin: 0 auto"></div>
                <div style="text-align: center;position: center"><img :src="t.image" :style="turretStyle"></div>
              </div>
              --->
              <div v-for="ff in Object.keys(unit[ee][t])" :key="ff">
                <span :title="template[ee][ff].des">{{ template[ee][ff].trans }}</span>
                <input type="checkbox" v-if="template[ee][ff].type==='bool'" v-model="unit[ee][t][ff]">
                <input type="checkbox" v-if="template[ee][ff].type==='LogicBoolean'" v-model="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='string'" v-model="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='int'" v-model.number="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='float'" v-model.number="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='time'" v-model="unit[ee][t][ff]">
                <div class="image file" v-if="template[ee][ff].type==='file (image)'">
                  <img :src="unit[ee][t][ff]" alt="">
                  <input type="file" accept="image/png" @change="base64encrypt(ee, ff)"
                         :id="template[ee].sectionKey+ t + ff">
                </div>
                <select v-if="template[ee][ff].type==='enum'" v-model="unit[ee][t][ff]">
                  <option v-for="g in template[ee][ff].enum" :key="g">{{ g }}</option>
                </select>
              </div>
            </div>
            <button @click="void(0)">新建效果</button>
          </template>
          <template v-else>
            <template v-for="ff in Object.keys(unit[ee])" :key="ff">
              <div v-if="true || template[ee][ff].isNecessary || template[ee][ff].isShow" class="unitdata">
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
                <select v-if="template[ee][ff].type==='effect'" v-model="unit[ee][ff]">
                  <option v-for="g in rangeArray(unit.effect)" :key="'eff' + g">{{ g }}</option>
                </select>
              </div>
            </template>
          </template>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
import {toRaw} from "vue";
import JSZip from "jszip";
import {saveAs} from "file-saver";
import {turretTmpl} from "@/components/TurretTmpl";
import {projectileTmpl} from "@/components/ProjectileTmpl";
import {legTmpl} from "@/components/LegTmpl";

export default {
  name: "UnitEditor",
  data() {
    return {
      templates: null,
      tmplSelect: null,
      unitSelect: null,
      unit: {},
      template: null,
      output: null,
      outZip: new JSZip(),
      units: {}
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
    this.$watch("unitSelect", (newValue) => {
      this.unit = this.units[newValue]
    })
    /*
    this.$watch('tmplSelect', (newValue) => {
      fetch("/templates/" + newValue.en + ".json", {
        method: "GET",
      }).then(resp => resp.json()).then(res => {
        this.unit = res
        for (let i in this.unit) {
          // stop using typical methods to deal with turrets here
          if (i === 'turret' || i === 'projectile' || i === 'leg' || i === 'effect') continue
          this.template[i].show = false
          for (let j in this.unit[i]) {
            console.log(i + j)
            this.template[i][j].isShow = true
          }
        }
      })
    })*/
  },
  methods: {
    newUnit() {
      let name = prompt("输入单位名称（不要与已有的重复！）")
      if (name) {
        fetch("/templates/" + this.tmplSelect.en + ".json", {
          method: "GET",
        }).then(resp => resp.json()).then(res => {
          this.units[name] = res
          for (let i in this.unit) {
            // stop using typical methods to deal with turrets here
            if (i === 'turret' || i === 'projectile' || i === 'leg' || i === 'effect') continue
            this.template[i].show = false
            for (let j in this.unit[i]) {
              console.log(i + j)
              this.template[i][j].isShow = true
            }
          }
        })
      }
    },
    saveUnit() {
      this.units[this.unitSelect]=this.unit
    },
    deleteUnit() {
      let res = confirm("确认删除" + this.unitSelect + "吗？")
      if (res) {
        delete this.units[this.unitSelect]
        this.unitSelect = null
      }
    },
    allModOutput() {
      let product = new JSZip()
      for (let u in this.units) {
        let result = ""
        let unitFolder = product.folder(u)
        let raw = toRaw(this.units[u])
        for (let ee in raw) {
          if (ee in {"core": {}, "graphics": {}, "attack": {}, "movement": {}}) {
            result += "[" + this.template[ee].sectionKey + "]\n"
            for (let ff in raw[ee]) {
              if (this.template[ee][ff].type === "file (image)") {
                unitFolder.file(this.template[ee].sectionKey + ff + '.png', raw[ee][ff].substring(raw[ee][ff].indexOf(',') + 1), {base64: true})
                result += ff + ":" + this.template[ee].sectionKey + ff + '.png\n'
              } else {
                result += ff + ":" + raw[ee][ff] + "\n"
              }
            }
          } else if (ee in {"turret": {}, "projectile": {}, "leg": {}, "effect":{}}) {
            for (let i = 0; i < raw[ee].length; i++) {
              result += "[" + ee + "_" + i + "]\n"
              for (let ent in raw[ee][i]) {
                if (this.template[ee][ent].type === "file (image)" && raw[ee][i][ent].startsWith("data")) {
                  let filePth = ee + i + this.template[ee][ent].key + '.png'
                  unitFolder.file(filePth, raw[ee][i][ent].substring(raw[ee][i][ent].indexOf(',') + 1), {base64: true})
                  result += ent + ":" + filePth + '\n'
                } else {
                  result += ent + ":" + raw[ee][i][ent] + "\n"
                }
              }
            }
          }
        }
        unitFolder.file(this.units[u].core.name + '.ini', result)
      }
      product.generateAsync({type: "blob"})
          .then(function (content) {
            saveAs(content, 'mod.zip')
          })
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
        } else if (ee in {"turret": {}, "projectile": {}, "leg": {}, "effect":{}}) {
          for (let i = 0; i < raw[ee].length; i++) {
            result += "[" + ee + "_" + i + "]\n"
            for (let ent in raw[ee][i]) {
              if (this.template[ee][ent].type === "file (image)" && raw[ee][i][ent].startsWith("data")) {
                let filePth = ee + i + this.template[ee][ent].key + '.png'
                this.outZip.file(filePth, raw[ee][i][ent].substring(raw[ee][i][ent].indexOf(',') + 1), {base64: true})
                result += ent + ":" + filePth + '\n'
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
      this.unit.turret.push(turretTmpl)
    },
    newProjectile() {
      this.unit.projectile.push(projectileTmpl)
    },
    newLeg() {
      this.unit.leg.push(legTmpl)
    },
    rangeArray(l) {
      let res = []
      for (let i = 0; i < l.length; i++) {
        res.push(i)
      }
      return res
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
