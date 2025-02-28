<template>
  <div class="upload-container">
    <div>
      <div class="input-btns" style="margin-bottom: 10px">
        <input
          ref="input"
          type="file"
          style="display: none"
          @change="handleChange"
          :multiple="multiple"
        />
        <div v-if="img" class="upload-img">
          <!-- v-for="(file,index) in fileInfo.length>0?fileInfo: files" -->
          <div v-for="(file, index) in files" :key="index" class="img-item">
            <div class="operation">
              <div class="action">
                <Icon
                  type="md-eye"
                  @click="previewImg(index)"
                  class="view"
                ></Icon>
                <Icon
                  type="md-close"
                  @click="removeFile(index)"
                  class="remove"
                ></Icon>
              </div>
              <div class="mask"></div>
            </div>

            <img :src="getImgSrc(file, index)" :onerror="errorImg" />
          </div>
          <div
            v-show="!autoUpload || (autoUpload && files.length < maxFile)"
            class="img-selector"
            :class="getSelector()"
          >
            <div class="selector" @click="handleClick">
              <Icon type="ios-camera"></Icon>
            </div>
            <div
              v-if="!autoUpload"
              class="s-btn"
              :class="{ readonly: changed }"
              @click="upload"
            >
              <div>{{ loadText }}</div>
            </div>
          </div>
        </div>
        <Button v-else @click="handleClick" icon="ios-cloud-upload-outline"
          >选择{{ img ? "图片" : "文件" }}</Button
        >

        <Button
          v-if="!autoUpload && !img"
          type="info"
          :disabled="changed"
          icon="md-arrow-round-up"
          @click="upload(true)"
          :loading="loadingStatus"
          >上传文件</Button
        >
      </div>
      <slot></slot>
      <div v-if="desc">
        <Alert show-icon
          >{{ getText() }}文件大小不超过{{ maxSize || 3 }}M</Alert
        >
      </div>
      <slot name="content"></slot>
      <div v-if="!img">
        <ul class="upload-list" v-show="fileList">
          <li class="list-file" v-for="(file, index) in files" :key="index">
            <a>
              <span @click="fileOnClick(index, file)">
                <Icon :type="format(file)"></Icon>
                {{ file.name }}
              </span>
              <!-- <span
              v-show="file.size / 1024>0"
              style="margin-left:15px;"
              >大小{{(file.size / 1024).toFixed(2)}}KB</span>-->
            </a>
            <span @click="removeFile(index)" class="file-remove">
              <Icon type="md-close"></Icon>
            </span>
          </li>
        </ul>
      </div>
      <slot name="tip"></slot>
    </div>
  </div>
</template>
<script>
export default {
  components: {},
  props: {
    desc: {
      //是否显示默认介绍
      //是否多选
      type: Boolean,
      default: false,
    },
    fileInfo: {
      //用于接收上传的文件，也可以加以默认值，显示已上传的文件，用户上传后会覆盖默认值
      type: Array,
      default: () => {
        return [];
      }, //格式[{name:'1.jpg',path:'127.0.01/1.jpg'}]
    },
    downLoad: {
      //是否可以点击文件下载
      type: Boolean,
      default: true,
    },
    multiple: {
      //是否多选
      type: Boolean,
      default: false,
    },
    maxFile: {
      //最多可选文件数量，必须multiple=true，才会生效
      type: Number,
      default: 5,
    },
    maxSize: {
      //文件限制大小3M
      type: Number,
      default: 3,
    },

    autoUpload: {
      //选择文件后是否自动上传
      type: Boolean,
      default: true,
    },
    img: {
      //图片类型  img>excel>fileTypes三种文件类型优先级
      type: Boolean,
      default: false,
    },
    excel: {
      //excel文件
      type: Boolean,
      default: false,
    },
    fileTypes: {
      //指定上传文件的类型
      type: Array,
      default: () => {
        return [];
      },
    },
    url: {
      //上传的url
      type: String,
      default: "",
    },
    uploadBefore: {
      //返回false会中止执行
      //上传前
      type: Function,
      default: (files) => {
        return true;
      },
    },
    uploadAfter: {
      //返回false会中止执行
      //上传后
      type: Function,
      default: (result, files) => {
        return true;
      },
    },
    onChange: {
      //选择文件时  //返回false会中止执行
      type: Function,
      default: (files) => {
        return true;
      },
    },
    // clear: {
    //   //上传完成后是否清空文件列表
    //   type: Boolean,
    //   default: true
    // },
    fileList: {
      //是否显示选择的文件列表
      type: Boolean,
      default: true,
    },
    fileClick: {
      //点击文件事件
      type: Function,
      default: (index, file, files) => {
        return true;
      },
    },
    removeBefore: {
      //移除文件事件
      type: Function,
      default: (index, file, files) => {
        return true;
      },
    },
    append: {
      //此属性已废弃，多文件上传，默认追加文件
      type: Boolean,
      default: false,
    },
    compress: {
      //开启图片压缩,后面根据需要再完善
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      errorImg: 'this.src="' + require("@/assets/imgs/error-img.png") + '"',
      changed: false, //手动上传成功后禁止重复上传，必须重新选择
      model: true,
      files: [],
      bigImg: "",
      loadingStatus: false,
      loadText: "上传文件",
    };
  },
  created() {
    //默认有图片的禁止上传操作
    if (this.fileInfo) {
      this.changed = true;
    }
    this.cloneFile(this.fileInfo);
  },
  watch: {
    fileInfo: {
      handler(files) {
        this.cloneFile(files);
      },
      deep: true,
    },
  },
  methods: {
    cloneFile(files) {
      this.files = files.map((x) => {
        return {
          name: x.name || "未定义文件名",
          path: x.path,
        };
      });
    },
    previewImg(index) {
      //查看大图预览模式待完
      this.base.previewImg(this.getImgSrc(this.files[index]));
      //  window.open(this.getImgSrc((this.files.length>0?this.files:this.fileInfo)[index]));
    },
    getSelector() {
      if (this.autoUpload) {
        return "auto-selector";
      }
      return "submit-selector";
    },
    getImgSrc(file, index) {
      if (file.hasOwnProperty("path")) {
        if (this.base.isUrl(file.path)) {
          return file.path;
        }
        //2020.12.27增加base64图片操作
        if (file.path.indexOf("/9j/") != -1) {
          return "data:image/jpeg;base64," + file.path;
        }
        if (file.path.substr(0, 1) == "/") {
          file.path = file.path.substr(1);
        }
        return this.http.ipAddress + file.path;
      }
      return window.URL.createObjectURL(file);
    },
    fileOnClick(index, file) {
      if (!this.fileClick(index, file, this.files)) {
        return;
      }
      //点击不下载
      if (!this.downLoad) {
        return;
      }
      if (!file.path) {
        this.$Message.error("请先上传文件");
        return;
      }
      this.base.dowloadFile(
        file.path,
        file.name,
        {
          Authorization: this.$store.getters.getToken(),
        },
        this.http.ipAddress
      );
    },
    getText() {
      if (this.img) {
        return "只能上传图片,";
      } else if (this.excel) {
        return "只能上传excel文件,";
      }
    },
    handleClick() {
      this.$refs.input.click();
    },
    handleChange(e) {
      //this.compress开启图片压缩,后面根据需要再完善
      // this.clearFiles();
      var result = this.checkFile(e.target.files);
      if (!result) {
        return;
      }

      this.changed = false;
      //如果传入了FileInfo需要自行处理移除FileInfo
      if (!this.onChange(e.target.files)) {
        return;
      }
      for (let index = 0; index < e.target.files.length; index++) {
        const element = e.target.files[index];
        element.input = true;
      }
      if (!this.multiple) {
        this.files.splice(0);
      }
      this.files.push(...e.target.files);

      this.$refs.input.value = null;
      if (this.autoUpload && result) {
        this.upload(false);
      }
    },
    removeFile(index) {
      //如果传入了FileInfo需要自行处理移除FileInfo
      //t移除文件
      let removeFile = this.files[index];
      //删除的还没上传的文件
      if (removeFile.input) {
        this.files.splice(index, 1);
      } else {
        this.fileInfo.splice(index, 1);
      }
      if (!this.removeBefore(index, removeFile, this.fileInfo)) {
        return;
      }
    },
    clearFiles() {
      this.files.splice(0);
    },
    getFiles() {
      return this.files;
    },
    upload(vail) {
      if (vail && !this.checkFile()) return false;
      if (!this.url) {
        return this.$Message.error({ duration: 5, content: "没有配置好Url" });
      }
      if (!this.files || this.files.length == 0) {
        return this.$Message.error({ duration: 5, content: "请选择文件" });
      }
      if (!this.uploadBefore(this.files)) {
        return;
      }
      var forms = new FormData();
      this.files.forEach(function (file) {
        if (file.input) {
          forms.append("fileInput", file, file.name);
        }
      });
      // forms.append("fileInput", this.files);
      this.loadingStatus = true;
      this.loadText = "上传中..";
      this.http
        .post(this.url, forms, this.autoUpload ? "正在上传文件" : "")
        .then(
          (x) => {
            // this.$refs.uploadFile.clearFiles();
            this.loadingStatus = false;
            this.loadText = "上传文件";
            if (!this.uploadAfter(x, this.files)) {
              this.changed = false;
              return;
            } else {
              this.changed = true;
            }
            this.$Message.success({ duration: 5, content: x.message });
            this.changed = x.status;
            if (!x.status) {
              // this.files = null;
              return;
            }
            //单选清除以前的数据
            //  if (!this.multiple) {
            this.fileInfo.splice(0);
            // }
            this.fileInfo.push(
              ...this.files.map((file) => {
                return { name: file.name, path: x.data + file.name };
              })
            );
            // this.files.forEach((file) => {
            //   this.fileInfo.push({ name: file.name, path: x.data + file.name });
            // });
            // if (this.clear) {
            //  this.clearFiles();
            // }
          },
          (error) => {
            this.loadText = "上传文件";
            this.loadingStatus = false;
          }
        );
    },
    format(file, checkFileType) {
      const format = file.name.split(".").pop().toLocaleLowerCase() || "";
      let fileIcon = "ios-document-outline";
      if (this.fileTypes.length > 0 && checkFileType != undefined) {
        if (this.fileTypes.indexOf(format) != -1) {
          return true;
        }
        return false;
      }
      if (
        checkFileType &&
        !(checkFileType instanceof Array) &&
        checkFileType != "img" &&
        checkFileType != "excel"
      ) {
        if (checkFileType.indexOf(format) > -1) {
          return true;
        } else {
          return false;
        }
      }

      if (
        checkFileType == "img" ||
        ["gif", "jpg", "jpeg", "png", "bmp", "webp"].indexOf(format) > -1
      ) {
        if (checkFileType == "img") {
          if (
            ["gif", "jpg", "jpeg", "png", "bmp", "webp"].indexOf(format) > -1
          ) {
            return true;
          } else {
            return false;
          }
        }
        fileIcon = "ios-image";
      }
      if (
        ["mp4", "m3u8", "rmvb", "avi", "swf", "3gp", "mkv", "flv"].indexOf(
          format
        ) > -1
      ) {
        fileIcon = "ios-film";
      }
      if (["mp3", "wav", "wma", "ogg", "aac", "flac"].indexOf(format) > -1) {
        fileIcon = "ios-musical-notes";
      }
      if (["doc", "txt", "docx", "pages", "epub", "pdf"].indexOf(format) > -1) {
        fileIcon = "md-document";
      }
      if (
        checkFileType == "excel" ||
        ["numbers", "csv", "xls", "xlsx"].indexOf(format) > -1
      ) {
        if (checkFileType == "excel") {
          if (["numbers", "csv", "xls", "xlsx"].indexOf(format) > -1) {
            return true;
          } else {
            return false;
          }
        }
        fileIcon = "ios-podium";
      }
      if (["keynote", "ppt", "pptx"].indexOf(format) > -1) {
        fileIcon = "ios-videocam";
      }

      return fileIcon;
    },
    beforeUpload() {},
    checkFile(inputFiles) {
      const files = this.files;

      if (
        this.multiple &&
        files.length + (inputFiles || []).length > (this.maxFile || 5)
      ) {
        this.$Message.error({
          duration: 5,
          content:
            "最多只能选【" +
            (this.maxFile || 5) +
            "】" +
            (this.img ? "张图片" : "个文件") +
            "",
        });
        return false;
      }
      if (!inputFiles) {
        inputFiles = this.files.filter((x) => {
          return x.input;
        });
      }
      let names = [];
      for (let index = 0; index < inputFiles.length; index++) {
        const file = inputFiles[index];
        if (names.indexOf(file.name) != -1) {
          file.name = "(" + index + ")" + file.name;
        }
        names.push(file.name);
        if (this.img && !this.format(file, "img")) {
          this.$Message.error({
            duration: 5,
            content: "选择的文件【" + file.name + "】只能是图片格式",
          });
          return false;
        }
        if (this.excel && !this.format(file, "excel")) {
          this.$Message.error({
            duration: 5,
            content: "选择的文件【" + file.name + "】只能是excel文件",
          });
          return false;
        }
        if (
          this.fileTypes &&
          this.fileTypes.length > 0 &&
          !this.format(file, this.fileTypes)
        ) {
          this.$Message.error({
            duration: 5,
            content:
              "选择的文件【" +
              file.name +
              "】只能是【" +
              this.fileTypes.join(",") +
              "】格式",
          });
          return false;
        }
        if (file.size > (this.maxSize || 3) * 1024 * 1024) {
          this.$Message.error({
            duration: 5,
            content:
              "选择的文件【" +
              file.name +
              "】不能超过:" +
              (this.maxSize || 3) +
              "M",
          });
          return false;
        }
      }
      return true;
    },
  },
};
</script>
<style lang="less" scoped>
.upload-list {
  list-style: none;
  .list-file {
    padding: 4px;
    color: #515a6e;
    border-radius: 4px;
    transition: background-color 0.2s ease-in-out;
    overflow: hidden;
    position: relative;

    font-size: 13px;
    .file-remove {
      display: none;
      right: 0;
      //  margin-left: 50px;
      color: #0e9286;
    }
  }
  .list-file:hover {
    cursor: pointer;
    .file-remove {
      display: initial;
    }
    color: #2d8cf0;
  }
}
.upload-container {
  display: inline-block;
  width: 100%;
  // padding: 10px;

  // min-height: 250px;
  border-radius: 5px;
  .alert {
    margin-top: 43px;
  }
  .button-group > * {
    float: left;
    margin-right: 10px;
  }
  .file-info > span {
    margin-right: 20px;
  }
}
.upload-img {
  display: inline-block;
  .img-item:hover .operation {
    display: block;
  }
  .img-item,
  .img-selector {
    position: relative;
    cursor: pointer;
    margin: 0 10px 10px 0;
    float: left;
    width: 70px;
    height: 70px;
    border: 1px solid #c7c7c7;
    overflow: hidden;
    border-radius: 5px;
    // width: 100px;
    img {
      margin: 0;
      padding: 0;
      width: 100%;
      height: 100%;
      object-fit: cover;
    }

    .operation {
      display: none;
      position: absolute;
      top: 0;
      bottom: 0;
      left: 0;
      right: 0;
      .action {
        opacity: 0.6;
        text-align: right;
        background: #727779;
        font-size: 17px;
        position: absolute;
        z-index: 90;
        width: 100%;
        bottom: 3px;
        bottom: 0;
        color: #ded5d5;
        padding-right: 7px;
        padding-bottom: 3px;
        line-height: 23px;
        // i {
        //   border-radius: 50%;
        //   background: white;
        // }
      }
      .mask {
        opacity: 0.6;
        background: #9e9e9e;
        top: 0;
        width: 100%;
        height: 100%;
        position: absolute;
      }
    }
  }
  .img-selector {
    font-size: 50px;
    text-align: center;
    i {
      top: -4px;
      position: relative;
    }
  }

  .auto-selector {
    .selector {
      line-height: 64px;
    }
  }
  .selector {
    color: #a0a0a0;
  }
  .submit-selector {
    .s-btn {
      line-height: 22px;
      font-size: 12px;
      top: -7px;
      // padding: 2px;
      position: relative;
      background: #2db7f5;
      color: white;
    }
    .selector {
      line-height: 50px;
    }
    .readonly {
      background: #8c8c8c;
    }
  }
}
.big-model {
  width: 100%;
  height: 100%;
  position: relative;
  .m-img {
  }
  .mask {
    position: absolute;
    opacity: 0.6;
    background: #eee;
    top: 0;
    width: 100%;
    height: 100%;
    position: absolute;
  }
}

.auto-upload {
  z-index: 9999999;
  width: 100%;
  height: 100%;
  position: fixed;
  top: 0;
  left: 0;
  .j-content {
    text-align: center;
    font-size: 17px;
    top: 40%;
    position: absolute;
    z-index: 999;
    left: 0;
    right: 0;
    width: 240px;
    /* height: 100%; */
    margin: auto;
    background: white;
    /* bottom: 30px; */
    line-height: 50px;
    border-radius: 6px;
    border: 1px solid #d2d2d2;
  }
  .mask {
    cursor: pointer;
    opacity: 0.6;
    width: 100%;
    height: 100%;
    background: #101010;
  }
}
</style>
