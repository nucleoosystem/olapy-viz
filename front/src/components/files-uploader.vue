<template>
  <div class="container">
    <!--UPLOAD-->
    <form enctype="multipart/form-data" novalidate v-if="isInitial || isSaving">
      <!--<h1>Upload csv files</h1>-->
      <div class="dropbox">
        <input type="file" multiple :name="uploadFieldName" :disabled="isSaving"
               @change="filesChange($event.target.name, $event.target.files); fileCount = $event.target.files.length"
               accept="text/csv" class="input-file">
        <p v-if="isInitial">
          Drag your file(s) here to begin<br> or click to browse
        </p>
        <p v-if="isSaving">
          Uploading {{ fileCount }} files...
        </p>
      </div>
    </form>
    <div v-if="isFailed">
      <h2>Uploaded failed.</h2>
      <p>
        <a href="javascript:void(0)" @click="reset()">Try again</a>
      </p>
      <pre>{{ uploadError }}</pre>
    </div>
  </div>
</template>

<script>

  const STATUS_INITIAL = 0,
    STATUS_SAVING = 1,
    STATUS_SUCCESS = 2,
    STATUS_FAILED = 3;

  import axios from 'axios';

  export default {
    data() {
      return {
        uploadedFilesColumns: [],
        uploadError: null,
        currentStatus: null,
        uploadFieldName: "files",
        fileCount: null
      };
    },
    computed: {
      isInitial() {
        return this.currentStatus === STATUS_INITIAL;
      },
      isSaving() {
        return this.currentStatus === STATUS_SAVING;
      },
      isFailed() {
        return this.currentStatus === STATUS_FAILED;
      },
    },
    methods: {
      reset() {
        // reset form to initial state
        this.currentStatus = STATUS_INITIAL;
        this.uploadedFilesColumns = [];
        this.uploadError = null;
      },
      save(formData) {
        // upload data to the server
        this.currentStatus = STATUS_SAVING;
        axios.post("api/cubes/add", formData)
          .then(x => {
            this.uploadedFilesColumns = [].concat(x.data);
            // if (x.data.facts != null) {
            //   console.log(x.data.facts);
            //   this.uploadedFiles.push(x.data.facts);
            //   // this.$emit("UploadStatus", "success");
            // }
            // else {
            //   this.$emit("SelectInputStatus", "toConfig");
            // }
            this.$emit('uploadedFilesColumns', this.uploadedFilesColumns);
            this.currentStatus = STATUS_SUCCESS;
          })
          .catch(err => {
            this.uploadError = err.response;
            this.currentStatus = STATUS_FAILED;
            this.$emit("SelectInputStatus", "failed");
          });
      },
      filesChange(fieldName, fileList) {
        // handle file changes
        const formData = new FormData();

        if (!fileList.length) return;

        // append the files to FormData
        Array.from(Array(fileList.length).keys()).map(x => {
          formData.append(fieldName, fileList[x], fileList[x].name);
        });

        // save it
        this.save(formData);
      },
    },
    mounted() {
      this.reset();
    },
  };
</script>

<style lang="scss" scoped>
  .dropbox {
    outline: 2px dashed grey; /* the dash box */
    outline-offset: -10px;
    background: lightcyan;
    color: dimgray;
    /*padding: 10px 10px;*/
    min-height: 200px; /* minimum height */
    position: relative;
    cursor: pointer;
  }

  .input-file {
    opacity: 0; /* invisible but it's there! */
    width: 100%;
    height: 200px;
    position: absolute;
    cursor: pointer;
  }

  .dropbox:hover {
    background: lightblue; /* when mouse over to the drop zone, change color */
  }

  .dropbox p {
    font-size: 1.2em;
    text-align: center;
    padding: 50px 0;
  }

  .container {
    width: 100%;
    margin-top: 7px;
  }
</style>
