<template>
  <div>
    <div v-if="true">
      <div class="path">
        <div
          class="btn"
          @click="toPath"
        >返回上级
        </div>
        <div
          class="btn"
          data-index="0"
          :data-item="firstFloor"
          @click="toPath"
        >
          {{firstFloor.text}}
        </div>
        <div
          class="btn"
          :data-index="index + 1"
          :data-item="item"
          v-for="(item, index) in currentPath"
          :key="item.id"
          @click="toPath"
        >{{item.text}}</div>
      </div>
    </div>

    <!-- <div v-else>
      <div
        class="btn"
        v-if="currentPath.length !== 0"
        @click="toPath"
      >返回上级</div>
      <div
        class="btn"
        :data-index="index"
        v-for="(item, index) in currentPath"
        :key="index"
        @click="toPath"
      >{{item.text}}</div>
    </div> -->

    <scroll-view
      scroll-y
      class="list-view"
    >
      <div
        v-for="(item, index) in outValue"
        :key="item.id"
        class="list-view-item"
        @click="tapItem"
        :data-index="index"
        :data-item="item"
      >
        <div class="title">
          <van-icon
            :name="item.children ? 'column' :'location'"
            size="18px"
            color="#f0f00f"
            custom-class="icon-area-info"
          />
          {{item.text}}
        </div>
      </div>
    </scroll-view>
  </div>
</template>

<script>
import toTree from '@/common/toTree'

export default {
  props: {
    value: {
      type: Object,
      value: null
    },
    // 非树形数据，仅在value无传参时生效
    unnormalizedValue: {
      type: Array,
      value: []
    },
    fatherKey: {
      type: String,
      value: 'pid'
    },
    selfKey: {
      type: String,
      value: 'id'
    },
    rootValue: {
      type: null,
      value: null
    },
    pathMode: {
      type: String,
      value: 'mode1'
    },
    firstFloorTxt: {
      type: String,
      value: '第一级'
    },
    btnTxt: {
      type: String,
      value: '选择'
    },
    // 保持位置
    // 如果开启，数据动态更新后，会保持和更新前父元素对应的界面一致
    // 这里是通过记录下更新前的父元素的唯一标识符：id（或是selfKey中设定的键对应的值），在更新后找回显示出来
    // 所以就算更新后路径变了（父元素本来在第四层，变成了第二层之类），仍然能显示父元素的相应界面
    keepLoc: {
      type: Boolean,
      value: false
    }
  },
  data() {
    return {
      firstFloor: {},
      outValue: [],
      currentPath: [],
      // 判断当前是否已正在执行修改路径的方法
      isChange: false,
      // 映射value值，存放树形数据
      normalValue: [],
      currentFatherId: null
    }
  },
  methods: {
    initView() {
      if (this.keepLoc) {
        if (this.outValue.length > 0) {
          // 记录下当前界面元素的父节点id
          const fatherId = this.fatherKey
          this.currentFatherId = this.outValue[0][fatherId] || null
        }
      }
      // 优先使用value
      if (this.value.children.length > 0) {
        this.firstFloor = { text: this.value.text, id: this.value.id }
        this.normalValue = this.value.children
      } else {
        // 将unnormalizedValue标准化
        this.normalValue = this.normalizeValue()
      }

      if (this.keepLoc && this.currentFatherId !== null) {
        // 得到父元素的位置
        const fatherNodeLocation = this.searchLocById(this.currentFatherId)
        // 父元素的位置即是我们希望得到的当前路径
        const currentPath = fatherNodeLocation
        this.switchPath(currentPath)
      } else {
        // 设置初始的输出值
        this.outValue = this.normalValue
        this.currentPath = []
      }
    },
    tapItem(e) {
      // 如果正在执行修改路径的方法
      if (this.isChange) {
        return
      }
      this.isChange = true
      // 获取当前点击的索引
      const currentIndex = e.mp.currentTarget.dataset.index
      const currentItem = e.mp.currentTarget.dataset.item
      // 如果当前点击的标签还有下一级，就将路径改变
      if (this.outValue[currentIndex].children) {
        // 添加索引如路径
        this.currentPath = [
          ...this.currentPath,
          { text: currentItem.text, id: currentItem.id, index: currentIndex }
        ]
        this.selPath()
      }
      this.isChange = false
      this.$emit('nodeSelected', e.mp.currentTarget.dataset.item)
    },
    // 选择路径
    // pathsIndex 是 paths 的索引
    selPath(pathsIndex = this.currentPath.length - 1) {
      // 判断是否在第一级
      if (this.currentPath.length === 0) {
        return
      }
      // 根据路径修改 outValue
      let tmpValue = this.normalValue
      // 如果 pathsIndex 是 -1 就应该要回到第一级
      if (pathsIndex === -1) {
        this.currentPath = []
        this.outValue = tmpValue
        return
      }
      for (let i = 0; i <= pathsIndex; i++) {
        const item = this.currentPath[i].index
        tmpValue = tmpValue[item].children
      }
      // 更新 outValue , currentPath
      const endIndex = pathsIndex + 1
      this.outValue = tmpValue
      this.currentPath = this.currentPath.slice(0, endIndex)
    },
    toPath(e) {
      // 如果正在执行修改路径的方法
      if (this.isChange) {
        return
      }
      this.isChange = true
      const { dataset } = e.mp.currentTarget
      // 获取当前点击的索引
      let index1 = 0
      let item1 = {}
      if (dataset.index) {
        index1 = dataset.index
        item1 = dataset.item
      } else {
        index1 = this.currentPath.length - 1

        if (index1 > 0) {
          const temp = this.currentPath[index1 - 1]
          item1 = {
            text: temp.text,
            id: temp.id
          }
        } else {
          item1 = {
            text: this.firstFloor.text,
            id: this.firstFloor.id
          }
        }
      }
      this.$emit('nodeSelected', item1)
      this.selPath(index1 - 1)
      this.isChange = false
    },
    tapBtn(e) {
      this.$emit('tapBtn', e.mp.currentTarget.dataset.item)
    },
    // 将非标准值标准化
    normalizeValue() {
      return toTree({
        value: this.unnormalizedValue,
        fatherKey: this.fatherKey,
        selfKey: this.selfKey,
        rootValue: this.rootValue === null ? undefined : this.rootValue
      })
    },
    // 通过唯一标识符找到元素的所在位置
    // 这里特意用了loc（location）表示位置而不是path，二者的区别是loc还包括了元素自身，而path不包括元素自身
    // 如树形数据[{id:1,title:'1'}],id:1的位置是[1]，而它的路径是[]
    searchLocById(id) {
      if (id == null) {
        return null
      }
      const path = JSON.parse(JSON.stringify(this.normalValue))
      for (let i = 0; i < path.length; i++) {
        path[i].location = [i]
        if (id === path[i][this.selfKey]) {
          return path[i].location
        }
      }
      let stark = []
      stark = stark.concat(path)
      while (stark.length) {
        const temp = stark.shift()
        if (temp.children) {
          for (let j = 0; j < temp.children.length; j++) {
            temp.children[j].location = [...temp.location, j]
            if (id === temp.children[j][this.selfKey]) {
              return temp.children[j].location
            }
          }
          // 当前节点有子节点时，将子节点放到当前的栈的前面
          stark = temp.children.concat(stark)
        }
      }
      // 如果找不到对应id的位置
      return null
    },
    // 直接跳转到指定路径
    switchPath(pathArr = []) {
      let tmpValue = JSON.parse(JSON.stringify(this.normalValue))
      const currentPath = []
      for (let i = 0; i < pathArr.length; i++) {
        const index = pathArr[i]
        tmpValue = tmpValue[index].children
        currentPath.push({
          text: tmpValue[index].text,
          id: tmpValue[index].id,
          index
        })
      }
      // 更新 outValue , currentPath
      this.outValue = tmpValue
      this.currentPath = currentPath
    }
  },
  mounted() {
    this.initView()
  }
}
</script>

<style>
.path {
  display: flex;
  padding: 0 0 0 10px;
  min-height: 30px;
  flex-wrap: wrap;
}
.path .btn {
  margin-right: 20px;
  margin-bottom: 10px;
  padding: 0 5px;
  height: 20px;
  border-radius: 2px;
  background: #4b0;
  color: #fff;
  font-size: 10px;
  line-height: 20px;
}
.list-view {
  width: 100%;
  max-height: 250px;
  min-height: 250px;
}
.list-view-item {
  position: relative;
  display: flex;
  flex-direction: row;
  box-sizing: border-box;
  margin-top: -1px;
  width: 100%;
  height: 40px;
  border: 1px solid #e5e5e5;
  border-radius: 3px;
}
.list-view-item .title {
  padding-left: 20rpx;
  font-size: 14px;
  line-height: 40px;
}
.list-view-item .tip {
  font-size: 10px;
  line-height: 40px;
}
.list-view-item .btn {
  position: absolute;
  top: 7px;
  right: 20rpx;
  padding: 0 5px;
  height: 25px;
  border-radius: 3px;
  background: #4b0;
  color: #fff;
  font-size: 14px;
  line-height: 25px;
}

.icon-area-info {
  margin-right: 0;
}
</style>
