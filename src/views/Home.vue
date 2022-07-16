|<script setup>
// 导入的组件
import {
  ElSelect,
  ElOption,
  ElButton,
  ElTooltip,
  ElLoading,
  ElDrawer,
  ElMessage,
  ElMessageBox,
} from "element-plus";

import { ref, reactive, onMounted } from "vue";

import axios from "axios";

let rowOne = reactive([]);
let rowTwo = reactive([]);
//6行

// 抽屉js
const formLabelWidth = "80px";
let timer;

const table = ref(false);
const dialog = ref(false);
const loading = ref(false);

const open = () => {
  ElMessageBox.confirm("确定启用LH-1-1-1-2库位吗?", "是否启用", {
    confirmButtonText: "确定",
    cancelButtonText: "取消",
    type: "warning",
  })
    .then(() => {
      ElMessage({
        type: "success",
        message: "启用成功",
      });
    })
    .catch(() => {
      ElMessage({
        type: "info",
        message: "已取消操作",
      });
    });
};

// 抽屉表格数据

const gridData = [
  {
    date: "2016-05-02",
    name: "Peter Parker",
    address: "Queens, New York City",
  },
  {
    date: "2016-05-04",
    name: "Peter Parker",
    address: "Queens, New York City",
  },
  {
    date: "2016-05-01",
    name: "Peter Parker",
    address: "Queens, New York City",
  },
  {
    date: "2016-05-03",
    name: "Peter Parker",
    address: "Queens, New York City",
  },
];

// const drawerRef = ref<InstanceType<typeof ElDrawer>>()
// const onClick = () => {
//   drawerRef.value!.close()
// }

const handleClose = (done) => {
  if (loading.value) {
    return;
  }
  ElMessageBox.confirm("Do you want to submit?")
    .then(() => {
      loading.value = true;
      timer = setTimeout(() => {
        done();
        // 动画关闭需要一定的时间
        setTimeout(() => {
          loading.value = false;
        }, 400);
      }, 2000);
    })
    .catch(() => {
      // catch error
    });
};

const cancelForm = () => {
  loading.value = false;
  dialog.value = false;
  clearTimeout(timer);
};
//抽屉js以上

//dialog-正视图弹窗
// 是否展示弹窗表格
let dialogTableVisible = ref(false);
// 弹窗的数据
//编号-物料编码-物料名称-数量，
// let gridData = reactive({data:[]})
// let gridData = reactive([]);
//获取所有库位信息
async function getList() {
  let loading = ElLoading.service({
    lock: true,
    text: "加载中",
    background: "rgba(0, 0, 0, 0.8)",
  });
  let data = await axios({
    method: "post",
    url: "http://39.170.118.36:8001/api/mingdao/storagePosition",
    data: {
      appKey: "97fd9e6f1508d195",
      sign: "ZWUzNzA1ZDA0NjM2YTcwMTcyNmRhZjA0ZGRiZjM1NWU0YWFiNjAwMjJkMzlkYTdhNDI2OWFjNzdhMzg2NTI1NQ==",
      worksheetId: "kuwei",
      // viewId: "628f2e45b637eb4754763338",
      pageSize: 999,
      pageIndex: 1,
    },
  });
  //一次拿到所有数据
  console.log(data);
  let arr = data.data.data.rows;
  //两行数据
  // let rowO = optionStatus[value.value].rowOne;
  // let rowT = optionStatus[value.value].rowTwo;

  if (currentView.value == 0) {
    // debugger
    fView[storyValue.value - 1].forEach((it, idx) => {
      // debugger

      // 每一层
      let ceng = arr.filter((it) => {
        return it.ceng == storyValue.value && it.pai == idx + 1;
      });

      let arr_ = [];
      // 每一列
      for (let i = 0; i < 9; i++) {
        let result = ceng.filter((it) => {
          if (it.lie == i + 1) {
            return true;
          } else {
            return false;
          }
        })[0];
        arr_.push(result ? result : {});
      }

      arr_.forEach((it, idx2) => {
        fView[storyValue.value - 1][idx].splice(idx2, 1, it);
      });
    });
  } else if (currentView.value == 1) {
    // 正视视角
    // 每一排
    zView[pai.value].forEach((it, idx) => {
      // debugger

      // 每一层
      let ceng = arr.filter((it) => {
        return it.pai == pai.value + 1 && it.ceng == idx + 1;
      });

      let arr_ = [];
      // 每一列
      for (let i = 0; i < zView[pai.value][0].length; i++) {
        let result = ceng.filter((it) => {
          if (it.lie == i + 1) {
            return true;
          } else {
            return false;
          }
        })[0];
        arr_.push(result ? result : {});
      }

      arr_.forEach((it, idx2) => {
        zView[pai.value][idx].splice(idx2, 1, it);
      });
    });
  } else if (currentView.value == 2) {
    // 侧视图
    cView[lie.value - 1].forEach((it, idx) => {
      // debugger

      // 每一层
      let ceng = arr.filter((it) => {
        return it.lie == lie.value && it.ceng == idx + 1;
      });

      let arr_ = [];
      // 每一列
      for (let i = 0; i < 4; i++) {
        let result = ceng.filter((it) => {
          if (it.pai == i + 1) {
            return true;
          } else {
            return false;
          }
        })[0];
        arr_.push(result ? result : {});
      }

      arr_.forEach((it, idx2) => {
        cView[lie.value - 1][idx].splice(idx2, 1, it);
      });
    });
  }

  setTimeout(() => {
    loading.close();
  }, 500);
}

let positionCode = ref("");
let boxCode = ref("");

async function getDetail(it) {
  if (positionAction) {
    positionAction === 1 ? (startPosition.data = it) : (endPosition.data = it);
    console.log(startPosition.data);
    return;
  }
  if (it.positionStatus !== "有货") return;

  positionCode.value = it.positionCode;
  boxCode.value = it.boxCode;
  // console.log(positionCode)
  let data = await axios({
    method: "post",
    url: "http://39.170.118.36:8001/api/mingdao/positionDetail",
    data: {
      appKey: "97fd9e6f1508d195",
      sign: "ZWUzNzA1ZDA0NjM2YTcwMTcyNmRhZjA0ZGRiZjM1NWU0YWFiNjAwMjJkMzlkYTdhNDI2OWFjNzdhMzg2NTI1NQ==",
      worksheetId: "628f2e45b637eb47547632ba",
      filters: [
        {
          controlId: "positionCode",
          dataType: 32,
          filterType: 2,
          // 变量
          value: it.positionCode,
        },
      ],
    },
  });
  //一次拿到所有数据
  console.log(data);
  console.log("上面是data");
  console.log(positionCode);
  console.log(boxCode);
  console.log("Hello");
  //    data.data.data.rows = [
  //   {
  //     no: "1",
  //     itemCode: "A0101",
  //     itemName: "测试物料1",
  //     count: "8",
  //   },

  // ];
  // debugger
  gridData.length = 0;
  gridData.push(...data.data.data.rows);
  // gridData[0].no = 110
  console.log(gridData);
  dialogTableVisible.value = true;
}

async function movePosition() {
  let data = await axios({
    method: "post",
    url: "http://39.170.118.36:8001/api/mingdao/movePosition",
    data: {
      appKey: "97fd9e6f1508d195",
      sign: "ZWUzNzA1ZDA0NjM2YTcwMTcyNmRhZjA0ZGRiZjM1NWU0YWFiNjAwMjJkMzlkYTdhNDI2OWFjNzdhMzg2NTI1NQ==",
      worksheetId: "movetrayaction",
      controls: [
        {
          controlId: "62b0485c3e106f528a203814",
          value: "文本",
        },
        {
          controlId: "startPosition",
          value: startPosition.data.positionCode,
        },
        {
          controlId: "endPosition",
          value: endPosition.data.positionCode,
        },
        {
          controlId: "ownerid",
          value: "66b680d4-f772-4ede-ac03-c35bc17ed9df",
        },
        {
          controlId: "moveType",
          value: "人工移库",
        },
        {
          controlId: "moveStatus",
          value: "待移库",
        },
      ],
    },
  });
  console.log(data);
  alert("移库成功");
  moveActionBoxClose();
  viewSwitch(1);
}
async function lockSwitch() {
  let data = await axios({
    method: "post",
    url: "http://39.170.118.36:8001/api/mingdao/lockSwitch",
    data: {
      appKey: "97fd9e6f1508d195",
      sign: "ZWUzNzA1ZDA0NjM2YTcwMTcyNmRhZjA0ZGRiZjM1NWU0YWFiNjAwMjJkMzlkYTdhNDI2OWFjNzdhMzg2NTI1NQ==",
      worksheetId: "lockSwitch",
      controls: [
        {
          controlId: "positionCode",
          value: "LH-1-1-1-2",
        },
        {
          controlId: "positionStatus",
          value: "有货",
        },
        {
          controlId: "isLock",
          value: "是(启用)",
        },
        {
          controlId: "ownerid",
          value: "66b680d4-f772-4ede-ac03-c35bc17ed9df",
        },
      ],
    },
  });
  console.log(data);
  alert("移库成功");
  moveActionBoxClose();
  viewSwitch(1);
}
//会先判断是否启用，没启用的库位（否），将会锁定，更换视图，优先级高于所有状态，直接return跳出
// 状态判断
function checkStatus(sta) {
  let result = "";
  if (sta.locked === "否") {
    return "locked";
  }

  if (sta.positionStatus === "无货") {
    result = "empty";
  } else if (sta.positionStatus === "容器占用") {
    result = "emptyTray";
  } else if (sta.positionStatus === "有货") {
    result = "occupy";
  } else if (sta.positionStatus === "作业中") {
    result = "working";
  }
  return result;
}

function moveTry() {
  if (
    startPosition.data.positionStatus !== "有货" &&
    startPosition.data.positionStatus !== "容器占用"
  ) {
    alert("起点库位异常，请重选");
    return;
  }
  if (endPosition.data.positionStatus !== "无货") {
    alert("终点库位异常，请重选");
    return;
  }
  // alert("起始库位" + "\n 终点库位：");
  movePosition();
}

let optionStatus = reactive([
  {
    // 长料框C成品一区56排，12行
    // 右区
    count: 9,
    rowOne: 1,
    rowTwo: 2,
  },
  {
    //左区
    count: 8,
    rowOne: 3,
    rowTwo: 4,
  },
]);

//默认第一排
let value = ref("0");
// 默认第一层
let storyValue = ref("0");
// 俯视图-层下拉框数

let stories = ref([
  {
    value: "1",
    label: "1层",
  },
  {
    value: "2",
    label: "2层",
  },
  {
    value: "3",
    label: "3层",
  },
  {
    value: "4",
    label: "4层",
  },
  {
    value: "5",
    label: "5层",
  },
  {
    value: "6",
    label: "6层",
  },
]);

// 侧视图-列下拉框
let lie = ref("1");
let lieOptions = ref([
  {
    value: "1",
    label: "1列",
  },
  {
    value: "2",
    label: "2列",
  },
  {
    value: "3",
    label: "3列",
  },
  {
    value: "4",
    label: "4列",
  },
  {
    value: "5",
    label: "5列",
  },
  {
    value: "6",
    label: "6列",
  },
  {
    value: "7",
    label: "7列",
  },
  {
    value: "8",
    label: "8列",
  },
  {
    value: "9",
    label: "9列",
  },
]);

//切换视角按钮
let currentView = ref(1);
function viewSwitch(value) {
  currentView.value = value;
  getList();
}

// let showShadow = ref(0);
// showShadow = "hidden";

let pai = ref(0);
// 正视图有4排
let paiOptions = ref([
  {
    value: 0,
    label: "1排",
  },
  {
    value: 1,
    label: "2排",
  },
  // 注意3和4排只有8列
  {
    value: 2,
    label: "3排",
  },
  {
    value: 3,
    label: "4排",
  },
]);
//正视格子
let zView = reactive([
  [
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
  ],
  [
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
  ],
  [
    [{}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}],
  ],
  [
    [{}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}],
  ],
]);
// 侧视格子

let cView = reactive([
  [
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
  ],
  [
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
  ],
  [
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
  ],
  [
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
  ],
  [
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
  ],
  [
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
  ],
  [
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
  ],
  [
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
    [{}, {}, {}, {}],
  ],
  [
    [{}, {}, "", ""],
    [{}, {}, "", ""],
    [{}, {}, "", ""],
    [{}, {}, "", ""],
    [{}, {}, "", ""],
    [{}, {}, "", ""],
  ],
]);

// 俯视格子
let fView = reactive([
  [
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, ""],
    [{}, {}, {}, {}, {}, {}, {}, {}, ""],
  ],
  [
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, ""],
    [{}, {}, {}, {}, {}, {}, {}, {}, ""],
  ],
  [
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, ""],
    [{}, {}, {}, {}, {}, {}, {}, {}, ""],
  ],
  [
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, ""],
    [{}, {}, {}, {}, {}, {}, {}, {}, ""],
  ],
  [
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, ""],
    [{}, {}, {}, {}, {}, {}, {}, {}, ""],
  ],
  [
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, {}],
    [{}, {}, {}, {}, {}, {}, {}, {}, ""],
    [{}, {}, {}, {}, {}, {}, {}, {}, ""],
  ],
]);

let startPosition = reactive({ data: { positionCode: "" } });
let endPosition = reactive({ data: { positionCode: "" } });
let positionAction = "";
let moveActionBoxFlag = ref(false);
function moveActionBoxClose() {
  startPosition.data = { positionCode: "" };
  endPosition.data = { positionCode: "" };
  positionAction = "";
  moveActionBoxFlag.value = false;
}
function startPositionFocus() {
  positionAction = 1;
  console.log(positionAction);
}

function endPositionFocus() {
  positionAction = 2;
  console.log(positionAction);
}
function moveAction() {
  alert("移库操作");
}
function isLock() {
  alert("锁定/解锁");
}
function changeStatus() {
  alert("库位状态变更为XX");
}
//默认第一列，记得是挂载上去，就会默认先加载一次
onMounted(() => {
  // 默认请求第一行
  getList();
  // addEmpty();
  storyValue.value = "1";
});
</script>

<template>
  <div class="main">
    <!-- views switch buttons-->
    <!-- 角度切换按钮 -->
    <span class="viewSwitch">
      <button
        :class="['viewsBtn', currentView === 0 ? 'active' : '']"
        @click="viewSwitch(0)"
      >
        俯视
      </button>
      <button
        :class="['viewsBtn', currentView === 1 ? 'active' : '']"
        @click="viewSwitch(1)"
      >
        正视
      </button>
      <button
        :class="['viewsBtn', currentView === 2 ? 'active' : '']"
        @click="viewSwitch(2)"
      >
        侧视
      </button>
      <br /><br />
      <!-- 选择库位 -->

      <div class="selectPart" v-show="currentView === 0">
        <!-- 俯视选层 -->
        <el-select
          class="story"
          v-model="storyValue"
          placeholder="层"
          @change="getList"
          :disabled="!value"
        >
          <el-option
            v-for="item in stories"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
        <!-- 根据是否选区，来判断是否可以刷新 -->
        <el-button
          :disabled="!value"
          class="request"
          type="primary"
          @click="getList"
          plain
          >刷新</el-button
        >

        <!-- 弹窗 库位信息按钮-->
        <!-- <el-button plain @click="open"> 库位信息1 </el-button> -->
        <!-- <el-button :plain="true" @click="moveActionBoxFlag = true"
          >移库operation1</el-button
        > -->
      </div>
      <div class="selectPart" v-show="currentView === 1">
        <!-- 正视选排 -->
        <!-- <el-select v-model="pai" placeholder="排" @change="addEmpty"> -->
        <el-select
          class="story"
          v-model="pai"
          placeholder="排"
          @change="getList"
        >
          <el-option
            v-for="item in paiOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>

        <!-- 根据是否选区，来判断是否可以刷新 -->
        <!-- 刷新按键 -->
        <el-button
          :disabled="!value"
          class="request"
          type="primary"
          @click="getList"
          plain
          >刷新</el-button
        >
      </div>
      <div class="selectPart" v-show="currentView === 2">
        <!-- 侧视选列 -->
        <el-select
          class="story"
          v-model="lie"
          placeholder="列"
          @change="getList"
        >
          <el-option
            v-for="item in lieOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>

        <!-- 根据是否选区，来判断是否可以刷新 -->
        <el-button
          :disabled="!value"
          class="request"
          type="primary"
          @click="getList"
          plain
          >刷新</el-button
        >
        >
      </div>
    </span>
    <!-- 中间的大盒子 -->
    <div class="boxesContainerWrap">
      <!-- 俯视图 -->
      <div class="fViewContainer" v-if="currentView === 0">
        <div class="boxesContainer f" v-if="currentView === 0">
          <div class="columnNo">
            <div
              class="rowNumbers box"
              v-for="(item, idx) in fView[0][0]"
              :key="idx"
            >
              {{ idx + 1 + "列" }}
            </div>
          </div>
          <div class="rowOneWrap">
            <div
              class="rowOne"
              v-for="(item, idx) in fView[storyValue - 1]"
              :key="idx"
            >
              <div
                class="box"
                :class="[
                  'box',
                  checkStatus(it2),
                  it2.positionCode === startPosition.data.positionCode
                    ? 'active'
                    : '',
                  it2.positionCode === endPosition.data.positionCode
                    ? 'active_end'
                    : '',
                ]"
                v-for="(it2, idx) in item"
                @click="getDetail(it2)"
                :key="idx"
              >
                <el-tooltip
                  :content="it2.positionCode"
                  v-if="it2.positionCode"
                  placement="top"
                  :show-after="300"
                >
                  <el-button
                    style="
                      opacity: 0;
                      margin-left: -1px;
                      width: 50px;
                      height: 100%;
                    "
                  ></el-button>
                </el-tooltip>
              </div>
            </div>
          </div>
          <div class="wrap">
            <div
              class="rowNo"
              v-for="(item, idx) in fView[storyValue - 1]"
              :key="idx"
            >
              <p>{{ idx + 1 + "排" }}</p>
            </div>
          </div>
        </div>
      </div>
      <!-- 正视图 -->
      <div class="zViewContainer" v-if="currentView === 1">
        <div :class="['topShadow', currentView !== 1 ? 'hide' : '']"></div>
        <div class="boxesContainer" v-if="currentView === 1">
          <div class="columnNo">
            <div
              class="rowNumbers box"
              v-for="(item, idx) in zView[pai][0]"
              :key="idx"
            >
              {{ idx + 1 + "列" }}
            </div>
          </div>
          <div class="rowOneWrap">
            <div class="rowOne" v-for="(item, idx) in zView[pai]" :key="idx">
              <div
                class="box"
                :class="[
                  'box',
                  checkStatus(it2),
                  it2.positionCode == startPosition.data.positionCode
                    ? 'active'
                    : '',
                  it2.positionCode == endPosition.data.positionCode
                    ? 'active_end'
                    : '',
                ]"
                v-for="(it2, idx) in item"
                @click="getDetail(it2)"
                :key="idx"
              >
                <el-tooltip
                  :content="it2.positionCode"
                  v-if="it2.positionCode"
                  placement="top"
                  :show-after="300"
                >
                  <el-button
                    style="
                      opacity: 0;
                      margin-left: -1px;
                      width: 50px;
                      height: 100%;
                    "
                  ></el-button>
                </el-tooltip>
              </div>
            </div>
          </div>
          <div class="wrap">
            <div class="rowNo" v-for="(item, idx) in zView[pai]" :key="idx">
              <p>{{ idx + 1 + "层" }}</p>
            </div>
          </div>
          <div id="rightShadow"></div>
        </div>
      </div>
      <!-- 侧视图-->
      <div class="cViewContainer" v-if="currentView === 2">
        <div class="boxesContainer" v-if="currentView === 2">
          <div class="columnNo">
            <div
              class="rowNumbers box"
              v-for="(item, idx) in cView[0][0]"
              :key="idx"
            >
              {{ idx + 1 + "排" }}
            </div>
          </div>
          <div class="rowOneWrap">
            <div
              class="rowOne"
              v-for="(item, idx) in cView[lie - 1]"
              :key="idx"
            >
              <div
                class="box"
                :class="[
                  'box',
                  checkStatus(it2),
                  it2.positionCode == startPosition.data.positionCode
                    ? 'active'
                    : '',
                  it2.positionCode == endPosition.data.positionCode
                    ? 'active_end'
                    : '',
                ]"
                v-for="(it2, idx) in item"
                @click="getDetail(it2)"
                :key="idx"
              >
                <el-tooltip
                  :content="it2.positionCode"
                  v-if="it2.positionCode"
                  placement="top"
                  :show-after="300"
                >
                  <el-button
                    style="
                      opacity: 0;
                      margin-left: -1px;
                      width: 50px;
                      height: 100%;
                    "
                  ></el-button>
                </el-tooltip>
              </div>
            </div>
          </div>
          <div class="wrap">
            <div class="rowNo" v-for="(item, idx) in cView[lie - 1]" :key="idx">
              <p>{{ idx + 1 + "层" }}</p>
            </div>
          </div>
        </div>
      </div>
      <div class="moveActionBox" v-if="moveActionBoxFlag">
        <div class="closeWrap">
          <el-button
            type="danger"
            round
            @click="moveActionBoxClose()"
            class="closeBtn"
            size="small"
            >X</el-button
          >
        </div>
        <p>移库操作</p>
        <input
          class="inputPosition"
          @focus="startPositionFocus()"
          v-model="startPosition.data.positionCode"
          readonly
          placeholder="起始库位"
        />
        <br />
        <input
          class="inputPosition"
          @focus="endPositionFocus()"
          v-model="endPosition.data.positionCode"
          readonly
          placeholder="终点库位"
        />
        <br />
        <br /><br />
        <el-button type="primary" round @click="moveTry" class="moveBtn"
          >移库BTN</el-button
        >
      </div>
      <div class="operationBox">
        <!-- <div>操作台</div> -->
        <!-- <el-button
          class="operationBtn"
          type="primary"
          :plain="true"
          @click="moveActionBoxFlag = true"
          >移库</el-button
        >
        <el-button
          class="operationBtn"
          type="primary"
          :plain="true"
          @click="isLock()"
          >锁定/解锁</el-button
        >
        <el-button
          class="operationBtn"
          type="primary"
          :plain="true"
          @click="changeStatus()"
          >库位状态变更</el-button
        > -->
        <!-- 抽屉按钮 -->
        <!-- <el-button text @click="table = true">抽屉按钮</el-button> -->
        <!-- <el-button text @click="dialog = true"
          >Open Drawer with nested form</el-button
        > -->
        <el-drawer
          v-model="dialogTableVisible"
          :title="`库位：${positionCode} 料框：${boxCode} `"
          direction="btt"
          size="60%"
        >
          <span>料框:</span>
          <div class="operationsBtnContainer">
            <el-button
              class="operationBtn"
              type="primary"
              :plain="true"
              @click="moveActionBoxFlag = true"
              >移库</el-button
            >
            <!-- 点击移库，抽屉关闭，并且选中当前库位为起始库位
                dialogTableVisible =!dialogTableVisible  -->

            <el-button
              class="operationBtn"
              type="primary"
              :plain="true"
              @click="isLock()"
              >锁定/解锁</el-button
            >
            <el-button
              class="operationBtn"
              type="primary"
              :plain="true"
              @click="changeStatus()"
              >库位状态变更</el-button
            >
            <el-button text @click="open">启用库位/锁定库位</el-button>
          </div>
          <!-- 抽屉   在框物料信息 -->
          <el-table :data="gridData">
            <!-- 弹窗内容 -->
            <el-table-column property="no" label="编号" width="60">
              <template v-slot="scope">
                <div>{{ scope.$index + 1 }}</div>
              </template>
            </el-table-column>
            <el-table-column property="itemCode" label="物料编码" width="200" />
            <el-table-column property="iName" label="物料名称" />
            <el-table-column property="count" label="数量" width="60" />
          </el-table>
        </el-drawer>
      </div>
    </div>
    <!-- 展示图 -->
    <div class="displayContainer">
      <div class="displayBox">
        <div class="empty dBoxes"></div>
        <p class="pInfo">无货</p>
      </div>

      <div class="displayBox">
        <div class="occupy dBoxes"></div>
        <p class="pInfo">有货</p>
      </div>

      <div class="displayBox">
        <div class="emptyTray dBoxes"></div>
        <p class="pInfo">容器占用</p>
      </div>

      <div class="displayBox">
        <div class="working dBoxes"></div>
        <p class="pInfo">作业中</p>
      </div>

      <div class="displayBox">
        <div class="locked dBoxes"></div>
        <p class="pInfo">锁定</p>
      </div>
    </div>

    <!-- 状态展示 -->
    <br />
    <br />

    <!-- <el-button text @click="dialogTableVisible = true">库位明细</el-button> -->
    <!-- <el-dialog
      v-model="dialogTableVisible"
      :title="`${positionCode}  在箱物料`"
    > -->
    <!-- <el-table :data="gridData"> -->
    <!-- 弹窗内容 -->
    <!-- <el-table-column property="no" label="编号" width="60">
          <template v-slot="scope">
            <div>{{ scope.$index + 1 }}</div>
          </template>
        </el-table-column>
        <el-table-column property="itemCode" label="物料编码" width="140" />
        <el-table-column property="iName" label="物料名称" width="420" />
        <el-table-column property="count" label="数量" />
      </el-table>
    </el-dialog> -->
  </div>
</template>

<style lang="scss">
.el-message {
  --el-message-min-width: 320px !important;
}

label {
  width: 400px;
  height: 30px;
  float: left;
  text-align: center;
  line-height: 30px;
  border-right: 1px solid brown;
  border-bottom: 1px solid brown;
}

label div {
  width: 100%;
  height: 100px;
  position: absolute;
  left: 0;
  top: 31px;
  background: #b3d8ff;
  display: none;
}

label input {
  width: 0;
}

input:checked + div {
  display: block;
}

.viewsBtn {
  width: 100px;
  height: 40px;
  margin-left: 5px;
  color: #409efe;
  box-sizing: border-box;
  border-color: #b3d8ff;
  border-radius: 10px;
  background-color: #ffffff;
}
.viewsBtn.active {
  // background: rgb(130, 235, 130);
  background: rgb(130, 235, 130);
  color: black;
}

.main {
  // border: 1px solid blue;
  min-width: 990px;
  max-width: 1000px;
  padding: 40px;
  height: 100px;
  padding: 0.8% 0 0 0;
  margin: 0 auto;
  // display:flex;

  .viewSwitch {
    display: flex;
  }

  .selectPart {
    min-width: 800px;
    // padding-left: 300px;

    .story {
      margin: 0px 10% 0 20%;
    }

    .request {
      margin-left: 20px;
    }
  }

  .topShadow {
    // visibility: hidden;
    min-width: 800px;
    // margin-top: 40px;
    // position: relative;
    background-color: rgb(214, 207, 207);
    width: 100%;
    height: 31px;
    padding-top: 1px;
    margin-left: 10px;
    transform: skew(-35deg);
    z-index: 999;
    &.hide {
      opacity: 0;
    }
  }

  #rightShadow {
    // visibility: hidden;
    background-color: #b6aeae;
    width: 35px;
    height: 257px;
    transform: rotate(-55deg) skew(-55deg);
    position: absolute;
    top: 17.2%;
    left: 99.8%;
    z-index: 999;
  }

  .boxesContainer {
    // display: inline-block;
    // position: relative;
    z-index: 1;
    color: gray;
    // background-color: rgba(209, 206, 206, 0.938);
    border: 6px solid rgb(182, 181, 181);
    border-right: 6px solid rgb(182, 174, 174);
    border-top: 6px solid rgb(214, 207, 207);
    border-left: 6px solid rgb(214, 207, 207);
    border-bottom: 6px solid rgb(214, 207, 207);
    // border-top: 10px solid red;
    border-radius: 3px;

    width: 90%;
    // height: 220px;
    height: 400px;
    margin-top: -4px;
    padding: 20px 0 20px 30px;
    position: relative;
    min-width: 800px;
    &.f {
      .rowOneWrap {
        flex-direction: column;
      }
      .wrap {
        flex-direction: column;
      }
      .rowOne {
        // flex-direction: row;
      }
    }
    .rowOneWrap {
      display: flex;
      flex-direction: column-reverse;
    }
    .columnNo,
    .rowOne,
    .rowTwo {
      margin-top: 10px;
      padding: 0 20px 0 20px;
      // border: 1px solid green;
      display: flex;
      flex-direction: row-reverse;
      justify-content: space-around;

      // 未渲染
      .box {
        width: 50px;
        height: 50px;
        border: 1px solid lightgray;
        border-radius: 10px;
        // margin: 20px 0 20px 0;
        background-size: 100%;
        background-repeat: no-repeat;
        font-size: 6%;
        // 呼吸效果
        &.active {
          animation: myBreath 1s linear infinite;
          border: 6px outset rgb(236, 92, 102);
        }
        &.active_end {
          // 选中后
          animation: myBreath 1s linear infinite;
          border: 6px outset lightgreen;
        }
        &.hide {
          opacity: 0;
        }

        &.empty {
          background-image: url(../assets/empty.svg);
        }

        &.emptyTray {
          background-image: url(../assets/emptyTray.svg);
        }

        &.occupy {
          background-image: url(../assets/occupy.svg);
        }

        &.locked {
          background-image: url(../assets/locked.svg);
        }

        &.working {
          background-image: url(../assets/working.svg);
        }
      }

      .rowNumbers {
        border: none;
        text-align: center;
        height: 20px;
      }
    }

    .columnNo {
      margin-top: 0px;
    }

    .wrap {
      position: absolute;
      font-size: 12px;
      left: 0px;
      top: 45px;
      display: flex;
      flex-direction: column-reverse;

      .rowNo {
        width: 50px;
        line-height: 38px;
        text-align: center;
      }
    }
  }
  .boxesContainerWrap {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-top: 2%;
    width: 100%;
    .operationBox {
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      margin-left: 30px;
      padding-left: 30px;
      width: 15%;
      // border: 2px solid red;
      height: 200px;
      .operationsBtnContainer {
        // margin: 5px 0 20px 0;
      }
      .operationBtn {
        // background: #9dc6e8;
        width: 120px;
        // border-radius: 20px;
      }
    }
  }
  .fViewContainer,
  .zViewContainer,
  .cViewContainer {
    margin-left: 4%;
  }

  .displayContainer {
    width: 85%;
    margin: 1% 0 0 3%;
    display: flex;
    // flex-direction: column;
    justify-content: space-around;
    // min-width: 800px;

    .displayBox {
      width: 70px;
      height: 100px;
      // margin: 10px;
      // border: 2px solid red;
      text-align: center;
      left: 50%;
      .pInfo {
        // padding: -10px;
        margin: -10px;
      }

      //3D新增
      %test {
        position: absolute;
        content: "";
        transition: all 0.5s;
        transition: all 0.5s;
      }

      .dBoxes {
        margin: 40px 0px 0 25px;

        width: 60px;
        height: 60px;
        background-repeat: no-repeat;
        background-size: 100%;
        transform: translate(-50%, -50%);
        text-decoration: none;
        font-size: 4vw;
        transition: all 0.5s;
        background-color: #3498db;

        //3D新增-下方
        &::before {
          @extend %test;
          bottom: -10px;
          height: 10px;
          width: 100%;
          left: 5px;
          transform: skewX(45deg);
          background-color: darken(#3498db, 26%);
        }

        // 3D右侧
        &::after {
          @extend %test;
          right: -10px;
          height: 100%;
          width: 10px;
          bottom: -5px;
          transform: skewY(45deg);
          background-color: darken(#3498db, 16%);
        }
      }
      // 各个库位状态

      .empty {
        background-image: url(../assets/empty.svg);
      }

      .emptyTray {
        background-image: url(../assets/emptyTray.svg);
      }

      .occupy {
        background-image: url(../assets/occupy.svg);
      }

      .locked {
        background-image: url(../assets/locked.svg);
      }

      .working {
        background-image: url(../assets/working.svg);
      }
    }
  }
  .el-input--suffix {
    margin-left: 5px;
  }
  .el-button--primary.is-plain {
    margin-left: 50px;
  }

  // 移库
  .moveActionBox {
    // position: absolute;
    // right: 10px;
    // top: 100px;
    width: 10%;
    height: 58%;
    border: 6px solid rgb(182, 181, 181);
    margin: 10px 10px 10px 50px;
    padding: 10px;
    .closeWrap {
      display: flex;
      justify-content: flex-end;
    }
    .moveBtn {
      background: lightskyblue;
      color: #fff;
      width: 100px;
      text-align: center;
      // height: 50px;
      line-height: 50px;
      border-radius: 32px;
      animation: myBreath 1.5s linear infinite;
    }
    @keyframes myBreath {
      0% {
        transform: scale(0.78);
      }
      50% {
        transform: scale(100%);
      }
      100% {
        transform: scale(0.78);
      }
    }
    .inputPosition {
      border: 1px solid lightblue;
      width: 80px;
      border-radius: 17px;
      margin: 0 0 10px 0;
      padding: 10px;
    }
  }
}
</style>
