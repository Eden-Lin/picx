<template>
  <div class="selector-wrapper" v-if="getImageCardCheckedNum">
    <div class="selector-left-box">
      <el-checkbox
        :label="checked ? '取消全选' : '全选'"
        v-model="checked"
        @change="triggleFullCheck"
      ></el-checkbox>
      <div class="item">已选择 {{ getImageCardCheckedNum }} 张图片</div>
      <div class="item cancel-select-btn" @click="cancelPick">取消选择</div>
    </div>
    <div class="selector-right-box">
      <el-tooltip placement="top" content="批量复制外链">
        <i class="el-icon-copy-document btn-icon" @click="copyLink"></i>
      </el-tooltip>
      <el-tooltip placement="top" content="批量删除图片">
        <i class="el-icon-delete btn-icon" @click="batchDeleteImage"></i>
      </el-tooltip>
    </div>
    <textarea class="temp-batch-externalink"></textarea>
  </div>
</template>
<script lang="ts" setup>
import { computed, onMounted, defineEmits, watch, ref } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { useStore } from '@/store'
import { UploadedImageModel } from '@/common/model/upload.model'
import copyExternalLink from '@/common/utils/copy-external-link'
import { delelteBatchImage } from '@/common/utils/delete-image-card'
import { ExternalLinkType } from '@/common/model/externalLink.model'
import { deleteStatusEnum } from '@/common/model/delete.model'

// eslint-disable-next-line no-undef
const props = defineProps({
  currentDirImageList: {
    type: Array,
    default: () => []
  }
})

const emits = defineEmits(['update:initImageList'])

const store = useStore()
const checked = ref(false)

const getImageCardCheckedArr = computed(() => store.getters.getImageCardCheckedArr)
const userConfigInfo = computed(() => store.getters.getUserConfigInfo).value
const getImageCardCheckedNum = computed(() => getImageCardCheckedArr.value.length || 0)

watch(
  () => getImageCardCheckedNum.value,
  (newVal, oldVal) => {
    const newValCheckedNum = props.currentDirImageList.length
    if (newVal === newValCheckedNum) {
      checked.value = true
    } else {
      checked.value = false
    }
  }
)

function copyLink() {
  copyExternalLink(getImageCardCheckedArr.value, ExternalLinkType.cdn, '批量复制链接成功')
}

function cancelPick() {
  props.currentDirImageList.forEach((item: UploadedImageModel) => {
    if (item.checked) {
      // eslint-disable-next-line no-param-reassign
      item.checked = false
    }
  })
}

async function batchDeleteImage() {
  if (getImageCardCheckedArr.value?.length > 0) {
    ElMessageBox.confirm('是否批量删除已选中的图片？', '删除提示', {
      confirmButtonText: `确定`,
      cancelButtonText: `取消`,
      iconClass: `el-icon-warning`
    })
      .then(async () => {
        const res = await delelteBatchImage(getImageCardCheckedArr.value, userConfigInfo)
        if (res === deleteStatusEnum.allDeleted) {
          ElMessage.success('批量删除成功！')
        }
      })
      .catch(() => {
        console.log('取消批量删除')
      })
  } else {
    ElMessage.warning('请先选择图片')
  }
}

function triggleFullCheck() {
  let checkedImgArr: Array<UploadedImageModel> = []
  props.currentDirImageList.forEach((item: UploadedImageModel) => {
    // eslint-disable-next-line no-param-reassign
    item.checked = checked.value
  })
  checkedImgArr = props.currentDirImageList as any
  store.commit('REPLACE_IMAGE_CARD', { checkedImgArr })
}
onMounted(() => {
  triggleFullCheck()
})
</script>

<style scoped lang="stylus">
@import 'image-selector.styl';
</style>
