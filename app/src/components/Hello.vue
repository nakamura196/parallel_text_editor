<template>
  <span>

    

    <template v-for="(obj, key) in elements">
      <template v-if="obj.text">
        <span :key="key">{{obj.text}}</span>
      </template>
      <template
        v-else-if="obj.name == 'p' || 
        obj.name == 'head' || 
        obj.name == 'ab' || 
        obj.name == 'list' || 
        obj.name == 'listPerson' || 
        obj.name == 'item' || 
        obj.name == 'person'"
      >
        <span :class="'tei-'+obj.name" class="my-5" :key="key">
          <Hello
            v-on:parentMessage="messageLog"
            :key="key"
            v-if="obj.elements"
            :elements="obj.elements"
            :parent="obj.name"
          ></Hello>
        </span>
      </template>
      <template v-else-if="obj.name == 'date'">
        <v-tooltip bottom :key="key">
          <template v-slot:activator="{ on }">
            <span v-on="on" :style="style(obj.name)">
              <Hello :key="key" v-if="obj.elements" :elements="obj.elements" :parent="obj.name"></Hello>
            </span>
          </template>
          <span>{{obj.attributes}}</span>
        </v-tooltip>
      </template>
      <template v-else-if="obj.name == 'teiHeader'">
        <!-- 
        <v-sheet :key="key" class="pa-5 ma-5" color="grey lighten-3">
          <Hello
            v-on:parentMessage="messageLog"
            :key="key"
            v-if="obj.elements"
            :elements="obj.elements"
            :parent="obj.name"
          ></Hello>
        </v-sheet>
        -->
      </template>
      <template v-else-if="obj.name == 'title'">
        <h2 :key="key" class="my-5">
          <Hello
            v-on:parentMessage="messageLog"
            :key="key"
            v-if="obj.elements"
            :elements="obj.elements"
            :parent="obj.name"
          ></Hello>
        </h2>
      </template>
      <template
        v-else-if="(obj.name == 'eaj:ruby') || (obj.attributes && obj.attributes.type == 'ruby')"
      >
        <ruby :key="key">
          <Hello :elements="obj.elements[0].elements"></Hello>
          <rt>{{obj.elements[2].elements[0].text}}</rt>
        </ruby>
      </template>
      <template
        v-else-if="obj.name == 'persName' || 
        obj.name == 'placeName' || 
        obj.name == 'name' || 
        obj.name == 'rs' ||
        obj.name == 'speaker' || 
        obj.name == 'bibl'
        "
      >
        <v-tooltip bottom :key="key">
          <template v-slot:activator="{ on }">
            <span
              v-on="obj.attributes ? on : null"
              :class="'tei-'+obj.name"
              :key="key"
              :style="parent != 'person' && parent != 'respStmt' ? style(obj.name) : 'margin-right : 10px; margin-left : 10px;'"
              @click="parent != 'person' && parent != 'respStmt' ? $emit('parentMessage', obj) : ''"
              :id="obj.id"
            >
              <Hello
                v-on:parentMessage="messageLog"
                :key="key"
                v-if="obj.elements"
                :elements="obj.elements"
                :parent="obj.name"
              ></Hello>
            </span>
          </template>
          <span>{{obj.attributes}}</span>
        </v-tooltip>
      </template>

      <!-- 青空自動TEI用 -->
      <template v-else-if="obj.name == 'seg' && obj.attributes && obj.attributes.rendition">
          <span :class="'tei-'+obj.attributes.rendition" :style="obj.attributes.style" :key="key">
            <Hello
              v-on:parentMessage="messageLog"
              :key="key"
              v-if="obj.elements"
              :elements="obj.elements"
              :parent="obj.name"
            ></Hello>
          </span>
      </template>

      <!-- Main @click="anchorClick(obj)" -->

      <template v-else-if="obj.name == 'anchor'">
        
        <span :key="key" 
        v-on:mouseover="mouseover(obj.attributes['corresp'].split('#')[1])" :style="obj.attributes['corresp'].split('#')[1] === lineId ? 'background-color : rgba(255,165,0, 0.5);' : ''"
        >
          <v-icon color="blue darken-2">mdi-anchor</v-icon>
        </span>
      </template>

      <template v-else-if="obj.name == 'seg' && obj.attributes.corresp">
        <span :key="key">

          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
              <v-btn icon color="primary" 
              @click="idClick(obj)" dark v-on="on"
              class="mb-5"
              >
              <v-icon>mdi-tooltip-edit</v-icon>
            </v-btn>
            </template>
            {{obj.attributes.corresp}}
          </v-tooltip>
          
          <span :class="'tei-'+obj.name" @click="anchorClick(obj)">
            <span
              v-if="obj.attributes && obj.attributes.facs && obj.attributes.facs.startsWith('#')"
              @click="$emit('parentMessage', obj)"
            >
            <img
                src="https://iiif.dl.itc.u-tokyo.ac.jp/images/iiif.png"
                style="width: 30px"
                class="mr-2"
              />
            </span>
            <Hello
              v-on:parentMessage="messageLog"
              :key="key"
              v-if="obj.elements"
              :elements="obj.elements"
              :parent="obj.name"
            ></Hello>
          </span>
        </span>
      </template>

      <!-- Sub 
      :style="obj.attributes['xml:id'] === subLineId ? 'color : #ff5252; font-weight: bold;' : ''"
      -->

      <template v-else-if="obj.name == 's'">
        <component :is="childDiv(obj) == 'div' ? 'div' : 'span'" :key="key" :style="obj.attributes['xml:id'] === lineId ? 'background-color : rgba(255,165,0, 0.5);' : ''"
        v-on:mouseover="mouseover(obj.attributes['xml:id'])" @click="selectSub(obj.attributes['xml:id'])"
        >
          <span style="color : #1976D2;" :style="obj.attributes['xml:id'] === subLineId ? 'color : #ff5252;' : ''">{{obj.attributes["xml:id"]}}</span>
          
          <span :class="'tei-'+obj.name" >
            <Hello
              v-on:parentMessage="messageLog"
              :key="key"
              v-if="obj.elements"
              :elements="obj.elements"
              :parent="obj.name"
            ></Hello>
          </span>
        </component>
      </template>

      <template v-else-if="obj.name != 'figure'">
        <span :class="'tei-'+obj.name" :key="key">
          <!--
          <span
            v-if="obj.attributes && obj.attributes.facs && obj.attributes.facs.startsWith('#')"
            @click="$emit('parentMessage', obj)"
          >
            <img
                src="https://iiif.dl.itc.u-tokyo.ac.jp/images/iiif.png"
                style="width: 30px"
                class="mr-2"
              />
          </span>
          -->
          <Hello
            v-on:parentMessage="messageLog"
            :key="key"
            v-if="obj.elements"
            :elements="obj.elements"
            :parent="obj.name"
          ></Hello>
        </span>
      </template>
    </template>

    
  </span>
</template>

<script>

import { uuid } from "vue-uuid";

export default {
  name: "Hello",
  props: ["elements", "flg", "parent"],
  data() {
    return {
      highlight_line : "",
      sharedState: this.$store.getters,
    };
  },
  computed: {
    lineId : {
      get() { return this.$store.getters.lineId}
    },
    subLineId : {
      get() { return this.$store.getters.subLineId}
    }
  },
  mounted() {
    this.init();
  },
  /*
  watch: {
    elements: function() {
      this.init();
    }
  },
  */
  methods: {
    init() {
      let elements = this.elements;
      if (elements != null) {
        for (let i = 0; i < elements.length; i++) {
          let obj = elements[i];
          obj.id = uuid.v1();
        }
      }
    },
    messageLog(dat) {
      this.$emit("parentMessage", dat);
    },
    mouseover: function(line_id){
      this.highlight_line = line_id
      this.$store.dispatch("setLineId", line_id)
    },
    selectSub: function(line_id){
      this.$store.dispatch("setSubLineId", line_id)
    },
    idClick: function(obj){
      this.$store.dispatch("setDialogFlag", true)
      this.$store.dispatch("setDialogData", obj)
    },
    anchorClick: function(){
      console.log("scroll !!!")
    },
    childDiv: function(obj){
      if(!obj.elements[0].attributes){
        return null
      } else {
        if(obj.elements[0].attributes.rendition == "div"){
          return "div"
        } else {
          return "span"
        } 
      }
    }
  }
};
</script>