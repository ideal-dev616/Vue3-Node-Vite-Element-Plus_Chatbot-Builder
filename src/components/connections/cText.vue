<template>
  <div ref="el" class="connect-text-wrapper" :style="`left: ${Math.abs(connectionXCenter - width/2)}px; top: ${connectionYCenter - height/2}px`">
    <input 
    class="title reactive" 
    type='text' 
    value=""
    @keyup="onKeyUp">
  </div>
</template>

<script>

export default {
    props: [ 'text', 'connectionXCenter', 'connectionYCenter' ],
    components: {
    },
    data() {
      return {
        width: 40,
        height: 30,
      }
    },
    mounted() {
      this.updateSize();
      console.log("cConnection Text Size => w: ", this.width + ", h: " + this.height);
    },
    methods: {
      updateSize(){
        this.width = this.$refs.el.clientWidth;
        this.height = this.$refs.el.clientHeight;
      },
      onKeyUp(e){
        if(e.target.value.length < 4){
          e.target.style.width = 40 + 'px';
          this.width = 40;
        } else {
          const txtfieldW = 17 + (e.target.value.length * 6.3);
          e.target.style.width = txtfieldW + 'px';
          this.width = txtfieldW;
        }
      },
    },
    // setup(){}
}
</script>

<style scoped>
  .connect-text-wrapper {
    width: auto;
    position: absolute;
    border: solid 1px #4fa186;
    border-radius: 5px;
  }

  input.reactive {
    width: 40px;
    color: #bbb;
    background: rgb(43, 44, 48);
    border: 0px;
    outline: 0;
    padding: 3px 6px;
    text-align: center;
  }
  input.reactive:focus{
    outline: 0;
  }

</style>