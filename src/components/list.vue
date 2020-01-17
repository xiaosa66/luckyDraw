<template>
  <div class="list">
    <ul id="gallery">
      <li
        v-for="(people,pIndex) in peoples"
        :key="people.name"
        class="loaded"
        :class="[people.top?'top':'',people.sec?'sec':'',people.thi?'thi':'']"
      >
        <img :src="people.avatar || `/photo/${pIndex+1}.jpg`" :alt="people.name">
      </li>
    </ul>
    <div style="position: fixed;bottom: 10px;right:10px;">
      <button
        v-if="winnerHistory.length"
        class="button-jumbo"
        @click="toggleHistory"
      >
        history
      </button>
      <button
        class="button-jumbo"
        @keyup.enter="toggleBtn"
        @click="toggleBtn"
      >{{ drawing?'stop':'start' }}
      </button>
    </div>
    <div v-if="historyVisible" class="background" @click="toggleHistory">
      <div v-for="(item) in winnerHistory" :key="'_history'+item.name" class="winner-list">

        <div class="section">
          <div>
            <div class="img-box">
              <img :src="item.topPrize.avatar" :alt="item.topPrize.name">
            </div>
            <div class="info-box">
              <div>{{ item.topPrize.name }}</div>
            </div>
          </div>
        </div>
        <div class="section">
          <div v-for="subItem in item.secondPrize" :key="'_sec'+subItem.name">
            <div class="img-box">
              <img :src="subItem.avatar" :alt="subItem.name">
            </div>
            <div class="info-box">
              <div>{{ subItem.name }}</div>
            </div>
          </div>
        </div>
        <div class="section">
          <div v-for="subItem in item.thirdPrize" :key="'thi'+subItem.name">
            <div class="img-box">
              <img :src="subItem.avatar" :alt="subItem.name">
            </div>
            <div class="info-box">
              <div>{{ subItem.name }}</div>
            </div>
          </div>
        </div>

        <!--        <div v-for="subItem in item" :key="'_s'+subItem.name">-->
        <!--          <div class="img-box">-->
        <!--            <img :src="subItem.avatar" :alt="item.name">-->
        <!--          </div>-->
        <!--          <div class="info-box">-->
        <!--            <div>{{ item.name }}</div>-->
        <!--          </div>-->
        <!--        </div>-->
      </div>
    </div>
  </div></template>

<script>
import '@/style/animate.css'
import $ from 'jquery'
let photo_num
// eslint-disable-next-line no-unused-vars
let file_num
let timer_big
let timer_small
let bounce
export default {
  name: 'List',
  props: {
    setting: {
      type: Object,
      default: function() {
        return {
          timeout: 10,
          once: false,
          title: '抽奖',
          subtitle: null,
          api: null,
          showBtn: true,
          speed: 350,
          number: 1,
          _round: 0
        }
      }
    },
    totalRoundNum: {
      type: Number,
      default: 3
    },
    topPrizeNum: {
      type: Number,
      default: 1
    },
    secondPrizeNum: {
      type: Number,
      default: 2
    },
    maxTime: {
      type: Number,
      default: 3
    },
    thirdPrizeNum: {
      type: Number,
      default: 3
    }
  },
  data() {
    return {
      historyVisible: false,
      peoples: [
        {
          name: '111',
          hireDate: true,
          group: 1,
          top: false,
          sec: false,
          thi: false,
          avatar: '/photo/1.jpg'
        }, {
          name: '122',
          hireDate: false,
          group: 1,
          top: false,
          sec: false,
          thi: false,
          avatar: '/photo/1.jpg'
        }, {
          name: '222',
          hireDate: true,
          group: 1,
          top: false,
          sec: false,
          thi: false,
          avatar: '/photo/1.jpg'
        }, {
          name: '211',
          hireDate: false,
          group: 2,
          top: false,
          sec: false,
          thi: false,
          avatar: '/photo/1.jpg'
        }, {
          name: '311',
          hireDate: true,
          group: 3,
          top: false,
          sec: false,
          thi: false,
          avatar: '/photo/1.jpg'
        }, {
          name: '312',
          hireDate: true,
          group: 3,
          top: false,
          sec: false,
          thi: false,
          avatar: '/photo/1.jpg'
        }, {
          name: '313',
          hireDate: true,
          group: 3,
          top: false,
          sec: false,
          thi: false,
          avatar: '/photo/1.jpg'
        }, {
          name: '314',
          hireDate: true,
          group: 3,
          top: false,
          sec: false,
          thi: false,
          avatar: '/photo/1.jpg'
        }, {
          name: '315',
          hireDate: true,
          group: 3,
          top: false,
          sec: false,
          thi: false,
          avatar: '/photo/1.jpg'
        }],
      drawPool: [],
      drawing: false,
      winners: [],
      winnerList: [],
      winnerHistory: [],
      timeout: undefined
    }
  },
  mounted() {
    photo_num = this.peoples.length
    file_num = this.peoples.length
    this.drawPool = JSON.parse(JSON.stringify(this.peoples))
    this.bounceStyle()
  },
  methods: {
    toggleBtn() {
      if (this.drawing) {
        this.handleStopLuckyDraw()
      } else {
        this.handleStartLuckyDraw()
      }
    },
    bounceStyle() {
      // 跳动样式
      bounce = setInterval(function() {
        $('#gallery li:eq(' + Math.ceil(Math.random() * photo_num) + ')')
          .addClass('animated bounce')
          .one('webkitAnimationEnd mozAnimationEnd MSAnimationEnd oanimationend animationend', function() {
            $(this)
              .removeClass('animated bounce')
          })
      }, 600)
    },
    startAnimation() {
      this.showDrawing()
      this.timeout = setTimeout(() => {
        this.toggleBtn()
      }, this.maxTime * 1000)
    },
    handleStartLuckyDraw() {
      this.drawing = true
      this.startAnimation()
    },
    handleStopLuckyDraw() {
      this.drawing = false
      if (this.timeout) {
        clearTimeout(this.timeout)
      }
      this.luckyDraw()
      this.showDrawed()
    },
    getTopPrize() {
      const vv = this.drawPool.filter(p => {
        return p.hireDate
      })
      const topPrizePool = this.washCards(vv)
      for (const i in topPrizePool) {
        console.log(topPrizePool[i].name)
      }
      for (let i = 0, len = topPrizePool.length; i < len + 1; i++) {
        if (this.checkTopGroup(topPrizePool[i])) {
          this.drawPool.splice(this.drawPool.indexOf(topPrizePool[i]), 1)
          return topPrizePool[i]
        }
        if (i === len + 1) {
          alert('无法分配一等奖')
          return {}
        }
      }
    },
    luckyDraw() {
      const topPrize = this.getTopPrize()
      const normalPrizePool = this.washCards(this.drawPool)
      if (normalPrizePool.length < (this.secondPrizeNum + this.thirdPrizeNum)) {
        alert('奖池长度不足以分配')
        return
      }
      const secondPrize = normalPrizePool.splice(0, this.secondPrizeNum)
      const thirdPrize = normalPrizePool.splice(0, this.thirdPrizeNum)
      console.log('一等奖')
      console.log(topPrize.name)
      console.log('二等奖')
      for (const i in secondPrize) {
        console.log(secondPrize[i].name)
      }
      console.log('三等奖')
      for (const i in thirdPrize) {
        console.log(thirdPrize[i].name)
      }
      // console.log(secondPrize)
      // console.log(thirdPrize)
      this.winnerHistory.push({ topPrize, secondPrize, thirdPrize })
    },
    showDrawing() {
      timer_big = setInterval(function() {
        $('#gallery li.focus').removeClass('focus top')
        $('#gallery li:eq(' + Math.ceil(Math.random() * photo_num) + ')').addClass('top')
      }, 10)
      timer_small = setInterval(function() {
        $('#gallery li.focus').removeClass('focus top')
        $('#gallery li:eq(' + Math.ceil(Math.random() * photo_num) + ')').addClass('sec')
      }, 5)
    },
    showDrawed() {
      const arr = this.winnerHistory
      clearInterval(timer_big)
      clearInterval(timer_small)
      clearInterval(bounce)
      for (const i in arr) {
        for (const j in this.peoples) {
          const people = this.peoples[j]
          if (arr[i].topPrize.name === people.name) {
            people.top = true
          }
          for (const k in arr[i].secondPrize) {
            if (arr[i].secondPrize[k].name === people.name) {
              people.sec = true
            }
            for (const k in arr[i].thirdPrize) {
              if (arr[i].thirdPrize[k].name === people.name) {
                people.thi = true
              }
            }
          }
        }
      }
      // $('#gallery li.focus').addClass('hover')
    },
    checkTopGroup(topPrize) {
      if (this.winnerHistory.length) {
        for (const i in this.winnerHistory) {
          const item = this.winnerHistory[i]
          if (item.group === topPrize.group) {
            return false
          } else {
            return true
          }
        }
      } else {
        return true
      }
    },
    rangeRandom(startNum, endNum) {
      return Math.ceil(Math.random() * endNum - startNum)
    },
    washCards(arr) {
      return arr.sort(() => Math.random() > 0.5 ? -1 : 1)
    },
    toggleHistory() {
      this.historyVisible = !this.historyVisible
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    body, div, dl, dt, dd, ul, ol, li, h1, h2, h3, h4, h5, h6, pre,
    code, form, fieldset, legend, input, textarea, p, blockquote, th, td {
        margin: 0;
        padding: 0;
    }

    html {
        height: 100%;
        overflow: hidden;
        background: #000;
    }

    body {
        font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
        min-height: 100%;
        padding: 0;
    }

    #gallery {
        padding: 50px;
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;
    }
    .winner-list {
      display: flex;
      flex-direction: column;
    }
    .section {
      display: flex;
      width: 100%;
      justify-content: center;
      align-items: center;
      flex-direction: row;
    }
    .background {
      height: 100%;
      width: 100%;
      opacity: 80%;
      background-color: black;
      position: absolute;
      top: 0;
      z-index: 3;
    }
    .img-box {
      margin: 30px;
      width: 256px;
      height: 256px;
      border: 6px solid #FFC825;
    }
    .img-box img {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }
    .info-box {
      font-size: 28px;
    }

    #gallery li {
        width: 10%;
        box-sizing: border-box;
        list-style: none;
        perspective: 100px;
        -webkit-perspective: 100px;
        padding: 1px;
        position: relative;
        transition: .1s;
        -webkit-transition: 0.1s;
    }
    .top {
      z-index: 2;
      transform: scale(1.5);
      -webkit-transform: scale(1.5);
    }
    .sec {
      z-index: 2;
      transform: scale(1.2);
      -webkit-transform: scale(1.2);
    }
    .thi {
      z-index: 2;
      transform: scale(1.1);
      -webkit-transform: scale(1.1);
    }
    #gallery li img {
        display: block;
        width: 100%;
        transition: .3s ease-out;
        -webkit-transition: 0.3s ease-out;
    }

    .button-jumbo {
        font-size: 40px;
        width: 200px;
        box-shadow: 0 7px 0 #8bc220, 0 8px 3px rgba(0, 0, 0, 0.3);
        height: 60px;
        line-height: 60px;
        padding: 0 60px;
        position: relative;
        top: 0;
        border-radius: 200px;
        background-color: #a5de37;
        border-color: #a5de37;
        color: white;
        margin: 0;
        display: inline-block;
        cursor: pointer;
        border: none;
    }
</style>
