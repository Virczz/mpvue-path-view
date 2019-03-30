<template>
  <div>
    <!-- 头部 -->
    <div class="tool-bar">
      <div
        class="tool_cancel"
        @click="onPickerCancel"
      >取消</div>
      <div class="tool_title">{{checkedValue}}</div>
      <div
        class="tool_confirm"
        @click="onPickerConfirm"
      >确认</div>
    </div>
    <path-view
      :value="dbInfo"
      @nodeSelected="selThis"
    ></path-view>

  </div>

</template>

<script>
import pathView from "@/components/pathView";
import dbInfo from "@/common/dbInfo";

export default {
  components: {
    pathView
  },
  data() {
    return {
      dbInfo,
      checkedValue: "中国"
    };
  },

  methods: {
    selThis(e) {
      this.checkedValue = e.text;
    }
  },
  mounted() {}
};
</script>

<style>
.head-area {
  display: flex;
  flex: row;
  height: 40px;
  line-height: 40px;
  margin-left: 10px;
  margin-right: 10px;
}

.acount-area {
  flex: 1;
  text-align: center !important;
  height: 34px !important;
  line-height: 32px !important;
}

.account-label {
  max-width: 60px;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.icon-account_manager {
  margin-bottom: 5px;
  vertical-align: middle;
}
.icon-account_manager1 {
  margin-right: 5px;
  margin-bottom: 5px;
  vertical-align: middle;
}

.area-info_wrap {
  display: flex;
  border: 1px solid red;
  height: 32px;
  line-height: 32px;
}

.area-info_site {
  flex: 1;
}
.icon-area-info {
  margin-right: 10px;
}

.tool-bar {
  display: flex;
  height: 44px;
  line-height: 44px;
  justify-content: space-between;
}

.ellipsis {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.tool_title {
  max-width: 50%;
  font-size: 16px;
  font-weight: 500;
  text-align: center;
}

.tool_cancel,
.tool_confirm {
  padding: 0 15px;
  font-size: 14px;
  color: #1989fa;
}
.checkbox_area {
  font-size: 14px;
  text-align: center;
  border-top: 1px solid gray;
  border-bottom: 1px solid gray;
  margin: 5px 5px;
}
</style>
