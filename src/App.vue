<script setup>
import { reactive, computed } from "@vue/reactivity"
import "element-plus/es/components/message/style/css"
import { ElMessage } from "element-plus"
import ClipboardJS from "clipboard"
import { useDark, useToggle } from "@vueuse/core"

const isDark = useDark()
const toggleDark = useToggle(isDark)
setTimeout(() => {
  const hour = new Date().getHours()
  toggleDark(hour >= 19)
})

// 密码区长度，128位
const size = 108
// 每行长度
const separateSize = 27
const strs = ["*", "/", "-", "+", ">", "<"]
const numbers = ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
const maps = [...strs, ...numbers]

var clipboard = new ClipboardJS(".copy")

clipboard.on("success", function (e) {
  ElMessage({
    message: "已复制到剪贴板",
    type: "success",
  })
})

clipboard.on("error", function (e) {
  ElMessage({
    message: "请手动粘贴",
    type: "error",
  })
})
/** 生成0-len的一个数 */
function getRandom(len) {
  return Math.round(Math.random() * len)
}

/** 随机字符 */
function getRandomStr(arr) {
  const len = arr.length - 1
  const random = getRandom(len)
  return arr[random]
}

/** 状态 */
const state = reactive({
  // 密码区，128位
  contents: [],
})

/** 密码区生成 */
async function genrate() {
  const contents = []
  contents.length = size
  // 前13位用时间戳占位
  const now = `${Date.now()}`
  const nows = now.split("")
  // 将时间戳转为需要的字符，随机插入
  nows.forEach((ele, index) => {
    contents[getRandom(size)] = maps[ele]
  })
  // 剩下的内容使用随机值占满
  for (let i = 0; i < size; i++) {
    // 已有值，忽略
    if (contents[i] !== undefined) continue
    // 赋值
    contents[i] = getRandomStr(maps)
  }
  state.contents = contents
  // console.log("contents", contents)
  // await nextTick()
  // clipboard.text(document.querySelector("#text-area"))
}

genrate()
// 字符串
const contentStr = computed(() => state.contents.join(""))
// 格式化，分4行，每行32位
const formatContents = computed(() => {
  let str = ""
  state.contents.forEach((ele, index) => {
    str += ele
    if ((index + 1) % separateSize === 0 && index !== size - 1) {
      str += ","
    }
  })
  return str.split(",")
})
</script>

<template>
  <el-card class="box-card">
    <template #header>
      <div class="card-header">
        <span>发票(密码区)</span>
      </div>
    </template>
    <div class="copy" :data-clipboard-text="contentStr">
      <template v-for="(item, index) in formatContents" :key="index">
        <div>
          <span class="item" v-for="(ele, index) in item" :key="index">
            {{ ele }}
          </span>
        </div>
      </template>
    </div>
    <el-divider />
    <div class="operation">
      <el-space>
        <el-button class="copy copy_color" :data-clipboard-text="contentStr"
          >剪贴</el-button
        >
        <el-button type="primary" @click="genrate">生成</el-button>
      </el-space>
    </div>
  </el-card>
</template>

<style scoped>
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 14px;
  font-weight: bold;
}

.el-divider--horizontal {
  margin: 24px 0 16px 0;
}

.item {
  display: inline-block;
  width: 12px;
}

.box-card {
  width: 376px;
}
.operation {
  /* margin-top: 16px; */
  display: flex;
  justify-content: end;
  align-items: center;
}
.dark .copy_color:not(:hover) {
  border-color: white;
}
.dark .copy_color:hover{
  border-color: #409eff;
}
</style>
