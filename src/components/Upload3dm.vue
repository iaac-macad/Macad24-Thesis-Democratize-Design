<template>
    <div>

        <input name="uploadfile"  type="file" @change="uploadFile" />
        <!-- <button v-if="selectedFile" @click="uploadFile">Upload File</button> -->


        <!-- <div class="button">
            <label for='upload3dm' class='customUpload'
              title="Upload the preprocessed .3dm model to calculate the coverage of the sensors">Upload 3dm</label>
            <input type="file" accept=".3dm" id="upload3dm" hidden />
            <span id="3dm-chosen" class="fileText">No file chosen</span>
          </div> -->

    </div>
</template>
  
<script setup>
import { ref } from 'vue';
import { base64ByteArray } from "@/scripts/compute.js";

let uploadDoc = ref(null);

const emits = defineEmits(['encoded3dm']);

const selectedFile = ref(null);

const uploadFile = async (event) => {
    selectedFile.value = event.target.files[0];
    if (!selectedFile.value) {
        return;
    }else{
        console.log(selectedFile.value);
    }


    // get file
    let  file = selectedFile.value

    // try to open 3dm file
    const buffer3dm = await file.arrayBuffer()
    const arr = new Uint8Array(buffer3dm)
    const b64ba = base64ByteArray(arr)

    uploadDoc.value = b64ba

    emits('encoded3dm', uploadDoc.value);

    // selectedFile.value = null;

}
</script>
  
<style scoped>
/* Add styles here if needed */
</style>