<template>
  <div class="flex flex-col items-center mb-8">
    <h1 class="text-gray-800 text-xl md:text-2xl font-semibold my-6">Upload File</h1>
    <choose-file ref="chooseFile" @VitekId="getVitekId" @AddFile="getFile" />
    <button
      ref="uploadButton"
      @click="showPopUpConfirm"
      :disabled="!file || !vitek_id"
      :class="{ 'opacity-50 cursor-not-allowed': !file || !vitek_id }"
      class="bg-blue-2 text-white-1 font-medium py-2 px-6 rounded"
    >
      Upload
    </button>
    <file-upload-log ref="fileUploadLog"/>
  </div>

  <!-- Pop-Up -->
  <pop-up
    :showPopUp="show_popup_confirm"
    @OnConfirm="onConfirm"
    @OnCancel="onCancel"
  >
    <template v-slot:popup-header>
      <h2 class="font-sarabun font-bold">Confirm File Upload</h2>
    </template>
    <template v-slot:popup-body>
      <p class="font-sarabun">คุณต้องการจะอัปโหลดไฟล์ "{{ file.name }}" ใช่หรือไม่?</p>
    </template>
  </pop-up>

  <!-- Loader -->
  <div
    v-if="show_loading"
    class="fixed right-0 left-0 top-0 z-50 flex justify-center items-center h-full opacity-25 bg-black"
  >
    <loader />
  </div>
</template>

<script>
import ChooseFile from "../components/ChooseFile.vue";
import FileUploadLog from "../components/FileUploadLog.vue";
import PopUp from "../components/PopUp.vue"
import Loader from "../components/Loader.vue";

export default {
  name: "UploadFilePage",
  components: {
    ChooseFile,
    FileUploadLog,
    PopUp,
    Loader,
  },
  data() {
    return {
      file: null,
      vitek_id: null,
      show_popup_confirm: false,
      show_loading: false,
    };
  },
  methods: {
    getVitekId(vitek_id) {
      this.vitek_id = vitek_id;
      // console.log(this.vitek_id)
    },
    getFile(file) {
      this.file = file;
      // console.log(this.file)
    },
    uploadFile() {
      this.show_loading = true;
      let params = { vitek_id: this.vitek_id };
      let formData = new FormData();
      formData.append("in_file", this.file);
      this.axios
        .post(`${this.host}/api/upload`, formData, { params })
        .then((response) => {
          if (response.data.status == "success") {
            // console.log(response.data.data);
            this.$refs.chooseFile.clearFile();
            this.$refs.fileUploadLog.getLogs();
            this.$refs.fileUploadLog.currentPage = 1;
            this.show_loading = false;
          } else {
            this.show_loading = false;
          }
        })
        .catch((error) => {
          console.log(error);
          this.show_loading = false;
        });
    },
    showPopUpConfirm() {
      this.show_popup_confirm = true;
    },
    onConfirm() {
      this.show_popup_confirm = false;
      this.uploadFile();
      // this.gotoLog();
    },
    onCancel() {
      this.show_popup_confirm = false;
    },
    gotoLog() {
      let top = this.$refs.uploadButton.offsetTop;
      window.scrollTo(0, top-20);
    }
  },
};
</script>

<style scoped></style>
