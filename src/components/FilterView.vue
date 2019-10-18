<template>
  <!--  蒙层  当排序标记和筛选标记都为true时，才会显示蒙层模版   并且 点击自己的执行 隐藏方法-->
  <div :class="{'open':isSort || isScreen}" @click.self="hideView">
    <!-- 导航 -->
    <div v-if="filterData" class="filter_wrap">
      <aside class="filter">
        <!-- :class="{'filter-bold':currentFilter==index}"   点击的和遍历的索引如果相同，就添加类名 -->
        <div
          class="filter-nav"
          v-for="(item,index) in filterData.navTab"
          :key="index"
          :class="{'filter-bold':currentFilter==index}"
          @click="filterSort(index)"
        >
          <span>{{item.name}}</span>
          <i v-if="item.icon" :class="'fa fa-'+item.icon"></i>
        </div>
      </aside>
    </div>

    <!-- 综合排序 的下拉菜单  通过 isSort 标记是否显示 出来 -->
    <section class="filter-extend" v-if="isSort">
      <ul>
        <li v-for="(item,index) in filterData.sortBy" :key="index" @click="selectSort(item,index)">
          <span :class="{'selectName':currentSort == index}">{{item.name}}</span>
          <i v-show="currentSort == index" class="fa fa-check"></i>
        </li>
      </ul>
    </section>

    <!-- 筛选 -->
    <section class="filter-extend" v-if="isScreen">
      <div class="filter-sort">
        <div v-for="(screen,index) in filterData.screenBy" :key="index" class="morefilter">
          <p class="title">{{screen.title}}</p>
          <ul>
            <li
              v-for="(item,i) in screen.data"
              :key="i"
              :class="{'selected':item.select}"
              @click="selectScreen(item,screen)"
            >
              <img v-if="item.icon" :src="item.icon" alt />
              <span>{{item.name}}</span>
            </li>
          </ul>
        </div>
      </div>
      <div class="morefilter-btn">
        <span @click="clearFilter" :class="{'edit':edit}" class="morefilter-clear">清空</span>
        <span @click="filterOk" class="morefilter-ok">确定</span>
      </div>
    </section>
  </div>
</template>

<script>
export default {
  name: "FilterView",
  data() {
    return {
      currentFilter: 0, // 导航的当前索引 初始为 0
      isSort: false, // 是否显示综合排序的下拉菜单 标记
      currentSort: 0, // 存储 综合排序点击下拉菜单 的索引   初始为0
      isScreen: false
    };
  },
  // 接收获取到的导航数据
  props: {
    filterData: Object
  },

  computed: {
    // 计算属性  deit  如果有选中状态的则让清空按钮显示。 如果没有选中按钮，则让清空按钮为禁用状态。
    //   首先 遍历所有数据， 判断 每一个的select是否为选 中为true,  就让edit为true,
    edit() {
      let edit = false;
      this.filterData.screenBy.forEach(screen => {
        screen.data.forEach(item => {
          if (item.select) {
            edit = true;
          }
        });
      });
      return edit;
    }
  },

  methods: {
    // 点击导航 执行的方法  传入index
    filterSort(index) {
      this.currentFilter = index;
      switch (index) {
        case 0:
          this.isSort = true; //显示下拉菜单
          this.$emit("searchFixed", true); // 并让搜索商家框 置顶
          break;
        case 1:
          // 如果点击的是 距离最近，则触发 updata方法  将数据中的条件值 传递过去。
          this.$emit("update", {
            condition: this.filterData.navTab[1].condition
          });
          this.hideView(); // 执行隐藏方法
          break;
        case 2:
          this.$emit("update", {
            condition: this.filterData.navTab[2].condition
          });
          this.hideView();
          break;
        case 3:
          this.isScreen = true; // 显示筛选 菜单
          this.isSort = false;
          this.$emit("searchFixed", true); // 并让搜索商家框 置顶
          break;
        default:
          this.hideView(); //  默认执行隐藏
          break;
      }
    },

    // 蒙层的隐藏方法
    hideView() {
      this.isSort = false;
      this.isScreen = false;
      this.$emit("searchFixed", false);
    },

    // 综合排序-下拉菜单的点击方法
    selectSort(item, index) {
      this.currentSort = index;
      //  让选中下拉菜单名称 赋值给 导航中的名称。
      this.filterData.navTab[0].name = this.filterData.sortBy[index].name;
      this.hideView();

      // 更新数据   传入 当前点击的那个一项的 条件值   item.code
      this.$emit("update", { condition: item.code });
    },

    //  筛选 下拉菜单的点击方法   item: data中的每一项    screen是screenBy中的每一项
    selectScreen(item, screen) {
      // MPI 代表可多选  ！== MPI  就是单选
      if (screen.id !== "MPI") {
        // 遍历data中的数据，让每一个的select都为false
        screen.data.forEach(ele => {
          ele.select = false;
        });
      }
      // 每点击的一项 进行取返  为 true
      item.select = !item.select;
    },

    // 清空方法
    clearFilter() {
      // 遍历所有的筛选 数据，让所有的select都为false
      this.filterData.screenBy.forEach(screen => {
        screen.data.forEach(item => {
          item.select = false;
        });
      });
    },

    // 确定方法
    filterOk() {
      let screenData = {
        MPI: "",
        offer: "",
        per: ""
      };
      let mpiStr = ""; // 存储 多选拼接字符串
      this.filterData.screenBy.forEach(screen => {
        screen.data.forEach((item, index) => {
          if (item.select) {
            // 两种情况 1.多选 2.单选
            if (screen.id !== "MPI") {
              // 单选
              screenData[screen.id] = item.code;
            } else {
              // 多选 fengniao,pinpai
              mpiStr += item.code + ",";
              screenData[screen.id] = mpiStr;
            }
          }
        });
      });

      this.$emit("update", { condition: screenData });

      this.hideView();
    }
  }
};
</script>

<style scoped>
.filter_wrap {
  background: #fff;
  position: sticky;
  top: 54px;
  z-index: 10;
}
.filter {
  position: relative;
  border-bottom: 1px solid #ddd;
  line-height: 10.4vw;
  z-index: 101;
  height: 10.666667vw;
  display: flex;
  justify-content: space-around;
}
.filter-nav {
  flex: 1;
  text-align: center;
  color: #666;
  font-size: 0.8333rem;
}
.filter-nav i {
  width: 1.6vw;
  height: 0.8vw;
  margin-left: 1.333333vw;
  margin-bottom: 0.533333vw;
  fill: #333;
  will-change: transform;
}

.filter-bold {
  font-weight: 600;
  color: #333;
}

/* mask 蒙层 */
.open {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  transition: all 0.3s ease-in-out;
  z-index: 3;
}

.filter-extend {
  background-color: #fff;
  color: #333;
  padding-top: 2.133333vw;
  position: absolute;
  width: 100%;
  z-index: 4;
  left: 0;
  top: 24.533333vw;
}
.filter-extend li {
  position: relative;
  padding-left: 5.333333vw;
  line-height: 10.666667vw;
  overflow: hidden;
}
.fa-check {
  float: right;
  color: #009eef;
  margin-right: 3.733333vw;
  line-height: 10.666667vw;
}

.selectName {
  color: #009eef;
}
/* 筛选 */
.filter-sort {
  background: #fff;
  padding: 0 2.666667vw;
  line-height: normal;
}
.morefilter {
  margin: 2.666667vw 0;
  overflow: hidden;
}
.morefilter .title {
  margin-bottom: 2vw;
  color: #666;
  font-size: 0.5rem;
}
.morefilter ul {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  font-size: 0.8rem;
}
.morefilter li {
  box-sizing: border-box;
  width: 30%;
  height: 9.333333vw;
  line-height: 9.333333vw;
  margin: 0.8vw 1%;
  background: #fafafa;
}
.morefilter li img {
  width: 3.466667vw;
  height: 3.466667vw;
  vertical-align: middle;
  margin-right: 0.8vw;
}
.morefilter li span {
  margin-left: 2%;
  vertical-align: middle;
}

.morefilter-btn {
  display: flex;
  justify-content: space-around;
  align-items: center;
  background-color: #fafafa;
  box-shadow: 0 -0.266667vw 0.533333vw 0 #ededed;
  line-height: 11.466667vw;
  box-sizing: border-box;
}
.morefilter-btn span {
  font-size: 0.826667rem;
  text-align: center;
  text-decoration: none;
  flex: 1;
}
.morefilter-clear {
  color: #ddd;
  background: #fff;
}
.morefilter-ok {
  color: #fff;
  background: #00d762;
  border: 0.133333vw solid #00d762;
}

.selected {
  color: #3190e8 !important;
  background-color: #edf5ff !important;
}

.edit {
  color: #333 !important;
}
</style>
