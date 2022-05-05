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
        <div id="drawflow" @drop="drop($event)" @dragover="allowDrop($event)"></div>
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
import Drawflow from 'drawflow'
import styleDrawflow from 'drawflow/dist/drawflow.min.css'
import style from '../assets/style.css' 
import { onMounted, shallowRef, h, getCurrentInstance, render, readonly, ref } from 'vue'
import CSpeak from './nodes/cSpeak.vue'
import CAudio from './nodes/cAudio.vue'
import CDisplay from './nodes/cDisplay.vue'

export default {
  name: 'drawflow',
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
   
    const editor = shallowRef({})
    const dialogVisible = ref(false)
    const dialogData = ref({})
    const Vue = { version: 3, h, render };
    const internalInstance = getCurrentInstance()
    internalInstance.appContext.app._context.config.globalProperties.$df = editor;
   
    function exportEditor() {
      dialogData.value = editor.value.export();
      dialogVisible.value = true;
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
        if(end_pos_y - start_pos_y <= 20 && end_pos_y - start_pos_y >= -20){
          return(
            ' M ' + start_pos_x + ' ' + start_pos_y + 
            ' L ' + end_pos_x + ' ' + end_pos_y
          )
        }else if(end_pos_y - start_pos_y > 20) {
          if(end_pos_x - start_pos_x <= 20 && end_pos_x - start_pos_x >= -20){
            return(
              ' M ' + start_pos_x + ' ' + start_pos_y + 
              ' L ' + end_pos_x + ' ' + end_pos_y
            )
          }else if(end_pos_x - start_pos_x > 20){
            return (
              ' M ' + start_pos_x + ' ' + start_pos_y + 
              ' L ' + (center_x - 10) + ' ' + start_pos_y +
              ' M ' + (center_x - 10) + ' ' + start_pos_y +
              ' Q ' + center_x + ' ' + start_pos_y + ' ' + center_x + ' ' + (start_pos_y + 10) + 
              ' L ' + center_x + ' ' + (end_pos_y - 10) +
              ' M ' + center_x + ' ' + (end_pos_y - 10) +
              ' Q ' + center_x + ' ' + end_pos_y + ' ' + (center_x + 10) + ' ' + end_pos_y +
              ' L ' + end_pos_x + ' ' + end_pos_y
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
              ' L ' + end_pos_x + ' ' + end_pos_y
            )
          }
        }else{
          if(end_pos_x - start_pos_x <= 20 && end_pos_x - start_pos_x >= -20){
            return(
              ' M ' + start_pos_x + ' ' + start_pos_y + 
              ' L ' + end_pos_x + ' ' + end_pos_y
            )
          } else if(end_pos_x - start_pos_x > 20){
            return (
              ' M ' + start_pos_x + ' ' + start_pos_y + 
              ' L ' + (center_x - 10) + ' ' + start_pos_y +
              ' M ' + (center_x - 10) + ' ' + start_pos_y +
              ' Q ' + center_x + ' ' + start_pos_y + ' ' + center_x + ' ' + (start_pos_y - 10) + 
              ' L ' + center_x + ' ' + (end_pos_y + 10) +
              ' M ' + center_x + ' ' + (end_pos_y + 10) +
              ' Q ' + center_x + ' ' + end_pos_y + ' ' + (center_x + 10) + ' ' + end_pos_y +
              ' L ' + end_pos_x + ' ' + end_pos_y
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
              ' L ' + end_pos_x + ' ' + end_pos_y
            )
          }
        }
      }
      editor.value.start();
      
      editor.value.registerNode('CSpeak', CSpeak, {}, {});
      editor.value.registerNode('CAudio', CAudio, {}, {});
      editor.value.registerNode('CDisplay', CDisplay, {}, {});

      editor.value.import({"drawflow":{"Home":{"data":{"5":{"id":5,"name":"CAudio","data":{"script":"(req,res) => {\n console.log(req);\n}"},"class":"CAudio","html":"CAudio","typenode":"vue","inputs":{"input_1":{"connections":[{"node":"6","input":"output_1"}]}},"outputs":{"output_1":{"connections":[]}},"pos_x":1000,"pos_y":117},"6":{"id":6,"name":"CSpeak","data":{"url":"localhost/add", "method": "post"},"class":"CSpeak","html":"CSpeak","typenode":"vue","inputs":{"input_1":{"connections":[{"node":"6","input":"output_1"}]}},"outputs":{"output_1":{"connections":[{"node":"5","output":"input_1"}]}},"pos_x":137,"pos_y":89}}}}})
    })

    return {
      exportEditor, listNodes, drag, drop, allowDrop, dialogVisible, dialogData
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