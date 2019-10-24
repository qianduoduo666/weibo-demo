<template>
  <div>
    <h2>仿微博的@链接功能</h2>
    <!-- 输入框 -->
    <h4>发微博时的输入框</h4>
    <div style="position:relative;">
      <el-input
        ref="indexDef"
        class="m10"
        style="width: 80%;"
        type="textarea"
        :autosize="{ minRows: 4, maxRows: 8}"
        size="small"
        v-model="indexDef"
        placeholder="请输入指标定义"
      ></el-input>
      <!-- @的下拉框 -->
      <div id="popdiv" v-show="isShowPop" class="boxV p5 pop-style">
        <span
          v-for="(item, index) in dropData"
          :key="index"
          @click="handleSelectPop(item.name)"
        >{{ item.name }}</span>
      </div>
      <!-- 用来@弹框定位的隐藏的div -->
      <div id="testdiv" class="el-textarea el-input--small el-textarea__inner m10 testdiv"></div>
    </div>

    <el-button type="primary" size="small" @click="handleComfirmClick">发布</el-button>
    <h4>显示发布的文章结果</h4>
    <!-- 显示结果 -->
    <div id="content-div" class="content-div"></div>
  </div>
</template>

<script>
export default {
  name: "Home",
  data() {
    return {
      indexDef: "",
      isShowPop: false,
      isSelectPop: false,
      cursorIndex: 0, //记录光标的位置
      dropData: [
        { name: "肖战", id: "2" },
        { name: "王一博", id: "3" },
        { name: "林俊杰", id: "6" },
        { name: "何炅", id: "7" },
        { name: "白敬亭", id: "14" }
      ], //下拉框的数据
      selectItems: [] //存储下拉框选中的数据
    };
  }, // end data
  watch: {
    indexDef(val) {
      if (this.isSelectPop) {
        this.isSelectPop = false; //说明是选中pop的处理，那么不走下面的逻辑
        return;
      }
      var textareaDom = this.$refs.indexDef.$el.querySelector("textarea");
      //获取光标所在位置
      this.cursorIndex = 0; //清空
      if (textareaDom.selectionStart || textareaDom.selectionStart == "0") {
        this.cursorIndex = textareaDom.selectionStart;
      }
      if (this.cursorIndex) {
        //说明不是在最开始的位置，那么判断光标前一个字符是不是@
        let chart = val.charAt(this.cursorIndex - 1);
        if (chart == "@") {
          //说明是@字符，替换@
          let spanId = "spanId";
          let replaceStr = "<span id='" + spanId + "'>@</span>";
          var value = val; //每次都重新赋值了，所以span的创建在div上只会有一个，也就是当前光标所在的@的位置
          value =
            value.substr(0, this.cursorIndex - 1) +
            replaceStr +
            value.substr(this.cursorIndex, value.length);
          var div = document.getElementById("testdiv");
          div.innerHTML = value;

          var span = document.getElementById(spanId);
          let postop = span.offsetTop;
          let posleft = span.offsetLeft;

          this.isShowPop = true;
          var popdiv = document.getElementById("popdiv");
          popdiv.style.left = posleft + 25 + "px";
          popdiv.style.top = postop + 25 + "px";
        } else {
          this.isShowPop = false;
        }
      }
    }
  }, // end watch
  created() {
    //初始化
    this.$nextTick(() => {
      var textareaDom = this.$refs.indexDef.$el.querySelector("textarea");
      var testdiv = document.getElementById("testdiv");
      testdiv.style.top = 0;
      testdiv.style.left = 0;
      testdiv.style.height = textareaDom.style.height;

      //增加事件
      // var div = document.getElementById("content-div");
    });
  }, // end created
  methods: {
    //处理选择的数据
    handleSelectPop(val) {
      //插入数据
      this.indexDef =
        this.indexDef.substr(0, this.cursorIndex - 1) +
        " " +
        "@" +
        val +
        " " +
        this.indexDef.substr(this.cursorIndex, this.indexDef.length);
      this.indexDef.substr(this.cursorIndex, this.indexDef.length);
      this.isShowPop = false;
      this.isSelectPop = true;
    },
    //确定按钮的点击事件
    handleComfirmClick() {
      this.selectItems = [];
      this.dropData.forEach(item => {
        if (this.indexDef.indexOf(item.name) != -1) {
          //说明包含选中的数据，但是数据不一定只出现一次，有可能出现多次，那么需要获取所有的
          let positions = this.getStringAllIndex(this.indexDef, item.name);
          positions.forEach(index => {
            this.selectItems.push({
              name: "@" + item.name,
              id: item.id,
              index: index - 1
            });
          });
        }
      });
      this.selectItems.sort((a, b) => {
        return a.index - b.index;
      });
      //显示结果
      this.handleResultShow();
    },
    //获取字符串出现的所有位置
    getStringAllIndex(str, char) {
      var positions = [];
      var pos = str.indexOf(char);
      while (pos > -1) {
        positions.push(pos);
        pos = str.indexOf(char, pos + 1);
      }
      return positions;
    },
    //获取显示的结果
    handleResultShow() {
      let arr = this.handleStringSplit(this.indexDef, this.selectItems);
      var div = document.getElementById("content-div");
      div.innerHTML = ""; //清空数据
      arr.forEach((item, index) => {
        if (typeof item == "string") {
          //说明是字符串
          div.innerHTML += item;
        } else {
          //说明是对象
          let alink = `<a id=${item.id} style='margin: 0px 5px;'>${item.name}</a>`;
          div.innerHTML += alink;
          let _this = this;
          this.$nextTick(() => {
            let alinkDom = document.getElementById(item.id);
            alinkDom.addEventListener("click", function() {
              _this.handleLinkClick(item);
            });
          });
        }
      });
    },
    //拆分字符串
    handleStringSplit(str, splits) {
      var newArr = [];
      if (splits.length) {
        //说明是有@的数据
        var newstr = str;
        var tempChart = splits[0]; //取出第一个
        for (var i = 0; i < splits.length; i++) {
          newstr = newstr.split(splits[i].name).join(tempChart.name);
        }
        let newstrArr = newstr.split(tempChart.name);
        splits.forEach((item, index) => {
          //两个数组拼接
          if (newstrArr[index]) {
            //说明有值，那么添加，过滤掉空值
            newArr.push(newstrArr[index]);
          }
          newArr.push(item);
        });
        //添加最后一个，能够直接这样做，是因为拆分是根据splits这个来的，那么最后得到的拆分数组的个数，一定是比splits这个数组多一个
        if (newstrArr[newstrArr.length - 1]) {
          newArr.push(newstrArr[newstrArr.length - 1]);
        }
      } else {
        newArr = [str];
      }
      return newArr;
    },
    handleLinkClick(item) {
      this.$alert("你@的是" + item.name, "链接");
    }
  } // end methods
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.pop-style {
  position: absolute;
  width: 200px;
  height: 200px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  background-color: aqua;
}
.testdiv {
  position: absolute;
  overflow: hidden;
  z-index: -9999;
  width: 80%;
  height: 88px;
  /*强制中文换行*/
  white-space: pre-wrap;
  /*只对英文起作用，以单词作为换行依据*/
  word-break: break-all;
}
.m10 {
  margin: 10px;
}
.p5 {
  padding: 5px;
}
.boxV {
  display: flex;
  /*flex-flow: coloum;*/
  flex-direction: column;
}
.content-div {
  margin: 10px;
  width: 80%;
  height: 88px;
  border: 1px solid gray;
  padding: 10px;
  line-height: 23px;
  word-wrap: break-word;
}
</style>
<style>
div a {
  color: blue;
  text-decoration: underline;
  cursor: pointer;
}
</style>
