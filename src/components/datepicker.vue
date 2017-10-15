<template>
    <div class="datepicker" :class="{active:isActive}" onselectstart="return false">
        <slot>
            <div @click="show()">请选择</div>
        </slot>
        <div class="datepicker-mask" @click="cancel()" :class="{active:isActive}"></div>
        <transition :name="transition">
            <div class="datepicker-content" v-if="isActive">
                <div class="datepicker-buttons">
                    <span class="button-cancel" @click="cancel()">取消</span>
                    <span class="button-current" @click="setDate()" v-if="showDays">今天</span>
                    <span class="button-current" @click="setMonth()" v-if="!showDays&&showMonths">本月</span>
                    <span class="button-current" @click="setYear()" v-if="!showDays&&!showMonths">本年</span>
                    <span class="button-select" @click="select()">确定</span>
                </div>
                <div class="selected-date">
                    <span class="button-prev" @click="prev()">&lt;</span>
                    <div class="date">
                        <span @click="changeTab(TABS.YEAR)" v-text="formatYear(currentYear)" :class="{active:yearTabIsActive}"></span>
                        <span @click="changeTab(TABS.MONTH)" v-text="formatMonth(currentMonth)" :class="{active:monthTabIsActive}" v-if="showMonths"></span>
                        <span @click="changeTab(TABS.DAY)" v-text="formatDay(currentDay)" :class="{active:dayTabIsActive}" v-if="showDays"></span>
                    </div>
                    <span class="button-next" @click="next()">&gt;</span>
                </div>
                <div class="datepicker-items" :class="{changing:changing}">
                    <ul class="years" v-show="yearTabIsActive">
                        <li v-for="(item,index) in years" @click="selectYear(item)" v-text="item" :class="{item:true,active:currentYear==item}" :key="index"></li>
                    </ul>
                    <ul class="months" v-show="showMonths&&monthTabIsActive">
                        <li v-for="(item,index) in months" @click="selectMonth(item)" v-text="formatMonth(item)" :class="{item:true,active:currentMonth==item}" :key="index"></li>
                    </ul>
                    <ul class="days" v-show="showDays&&dayTabIsActive">
                        <li class="week" v-for="(item,index) in Weeks" v-text="item" :key="index"></li>
                        <li v-for="(item,index) in days" @click="selectDay(item)" v-text="item.day" :class="{item:true,prev:item.prev,next:item.next,active:isActiveDay(item)}" :key="index"></li>
                    </ul>
                </div>
            </div>
        </transition>
    </div>
</template>
<script>
//import Hammer from 'hammerjs'

const TABS = { YEAR: 1, MONTH: 2, DAY: 3 }
const WEEKS = ['一', '二', '三', '四', '五', '六', '日']
 
const SHOW_YEAR_COUNT = 20

export default {
    name: 'DatePicker',
    props: {
        value: String,
        year: Number,
        month: Number,
        day: Number,
        format: String,
        label: String,
        placeholder: {
            type: String,
            default: '请选择'
        },
        transition: {
            type: String,
            default: 'fadeInTop'
        },
        showMonths: {
            type: Boolean,
            default: true
        },
        showDays: {
            type: Boolean,
            default: true
        }
    },
    data() {
        return {
            isActive: false,        
            years: [],           
            months: [],
            days: [],
            currentYear: 0,
            currentMonth: 0,
            currentDay: 0,
            selectedText: '',
            swipeleft: false,
            swiperight: false,
            activeTab: TABS.DAY,
            changing: false,
            TABS,
            Weeks: WEEKS
        }
    },
    computed: {
        yearTabIsActive() {
            return this.activeTab == TABS.YEAR
        },
        monthTabIsActive() {
            return this.activeTab == TABS.MONTH
        },
        dayTabIsActive() {
            return this.activeTab == TABS.DAY
        }
    },
    methods: {
        changeTab(tab) {
            this.activeTab = tab
            if (tab == TABS.DAY) {
                this.createDays()
            } else if (tab == TABS.MONTH) {
                if (this.months.length == 0) {
                    this.createMonths()
                }
            } else {
                if (this.years.length == 0) {
                    this.createYears(this.currentYear)
                }
            }
        },
        formatYear(year) {
            return year + '年'
        },
        formatMonth(month) {
            return month + 1 + '月'
        },
        formatDay(day) {
            return day + '日'
        },
        isActiveDay(dayItem) {
            return this.currentMonth == dayItem.month && this.currentDay == dayItem.day
        },
        prev() {
            this.changing = true
            if (this.yearTabIsActive) {
                this.createYears(this.startYear - 1)
            } else {
                if (this.currentMonth == 0) {
                    this.currentYear--
                    this.currentMonth = 11
                } else {
                    this.currentMonth--
                }
                if (this.dayTabIsActive) {
                    this.createDays()
                }
            }
            setTimeout(() => {
                this.changing = false
            }, 250)

        },
        next() {
            this.changing = true
            if (this.yearTabIsActive) {
                this.createYears(this.endYear + SHOW_YEAR_COUNT)
            } else {
                if (this.currentMonth == 11) {
                    this.currentYear++
                    this.currentMonth = 0
                } else {
                    this.currentMonth++
                }
                if (this.dayTabIsActive) {
                    this.createDays()
                }
            }
            setTimeout(() => {
                this.changing = false
            }, 250)
        },
        createYears(endYear) {
            this.startYear = endYear - SHOW_YEAR_COUNT + 1
            this.endYear = endYear
            this.years = []
            for (var i = this.startYear; i <= endYear; i++) {
                this.years.push(i)
            }
        },
        createMonths() {
            let months = []
            for (var i = 0; i <= 11; i++) {
                this.months.push(i)
            }
        },

        createDays() {
            this.days = this.getDays(this.currentYear, this.currentMonth)
        },
        getDays(year, month, startWeek = 1) {
            let days = []

            let lastDayOfMonth = new Date(year, month + 1, 0).getDate()

            let firstDayOfMonth = new Date(year, month, 1)

            let firstDayOfWeek = firstDayOfMonth.getDay()

            if (firstDayOfWeek == 0) { //0表示星期天,这里从周一开始排列,改为7
                firstDayOfWeek = 7
            }
            let prevMonthDays = firstDayOfWeek - startWeek - 1//从指定星期开始计算
            //如果第一天不是起始周,计算上个月补位的天数
            if (firstDayOfWeek != startWeek) {
                firstDayOfMonth.setDate(-1 * prevMonthDays)
                let prevMonth = new Date(firstDayOfMonth.getFullYear(), firstDayOfMonth.getMonth() + 1, 0)
                let prevMonthEndDate = prevMonth.getDate()
                //补上个月时间
                for (var i = firstDayOfMonth.getDate(); i <= prevMonthEndDate; i++) {
                    days.push({
                        day: i,
                        prev: true,
                        month: month - 1
                    })
                }
            }
            for (var i = 1; i <= lastDayOfMonth; i++) {
                days.push({
                    day: i,
                    month: month
                })
            }
            //按照要显示总数,计算补位下个月天数
            let nextMonthDays = 6 * 7 - lastDayOfMonth - prevMonthDays - 1
            for (let i = 1; i <= nextMonthDays; i++) {
                days.push({
                    day: i,
                    next: true,
                    month: month + 1
                })
            }
            return days
        },
        selectMonth(month) {
            this.currentMonth = month
            if (this.showDays) {
                this.changeTab(TABS.DAY)
            }
        },
        selectYear(year) {
            this.currentYear = year
            if (this.showMonths) {
                this.changeTab(TABS.MONTH)
            }
        },
        selectDay(dayItem) {
            this.currentDay = dayItem.day
            this.currentMonth = dayItem.month
        },
        setDate(date) {
            this.setCurrent(date || new Date())
            this.activeTab = TABS.DAY
            this.createDays()
        },
        setMonth(date) {
            this.setCurrent(date || new Date())
            this.activeTab = TABS.MONTH
            this.createMonths()
        },
        setYear(date) {
            this.setCurrent(date || new Date())
            this.activeTab = TABS.YEAR
            this.createYears(this.currentYear)
        },
        setCurrent(date) {
            this.currentYear = date.getFullYear()
            this.currentMonth = date.getMonth()
            this.currentDay = date.getDate()
        },
        getCurrent() {
            return new Date(this.currentYear, this.currentMonth, this.currentDay)
        },
        select() {
            let date = this.getCurrent()
            let value = this.updateValue()
            this.$emit('select', date)
            this.$emit('input', value)
            this.cancel()
        },
        updateValue(date) {
            let format = this.format
            if (!format) {
                if (this.showDays) {
                    format = 'yyyy-MM-dd'
                } else if (this.showMonths) {
                    format = 'yyyy-MM'
                } else {
                    format = 'yyyy'
                }
            }
            date = date || this.getCurrent()
            let updateValue = '';//FormatHelper.formatDate(date, format)
            this.selectedText = updateValue
            return updateValue
        },
        show() {
            this.isActive = true
            // this.$nextTick(() => {
            //     this.initGesture()
            // })
        },
        cancel() {
            this.isActive = false
        },
        initGesture() {
            let container = this.$el.querySelector('.items')
            if (!container) {
                //console.log('初始化手势失败')
                return
            }
            let hammer = new Hammer(container)
            hammer.on('swipeleft', (event) => {
                this.next()
            })
            hammer.on('swiperight', (event) => {
                this.prev()
            })
        },
        init() {

            let date = this.value && new Date(this.value)
            if (this.showDays) {
                this.setDate(date)
            } else if (this.showMonths) {
                this.setMonth(date)
            } else {
                this.setYear(date)
            }
            if (this.value) {
                this.updateValue()
            }

        }
    },
    watch: {
        value() {
            this.init()
        }
    },
    created() {
        this.init()

    }
}
</script>
