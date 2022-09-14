<template>
  <button @click="debugg" v-if="true">debug</button>
  <div id="navi">
    <div>
      <span>选择一个模板</span>
      <select v-model="tmplSelect">
        <option v-for="t in templates" :key="t" :value="t">{{ t.cn }}</option>
      </select>
      <button @click="newUnit">新建单位</button>
    </div>
    <div id="project buttons">
      <input type="file" style="display: none" id="projectFile" @change="loadProject">
      <button @click="clickLoadProject">载入工程</button>
      <button @click="saveProject">保存工程</button>
    </div>
    <span>选择一个单位来进行编辑：</span>
    <select v-model="unitSelect">
      <option v-for="e in Object.keys(project.units)" :value="e" :key="'unit'+e">
        {{ e }}
      </option>
    </select>
    <button @click="showTurret">刷新</button>
    <button @click="saveUnit">保存单位</button>
    <button @click="deleteUnit">删除单位</button>
    <button @click="allModOutput">导出zip</button>
  </div>
  <div id="uniteditor" v-if="unit" style="overflow: auto; height: 80%;width: auto">
    <template v-if="tmplSelect">
      <div v-for="ee in Object.keys(unit)" :key="ee.id">
        <button @click="template[ee].show=!template[ee].show" style="width: 50%;text-align: left"
                v-if="template[ee].sectionName">{{
            template[ee].sectionName
          }}
        </button>
        <div v-if="template[ee].show">
          <template v-if="ee === 'turret' || ee === 'projectile'">
            <div v-for="t in rangeArray(unit[ee])" :key="t">
              <h2>{{ ee }}{{ t }}</h2>
              <button @click="unit[ee].splice(t,1)">删除{{ template[ee].sectionName }}</button>
              <div v-if="ee === 'turret'" :id="ee + t">
                <canvas :id="ee + 'canvas' + t"></canvas>
              </div>
              <div v-for="ff in Object.keys(unit[ee][t])" :key="ff">
                <TransAndDes :trans="template[ee][ff].trans" :des="template[ee][ff].des" />
                <input type="checkbox" v-if="template[ee][ff].type==='bool'" v-model="unit[ee][t][ff]">
                <input type="checkbox" v-if="template[ee][ff].type==='LogicBoolean'" v-model="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='string'" v-model="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='int'" v-model.number="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='float'" v-model.number="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='time'" v-model="unit[ee][t][ff]">
                <div class="image file" v-if="template[ee][ff].type==='file (image)'">
                  <img :src="unit[ee][t][ff]" alt="">
                  <input type="file" accept="image/png" @change="base64encrypt(ee, t, ff)"
                         :id="template[ee].sectionKey+ t + ff">
                </div>
                <select v-if="template[ee][ff].type==='enum'" v-model="unit[ee][t][ff]">
                  <option v-for="g in template[ee][ff].enum" :key="g">{{ g }}</option>
                </select>
                <select v-if="template[ee][ff].type==='projectile'" v-model="unit[ee][t][ff]">
                  <option v-for="g in rangeArray(unit.projectile)" :key="'pro' + g">{{ g }}</option>
                </select>
              </div>
              <div class="newProp">
                <input type="text"
                       :id="ee + t + 'addItem'"
                       @change="textTriggeredHint(ee, '#' + ee + t + 'addItem')"
                @focus="textTriggeredHint(ee, '#' + ee + t + 'addItem')">
                <button @click="newItem(ee, t, '#' + ee + t + 'addItem')">+</button>
              </div>
            </div>
            <button @click="newComponentFunctions[ee]">新建{{ template[ee].sectionName }}</button>
          </template>
          <!--
          <template v-else-if="ee==='projectile'&&unit[ee]">
            <div v-for="t in rangeArray(unit.projectile)" :key="t">
              <h2>projectile{{ t }}</h2>
              <button @click="unit.projectile.splice(t,1)">删除炮弹</button>
              <div v-for="ff in Object.keys(unit.projectile[t])" :key="ff">
                <TransAndDes :trans="template.projectile[ff].trans" :des="template.projectile[ff].des" />
                <input type="checkbox" v-if="template[ee][ff].type==='bool'" v-model="unit.projectile[t][ff]">
                <input type="checkbox" v-if="template[ee][ff].type==='LogicBoolean'" v-model="unit.projectile[t][ff]">
                <input type="text" v-if="template[ee][ff].type==='string'" v-model="unit.projectile[t][ff]">
                <input type="text" v-if="template[ee][ff].type==='int'" v-model.number="unit.projectile[t][ff]">
                <input type="text" v-if="template[ee][ff].type==='float'" v-model.number="unit.projectile[t][ff]">
                <input type="text" v-if="template[ee][ff].type==='time'" v-model="unit.projectile[t][ff]">
                <div class="image file" v-if="template[ee][ff].type==='file (image)'">
                  <img :src="unit.projectile[t][ff]" alt="">
                  <input type="file" accept="image/png" @change="base64encrypt(ee, t, ff)"
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
              <div v-for="ff in Object.keys(unit[ee][t])" :key="ff">
                <TransAndDes :trans="template[ee][ff].trans" :des="template[ee][ff].des" />
                <input type="checkbox" v-if="template[ee][ff].type==='bool'" v-model="unit[ee][t][ff]">
                <input type="checkbox" v-if="template[ee][ff].type==='LogicBoolean'" v-model="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='string'" v-model="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='int'" v-model.number="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='float'" v-model.number="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='time'" v-model="unit[ee][t][ff]">
                <div class="image file" v-if="template[ee][ff].type==='file (image)'">
                  <img :src="unit[ee][t][ff]" alt="">
                  <input type="file" accept="image/png" @change="base64encrypt(ee, t, ff)"
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
              <div v-for="ff in Object.keys(unit[ee][t])" :key="ff">
                <TransAndDes :trans="template[ee][ff].trans" :des="template[ee][ff].des" />
                <input type="checkbox" v-if="template[ee][ff].type==='bool'" v-model="unit[ee][t][ff]">
                <input type="checkbox" v-if="template[ee][ff].type==='LogicBoolean'" v-model="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='string'" v-model="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='int'" v-model.number="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='float'" v-model.number="unit[ee][t][ff]">
                <input type="text" v-if="template[ee][ff].type==='time'" v-model="unit[ee][t][ff]">
                <div class="image file" v-if="template[ee][ff].type==='file (image)'">
                  <img :src="unit[ee][t][ff]" alt="">
                  <input type="file" accept="image/png" @change="base64encrypt(ee, t, ff)"
                         :id="template[ee].sectionKey+ t + ff">
                </div>
                <select v-if="template[ee][ff].type==='enum'" v-model="unit[ee][t][ff]">
                  <option v-for="g in template[ee][ff].enum" :key="g">{{ g }}</option>
                </select>
              </div>
            </div>
            <button @click="void(0)">新建效果</button>
          </template>
    --->
          <template v-else>
            <template v-for="ff in Object.keys(unit[ee])" :key="ff">
              <div v-if="true || template[ee][ff].isNecessary || template[ee][ff].isShow" class="unitdata">
                <TransAndDes :trans="template[ee][ff].trans" :des="template[ee][ff].des" />
                <input type="checkbox" v-if="template[ee][ff].type==='bool'" v-model="unit[ee][ff]">
                <input type="checkbox" v-if="template[ee][ff].type==='LogicBoolean'" v-model="unit[ee][ff]">
                <input type="text" v-if="template[ee][ff].type==='string'" v-model="unit[ee][ff]">
                <input type="text" v-if="template[ee][ff].type==='int'" v-model.number="unit[ee][ff]">
                <input type="text" v-if="template[ee][ff].type==='float'" v-model.number="unit[ee][ff]">
                <input type="text" v-if="template[ee][ff].type==='time'" v-model="unit[ee][ff]">
                <div class="image file" v-if="template[ee][ff].type==='file (image)'">
                  <img :src="unit[ee][ff]" alt="">
                  <input type="file" accept="image/png" @change="base64encrypt(ee, null, ff)"
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
            <div class="newProp">
              <input type="text"
                     :id="ee + 'addItem'"
                     @change="textTriggeredHint(ee, '#' + ee + 'addItem')"
                     @focus="textTriggeredHint(ee, '#' + ee + 'addItem')">
              <button @click="newItem(ee, null, '#' + ee + 'addItem')">+</button>
            </div>
          </template>
        </div>
      </div>
    </template>
  </div>
  <span id="hint" style="display: inline-block;position: fixed;bottom: 0;right: 0;width: 20%;overflow: auto;background-color: #EFF1FE">Hint</span>
</template>

<script>
import {toRaw} from "vue";
import JSZip from "jszip";
import {saveAs} from "file-saver";
import {turretTmpl} from "@/components/TurretTmpl";
import {projectileTmpl} from "@/components/ProjectileTmpl";
import {legTmpl} from "@/components/LegTmpl";
import TransAndDes from "@/components/TransAndDes";

export default {
  name: "UnitEditor",
  components: {TransAndDes},
  data() {
    return {
      templates: null,
      project:{units: {}},
      tmplSelect: null,
      unitSelect: null,
      unit: {},
      template: null,
      interval: null,
      showHint: false,
      newComponentFunctions: {
        turret: this.newTurret,
        projectile: this.newProjectile,
        leg: this.newLeg
      }
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
      this.unit = this.project.units[newValue]
      this.interval = setInterval(() => {this.showTurret()}, 200)
      //this.showTurret()
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
  watch: {
    /*"this.unit.turret": {
      handler() {
        this.showTurret()
      },
      deep: true
    },*/
  },
  methods: {
    textTriggeredHint(pos, patPos) {
      console.log("patPos="+patPos)
      let pat = document.querySelector(patPos).value
      let con = ""
      const re = new RegExp(pat + "*", "i")
      for (let entry in this.template[pos]) {
        console.log("pos="+pos)
        console.log("ent="+entry)
        if (re.test(entry)) {
          con += "<p onclick='document.querySelector("+patPos+").value="+entry+"'>" + entry + " " + this.template[pos][entry].trans + "</p>"
        }
      }
      document.querySelector("#hint").innerHTML = con
    },
    debugg() {
      console.log(this.$data)
    },
    showTurret() {
      for (let tur in Object.keys(this.unit.turret)) {
        let bgImg = new Image()
        bgImg.src = this.unit.graphics.image
        let turImg = new Image()
        turImg.src = this.unit.turret[tur].image
        let c = document.getElementById("turretcanvas" + tur)
        let ctx = c.getContext("2d")
        c.width = bgImg.width
        c.height = bgImg.height
        ctx.drawImage(bgImg, 0, 0)
        ctx.translate(bgImg.width/2 + this.unit.turret[tur].x, bgImg.height/2 - this.unit.turret[tur].y)
        ctx.rotate(Date.now()/1000%360)
        ctx.drawImage(turImg, - turImg.width/2, - turImg.height/2)
      }
    },
    newUnit() {
      if (this.tmplSelect === null) {
        alert("请先选择一个模板")
        return
      }
      let name = prompt("输入单位名称（不要与已有的重复！）")
      if (name) {
        fetch("./templates/" + this.tmplSelect.en + ".json", {
          method: "GET",
        }).then(resp => resp.json()).then(res => {
          this.project.units[name] = res
          /*for (let i in this.unit) {
            // stop using typical methods to deal with turrets here
            if (i === 'turret' || i === 'projectile' || i === 'leg' || i === 'effect') continue
            this.template[i].show = false
            for (let j in this.unit[i]) {
              console.log(i + j)
              this.template[i][j].isShow = true
            }
          }*/
        })
      }
    },
    newItem(pos1, pos2, item) {
      console.log("pos1=" + pos1)
      console.log("pos2=" + pos2)
      console.log("item=" + item)
      let t = document.querySelector(item).value
      console.log("t="+t)
      let v
      try {
        v = this.template[pos1][t].value
      }
      catch (e) {
        console.log("not found matching default value")
        return
      }
      if (pos2 === null) {
        this.unit[pos1][t] = v
      }
      else {
        this.unit[pos1][pos2][t] = v
      }
    },
    clickLoadProject() {
      document.getElementById("projectFile").click()
    },
    loadProject() {
      const reader = new FileReader()
      reader.readAsText(document.getElementById("projectFile").files[0])
      const outThis = this;
      reader.onload = function () {
        outThis.project = JSON.parse(this.result)
      }
    },
    saveProject() {
      let f = new File([JSON.stringify(this.project)], "project.json", { type: "text/plain;charset=utf-8" })
      saveAs(f)
    },
    saveUnit() {
      if (this.unitSelect === null) return
      this.project.units[this.unitSelect]=this.unit.clone()
    },
    deleteUnit() {
      let res = confirm("确认删除" + this.unitSelect + "吗？")
      if (res) {
        delete this.project.units[this.unitSelect]
        this.unitSelect = null
      }
    },
    allModOutput() {
      let product = new JSZip()
      for (let u in this.project.units) {
        let result = ""
        let unitFolder = product.folder(u)
        let raw = toRaw(this.project.units[u])
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
        unitFolder.file(this.project.units[u].core.name + '.ini', result)
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
    base64encrypt(e, f, g) {
      const reader = new FileReader()
      reader.readAsDataURL(document.getElementById(f===null?(this.template[e].sectionKey+ g):(this.template[e].sectionKey+f+g)).files[0])
      const outThis = this;
      reader.onload = function () {
        if (f===null) {outThis.unit[e][g] = this.result}
        else {outThis.unit[e][f][g] = this.result}
        this.showTurret()
      }
    },
    newTurret() {
      this.unit.turret.push(JSON.parse(JSON.stringify(turretTmpl)))
    },
    newProjectile() {
      this.unit.projectile.push(JSON.parse(JSON.stringify(projectileTmpl)))
    },
    newLeg() {
      this.unit.leg.push(JSON.parse(JSON.stringify(legTmpl)))
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
    fetch("./templates/templates.txt", {
      method: "GET",
    }).then(resp => resp.json()).then(res => this.templates = res)
    fetch("./templates/tmpl.json", {
      method: "GET",
    }).then(resp => resp.json()).then(res => {
      this.template = res
    })
  },
}
</script>

<style scoped>
.unitdata {
  margin-left: 0;
}
</style>
