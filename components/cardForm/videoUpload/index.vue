<template>
  <div>
    <el-upload
      class="video-uploader"
      :action="action"
      name="vediourl"
      :show-file-list="false"
      :on-success="handleVideoSuccess"
      :before-upload="beforeUploadVideo"
      :on-progress="uploadVideoProcess">
      <video
        v-if="bindData != '' && videoFlag == false"
        :src="bindData"
        class="videobox"
        controls="controls">
        您的浏览器不支持视频播放
      </video>
      <i v-else-if="bindData =='' && videoFlag == false" class="el-icon-plus video-uploader-icon"></i>
      <el-progress
        v-if="videoFlag == true"
        type="circle"
        :percentage="videoUploadPercent"
        class="videoProgress">
      </el-progress>
    </el-upload>
    <P class="videoDer">请保证视频格式(flv、mp4、avi、wmv、rmvb)正确，且不超过{{limit}}M</P>
  </div>
</template>
<script>
export default {
  name: "VideoUpload",
  props: {
    value: {
      type: String,
      default: ""
    },
    action: {
      type: String,
      default: ""
    },
    name: {
      type: String,
      default: "vediourl"
    },
    limit: {
      type: Number,
      default: 200
    }
  },
  methods: {
    beforeUploadVideo(file) {
      const limit = file.size / 1024 / 1024 < this.limit;
      if (
        [
          "video/quicktime",
          "video/mp4",
          "video/flv",
          "video/avi",
          "video/wmv",
          "video/rmvb"
        ].indexOf(file.type) == -1
      ) {
        this.$alert(
          "常用的视频格式有：mkv、mp4、flv、avi、wmv、rmvb。请选择以上视频格式进行上传！",
          "请选择正确的视频格式",
          {
            confirmButtonText: "朕知道了"
          }
        );
        return false;
      }
      if (!limit) {
        this.$message.error("视频大小不能超过" + this.limit + "MB哦!");
        return false;
      }
    },
    uploadVideoProcess(event, file, fileList) {
      this.videoFlag = true;
      this.videoUploadPercent = file.percentage.toFixed(0);
    },
    handleVideoSuccess(res, file) {
      // 获取上传图片地址
      this.videoFlag = false;
      this.videoUploadPercent = 0;
      if (res.code == 200) {
        this.bindData = res.data.src;
      } else {
        this.$message.error("视频上传失败，请重新上传！");
      }
    }
  },
  data() {
    return {
      bindData: this.value,
      videoFlag: false, // video process
      videoUploadPercent: 0 // video process
    };
  },
  watch: {
    videoUploadPercent(val) {
      this.videoUploadPercent = Number(val);
    },
    value(value) {
      this.bindData = value;
    },
    bindData(value) {
      this.$emit("input", value);
    }
  }
};
</script>
<style rel='stylesheet/scss' lang="scss" scoped>
.video-uploader {
  display: flex;
  justify-content: center;
  background-color: #fbfdff;
  border: 1px dashed #c0ccda;
  border-radius: 6px;
  box-sizing: border-box;
  width: 320px;
  height: 200px;
  cursor: pointer;
  line-height: 200px;
  vertical-align: top;
}
.video-uploader .el-upload:hover {
  border-color: #409eff;
}
.video-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 108px;
  height: 108px;
  line-height: 108px;
  text-align: center;
}
.videobox {
  width: 320px;
  height: 200px;
}
.videoDer {
  margin: 2px 0 0 0;
  color: #ccc;
  font-size: 12px;
  user-select: none;
}
.videoProgress {
  margin-top: calc(50% - 30px);
}
</style>