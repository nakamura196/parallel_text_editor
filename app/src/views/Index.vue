<template>
  <v-app>
    <v-content>
      <v-toolbar :dark="true" flat>
        <v-toolbar-title>TEI Anchor Editor</v-toolbar-title>
        <v-spacer></v-spacer>
        <v-btn icon @click="export2">
          <v-icon>mdi-export</v-icon>
        </v-btn>
      </v-toolbar>

      <div :style="'height: '+height+'px; overflow-wrap: break-word; word-break: break-all;'">
        <splitpanes class="default-theme">
          <pane min-size="20">
            <v-card class="scroll vertical pa-5" id="main" style="background-color : #616161">
              <v-card class="pa-5">
                  <Hello v-on:parentMessage="messageLog" v-if="data2" :elements="data2.elements"></Hello>
                  </v-card>                  </v-card>
                
          </pane>
          <pane>
            <v-card class="scroll vertical pa-5" id="sub" style="background-color : #616161">
              <v-card class="pa-5">
                  <Hello v-on:parentMessage="messageLog" v-if="data" :elements="data.elements"></Hello>
                  </v-card>
                </v-card>
          </pane>
        </splitpanes>
      </div>
    </v-content>

    <v-row justify="center">
      <v-dialog v-model="dialogFlag" max-width="80%">
        <v-card class="pa-5" style="background-color : #616161">
          <template v-for="(element, index) in dialogData.elements">
            
            <template v-if="element.type=='text' && inputs.length > 0">
              <v-card :key="index" class="mb-5">
                <v-container class="pa-10">
                  <h3>{{element.text}}</h3>
                  <v-row class="mt-5">
                    <v-col cols="12" md="12">
                      <v-textarea
                        filled
                        v-model="inputs[index].text"
                        rows="1"
                        label="アンカーの挿入箇所より前のテキスト部分を入力してください。"
                        :hint="'ex. '+element.text.substring(0, element.text.length / 2)"
                      ></v-textarea>
                    </v-col>
                    
                  </v-row>
                  <v-row>
                    <v-col cols="12" md="12">
                      <v-text-field
                        label="挿入対象テキストのIDを入力してください。"
                        v-model="inputs[index].id"
                        filled
                        :hint="'ex. '+ (inputs[index].id ? inputs[index].id : 'http://example.org/id/123')"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12" md="12">
                      <v-btn
                        block
                        color="primary"
                        @click="add(index)"
                        x-large
                      >
                        登録
                      </v-btn>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card>
            </template>
          </template>

          <v-card class="mt-10 mb-5">
            <v-container class="pa-10">
              <h3>Optional: JSON Editor</h3>
              <v-jsoneditor class="my-5" v-model="json" :options="options" :plus="true" height="600px" @error="onError"/>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="primary darken-1" @click="updateDialogData()">更新</v-btn>
                <v-btn color="warning darken-1" text @click="$store.dispatch('setDialogFlag', false)">閉じる</v-btn>
              </v-card-actions>
            </v-container>
           </v-card>
        </v-card>
      </v-dialog>
    </v-row>

    <v-row justify="center">
      <v-dialog v-model="resultFlag" max-width="80%">
        
        <v-card class="pa-5">
          <h3 class="my-5">出力結果</h3>
          <v-textarea
            solo
            :value="result"
            rows="20"
          ></v-textarea>
        </v-card>
      </v-dialog>
    </v-row>

    
  </v-app>
</template>

<script>

// In your VueJS component.
import { Splitpanes, Pane } from "splitpanes";
import axios from "axios";
import VJsoneditor from 'v-jsoneditor/src/index'

import Hello from "../components/Hello.vue";

var convert = require("xml-js");

("use strict");
const crypto = require("crypto");
function md5hex(str /*: string */) {
  const md5 = crypto.createHash("md5");
  return md5.update(str, "binary").digest("hex");
}

export default {
  components: { Splitpanes, Pane, Hello, VJsoneditor },
  data: function() {
    return {
      e: null,
      data: null,
      data2: null,

      width: window.innerWidth,
      height: window.innerHeight,
      return_url: null,
      return_label: null,
      dialog_information: false,
      dialog_settings: false,
      url_main: "",
      url_sub: "",
      test: "",
      data_main: [],
      data_sub: [],
      data_main_e: [],
      data_sub_e: [],
      hightlights: [],
      m_s_id_map: [],
      s_m_id_map: [],
      w_l_id_map: {},
      l_w_id_map: {},
      image_map: {},
      label_main: "",
      label_sub: "",
      selected_manifests: [],
      mirador_path: "",
      direction: "vertical",
      layout: "2x2",

      json: [],

      inputs: [],

      options : {mode: 'code'},

      result : {},
      resultFlag : false
    };
  },
  computed: {
    subLineId : {
      get() { return this.$store.getters.subLineId},
      set(val) { this.$store.dispatch("setSubLineId", val)}
    },
    dialogFlag : {
      get() { return this.$store.getters.dialogFlag},
      set(val) { this.$store.dispatch("setDialogFlag", val)}
    },
    dialogData : {
      get() { return this.$store.getters.dialogData},
      set(val) { this.$store.dispatch("setDialogData", val)}
    }
  },
  mounted: function() {
    window.addEventListener("resize", this.handleResize);

    if (this.$route.query.u == null && location.hostname != "localhost") {
      location.href = "https://github.com/TEI-EAJ/parallel_text_viewer/";
    }

    let main = this.$route.query.main ? this.$route.query.main : "https://webpark5032.sakura.ne.jp/tmp/genji/01_main.xml" //"https://kouigenjimonogatari.github.io/tei/01.xml"
    let sub = this.$route.query.sub ? this.$route.query.sub : "https://webpark5032.sakura.ne.jp/tmp/genji/01.xml" //"https://genji.dl.itc.u-tokyo.ac.jp/data/tei/yosano/01.xml"

    this.url_sub = sub

    this.exec2main(main);
    this.exec2sub(sub);
  },
  watch: {
    $route() {
      this.search();
    },
    dialogData: function (val) {
      this.json = val.elements

      this.inputs = []

      if(this.json){
        for(let i = 0; i < this.json.length; i++){
          this.inputs.push({
            text : "",
            id : this.subLineId ? this.subLineId : ""
          })
        }
      }
      
    }
  },
  methods: {
    messageLog(message) {
      this.e = message;
      let line_id = Number(message.attributes.n)
      let line_uri = "https://w3id.org/kouigenjimonogatari/data/0"+('0000000000' + line_id).slice(-3)+"-01.json"
      let line_uuid = this.convert_uri(line_uri)
      this.clickIcon(line_uuid)
    },
    clickIcon(line_id) {
    
      let selected_manifests = this.selected_manifests;
      let params = [];

      for (let i = 0; i < selected_manifests.length; i++) {
        let selected_manifest = selected_manifests[i];
        if (!selected_manifest.selected) {
          continue;
        }
        let manifest_label = selected_manifest.label;
        let manifest_map = this.image_map[manifest_label].data;

        if (manifest_map[line_id]) {
          let member_id = manifest_map[line_id]
          let tmp = member_id.split("#xywh=")
          let canvas = tmp[0]
          let xywh = tmp[1].split(",")
          let y = Number(xywh[1]) - 150
          let h = Number(xywh[3]) + 150
          member_id = canvas+"#xywh="+xywh[0]+","+y + ","+xywh[2] + "," + h
          params.push({
            manifest: this.image_map[manifest_label].manifest,
            canvas: member_id
          });
        } 
      }
    },
    scroll(id) {
      let target_ids = [];
      let query = "";
      if (this.m_s_id_map[this.w_l_id_map[id]]) {
        target_ids = this.m_s_id_map[this.w_l_id_map[id]];
        query = "sub";
      } else if (this.l_w_id_map[this.s_m_id_map[id][0]]) {
        target_ids = this.l_w_id_map[this.s_m_id_map[id][0]];
        query = "main";
      }

      if (target_ids.length > 0) {
        let target_id = target_ids[0];
        if (this.direction == "vertical") {
          this.$SmoothScroll(
            document.querySelector("#" + target_id).getBoundingClientRect()
              .left +
              document.querySelector("#" + query).scrollLeft -
              document.querySelector("#" + query).getBoundingClientRect().left -
              document.querySelector("#" + query).getBoundingClientRect()
                .width /
                2 +
              document.querySelector("#" + target_id).getBoundingClientRect()
                .width,
            0.1,
            null,
            document.querySelector("#" + query),
            "x"
          );
        } else {
          this.$SmoothScroll(
            document.querySelector("#" + target_id).getBoundingClientRect()
              .top +
              document.querySelector("#" + query).scrollTop -
              document.querySelector("#" + query).getBoundingClientRect().top,
            0.1,
            null,
            document.querySelector("#" + query),
            "y"
          );
        }
      }
    },
    handleResize: function() {
      // resizeのたびにこいつが発火するので、ここでやりたいことをやる
      this.width = window.innerWidth;
      this.height = window.innerHeight;
    },
    
    exec2main(url) {

      axios
        .get(url, {
          //responseType: "document"
        })
        .then(response => {
          let xml_node = response.data;
          if (typeof xml_node == "string") {
            var dpObj = new DOMParser();
            xml_node = dpObj.parseFromString(xml_node, "text/xml");
          }
          let xml_str = new XMLSerializer().serializeToString(xml_node);
          var result = convert.xml2json(xml_str, { compact: false, spaces: 4 });
          this.data2 = JSON.parse(result);
        });
    },

    exec2sub(url/*, query_sub*/) {
      //console.log("query_sub", query_sub)

      axios
        .get(url, {
          //responseType: "document"
        })
        .then(response => {
          let xml_node = response.data;
          if (typeof xml_node == "string") {
            var dpObj = new DOMParser();
            xml_node = dpObj.parseFromString(xml_node, "text/xml");
          }
          this.xml = xml_node;
          let xml_str = new XMLSerializer().serializeToString(xml_node);
          var result = convert.xml2json(xml_str, { compact: false, spaces: 4 });
          this.data = JSON.parse(result);
        });

      
    },
    convert_uri: function(id) {
      let tmp = id.split("#");
      if (tmp.length != 2) {
        return "id-" + md5hex(id);
      }

      return "id-" + md5hex(tmp[0]) + "#" + tmp[1];
    },

    updateDialogData: function(){
      let newData = Object.create(this.dialogData)
      newData.elements = this.json
      this.dialogData = newData

      this.dialogFlag = false

      this.inputs = []
    },

    add(index) {
      let input = this.inputs[index]
      let elements = this.dialogData.elements

      let text = this.dialogData.elements[index].text
      let texts = [input.text, text.replace(input.text, "")]
  
      //削除
      elements.splice(index, 1)

      elements.splice(index, 0, {
          text : texts[0],
          type : "text"
      })

      elements.splice(index+1, 0, {
          name : "anchor",
          type : "element",
          attributes: {
            corresp : this.url_sub+"#"+input.id
          }
      })

      elements.splice(index+2, 0, {
          text : texts[1],
          type : "text"
      })

      this.dialogFlag = false

      this.inputs = []

      this.dialogData.elements = elements
    },
    export2(){
      var options = {compact: false, ignoreComment: true, spaces: 4};
      let result = convert.json2xml(this.data2, options)
      this.result = result
      this.resultFlag = true
    },
    onError(){
      //console.log("error")
    }
  },

  beforeDestroy: function() {
    window.removeEventListener("resize", this.handleResize);
  }
};
</script>

<style>
.scroll {
  height: 100%;
  /*overflow-y: auto;*/
  overflow-y: auto;
}

.vertical {
  -webkit-writing-mode: vertical-rl;
  -ms-writing-mode: tb-rl;
  writing-mode: vertical-rl;
}

html,
body,
#app {
  height: 100%;
  margin: 0;
}
</style>
