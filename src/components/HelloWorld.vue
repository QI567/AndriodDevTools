<template>
  <div class="hello">
    <el-upload
      class="upload-demo"
      drag
      action=""
      :limit="1"
      :auto-upload="false"
      :onChange="handleChange"
    >
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
    </el-upload>
    <div>
      <el-select v-model="type" placeholder="请选择">
        <el-option
          v-for="item in options"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        >
        </el-option>
      </el-select>
      <el-button @click="save">保存</el-button>
    </div>
    <div class="grid">
      <div class="item-row" :key="name" v-for="(item, name) in fileList">
        <el-image
          style="width: 100px; height: 100px"
          :src="item.img"
          fit="fit"
        />
        <span>{{ name }}</span>
        <el-input
          style="width: 200px"
          size="medium"
          v-model="item.modifyName"
        />
      </div>
    </div>
  </div>
</template>

<script>
import JSZip from "jszip";
export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  data() {
    return {
      type: "mipmap",
      fileList: {},
      fullFileList: [],
      options: [
        {
          value: "drawable",
          label: "drawable",
        },
        {
          value: "mipmap",
          label: "mipmap",
        },
      ],
    };
  },
  methods: {
    save() {
      var zip = new JSZip();
      const tempFiles = []
      for (let key in this.fullFileList) {
          if (!this.fullFileList[key].dir) {
            tempFiles.push(this.fullFileList[key]);
          }
      }
      let count = tempFiles.length;
      for (let file of tempFiles) {
        const arr = file.name.split("/");
        let dir = arr[0];
        dir = this.type + "-" + dir.split("-")[1];
        let name = arr[1];
        name = this.fileList[name].modifyName;
        file.name = dir+'/'+name;
        file.async("base64").then((base) => {
          count--
          var dir_zip = zip.folder(dir);
          dir_zip.file(name, base, {base64: true});
          if(count == 0) {
            zip.generateAsync({ type: "blob" }).then(function (content) {
              let url = window.URL.createObjectURL(new Blob([content]));
              let link = document.createElement("a");
              link.style.display = "none";
              link.href = url;
              link.setAttribute("download", 'mipmap.zip');
              document.body.appendChild(link);
              link.click();
            }).catch(e=>{
              console.log(e)
            })
          }
        })
      }
      
    },
    handleChange(file) {
      var zip = new JSZip();
      zip.loadAsync(file.raw, { optimizedBinaryString: true }).then((res) => {
        this.fullFileList = res.files;
        const tempFiles = [];
        for (let key in res.files) {
          //判断是否是目录
          if (!res.files[key].dir) {
            tempFiles.push(res.files[key]);
          }
        }
        const fileList = {};
        let count = tempFiles.length;
        for (let item of tempFiles) {
          const name = item.name.split("/")[1];
          const itemFile = zip.file(item.name);
          itemFile.async("base64").then((base) => {
            count--;
            fileList[name] = {
              name: name,
              modifyName: name,
              img: "data:image/png;base64," + base,
            };
            if (count == 0) {
              this.fileList = fileList;
            }
          });
        }
      });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.grid {
  display: flex;
  flex-direction: row;
  align-items: center;
  flex-wrap: wrap;
}
.item-row {
  width: 33.33%;
  display: flex;
  flex-direction: row;
  align-items: center;
}
</style>
