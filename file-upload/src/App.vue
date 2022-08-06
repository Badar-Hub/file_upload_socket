<template>
  <div>
    <input
      :modelValue="files"
      @change="uploadFile"
      type="file"
      multiple
      id="upload_input"
    />

    <!-- <div class="row">
      <div class="images" id="images"></div>
    </div> -->
    <div v-if="files && files.length > 0" class="row">
      <div v-for="(file, i) in files" :key="i" class="brdr-br q-ma-sm q-pa-md">
        <div class="row full-width">
          <q-icon name="grid_view" class="q-my-auto q-mr-sm" size="md" />
          <h6 class="q-my-sm">{{ file.name }}</h6>
        </div>
        <p class="text-body2 q-my-xs">{{ file.size }}</p>
        <q-linear-progress size="10px" :value="file.progress" />
      </div>
    </div>
    <div id="images"></div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
// import axios from "axios";
import { io } from "socket.io-client";
import SocketIOFileUpload from "socketio-file-upload";

const files = ref([]);

function bytesToSize(bytes) {
  var sizes = ["Bytes", "KB", "MB", "GB", "TB"];
  if (bytes == 0) return "0 Byte";
  var i = parseInt(Math.floor(Math.log(bytes) / Math.log(1024)));
  return Math.round(bytes / Math.pow(1024, i), 2) + " " + sizes[i];
}

const uploadFile = (e) => {
  for (const file of e.target.files) {
    const fileModel = {
      name: file.name,
      size: bytesToSize(file.size),
      progress: 0,
    };
    console.log(fileModel);
    files.value.push(fileModel);
  }
};
onMounted(() => {
  const socket = io.connect("http://localhost:8000");
  const socketFileUpload = new SocketIOFileUpload(socket);
  socketFileUpload.listenOnInput(document.getElementById("upload_input"));

  // Do something on upload progress:
  socketFileUpload.addEventListener("progress", function (event) {
    console.log("heel3qwf");
    const percent = (event.bytesLoaded / event.file.size) * 100;
    console.log(Array.from(files.value), event.file.name);
    const file = files.value.find((x) => x.name === event.file.name);
    if (file) {
      file.progress = percent;
    }
    console.log(event);
  });

  // Do something when a file is uploaded:
  socketFileUpload.addEventListener("complete", function (event) {
    console.log(event.detail.name);
    var img = document.createElement("img");
    img.setAttribute("style", "float:left;width:500px;height:300px");
    img.src = event.detail.name;
  });
});
</script>

<style lang="scss">
.brdr-br {
  border: 1px solid #929292;
  border-radius: 8px;
}
</style>
