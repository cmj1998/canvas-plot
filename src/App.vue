<template>
  <div id="app">
    <!-- 功能按钮组 -->
    <div id="max-header">
      <!-- 画笔 -->
      <el-button
        plain
        @click="controller = 1"
        title="画笔"
        icon="el-icon-edit"
        :class="[ controller==1?'pitch-btn':'' ]"
      ></el-button>
      <!-- 方形 -->
      <el-button plain @click="controller = 2" title="方形" :class="[ controller==2?'pitch-btn':'' ]">
        <i class="btn-icon" :style="{ 'border-color' : controller==2? '#fff':'#000'}"></i>
      </el-button>
      <!-- 圆 -->
      <el-button plain @click="controller = 3" title="圆" :class="[ controller==3?'pitch-btn':'' ]">
        <i
          class="btn-icon"
          style="border-radius:50%;"
          :style="{ 'border-color' : controller==3? '#fff':'#000'}"
        ></i>
      </el-button>
      <!-- 直线 -->
      <el-button plain @click="controller = 4" title="直线" :class="[ controller==4?'pitch-btn':'' ]">
        <i
          class="btn-icon"
          style="height:0"
          :style="{ 'border-color' : controller==4? '#fff':'#000'}"
        ></i>
      </el-button>
    </div>
    <div id="max-box" :style="{'cursor':isCursor?'default':'none'}">
      <!-- 画布 -->
      <canvas
        id="myCanvas"
        width="1300"
        height="800"
        @mousedown="canvas($event)"
        @mouseenter="controller == 10 ? isShowEraser = true :isShowEraser = false"
        @mouseleave="isShowEraser = false"
      ></canvas>
      <!-- 清空画布按钮 -->
      <el-button
        type="primary"
        icon="el-icon-delete"
        class="btn__del-canvas"
        @click="emptyCanvas"
        title="清空画布"
      ></el-button>
      <!-- 橡皮擦 -->
      <el-button
        type="primary"
        icon="el-icon-copy-document"
        class="btn__eraser-canvas"
        @click="eraser($event)"
        title="橡皮擦"
      ></el-button>
      <div id="eraser__class" v-show="isShowEraser"></div>
      <!-- 颜色选择 -->
      <div class="btn-col">
        <el-color-picker v-model="color1" title="颜色选择"></el-color-picker>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: "app",
  data() {
    return {
      c: null, //画布
      ctx: null, //画布的getContext("2d")
      maxBox: null, //包裹画布的盒子
      eraserClass: null, //橡皮擦样式
      isCursor: true, //是否显示鼠标
      isShowEraser: false, //显示橡皮擦
      color1: "#000", //颜色选择器
      controller: 1 //功能控制器
    };
  },
  methods: {
    //定义canvas,获取节点
    defCanvas() {
      this.c = document.getElementById("myCanvas");
      this.ctx = this.c.getContext("2d");
      this.maxBox = document.getElementById("max-box");
      this.eraserClass = document.getElementById("eraser__class");
    },
    //画布功能  1=画笔 2=方块
    canvas(eve) {
      switch (this.controller) {
        //调用画笔
        case 1:
          this.brush(eve);
          break;
        //调用方块
        case 2:
          this.diamonds(eve);
          break;
        //调用圆
        //调用直线
        case 4:
          this.solid(eve);
      }
    },
    //画笔 -1
    brush(eve) {
      this.ctx.beginPath(); //重置路径
      this.ctx.moveTo(eve.layerX, eve.layerY); //定点
      let _this = this;
      document.onmousemove = function(eve) {
        _this.ctx.lineTo(eve.layerX, eve.layerY); //划线坐标
        _this.ctx.strokeStyle = _this.color1; //颜色样式
        _this.ctx.stroke(); //绘制
      };
      document.onmouseup = function() {
        document.onmousemove = null; //清除移动事件
      };
    },
    //方块 -2
    diamonds(eve) {
      this.ctx.beginPath(); //重置路径
      let _this = this;
      let x = eve.layerX; //定点X位置
      let y = eve.layerY; //定点y位置
      let w;
      let h;
      var div = document.createElement("div"); //创建div
      div.style.position = "absolute";
      div.style.border = "1px solid #000";
      div.style.borderColor = this.color1;
      this.maxBox.appendChild(div); //插入节点
      document.onmousemove = function(eve) {
        w = eve.clientX - _this.maxBox.offsetLeft - x; //宽度为:当前x轴位置-父元素距离文档的Left位置-定点的x轴位置
        h = eve.clientY - _this.maxBox.offsetTop - y; //高度为:当前y轴位置-父元素距离文档的Top位置-定点的x轴位置
        div.style.left = x + "px"; //div的起始x轴位置为定点x轴的位置
        div.style.top = y + "px"; //div的起始y轴位置为定点y轴的位置
        if (w >= 0 && h >= 0) {
          //右下画方块
          console.log("正数");
          div.style.width = w + "px";
          div.style.height = h + "px";
        } else {
          //左上画方块
          div.style.width = Math.abs(w) + "px";
          div.style.height = Math.abs(h) + "px";
          div.style.left = parseInt(div.style.left) + w + "px";
          div.style.top = parseInt(div.style.top) + h + "px";
        }
      };
      document.onmouseup = function() {
        div.remove(); //鼠标抬起,删除div
        _this.ctx.rect(x, y, w, h); //定义矩形位置,宽高
        _this.ctx.strokeStyle = _this.color1;
        _this.ctx.stroke(); //绘制矩形
        document.onmousemove = null; //清除移动事件
      };
    },
    //圆 -3
    //直线 -4
    solid(eve) {},
    //橡皮擦 -10
    eraser(eve) {
      // this.ctx.save();
      // this.ctx.beginPath();
      // //1.圆的中心的 x 坐标 2.圆的中心的 y 坐标 3.圆的半径 4.起始角，以弧度计。（弧的圆形的三点钟位置是 0 度） 5.结束角，以弧度计  6.可选。规定应该逆时针还是顺时针绘图。False = 顺时针，true = 逆时针。
      // this.ctx.arc(50, 50, 50, 0, 2 * Math.PI);
      // this.ctx.clip();
      // this.ctx.clearRect(0, 0, this.c.width, this.c.height);
      // this.ctx.restore();
      this.isCursor = !this.isCursor;
      this.controller == 10 ? (this.controller = 1) : (this.controller = 10);
      this.isShowEraser = !this.isShowEraser;
      var _this = this;
      this.maxBox.onmousemove = function(eve) {
        _this.eraserClass.style.left =
          eve.clientX - _this.maxBox.offsetLeft + "px";
        _this.eraserClass.style.top =
          eve.clientY - _this.maxBox.offsetTop + "px";
      };
      this.c.onmousedown = function() {
        _this.c.onmousemove = function(eve) {
          _this.ctx.save();
          _this.ctx.beginPath();
          //1.圆的中心的 x 坐标 2.圆的中心的 y 坐标 3.圆的半径 4.起始角，以弧度计。（弧的圆形的三点钟位置是 0 度） 5.结束角，以弧度计  6.可选。规定应该逆时针还是顺时针绘图。False = 顺时针，true = 逆时针。
          _this.ctx.arc(eve.clientX - _this.maxBox.offsetLeft+10, eve.clientY - _this.maxBox.offsetTop+10, 10, 0, 2 * Math.PI);
          _this.ctx.clip();
          _this.ctx.clearRect(0, 0, _this.c.width, _this.c.height);
          _this.ctx.restore();
        };
        _this.c.onmouseup = function() {
          // _this.ctx.restore();
          // _this.ctx.save();
          _this.c.onmousemove = null;
        };
      };
    },
    //清空画布
    emptyCanvas() {
      // this.ctx.clearRect(0, 0, this.c.width, this.c.height);  //在给定的矩形内清除指定的像素
      this.c.width = this.c.width; //高度或宽度被重设时，画布内容就会被清空
      document.onmouseup = null; //解决点击清除按钮后切换功能(画笔,方块...),会显示清除前最后一个图像的BUG
    }
  },
  watch: {
    //监听功能控制的值,不为橡皮擦时(10),显示鼠标,隐藏橡皮擦
    controller: function(newData, oldData) {
      if (newData != 10) {
        this.isCursor = true;
        this.isShowEraser = false;
      }
    }
  },
  mounted() {
    this.defCanvas();
    let _this = this;
    this.maxBox.onmousedown = function(eve) {
      //鼠标右键的点击事件
      if (eve.which == 3) {
        console.log("右键点击");
        _this.isShowEraser = false;
        _this.isCursor = true;
        _this.controller = 1;
        _this.c.onmousedown = null
      }
    };
    //清除浏览器的默认右键点击提示
    document.oncontextmenu = function() {
      return false;
    };
  }
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
html,
body {
  overflow: hidden;
}
#app {
  overflow: hidden;
}
/* 主体上方按钮组 */
#max-header {
  position: absolute;
  /* top:0; */
  width: 100%;
  display: flex;
  justify-content: center;
}
/* 功能按钮选中样式 */
.pitch-btn {
  background: rgb(93, 107, 255) !important;
  color: #fff !important;
}
/* 功能按钮图标样式 */
.btn-icon {
  width: 16px;
  height: 16px;
  border: 1px solid #000;
  display: inline-block;
}
/* 主体画布盒子 */
#max-box {
  width: 1300px;
  height: 800px;
  position: relative;
  margin: 100px auto;
  overflow: hidden;
  border: 1px solid #000;
}
/* 画布 */
#myCanvas {
  /* border: 1px solid #000; */
  display: inline-block;
  overflow: hidden;
}
/* 清空画布 */
.btn__del-canvas {
  position: absolute;
  right: 0px;
  top: 0px;
}
/* 橡皮擦 */
.btn__eraser-canvas {
  position: absolute;
  right: 0px;
  top: 50px;
}
/* 橡皮擦的样式 */
#eraser__class {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  border: 1px solid #000;
  position: absolute;
  top: 0;
  left: 0;
  display: block;
}
.btn-col {
  position: absolute;
  top: 0px;
  right: 80px;
}
</style>
