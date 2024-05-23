<template>
  <div class="publish-show">
    <!-- 上传内容 -->
    <div class="container" v-click-outside="onClickOutside" :style="containerStyle">
      <div class="main" @click="clickInput">
        <custom-input placeholder="发表你的说说" ref="customInputRef" @custom-focus="footerShow = true">
        </custom-input>
      </div>
      <div class="footer" :class="footerShow ? 'footer-show' : ''">
        <div class="left">
          <emotion placement="bottom-start" @enter-emotion="(e: number) => enterEmotion(e)"></emotion>
          <i class="bi bi-card-image add-imgs" @click="showImgList = true"></i>
        </div>
        <div class="right">
          <el-button size="small" @click="customInputRef.setInnerHTML('')">清空</el-button>
          <el-button type="primary" size="small" @click="publisHandle">点击发表</el-button>
        </div>
      </div>
    </div>
    <!-- 上传图片视频 -->
    <el-card style="margin-top: 10px; position: relative" @click="clickImgList" shadow="never" v-show="showImgList">
      <el-button style="position: absolute; top: 10px; right: 10px; z-index: 1" icon="CloseBold" size="small"
        @click.stop="showImgList = false" />
      <el-upload ref="upload" class="el-upload" action="#" list-type="picture-card" :auto-upload="false"
        v-model:file-list="fileList">
        <template #file="{ file }">
          <img class="el-upload-list__item-thumbnail" v-if="file.fileType === FileType.image" :src="file.url" alt="" />
          <template v-if="file.fileType === FileType.video">
            <img class="el-upload-list__item-thumbnail" v-if="file.secondUrl" :src="file.secondUrl" alt="" />
            <div class="el-upload-list__item-thumbnail" v-else  :v-loading="true"></div>
          </template>
          <span class="el-upload-list__item-actions">
            <span class="el-upload-list__item-preview" @click="handlePictureCardPreview(file)">
              <el-icon v-if="file.fileType === FileType.image"><zoom-in /></el-icon>
              <el-icon v-if="file.fileType === FileType.video">
                <VideoPlay />
              </el-icon>
            </span>
            <span class="el-upload-list__item-delete" @click="handleRemove(file)">
              <el-icon>
                <Delete />
              </el-icon>
            </span>
          </span>
        </template>
        <div>
        </div>
        <el-icon class="el-icon--upload">
          <PictureFilled />
        </el-icon>
        <div class="el-upload__text">
          图片/视频
        </div>
      </el-upload>

      <el-dialog v-model="dialogVisible" width="900">
        <div class="media">
          <img v-if="dialogFile.fileType === FileType.image" :src="dialogFile.url" alt="Preview Image" fit="contain" />
          <video v-if="dialogFile.fileType === FileType.video" :src="dialogFile.url" autoplay controls />
        </div>
      </el-dialog>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue';
import CustomInput from "./CustomInput.vue";
import emotion from "./emotion/index.vue";
import type { UploadFile, UploadUserFile } from "element-plus";
import { ElMessage } from "element-plus";
import { imageType, videoType, FileType } from '@/utils/FileUtils/FileType';
import { handleVideoFileSelect } from '@/utils/FileUtils/VideoFile';


const props = defineProps<{
  backgroundColor?: string;
}>();

const emit = defineEmits<{
  (e: "publish-handle", content: string, imageFiles: UploadUserFile[]): void;
}>();

const publisHandle = async () => {
  console.log('fileList.value :>> ', fileList.value);
  // return;
  if (customInputRef.value.getInnerHTML() == "") {
    ElMessage.info({
      offset: 180,
      message: "请先输入要发表的内容",
      showClose: true,
    });
  } else {
    emit("publish-handle", customInputRef.value.getInnerHTML(), fileList.value);
    // await nextTick()

    // await upload.value!.submit()
    // footerShow.value = showImgList.value = false;
    // customInputRef.value.clearInnerHTML();
  }
};

//#region 文字部分

const containerStyle = {
  backgroundColor: props.backgroundColor,
};

const customInputRef = ref();

const clickInput = () => {
  customInputRef.value.focusTextArea();
};

const footerShow = ref(false); // 发表栏的选项是否展示
const notClickEmo = ref(true); // 不是点击表情
const notClickUpdImg = ref(true); // 不是点击上传图片
const closeFooterOptBox = computed(
  () => notClickEmo.value && notClickUpdImg.value && !showImgList.value
); // 是否关闭选项框

const onClickOutside = () => {
  if (closeFooterOptBox.value && customInputRef.value.getInnerHTML() == "") {
    footerShow.value = false;
  }
  notClickEmo.value = notClickUpdImg.value = true;
};

const enterEmotion = (e: number) => {
  console.log("fileList :>> ", fileList);
  notClickEmo.value = false;
  customInputRef.value.enterEmotion(`/src/assets/emotions/${e}.gif`);
};

//#endregion

//#region 上传图片

const upload = ref();

const showImgList = ref(false); // 显示上传图片列表
const clickImgList = () => {
  // 点击列表区域
  notClickUpdImg.value = false;
  footerShow.value = true;
};

const fileList = ref<UploadUserFile[]>([]);

const fileListLength = computed(() => fileList.value.length - 1)
watch(fileListLength, async (newLength, oldLength) => {
  // 添加图文件时 // 已知文件的添加和删除每次变化的数量为 1，添加文件是在数组的末尾。
  if (newLength > oldLength) {
    const file: any = fileList.value[fileListLength.value]
    if (imageType.has(file.raw.type)) {
      file['fileType'] = FileType.image
    }
    else if (videoType.has(file.raw.type)) {
      file['fileType'] = FileType.video
      const [secondUrl, secondFile] = await handleVideoFileSelect(file);
      file['secondUrl'] = secondUrl;
      file['secondRaw'] = secondFile;
    }
  }
})


const dialogFile = ref();
const dialogVisible = ref(false);

const handleRemove = (file: UploadFile) => {
  upload.value.handleRemove(file)
  console.log(fileList.value)

}
const handlePictureCardPreview = (file: any) => {
  dialogFile.value = file
  dialogVisible.value = true
}
//#endregion

</script>

<style lang="scss" scoped>
.publish-show {
  .container {
    border: 1px solid var(--el-border-color-light);
    border-radius: 4px;


    .main {
      border: 1.5px solid $base-gray;
      background-color: $base-gray;

      &:focus-within {
        background-color: white;
        border: 1.5px solid rgb(255, 191, 71);
        border: 1.5px solid #79bbff;
      }

      margin: 9px 10px;
      border-radius: 4px;
      min-height: 60px;
      max-height: 400px;
      overflow: auto;
      cursor: text;
      padding-bottom: 10px;
    }

    .footer {
      display: flex;
      // align-items: center;
      justify-content: space-between;
      padding: 0 10px;
      width: 100%;
      overflow: hidden;
      height: 0px;
      transition: all 0.3s;
      margin-bottom: 0;

      .left {
        display: flex;
        align-items: center;

        .add-imgs {
          font-size: 20px;
          margin-left: 8px;

          &:hover {
            color: #e4a4ff;
            cursor: pointer;
            transition: color 0.1s;
          }
        }
      }
    }

    .footer-show {
      // height: 27px;
      height: 24px;
      margin-bottom: 8px;
    }
  }

  .el-card {
    .el-upload {
      cursor: default;
    }

    .el-dialog {
      .media {
        width: 100%;
        height: 500px;
        background-color: $base-gray;
        display: flex;
        align-items: center;
        justify-content: center;

        img {
          width: 100%;
          height: 100%;
          object-fit: contain;
        }

        video {
          width: 100%;
          height: 100%;
          object-fit: contain;
        }
      }
    }
  }

  .imgs {
    height: 0;
  }
}
</style>