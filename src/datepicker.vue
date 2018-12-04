<template>
  <div class="w-datepicker" @click.stop="light()">
    <input class="w-date" :class="{ 'w-date-focus': focus }" :value="value" readonly />
    <transition name="w-ani">
    <div class="w-calendar" v-show="focus">
      <div class="w-calendar-header">
        <span class="w-calendar-year">
          <button class="w-calendar-btn" @click="updateCalendar(year -= 1)">&#60;</button>
          <strong>{{ year }}</strong>
          <button class="w-calendar-btn" @click="updateCalendar(year += 1)">&#62;</button>
        </span>
        <span class="w-calendar-month">
          <button class="w-calendar-btn" @click="updateCalendar(month > 1 && (month -= 1))">&#60;</button>
          <strong>{{ computedMonth }}</strong>
          <button class="w-calendar-btn" @click="updateCalendar(month < 12 && (month += 1))">&#62;</button>
        </span>
      </div>
      <table class="w-calendar-body">
        <tr class="w-calendar-week">
          <td style="color: red">Sun</td>
          <td>Mon</td>
          <td>Tues</td>
          <td>Wed</td>
          <td>Thur</td>
          <td>Fri</td>
          <td>Sat</td>
        </tr>
        <tr class="w-calendar-date" v-for="(week, index) in calendar" :key="index">
          <td v-for="(date, day) in week" :key="day"><button :style="{ color: day === 0 && !date.disabled ? 'red' : '' }" class="w-calendar-btn" :disabled="date.disabled" @click.stop="submit(date.value)">{{ date.value }}</button></td>
        </tr>
      </table>
    </div>
    </transition>
  </div>
</template>

<script>
import { formatDate } from "./util";
const DATE = new Date()
export default {
  name: 'w-calendar',
  props: {
    value: { type: String, default: formatDate(DATE, '-') },
    separator: { type: String, default: '-' },
  },
  model: {
    prop: 'value',
    event: 'change'
  },
  data() {
    return {
      focus: false,
      year: DATE.getFullYear(),
      month: DATE.getMonth() + 1,
      calendar: []
    }
  },
  methods: {
    dim() {
      this.focus = false
    },
    light() {
      this.focus = true
    },
    submit(date) {
      this.focus = false
      const dateStr = `${this.year}${this.separator}${this.month}${this.separator}${date < 10 ? ('0' + date) : date}`
      this.$emit('change', dateStr)
    },
    updateCalendar() {
      const year = this.year
      const month = this.month
      const calendar = []
      const days31 = [1,3,5,7,8,10,12]
      const isLeap = year => (year % 4 === 0 && year % 100 !== 0) || year % 400 === 0
      let lastDay
      if (month === 2) lastDay = isLeap(year) ? 29 : 28
      else if (days31.includes(month)) lastDay = 31
      else lastDay = 30
      const firstWeekDay = new Date(year + '-' + month + '-01').getDay() 
      for (let i = 1; i <= 31; i++) {
        if (i > lastDay) break
        const date = new Date(year + '-' + month + '-' + (i < 10 ? ('0' + i) : i))
        const weekNum = Math.floor((i + firstWeekDay - 1) / 7)
        if (!calendar[weekNum]) calendar[weekNum] = []
        const week = calendar[weekNum]
        const day = date.getDay()
        if (i === 2) {
          console.log(JSON.parse(JSON.stringify(weekNum)))
        }
        if (i === 1) {
          const lastMonth = (month - 1) || 12
          let lastDay
          if (lastMonth === 2) lastDay = isLeap(year) ? 29 : 28
          else if (days31.includes(lastMonth)) lastDay = 31
          else lastDay = 30
          for (let disabledDay = day - 1; disabledDay >= 0; disabledDay--) {
            week[disabledDay] = { value: lastDay - (day - disabledDay - 1), disabled: true }
          }
          console.log(JSON.parse(JSON.stringify(week)))
        }
        else if (i === lastDay) {
          const nextMonth = month + 1 === 13 ? 1 : month + 1
          for (let disabledDay = day + 1, date = 1; disabledDay < 7; disabledDay++, date++) {
            week[disabledDay] = { value: date, disabled: true }
          }
        }
        week[day] = { disabled: false, value: i }
      }
      this.calendar = calendar
    }
  },
  computed: {
    computedMonth() {
      return this.month < 10 ? ('0' + this.month) : this.month
    }
  },
  mounted() {
    this.year = Number(this.value.split(this.separator)[0])
    this.month = Number(this.value.split(this.separator)[1])
    this.updateCalendar()
    document.addEventListener('click', this.dim)
   },
  beforeDestroy() { 
    document.removeEventListener('click', this.dim) 
  }
}
</script>

<style>
.w-datepicker {
  position: relative;
  display: inline-block;
}
.w-date {
  border: .8px solid #ccc;
  border-radius: 4px;
  padding: 2px 3px;
  outline: none;
  transition: border .25s;
}
.w-date-focus {
  border-color: #0ae;
  box-shadow: 0 0 20px -5px #0ae;
}
.w-calendar {
  width: 230px;
  padding: 5px;
  position: absolute;
  overflow: hidden;
  background-color: #fff;
  border: 1px solid #ddd;
  border-radius: 5px;
  margin-top: 5px;
  z-index: 999;
}
.w-calendar-btn {
  border: none;
  border-radius: 4px;
  background-color: transparent;
  outline: none;
  cursor: pointer;
  transition: all .3s;
}
.w-calendar-btn:hover {
  background-color: #eee;
}
.w-calendar-btn:disabled {
  background-color: transparent!important;
  cursor: not-allowed;
}
.w-calendar-btn:active {
  background-color: #ddd;
}
.w-calendar-header {
  text-align: center;
  border-radius: 3px;
  height: 22px;
}
.w-calendar-body {
  font-size: 13px;
  width: 100%;
}
.w-calendar table {
  text-align: center;
}
.w-ani-enter, .w-ani-leave-to {
  opacity: 0;
  transform: scaleY(0) translateY(-100px);
}
.w-ani-enter-active, .w-ani-leave-active {
  transition: all .15s;
}
</style>
