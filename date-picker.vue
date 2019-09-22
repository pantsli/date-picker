<template>
    <div v-click-outside>
        <input type="text" :value="formatDate" @change="changeDate($event.target.value)">
        <div v-if="isVisible" class="panel">
            <div class="panel-header">
                <span @click="prevYear">&lt;</span>
                <span @click="prevMonth">&lt;&lt;</span>
                <span>{{time.year}}年</span>
                <span>{{time.month + 1}}月</span>
                <span @click="nextMonth">&gt;&gt;</span>
                <span @click="nextYear">&gt;</span>
            </div>
            <div class="panel-body">
                <div class="days">
                    <span class="cell" v-for="j in 7" :key="`_`+j">
                        {{weekDays[j-1]}}
                    </span>
                    <div v-for="i in 6" :key="i">
                        <span class="cell cell-day" 
                            :class="[{notCurrentMonth:!isCurrentMonth(getVisibleDay(i,j))},
                            {today: isToday(getVisibleDay(i,j))},
                            {select: isSelect(getVisibleDay(i,j))}
                            ]" 
                            @click="chooseDate(getVisibleDay(i,j))"
                            v-for="j in 7" :key="j">
                            {{getVisibleDay(i,j).getDate()}}
                        </span>
                    </div> 
                </div>
            </div>
            <div class="panel-footer" @click="setCurrentDate">今天</div>
        </div>
    </div>
</template>
<script>
import * as util from './util'
export default {
    directives: {
        clickOutside: {
            bind(el, bindings, vnode) {
                // 把事件绑定给document上，看一下点击的是否是当前这个元素内部
                let handler = (e) => {
                    console.log(e.target)
                    if (el.contains(e.target)) {
                        // 判断一下当前面板是否已经显示出来了
                        if (!vnode.context.isVisible) {
                            vnode.context.focus()
                        }
                    } else {
                        if (vnode.context.isVisible) {
                            vnode.context.blur()
                        }
                    }
                }
                el.handler = handler
                document.addEventListener('click', handler)
            },
            unbind(el) {
                document.removeEventListener('click', el.handler)
            }
        }
    },
    data () {
        let {year, month} = util.getYearMonthDay(this.value)
        return {
            isVisible: false,
            weekDays: ['日','一','二','三','四','五','六'],
            time: {year,month}
        }
    },
    props: {
        value: {
            type: Date,
            // 返回的默认值必须是一个函数类型
            default: () => new Date()
        }
    },
    methods: {
        focus() {
            this.isVisible = true
        },
        blur () {
            this.isVisible = false
        },
        getVisibleDay(i,j) {
            return this.visibleDays[(i-1)*7+(j-1)]
        },
        isCurrentMonth (date) {
            // let {year, month} = util.getYearMonthDay(this.value)
            let {year, month} = util.getYearMonthDay(util.getDate(this.time.year,this.time.month, 1))
            let {year: y, month: m} = util.getYearMonthDay(date)
            return year === y && month === m
        },
        isToday(date) {
            let {year, month, day} = util.getYearMonthDay(new Date())
            let {year: y, month: m, day: d} = util.getYearMonthDay(date)
            return year === y && month === m && day === d
        },
        chooseDate(date) {
            this.time = util.getYearMonthDay(date)
            this.$emit('input', date)
            this.blur()
        },
        isSelect(date) {
            let {year, month, day} = util.getYearMonthDay(this.value)
            let {year: y, month: m, day: d} = util.getYearMonthDay(date)
            return year === y && month === m && day === d
        },
        prevMonth() {
            let d = util.getDate(this.time.year,this.time.month, 1)
            d.setMonth(d.getMonth() - 1)
            this.time = util.getYearMonthDay(d)
        },
        prevYear(){
            let d = util.getDate(this.time.year,this.time.month, 1)
            d.setFullYear(d.getFullYear() - 1)
            this.time = util.getYearMonthDay(d)
        },
        nextMonth() {
            let d = util.getDate(this.time.year,this.time.month, 1)
            d.setMonth(d.getMonth() + 1)
            this.time = util.getYearMonthDay(d)
        },
        nextYear(){
            let d = util.getDate(this.time.year,this.time.month, 1)
            d.setFullYear(d.getFullYear() + 1)
            this.time = util.getYearMonthDay(d)
        },
        setCurrentDate() {
            let today = new Date()
            this.time = util.getYearMonthDay(today)
            this.$emit('input', today)
        },
        changeDate(dateStr) {
            console.log(dateStr)
            let [year,month,day] = dateStr.split('-')
            let date = util.getDate(year,month - 1, day)
            this.time = util.getYearMonthDay(date)
            this.$emit('input', date)
        }
    },
    computed: {
        formatDate(){
            let {year, month, day} = util.getYearMonthDay(this.value)
            return `${year}-${month + 1}-${day}`
        },
        visibleDays() {
            // 获取年月
            let {year, month} = util.getYearMonthDay(util.getDate(this.time.year,this.time.month, 1))
            // let {year, month} = util.getYearMonthDay(this.value)
            // 获取第一天
            let currentFirstDay=util.getDate(year,month,1)
            let week = currentFirstDay.getDay()
            // 当前开始的天数
            let oneDay = 24 * 60 *60 * 1000
            let startDay = currentFirstDay - week * oneDay 
            // 循环42天
            let arr = []
            for (let i=0; i< 42; i++) {
                arr.push(new Date(startDay + i*oneDay))
            }
            return arr
        }
        
    }
}
</script>
<style lang="stylus">
.panel
    width 32*7px;
    background white;
    position absolute;
    box-shadow 2px 2px 2px pink, -2px -2px -2px pink;
    border 1px solid pink;
    .panel-header
        display flex;
        justify-content space-around;
        height 30px;
        span
            cursor pointer;
            user-select none;
    .panel-body
        .cell
            display inline-flex;
            justify-content center;
            align-items center;
            width 32px;
            height 32px;
            font-weight bold;
            box-sizing border-box;
        .cell-day:hover,.select
            border 1px solid pink;
    .panel-footer
        height 30px;
        text-align center;
.notCurrentMonth
    color gray;
.today
    background red;
    color #fff;
    border-radius 4px;
</style>