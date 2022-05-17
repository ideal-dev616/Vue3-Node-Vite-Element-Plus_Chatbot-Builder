<template>
  <el-container>
    <el-header class="header">
      <h3>Chatbot Builder</h3>
      <el-button type="primary"   @click="exportEditor">Show Node Relation</el-button>
    </el-header>
    <el-container class="container">
      <el-aside width="250px" class="column">
        <ul>
          <li v-for="n in listNodes" :key="n" draggable="true" :data-node="n.item" @dragstart="drag($event)" class="drag-drawflow" >
            <div class="node" :style="`background: ${n.color}`" >{{ n.name }}</div>
          </li>
        </ul>
      </el-aside>
      <el-main>
        <el-button type="primary" @click="generateLineText" class="line-option" v-if="lineOptionVisible" :style="{top:lineOptionTop, left:lineOptionLeft}">
          <svg xmlns="http://www.w3.org/2000/svg" width="30px" height="20px" viewBox="0 0 16 16"><g fill="none" fill-rule="evenodd"><path d="M0 0h16v16H0z"></path><path fill="#66b1ff" d="M12.63 13c.097 0 .194.023.281.066l1.74.87a.63.63 0 01.001 1.127l-1.74.87A.63.63 0 0112 15.37V15H1.5a.5.5 0 010-1l10.5-.001v-.37a.63.63 0 01.63-.629zM8.168 0c.508 0 .96.32 1.13.798l3.653 10.268a.7.7 0 01-.659.934h-.115a1.2 1.2 0 01-1.143-.834l-.739-2.308a.6.6 0 00-.571-.417H6.242a.601.601 0 00-.573.42l-.72 2.298A1.2 1.2 0 013.804 12h-.102a.693.693 0 01-.653-.926L6.703.798A1.2 1.2 0 017.833 0h.335zm-.246 2.52a.284.284 0 00-.196.2c-.125.46-.242.847-.35 1.163l-.768 2.326a.6.6 0 00.569.788h1.665a.6.6 0 00.57-.787l-.81-2.466a17.903 17.903 0 01-.267-.837l-.057-.193a.286.286 0 00-.356-.194z"></path></g></svg>
        </el-button>
        <div id="drawflow" @drop="drop($event)" @dragover="allowDrop($event)">
        </div>
        <div v-for="connect in connectTextList" :key="connect.index">
          <CConnectionText :text="connect.text" :connectionXCenter="connectionXCenter" :connectionYCenter="connectionYCenter" />
        </div>
      </el-main>
    </el-container>
  </el-container>
  <el-dialog
    v-model="dialogVisible"
    title="Node Relation Details"
    width="50%"
  >
    <span>Data:</span>
    <pre><code>{{dialogData}}</code></pre>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="dialogVisible = false">Cancel</el-button>
        <el-button type="primary" @click="dialogVisible = false"
          >Confirm</el-button
        >
      </span>
    </template>
  </el-dialog>
</template>

<script>
import { onMounted, shallowRef, h, getCurrentInstance, render, readonly, ref } from 'vue'

import Drawflow from 'drawflow'
import styleDrawflow from 'drawflow/dist/drawflow.min.css'
import style from '../assets/style.css' 

import CSpeak from './nodes/cSpeak.vue'
import CAudio from './nodes/cAudio.vue'
import CDisplay from './nodes/cDisplay.vue'
import CConnectionText from './connections/cText.vue'

export default {
  name: 'drawflow',
  components: {
    CConnectionText 
  },
  methods: {

  },
  setup() {
    const listNodes = readonly([
      {
        name: 'Speak',
        color: '#49494970',
        item: 'CSpeak',
        input:1,
        output:1
      },
      {
        name: 'Audio',
        color: '#49494970',
        item: 'CAudio',
        input:1,
        output:1
      },
      {
        name: 'Display',
        color: '#49494970',
        item: 'CDisplay',
        input:1,
        output:1
      },
    ])
   
    const lineOptionVisible = ref(false)
    const lineOptionTop = ref("100px")
    const lineOptionLeft = ref("800px")
    const selectedNodeId = ref(-1)
    const nodeSelected = ref(false)
    const connectionSelected = ref(false)
    const selectedConnectionIds = shallowRef([])
    const connectionXCenter = ref(-1)
    const connectionYCenter = ref(-1)
    const editor = shallowRef({})
    const dialogVisible = ref(false)
    const dialogData = ref({})
    const Vue = { version: 3, h, render };
    const internalInstance = getCurrentInstance()
    const posCenter = shallowRef({ x: 400, y: 300 })
    const connectTextList = shallowRef([
      // {
      //   id: "",
      //   text: "123",
      //   pos: {
      //     x: 400,
      //     y: 300,
      //   },
      // }
    ])
    internalInstance.appContext.app._context.config.globalProperties.$df = editor;
   
    function exportEditor() {
      dialogData.value = editor.value.export();
      dialogVisible.value = true;
    }
    
    function generateLineText() {
      console.log("generate line text");
      let text = "text";
      for(let i = 0; i < connectTextList.value.length; i++){
        text += "OK!";
      }

      const newObject = {
        text: text,
      }

      connectTextList.value = [
        ...connectTextList.value,
        newObject,
      ];
    }

    const drag = (ev) => {
      if (ev.type === "touchstart") {
        mobile_item_selec = ev.target.closest(".drag-drawflow").getAttribute('data-node');
      } else {
        ev.dataTransfer.setData("node", ev.target.getAttribute('data-node'));
      }
    }
    const drop = (ev) => {
      if (ev.type === "touchend") {
        var parentdrawflow = document.elementFromPoint( mobile_last_move.touches[0].clientX, mobile_last_move.touches[0].clientY).closest("#drawflow");
        if(parentdrawflow != null) {
          addNodeToDrawFlow(mobile_item_selec, mobile_last_move.touches[0].clientX, mobile_last_move.touches[0].clientY);
        }
        mobile_item_selec = '';
      } else {
        ev.preventDefault();
        var data = ev.dataTransfer.getData("node");
        addNodeToDrawFlow(data, ev.clientX, ev.clientY);
      }
    }
    const allowDrop = (ev) => {
      ev.preventDefault();
    }

    let mobile_item_selec = '';
    let mobile_last_move = null;
    function positionMobile(ev) {
      mobile_last_move = ev;
    }

    function addNodeToDrawFlow(name, pos_x, pos_y) {
      pos_x = pos_x * ( editor.value.precanvas.clientWidth / (editor.value.precanvas.clientWidth * editor.value.zoom)) - (editor.value.precanvas.getBoundingClientRect().x * ( editor.value.precanvas.clientWidth / (editor.value.precanvas.clientWidth * editor.value.zoom)));
      pos_y = pos_y * ( editor.value.precanvas.clientHeight / (editor.value.precanvas.clientHeight * editor.value.zoom)) - (editor.value.precanvas.getBoundingClientRect().y * ( editor.value.precanvas.clientHeight / (editor.value.precanvas.clientHeight * editor.value.zoom)));
    
      const nodeSelected = listNodes.find(ele => ele.item == name);
      editor.value.addNode(name, nodeSelected.input,  nodeSelected.output, pos_x, pos_y, name, {}, name, 'vue');
    }

    onMounted(() => {
      var elements = document.getElementsByClassName('drag-drawflow');
      for (var i = 0; i < elements.length; i++) {
        elements[i].addEventListener('touchend', drop, false);
        elements[i].addEventListener('touchmove', positionMobile, false);
        elements[i].addEventListener('touchstart', drag, false );
      }
      
      const id = document.getElementById("drawflow");
      editor.value = new Drawflow(id, Vue, internalInstance.appContext.app._context);
      editor.value.createCurvature = function (
        start_pos_x,
        start_pos_y,
        end_pos_x,
        end_pos_y
      ) {
        const center_x = (end_pos_x - start_pos_x) / 2 + start_pos_x
        const center_y = (end_pos_y - start_pos_y) / 2 + start_pos_y
        if(end_pos_y - start_pos_y <= 30 && end_pos_y - start_pos_y >= -30){
          return(
            ' M ' + start_pos_x + ' ' + start_pos_y + 
            ' L ' + end_pos_x + ' ' + end_pos_y
          )
        }else if(end_pos_y - start_pos_y > 30) {
          if(end_pos_x - start_pos_x <= 30 && end_pos_x - start_pos_x >= -30){
            return(
              ' M ' + start_pos_x + ' ' + start_pos_y + 
              ' L ' + (end_pos_x - 15) + ' ' + end_pos_y +
              ' M ' + (end_pos_x - 15 + 4) + ',' + end_pos_y + ' ' + (end_pos_x - 15) + ',' + (end_pos_y + 2.5) + ' ' + (end_pos_x - 15) + ',' + (end_pos_y - 2.5) + 'z'
            )
          }else if(end_pos_x - start_pos_x > 30){
            return (
              ' M ' + start_pos_x + ' ' + start_pos_y + 
              ' L ' + (center_x - 10) + ' ' + start_pos_y +
              ' M ' + (center_x - 10) + ' ' + start_pos_y +
              ' Q ' + center_x + ' ' + start_pos_y + ' ' + center_x + ' ' + (start_pos_y + 10) + 
              ' L ' + center_x + ' ' + (end_pos_y - 10) +
              ' M ' + center_x + ' ' + (end_pos_y - 10) +
              ' Q ' + center_x + ' ' + end_pos_y + ' ' + (center_x + 10) + ' ' + end_pos_y +
              ' L ' + (end_pos_x - 15) + ' ' + end_pos_y +
              ' M ' + (end_pos_x - 15 + 4) + ',' + end_pos_y + ' ' + (end_pos_x - 15) + ',' + (end_pos_y + 2.5) + ' ' + (end_pos_x - 15) + ',' + (end_pos_y - 2.5) + 'z'
            )
          }else {
            return (
              ' M ' + start_pos_x + ' ' + start_pos_y + 
              ' L ' + (center_x + 10) + ' ' + start_pos_y +
              ' M ' + (center_x + 10) + ' ' + start_pos_y +
              ' Q ' + center_x + ' ' + start_pos_y + ' ' + center_x + ' ' + (start_pos_y + 10) + 
              ' L ' + center_x + ' ' + (end_pos_y - 10) +
              ' M ' + center_x + ' ' + (end_pos_y - 10) +
              ' Q ' + center_x + ' ' + end_pos_y + ' ' + (center_x - 10) + ' ' + end_pos_y +
              ' L ' + (end_pos_x - 15) + ' ' + end_pos_y +
              ' M ' + (end_pos_x - 15 + 4) + ',' + end_pos_y + ' ' + (end_pos_x - 15) + ',' + (end_pos_y + 2.5) + ' ' + (end_pos_x - 15) + ',' + (end_pos_y - 2.5) + 'z'
            )
          }
        }else{
          if(end_pos_x - start_pos_x <= 30 && end_pos_x - start_pos_x >= -30){
            return(
              ' M ' + start_pos_x + ' ' + start_pos_y + 
              ' L ' + (end_pos_x - 15) + ' ' + end_pos_y +
              ' M ' + (end_pos_x - 15 + 4) + ',' + end_pos_y + ' ' + (end_pos_x - 15) + ',' + (end_pos_y + 2.5) + ' ' + (end_pos_x - 15) + ',' + (end_pos_y - 2.5) + 'z'
            )
          } else if(end_pos_x - start_pos_x > 30){
            return (
              ' M ' + start_pos_x + ' ' + start_pos_y + 
              ' L ' + (center_x - 10) + ' ' + start_pos_y +
              ' M ' + (center_x - 10) + ' ' + start_pos_y +
              ' Q ' + center_x + ' ' + start_pos_y + ' ' + center_x + ' ' + (start_pos_y - 10) + 
              ' L ' + center_x + ' ' + (end_pos_y + 10) +
              ' M ' + center_x + ' ' + (end_pos_y + 10) +
              ' Q ' + center_x + ' ' + end_pos_y + ' ' + (center_x + 10) + ' ' + end_pos_y +
              ' L ' + (end_pos_x - 15) + ' ' + end_pos_y +
              ' M ' + (end_pos_x - 15 + 4) + ',' + end_pos_y + ' ' + (end_pos_x - 15) + ',' + (end_pos_y + 2.5) + ' ' + (end_pos_x - 15) + ',' + (end_pos_y - 2.5) + 'z'
            )
          }else{
            return (
              ' M ' + start_pos_x + ' ' + start_pos_y + 
              ' L ' + (center_x + 10) + ' ' + start_pos_y +
              ' M ' + (center_x + 10) + ' ' + start_pos_y +
              ' Q ' + center_x + ' ' + start_pos_y + ' ' + center_x + ' ' + (start_pos_y - 10) + 
              ' L ' + center_x + ' ' + (end_pos_y + 10) +
              ' M ' + center_x + ' ' + (end_pos_y + 10) +
              ' Q ' + center_x + ' ' + end_pos_y + ' ' + (center_x - 10) + ' ' + end_pos_y +
              ' L ' + (end_pos_x - 15) + ' ' + end_pos_y +
              ' M ' + (end_pos_x - 15 + 4) + ',' + end_pos_y + ' ' + (end_pos_x - 15) + ',' + (end_pos_y + 2.5) + ' ' + (end_pos_x - 15) + ',' + (end_pos_y - 2.5) + 'z'
            )
          }
        }
      }
      editor.value.start();

      editor.value.on("connectionSelected", function(connection){ // data.output_id, data.input_id, data.output_class, data.input_class
        lineOptionVisible.value = true;
        selectedConnectionIds.value.push(connection.output_id, connection.input_id);
        const nodeData = editor.value.drawflow.drawflow.Home.data;
        let posX = [], posY = [];
        Object.keys(nodeData).map((key) => {
          if(key === connection.output_id || key === connection.input_id){
            posX.push(nodeData[key].pos_x);
            posY.push(nodeData[key].pos_y);
          }
        });
        lineOptionTop.value = Math.min(...posY) + 40 + "px";
        lineOptionLeft.value = 370 + Math.min(...posX) + (Math.max(...posX) - Math.min(...posX)) / 2 + "px";

        connectionXCenter.value = 396 + Math.min(...posX) + Math.abs(posX[0] - posX[1]) / 2;
        connectionYCenter.value = 130 + Math.min(...posY) + Math.abs(posY[0] - posY[1]) / 2;
      })

      editor.value.on("connectionUnselected", function(id){
        console.log("connectionUnselected => ", id);
        lineOptionVisible.value = false;
      });

      editor.value.on("nodeSelected", function(id){
        console.log("nodeSelected => ", id);
        selectedNodeId.value = id;
        nodeSelected.value = true;
        lineOptionVisible.value = true;
      });

      editor.value.on("nodeUnselected", function(id){
        console.log("nodeUnselected => ", id);
        nodeSelected.value = false;
        lineOptionVisible.value = false;
      });

      editor.value.on("mouseMove", function(position){
        if((nodeSelected.value) && selectedConnectionIds.value.includes(selectedNodeId.value)){
          const nodeData = editor.value.drawflow.drawflow.Home.data;
          let posX = [], posY = [];
          posX.push(nodeData[selectedConnectionIds.value[0]].pos_x, nodeData[selectedConnectionIds.value[1]].pos_x);
          posY.push(nodeData[selectedConnectionIds.value[0]].pos_y, nodeData[selectedConnectionIds.value[1]].pos_y);
          lineOptionTop.value = Math.min(...posY) + 40 + "px";
          lineOptionLeft.value = 370 + (Math.max(...posX) + Math.min(...posX)) / 2 + "px";
          connectionXCenter.value = 396 + Math.min(...posX) + Math.abs(posX[0] - posX[1]) / 2;
          connectionYCenter.value = 130 + Math.min(...posY) + Math.abs(posY[0] - posY[1]) / 2;
        }
      });
      
      editor.value.registerNode('CSpeak', CSpeak, {}, {});
      editor.value.registerNode('CAudio', CAudio, {}, {});
      editor.value.registerNode('CDisplay', CDisplay, {}, {});

      editor.value.import(
      {"drawflow": {
        "Home": {
          "data": {
            "7": {
              "id": 7,
              "name": "CSpeak",
              "data": {},
              "class": "CSpeak",
              "html": "CSpeak",
              "typenode": "vue",
              "inputs": {
                "input_1": {
                  "connections": []
                }
              },
              "outputs": {
                "output_1": {
                  "connections": [
                    {
                      "node": "8",
                      "output": "input_1"
                    }
                  ]
                }
              },
              "pos_x": 276,
              "pos_y": 131
            },
            "8": {
              "id": 8,
              "name": "CAudio",
              "data": {},
              "class": "CAudio",
              "html": "CAudio",
              "typenode": "vue",
              "inputs": {
                "input_1": {
                  "connections": [
                    {
                      "node": "7",
                      "input": "output_1"
                    }
                  ]
                }
              },
              "outputs": {
                "output_1": {
                  "connections": []
                }
              },
              "pos_x": 863,
              "pos_y": 346
            }
          }
        }
      }})
    })

    return {
      exportEditor, generateLineText, listNodes, drag, drop, allowDrop, dialogVisible, dialogData, lineOptionVisible, lineOptionTop, lineOptionLeft, nodeSelected, connectionSelected, selectedConnectionIds, selectedNodeId, connectTextList, connectionXCenter, connectionYCenter
    }
  }
}

</script>
<style scoped>
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #494949;
}
.container {
  min-height: calc(100vh - 100px);
}
.line-option {
  position: absolute;
  width: 56px;
  height: 50px;
  box-shadow: rgb(73 73 73) 0px 0px 0px 2px, rgb(104 104 104) 0px 5px 16px 0px;
  background: #2d2d2d;
  padding: 8px;
  z-index: 2;
}
.column {
  border-right: 1px solid #494949;
}
.column ul {
  padding-inline-start: 0px;
  padding: 10px 10px;
}
.column li {
  background: transparent;
}

.node {
  border-radius: 8px;
  border: 2px solid #494949;
  display: block;
  height: 50px;
  line-height: 40px;
  padding: 3px 10px;
  margin: 12px 0px;
  cursor: grab;
}
.node:hover{
  box-shadow: rgb(251 251 251 / 20%) 0px 1px 3px 0px, rgb(233 233 233 / 4%) 0px 0px 0px 1px;
}
.node:active{
  transition: transform 0 ease 0;
  transform: rotate(-2deg);
  cursor: grabbing;
}
.node:dragable{
  cursor: grabbing;
}

#drawflow {
  width: 100%;
  height: 100%;
  text-align: initial;
  background: #2b2c30;
  background-size: 20px 20px;
  background-image: radial-gradient(#494949 1px, transparent 1px);
}
</style>