<template>
  <div class="_calendar-wrap">
    <div class="top_change">
      <div class="icon icon1" @click="PreMonth(myDate, false)"></div>
      <div class="date">{{ dateTop }}</div>
      <div class="icon icon2" @click="NextMonth(myDate, false)"></div>
    </div>
    <div class="main head">
      <div class="content_item" v-for="(tag, index) in textTop" :key="index">
        {{ tag }}
      </div>
    </div>
    <div class="main content">
      <div
        v-for="(item, index) in list"
        @click="clickDay(item, index)"
        class="content_item"
        :key="index"
      >
        <div
          class="item_date"
          :class="[
            { isMark: item.isMark },
            { 'other_day-hide': item.otherMonth !== 'nowMonth' },
            { 'want_day-hide': item.dayHide },
            { isToday: item.isToday },
            { chose_day: item.chooseDay },
            setClass(item),
          ]"
        >
          {{ item.id }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import timeUtil from './calendar';
export default {
  data() {
    return {
      myDate: [],
      list: [],
      historyChose: [],
      dateTop: '',
    };
  },
  props: {
    markDate: {
      type: Array,
      default: () => [],
    },
    markDateMore: {
      type: Array,
      default: () => [],
    },
    textTop: {
      type: Array,
      default: () => ['一', '二', '三', '四', '五', '六', '日'],
    },
    sundayStart: {
      type: Boolean,
      default: () => false,
    },
    agoDayHide: {
      type: String,
      default: `0`,
    },
    futureDayHide: {
      type: String,
      default: `2554387200`,
    },
  },
  created() {
    this.intStart();
    this.myDate = new Date();
  },
  methods: {
    intStart() {
      timeUtil.sundayStart = this.sundayStart;
    },
    setClass(data) {
      let obj = {};
      obj[data.markClassName] = data.markClassName;
      return obj;
    },
    clickDay: function (item, index) {
      if (item.otherMonth === 'nowMonth' && !item.dayHide) {
        this.getList(this.myDate, item.date);
      }
      if (item.otherMonth !== 'nowMonth') {
        item.otherMonth === 'preMonth'
          ? this.PreMonth(item.date)
          : this.NextMonth(item.date);
      }
    },
    ChoseMonth: function (date, isChosedDay = true) {
      date = timeUtil.dateFormat(date);
      this.myDate = new Date(date);
      this.$emit('changeMonth', timeUtil.dateFormat(this.myDate));
      if (isChosedDay) {
        this.getList(this.myDate, date, isChosedDay);
      } else {
        this.getList(this.myDate);
      }
    },
    PreMonth: function (date, isChosedDay = true) {
      date = timeUtil.dateFormat(date);
      this.myDate = timeUtil.getOtherMonth(this.myDate, 'preMonth');
      this.$emit('changeMonth', timeUtil.dateFormat(this.myDate));
      if (isChosedDay) {
        this.getList(this.myDate, date, isChosedDay);
      } else {
        this.getList(this.myDate);
      }
    },
    NextMonth: function (date, isChosedDay = true) {
      date = timeUtil.dateFormat(date);
      this.myDate = timeUtil.getOtherMonth(this.myDate, 'nextMonth');
      this.$emit('changeMonth', timeUtil.dateFormat(this.myDate));
      if (isChosedDay) {
        this.getList(this.myDate, date, isChosedDay);
      } else {
        this.getList(this.myDate);
      }
    },
    forMatArgs: function () {
      let markDate = this.markDate;
      let markDateMore = this.markDateMore;
      markDate = markDate.map(k => {
        return timeUtil.dateFormat(k);
      });
      markDateMore = markDateMore.map(k => {
        k.date = timeUtil.dateFormat(k.date);
        return k;
      });
      return [markDate, markDateMore];
    },
    getList: function (date, chooseDay, isChosedDay = true) {
      const [markDate, markDateMore] = this.forMatArgs();
      this.dateTop = `${date.getFullYear()}.${
        date.getMonth() * 1 > 8
          ? date.getMonth() + 1
          : '0' + (date.getMonth() + 1)
      }`;
      let arr = timeUtil.getMonthList(this.myDate);
      for (let i = 0; i < arr.length; i++) {
        let markClassName = '';
        let k = arr[i];
        k.chooseDay = false;
        const nowTime = k.date;
        const t = new Date(nowTime).getTime() / 1000;
        //看每一天的class
        for (const c of markDateMore) {
          if (c.date === nowTime) {
            markClassName = c.className || '';
          }
        }
        //标记选中某些天 设置class
        k.markClassName = markClassName;
        k.isMark = markDate.indexOf(nowTime) > -1;
        //无法选中某天
        k.dayHide = t < this.agoDayHide || t > this.futureDayHide;
        if (k.isToday) {
          this.$emit('isToday', nowTime);
        }
        let flag = !k.dayHide && k.otherMonth === 'nowMonth';
        if (chooseDay && chooseDay === nowTime && flag) {
          this.$emit('choseDay', nowTime);
          this.historyChose.push(nowTime);
          k.chooseDay = true;
        } else if (
          this.historyChose[this.historyChose.length - 1] === nowTime &&
          !chooseDay &&
          flag
        ) {
          k.chooseDay = true;
        }
      }
      this.list = arr;
    },
  },
  mounted() {
    this.getList(this.myDate);
  },
  watch: {
    markDate: {
      handler(val, oldVal) {
        this.getList(this.myDate);
      },
      deep: true,
    },
    markDateMore: {
      handler(val, oldVal) {
        this.getList(this.myDate);
      },
      deep: true,
    },
    agoDayHide: {
      handler(val, oldVal) {
        this.getList(this.myDate);
      },
      deep: true,
    },
    futureDayHide: {
      handler(val, oldVal) {
        this.getList(this.myDate);
      },
      deep: true,
    },
    sundayStart: {
      handler(val, oldVal) {
        this.intStart();
        this.getList(this.myDate);
      },
      deep: true,
    },
  },
};
</script>
<style lang="scss" scoped>
._calendar-wrap {
  width: 100%;
  margin: 0;
  padding-bottom: 8px;
  background: #fff;
  /* 日历头部 */
  @at-root .top_change {
    position: relative;
    height: 42px;
    line-height: 42px;
    width: 100%;
    padding: 0 16px;
    box-sizing: border-box;
    & > div {
      &.date {
        color: #333;
        font-size: 12px;
        text-align: center;
        width: 100%;
        border-bottom: 1px solid #ccc;
      }
      &.icon {
        position: absolute;
        top: 16px;
        border: 5px solid transparent;
        cursor: pointer;
        &.icon1 {
          border-right-color: #999;
          left: 180px;
          &:hover {
            border-right-color: #36abf2;
          }
        }
        &.icon2 {
          border-left-color: #999;
          right: 180px;
          &:hover {
            border-left-color: #36abf2;
          }
        }
        &.active {
          border-color: #36abf2;
        }
      }
    }
  }

  /* 日期列表 */
  @at-root .main {
    width: 100%;
    @include clearfix;
    & > .content_item {
      font-size: 12px;
      text-align: center;
      color: #333;
      float: left;
      width: calc(100% / 7);
    }
    &.head {
      height: 42px;
      line-height: 42px;
    }
    &.content {
      & > .content_item {
        height: 24px;
        line-height: 24px;
        margin: 4px 0;
        cursor: pointer;
        & > .item_date {
          width: 24px;
          height: 100%;
          margin: auto;
          border-radius: 3px;
          /*今天*/
          &.isToday {
            background: #5b9cf3;
            color: #fff;
          }
          /*选中的日期*/
          &.chose_day {
            background: #5b9cf3;
            color: #fff;
          }
          &.isMark {
          }
          &.other_day-hide {
            color: #ccc;
          }
          &.want_day-hide {
          }
        }
      }
    }
  }
}
</style>
