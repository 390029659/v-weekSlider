<template>
    <div>
        <div class="year_str" v-if="showYear">{{yearMonthStr}}</div>
        <div class="week-slider">
            <div
                class="sliders"
                ref="sliders"
                @touchstart="touchstartHandle"
                @touchmove="touchmoveHandle"
                @touchend="touchendHandle">
                <template v-for="(item, index) in dates">
                    <div class="slider"
                        :style="getTransform(index)"
                        @webkit-transition-end="onTransitionEnd(index)"
                        @transitionend="onTransitionEnd(index)">
                        <div class="day" v-for="day in getDaies(item.date)">
                            <div
								@click.stop="dayClickHandle(day)"
								:style="buildDateStyle(day.isToday, day.isDay, day.isDisabled)">
                                {{day.week}}<br><strong>{{day.date.split('-')[2]}}</strong>
                            </div>
                        </div>
                    </div>
                </template>
            </div>
        </div>
    </div>
</template>

<script>
import moment from 'moment'
export default {
    name: 'weekSlider',
    props: {
        defaultDate: {
            type: String,
            default: moment().format('YYYY-MM-DD')
        },
        showYear: {
            type: Boolean,
            default: false
        },
        activeBgColor: {
            type: String,
            default: 'rgba(182, 30, 40, 1)'
        },
        todayBgColor: {
            type: String
        },
        disabledTxtColor: {
            type: String,
            default: 'rgba(221, 221, 221, 1)'
        },
        activeTxtColor: {
            type: String,
            default: 'rgba(255, 255, 255, 1)'
        },
        todayTxtColor: {
			type: String
		},
		lang: {
			type: String,
			default: 'ch'
        },
        startdate: {
            type: String
        },
        enddate: {
            type: String
        }
    },
    data () {
        return {
            dates: [],
            direction: null,
            activeIndex: 1,
            isAnimation: false,
            yearMonthStr: '',
            start: {
                x: null,
                y: null
            },
            end: {
                x: null,
                y: null
            },
            distan: {
                x: 0,
                y: 0
            },
            selecteddate: moment().format('YYYY-MM-DD'),
			sliderWidth: 0,
			weekLanguages: {
				ch: ['日', '一', '二', '三', '四', '五', '六'],
				en: ['Sun', 'Mon', 'Tue', 'Wed', 'Thur', 'Fri', 'Sat']
			}
        }
    },
    watch: {
        activeIndex(val){
            this.yearMonthStr = moment(this.dates[val].date).format('YYYY-MM')
        }
	},
	computed: {
		todayStyle: function () {
			let vm = this
			return {
				color: vm.todayTxtColor ? vm.todayTxtColor : ''
			}
		}
	},
    mounted () {
        this.sliderWidth = this.$refs.sliders.offsetWidth
    },
    created () {
        let vm = this
        vm.selecteddate = vm.defaultDate
        vm.yearMonthStr = moment(vm.defaultDate).format('YYYY-MM')
        vm.dates.push(
            {
                date: moment(vm.defaultDate).subtract(7, 'd').format('YYYY-MM-DD'),
            },
            {
                date: vm.defaultDate,
            },
            {
                date: moment(vm.defaultDate).add(7, 'd').format('YYYY-MM-DD'),
            }
        )
    },
    methods: {
        /**
        *获取制定日期的当前周的日期数据
        */
        getDaies (date) {
            let vm = this,
                arr = []
            let weekOfDate = Number(moment(date).format('E'))
            let weeks = vm.weekLanguages[vm.lang]
            let today = moment()
            let defaultDay = moment(vm.defaultDate)
            if (weekOfDate === 7) {
                weekOfDate = 0
            }
            for (var i = 0; i < 7; i++) {
                let _theDate = moment(date).subtract(weekOfDate - i, 'd')
                let _disabled = false;
                if(vm.startdate){
                    if(_theDate < moment(vm.startdate)){
                        _disabled = true;
                    }
                }
                if(vm.enddate){
                    if(_theDate > moment(vm.enddate)){
                        _disabled = true;
                    }
                }
                arr.push({
                    date: _theDate.format('YYYY-MM-DD'),
                    week: weeks[i],
                    isToday: _theDate.format('YYYY-MM-DD') === today.format('YYYY-MM-DD'),
                    isDay: _theDate.format('YYYY-MM-DD') === vm.selecteddate,
                    isDisabled: _disabled
                })
            }
            return arr
        },

        /**
        *根据索引计算出样式
        */
        getTransform (index) {
            let vm = this
            let style = {}
            if (index === vm.activeIndex) {
                style['transform'] = 'translateX('+ vm.distan.x +'px)'
            }
            if (index < vm.activeIndex) {
                style['transform'] = 'translateX(-100%)'
            }
            if (index > vm.activeIndex) {
                style['transform'] = 'translateX(100%)'
            }
            style['transition'] = vm.isAnimation ? 'transform .5s ease-out' : 'none'
            return style
        },



        /**
         * touchstart handle
         */
        touchstartHandle (event) {
            let vm = this,
                touch = event.touches[0]
            vm.start.x = touch.pageX
            vm.start.y = touch.pageY
        },

        /**
         * touchmove handle
         */
        touchmoveHandle (event) {
            let vm = this,
                touch = event.touches[0]
            vm.isAnimation = true
            vm.end.x = touch.pageX
            vm.end.y = touch.pageY
            vm.distan.x = vm.end.x - vm.start.x
            vm.distan.y = vm.end.y - vm.start.y
            let dom = vm.distan.x < 0 ? this.$refs.sliders.children[2] : this.$refs.sliders.children[0]
            if(vm.activeIndex === 0){
                dom = vm.distan.x < 0 ? this.$refs.sliders.children[1] : this.$refs.sliders.children[0]
            }
            else if(vm.activeIndex === 2){
                dom = vm.distan.x < 0 ? this.$refs.sliders.children[2] : this.$refs.sliders.children[1]
            }
            if (vm.distan.x < 0) {
                dom.style['transform'] = 'translateX('+ (vm.sliderWidth + vm.distan.x) +'px)'
            }else {
                dom.style['transform'] = 'translateX('+ (-vm.sliderWidth + vm.distan.x) +'px)'
            }
        },

        /**
         * touchend handle
         */
        touchendHandle (event) {
            let vm = this,
                touch = event.changedTouches[0]
            vm.isAnimation = true
            vm.end.x = touch.pageX
            vm.end.y = touch.pageY
            vm.distan.x = vm.end.x - vm.start.x
            vm.distan.y = vm.end.y - vm.start.y

            vm.getTouchDirection(vm.distan.x, vm.distan.y)
            if (vm.direction === 'left') {
                if(vm.activeIndex === 0){
                    vm.activeIndex = 1	
                }
                else{
                    vm.activeIndex = 2
                }
            } else if (vm.direction === 'right') {
                if(vm.activeIndex === 2){
                    vm.activeIndex = 1	
                }
                else{
                    vm.activeIndex = 0
                }
            } else {
                // for (var i = 0; i < this.$refs.sliders.children.length; i++) {
                //     this.$refs.sliders.children[i].style['transform'] = 'translateX('+ (i*100 - 100) +'%)'
                // }
            }
            vm.distan.x = 0
            vm.distan.y = 0
            vm.direction = null
        },

        onTransitionEnd (index) {
            let vm = this
            vm.isAnimation = false
            if (index === 1 && this.activeIndex === 2) {
                if(vm.enddate){
                    let enddate = moment(vm.dates[vm.activeIndex].date).add(7, 'd')
                    let weekOfDate = Number(moment(enddate).format('E'))
                    if (weekOfDate === 7) {
                        weekOfDate = 0
                    }
                    enddate = moment(enddate).subtract(weekOfDate, 'd')
                    if(enddate > moment(vm.enddate)){
                        return false;
                    }
                }
                vm.dates.push({
                    date: moment(vm.dates[vm.activeIndex].date).add(7, 'd').format('YYYY-MM-DD')
                })
                vm.dates.shift()
                vm.activeIndex = 1
            }else if (index === 1 && this.activeIndex === 0) {
                if(vm.startdate){
                    let startdate = moment(vm.dates[vm.activeIndex].date).subtract(7, 'd')
                    let weekOfDate = Number(moment(startdate).format('E'))
                    if (weekOfDate === 7) {
                        weekOfDate = 0
                    }
                    startdate = moment(startdate).subtract(weekOfDate - 6, 'd')
                    if(startdate < moment(vm.startdate)){
                        return false;
                    }
                }
                
                vm.dates.unshift({
                    date: moment(vm.dates[vm.activeIndex].date).subtract(7, 'd').format('YYYY-MM-DD')
                })
                vm.dates.pop()
                vm.activeIndex = 1
            }
        },

        /**
         * getAngle 计算角度
         */
        getAngle (x, y) {
            return Math.atan2(y, x) * 180 / Math.PI;
        },

        /**
         * getTouchDirection 获取滑动方向
         */
        getTouchDirection (x, y) {
            let vm = this
            if (Math.abs(x) > 20) {
                let angle = vm.getAngle(x, y)
                if (angle >= -45 && angle <= 45) {//向右
                    vm.direction = 'right'
                } else if ((angle >= 135 && angle <= 180) || (angle >= -180 && angle < -135)) {//向左
                    vm.direction = 'left'
                }
            }
        },
        dayClickHandle (day) {
            if(!day.isDisabled){
                day.isDay = true;
                this.selecteddate = day.date;
                this.$emit('dateClick', day.date)
                this.$emit('update:defaultDate', day.date)
            }
        },

		/**
		 *生成日期样式
		 */
		buildDateStyle (isToday, isActive, isDisabled) {
			let vm = this
			let res = {}

			if (isToday) {
				res.color = vm.todayTxtColor || ''
				res.backgroundColor = vm.todayBgColor || ''
			}

			if (isActive) {
				res.color = vm.activeTxtColor || ''
				res.backgroundColor = vm.activeBgColor || ''
            }
            
            if(isDisabled){
                res.color = vm.disabledTxtColor || ''
				res.backgroundColor = ''
            }
			return res
		}
    }
}
</script>
<style scoped>
	.year_str {
		height: 36px;
		border-bottom: #ddd solid 1px;
		line-height: 36px;
		text-align: center;
	}
	.week-slider {
		width: 100%;
		height: 48px;
		overflow: hidden;
		padding: 10px 0;
	}
	.week-slider .sliders {
		position: relative;
	}
	.week-slider .sliders .slider {
		height: 48px;
		width: 100%;
		display: flex;
		position: absolute;
		top: 0;
		left: 0;
		overflow: hidden;
	}
	.week-slider .sliders .slider .day {
		flex: 1;
	}
	.week-slider .sliders .slider .day div {
		height: 36px;
		width: 48px;
		padding: 6px 0;
		margin: auto;
		text-align: center;
		line-height: 18px;
		font-size: 12px;
		border-radius: 50%;
	}
	.week-slider .sliders .slider .day div.sameDay {
		background-color: #999;
		color: #FFF;
	}
	.week-slider .sliders .slider .day div strong {
		font-size: 14px;
	}

</style>

