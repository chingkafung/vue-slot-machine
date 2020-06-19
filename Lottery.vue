<template>
  <div>
    <div class="bg">
      <div class="van-mech">
        <div class="van-slots-box">
          <div class="van-slot-box" v-for="item in slots" :key="item.title">
            <div class="van-slot-box-inner">
              <div class="van-slot-items" :style="{transform: 'translateY('+item.trans+'px)'}">
                <div class="van-slot-item-wrap">
                  <div
                    class="van-slot-item"
                    :style="{backgroundColor: item.items[item.items.length-1].bg}"
                  >{{item.items[item.items.length-1].name}}</div>
                </div>
                <div v-for="(sitem,index) in item.items" :key="index" class="van-slot-item-wrap">
                  <div class="van-slot-item" :style="{backgroundColor: sitem.bg}">{{sitem.name}}</div>
                </div>
                <div v-for="(sitem,index) in item.items" :key="index" class="van-slot-item-wrap">
                  <div class="van-slot-item" :style="{backgroundColor: sitem.bg}">{{sitem.name}}</div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <button class="van-btn" @click="clickBtn">button</button>
      </div>
    </div>
  </div>
</template>

<script>
(function() {
  var lastTime = 0;
  var vendors = ["webkit", "moz"];
  for (var x = 0; x < vendors.length && !requestAnimationFrame; ++x) {
    requestAnimationFrame = vendors[x] + "RequestAnimationFrame";
    cancelAnimationFrame =
      vendors[x] + "CancelAnimationFrame" ||
      vendors[x] + "CancelRequestAnimationFrame";
  }
  if (!requestAnimationFrame)
    requestAnimationFrame = function(callback) {
      var currTime = new Date().getTime();
      var timeToCall = Math.max(0, 16 - (currTime - lastTime));
      var id = setTimeout(function() {
        callback(currTime + timeToCall);
      }, timeToCall);
      lastTime = currTime + timeToCall;
      return id;
    };
  if (!cancelAnimationFrame)
    cancelAnimationFrame = function(id) {
      clearTimeout(id);
    };
})();
const nextRound =
  typeof window === "object" && window.requestAnimationFrame
    ? window.requestAnimationFrame
    : requestAnimationFrame;

export default {
  name: "",
  data: function() {
    return {
      picture: "",
      loadimage: [],
      style: "background:rgba(0,0,0,0)",
      isLogined: "false",
      isShow: false,
      isClient: false,
      isshowFree: false,
      isscaleBig: false,
      isMaskshow: false,
      lotteryNumber: 0,
      giftType: 0,
      price1: 100,
      address: "德基广场一期一楼中庭扶梯旁",
      selected: 1, // 指定结果
      isDown: false,
      isOdd: false,
      slotsData: [
        // 抽奖数据
        { name: "1", isActived: 0, bg: "red" },
        { name: "2", isActived: 0, bg: "orange" },
        { name: "3", isActived: 0, bg: "blue" }
      ],
      slots: [
        // 抽奖数据分组
        {
          title: "组1",
          trans: -75,
          items: []
        },
        {
          title: "组2",
          trans: -75,
          items: []
        },
        {
          title: "组3",
          trans: -75,
          items: []
        }
      ],
      slotsIndex: [-1, -1, -1], // 中奖索引值
      slotsOpts: null, // 抽奖临时内存区
      slotsStartedAt: null // 动画内存区
    };
  },
  components: {},
  watch: {
    // 指定中奖项
  },
  async created() {
   
    this.slots[0].items = this.slotsData.slice(0);
    this.slots[1].items = this.slotsData.slice(0);
    this.slots[2].items = this.slotsData.slice(0);
    this.slotsIndex = [1, 1, 1];
  },
  mounted() {
    const that = this;
  },
  methods: {
    getRanArr() {
      const _arr = [];
      for (let i = 0; i < 3; i++) {
        const _rd = Math.random() * 3;
        _arr.push(Math.floor(_rd));
      }
      return _arr;
    },
    toRules() {},
    toFree() {},
    async toReceipt() {},
    async toPrize() {},
    /**
     * 获取抽奖结果序列
     */
   

    /**
     * 点击抽奖
     */
    async clickBtn() {
      const that = this;
      //  如果在抽奖中...不能点击
      if (that.slotsOpts) {
        return false;
      }
      // 每一个奖品标签的高度，根据实际高度变化(注意 margin 和 padding)
      const itemHeight = 75;

      that.slotsOpts = that.slots.map((data, i) => {
        let choice = that.slotsIndex[i]; // 中奖了，根据序列返回结果
        if (that.slotsIndex[0] === -1) {
          choice = Math.floor(Math.random() * data.items.length); // 没中奖，随机返回结果
        }
        // 初始化定义动画动作结果
        const opts = {
          finalPos: choice * itemHeight, // 最终转动位置
          startOffset: 8000 + Math.random() * 500 + i * 1600, // 影响转的圈数
          height: data.items.length * itemHeight, // 总长度
          duration: 5000 + i * 3500, // 持续时间，每一个奖品持续8秒，以此累加
          // startOffset: 1000 + Math.random() * 500 + i * 1, // 影响转的圈数
          // height: data.items.length * itemHeight, // 总长度
          // duration: 10000 + i * 40000, // 持续时间，每一个奖品持续8秒，以此累加
          isFinished: false // 是否完成了
        };
        return opts;
      });

      nextRound(that.slotsAnimate); // 开启抽奖动画
    },

    /**
     * 抽奖动画
     * @param  {[number]}   timestamp 当前的方法持续的毫秒数
     */
    slotsAnimate: function(timestamp) {
      // 是否已经在转动了
      if (this.slotsStartedAt === null) {
        this.slotsStartedAt = timestamp; // 动画初始时间
      }

      const timeDiff = timestamp - this.slotsStartedAt; // 动画持续的时间

      this.slotsOpts.forEach((opt, i) => {
        // 完成后停止转动
        if (opt.isFinished) {
          return false;
        }

        const timeRemaining = Math.max(opt.duration - timeDiff, 0); // 总的持续时间 - 动画持续时间 = 剩下的时间,0表示结束
        const power = 3; // 动画转动的力度
        const offset =
          (Math.pow(timeRemaining, power) / Math.pow(opt.duration, power)) *
          opt.startOffset; // 偏移量
        const pos = -1 * Math.floor((offset + opt.finalPos) % opt.height); // 转动值

        // 如果持续时间大于总持续时间，则该项表示完成
        if (timeDiff > opt.duration) {
          opt.isFinished = true;
        }

        // 得到转动数据
        this.slots[i].trans = pos;
      });

      // 当所有项已经完成，则停止转动，回归初始状态
      if (this.slotsOpts.every(o => o.isFinished)) {
        this.slotsOpts = null;
        this.slotsStartedAt = null;
      } else {
        // 否则继续转动
        nextRound(this.slotsAnimate);
      }
    }
  }
};
</script>
<style  scoped>
.bg {
  width: 100%;
  height: 100vh;
  background-size: cover;
}

.van-slots-box {
  display: flex;
}
.van-slots-box-title {
  width: 200px;
  font-size: 15px;
  color: #fff;
  top: 72px;
  left: 87px;
  text-align: center;
}
.van-slot-box {
  margin-right: 19px;
}
.van-slot-box-inner {
  height: 110px;
  padding-top: 17px;
  box-sizing: border-box;
  overflow: hidden;
}
.van-slot-box-inner.move {
  background: linear-gradient(to right, #0be881 0%, #4bcffa 50%, #0be881 100%);
  background-size: 300%;
  animation: sbg 1s infinite alternate;
}
.van-slot-items {
  margin-top: -75px;
  /* padding: 0 10px; */
}
.van-slot-item-wrap {
  overflow: hidden;
}
.van-slot-item {
  height: 65px;
  width: 65px;
  margin: 5px 0;
  line-height: 65px;
  text-align: center;
  background: red;
  color: #fff;
  background-size: cover;
}
.sele {
  display: block;
  margin: 20px auto;
  width: 200px;
  height: 48px;
  border: 1px solid #eee;
  color: #555;
  outline: none;
  background: #fff;
}
.van-btn {
  width: 121px;
  height: 68px;
  border: none;
  outline: none;
}
@keyframes sbg {
  0% {
    background-position: 100%;
  }
  50% {
    background-position: -100%;
  }
  100% {
    background-position: 0;
  }
}
.btn-wrap {
  display: flex;
  justify-content: space-between;
}
.bottom {
  width: 336px;
  margin: 0 auto;
}
</style>

<style>
@media screen and (min-height: 700px) {
  .bg .lottery-module_header .lottery-module_title {
    margin-top: 80px;
  }
}
</style>