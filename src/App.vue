<template>
  <div id="app">
    <router-view />
    <close />
  </div>
</template>
<script>
import Close from "@/components/Close.vue";
let ipcRenderer = require("electron").ipcRenderer;

export default {
  name: "app",
  components: {
    Close
  },
  mounted() {
    //检查自动更新
    this.checkForUpdate();
  },
  methods: {
    checkForUpdate() {
      //检查自动更新
      ipcRenderer.send("checkForUpdate");

      ipcRenderer.on("message", (event, text) => {
        console.log(arguments);
        this.tips = text;
      });
      ipcRenderer.on("downloadProgress", (event, progressObj) => {
        console.log(progressObj);
        this.downloadPercent = progressObj.percent || 0;
      });
      ipcRenderer.on("isUpdateNow", () => {
        ipcRenderer.send("isUpdateNow");
      });
    }
  },
  beforeDestroy() {
    ipcRenderer.removeAll(["message", "downloadProgress", "isUpdateNow"]);
  }
};
</script>
<style lang="scss">
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font: 16px/2 microsoft yahei;
}
::-webkit-scrollbar {
  width: 8px;
  height: 1px;
}

::-webkit-scrollbar-thumb {
  /*滚动条里面小方块*/
  border-radius: 10px;
  box-shadow: inset 0 0 5px rgba(228, 57, 60, 0.2);
  background: #5f87d8;
  position: absolute;
}

::-webkit-scrollbar-track {
  /*滚动条里面轨道*/
  box-shadow: inset 0 0 5px rgba(228, 57, 60, 0.2);
  border-radius: 10px;
  background: #ededed;
  position: absolute;
}

.min_max_close {
  top: 0;
  position: absolute;
  right: 0;
  font-size: 22px;
  color: white;
  i {
    margin-right: 5px;
  }
}
</style>
