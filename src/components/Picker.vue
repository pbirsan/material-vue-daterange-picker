<template>
    <div class="mdrp-root" v-clickoutside="clickOutside">
        <div class="mdrp__activator" @click="togglePicker">
            <slot name="input">
                <default-activator ref="defaultActivator" :isPickerActive="pickerVisible" :value="buildActivatorValue()" readonly />
            </slot>
            <span v-show="showClearIcon" class="clear" @click="clearSelection($event);">
                <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24">
                    <path d="M24 20.188l-8.315-8.209 8.2-8.282-3.697-3.697-8.212 8.318-8.31-8.203-3.666 3.666 8.321 8.24-8.206 8.313 3.666 3.666 8.237-8.318 8.285 8.203z"
                          fill="currentColor" />
                </svg>
            </span>
        </div>
        <transition name="slide-fade" mode="out-in">
            <div class="mdrp__panel dropdown-menu" :class="pickerStyles()" v-show="pickerVisible">
                <calendar-ranges v-if="showPresets && presets" :canSelect="inRange" :presets="presets"
                                 @clickCancel="pickerVisible = false" @clickApply="clickApply" @clickPreset="clickPreset">
                </calendar-ranges>
                <calendar class="calendar left" location="left" :calendar-month="leftCalendarMonth_" :locale="locale" :start="start_"
                          :end="end_" :hover-start="hoverStart_" :hover-end="hoverEnd_" @clickNextMonth="clickNextMonth"
                          @clickPrevMonth="clickPrevMonth" @dateClick="dateClick" @hoverDate="hoverDate" @clickYearSelect="clickYearSelect">
                </calendar>
                <calendar class="calendar right" location="right" :calendar-month="rightCalendarMonth_" :locale="locale"
                          :start="start_" :end="end_" :hover-start="hoverStart_" :hover-end="hoverEnd_"
                          @clickNextMonth="clickNextMonth" @clickPrevMonth="clickPrevMonth" @dateClick="dateClick"
                          @hoverDate="hoverDate" @clickYearSelect="clickYearSelect">
                </calendar>
            </div>
        </transition>
    </div>
</template>

<script>
    import moment from 'moment';
    import Calendar from './Calendar.vue';
    import CalendarRanges from './Ranges.vue';
    import DefaultActivator from './widgets/activator.vue';

    import clickoutside from '../directives/clickoutside';

    import { defaultPresets } from '../constant/index.js';

    export default {
        name: 'v-md-date-range-picker',
        components: { Calendar, CalendarRanges, DefaultActivator },
        directives: { clickoutside },
        provide() {
            return {
                'picker': this,
            };
        },
        props: {
            // The beginning date of the initially selected date range.
            // If you provide a string, it must match the date format string set in your locale setting
            startDate: {
                type: Object | String,
            },
            // The end date of the initially selected date range.
            endDate: {
                type: Object | String,
            },
            // set custom date formatter
            dateFormat: {
                type: String
            },
            // Set predefined date ranges the user can select from.
            // The range of each object an array with two dates representing the bounds of the range.
            presets: {
                type: Array,
                default() {
                    return defaultPresets;
                },
            },
            // Whether the picker appears aligned to the left, to the right, or centered under the HTML element it's attached to.
            opens: {
                type: String,
                default: 'left',
            },
            // Displays "Custom Range" at the end of the list of predefined ranges, when the ranges option is used.
            // This option will be highlighted whenever the current date range selection does not match one of the predefined ranges.
            // Clicking it will display the calendars to select a new range.
            showCustomRangeLabel: {
                type: Boolean,
                default: false,
            },
            // Show year select boxes above calendars to jump to a specific year.
            showYearSelect: {
                type: Boolean,
                default: false,
            },
            // The minimum year shown in the dropdowns when showYearSelect is set to true.
            minYear: {
                type: String,
                default: moment().subtract(100, 'year').format('YYYY'),
            },
            // The maximum year shown in the dropdowns when showYearSelect is set to true.
            maxYear: {
                type: String,
                default: moment().add(100, 'year').format('YYYY'),
            },
            // Hide the apply and cancel buttons, and automatically apply a new date range as soon as two dates are clicked.
            autoApply: {
                type: Boolean,
                default: true,
            },
            // show label for the default activator (inputbox)
            showActivatorLabel: {
                type: Boolean,
                default: true,
            },
            // show animation bar for the default activator (inputbox)
            showActivatorBar: {
                type: Boolean,
                default: true,
            },
            showPresets: {
                type: Boolean,
                default: true,
            },
        },
        data() {
            const data = {
                locale: {
                    direction: 'ltr',
                    format: moment.localeData().longDateFormat('L'),
                    separator: ' - ',
                    applyLabel: 'Apply',
                    cancelLabel: 'Cancel',
                    weekLabel: 'W',
                    customRangeLabel: 'Custom Range',
                    daysOfWeek: moment.weekdaysMin(),
                    monthNames: moment.monthsShort(),
                    firstDay: 1, //moment.localeData().firstDayOfWeek(),
                    isDirty: false
                },
            };
            // TODO 这里的 props 究竟是放在 data 里面进行初始化好，还是放在生命周期中好呢？
            // https://github.com/ly525/blog/issues/252
            // https://github.com/ly525/blog/issues/258
            data.leftCalendarMonth_ = this.startDate ? moment(this.startDate) : moment();
            data.rightCalendarMonth_ = this.endDate ? moment(this.endDate) : moment();
            data.start_ = this.startDate ? moment(this.startDate) : moment();
            data.end_ = this.endDate ? moment(this.endDate) : moment();
            data.hoverStart_ = this.startDate ? moment(this.startDate) : moment();
            data.hoverEnd_ = this.endDate ? moment(this.endDate) : moment();
            // fix #14
            data.cloneStart = this.startDate ? moment(this.startDate) : moment();
            data.cloneEnd = this.endDate ? moment(this.endDate) : moment();

            data.startText = this.startDate ? (this.dateFormat ? moment(this.startDate).format(this.dateFormat) : moment(this.startDate).format(data.locale.format)) : "";
            data.endText = this.endDate ? (this.dateFormat ? moment(this.endDate).format(this.dateFormat) : moment(this.endDate).format(data.locale.format)) : "";
            data.inRange = false; // inRange means whether user click once, if user click once, set value true
            data.pickerVisible = false;
            data.renderComponent = true;

            data.showClearIcon = this.startDate ? true : false;

            // update day names order to firstDay
            if (data.locale.firstDay !== 0) {
                let iterator = data.locale.firstDay;
                while (iterator > 0) {
                    data.locale.daysOfWeek.push(data.locale.daysOfWeek.shift());
                    iterator--;
                }
            }

            return data;
        },
        methods: {
            clickYearSelect({ location, calendarMonth }) {
                this[`${location}CalendarMonth_`] = calendarMonth.clone();
            },
            clickNextMonth() {
                // TODO 如果有 linkedCalendars，需要更新代码
                // moment.js 的 add 和 sub tract 的改变自身的行为没有被 watch 到，原因是什么呢？
                this.leftCalendarMonth_ = this.leftCalendarMonth_.clone().add(1, 'month');
            },
            clickPrevMonth() {
                // TODO 如果有 linkedCalendars，需要更新代码
                this.leftCalendarMonth_ = this.leftCalendarMonth_.clone().subtract(1, 'month');
            },
            buildActivatorValue() {
                if (this.startText && this.endText) {

                    if (this.startText != this.endText) {
                        return `${this.startText} - ${this.endText}`;
                    }
                    return this.startText;
                }

                return "";
            },
            /**
             * TODO type of value
             */
            dateClick(value) {
                if (this.inRange) { // second click
                    // second click action(第二次点击)
                    this.inRange = false;
                    // if second click value is smaller than first, which means user clicked a previous date,
                    // so set the smaller date as start date, bigger one as end date
                    if (value.isBefore(this.start_)) {
                        this.end_ = this.start_;
                        this.start_ = value.clone();
                    } else {
                        this.start_ = this.start_;
                        this.end_ = value.clone();
                    }

                    //Update the picker only if the user did anything
                    this.isDirty = true;

                    // feature #49
                    if (this.autoApply) {
                        this.clickApply();
                    }
                } else { // first click
                    // first click action, set value as start and end(第一次点击, 设置起始值皆为点击的值)
                    this.inRange = true;
                    this.start_ = value.clone();
                    this.end_ = value.clone();
                    // Notice: If you watch start_, its callback function will be executed after end_ is assigned, which is exactly what we want.
                    // You can add a loop to test here
                    // In fact, the callback function is actually updateMonthCalendar, which is to update the date based on the values of start and end.
                    // So if the callback is callback after both the start_ and end_, that's right!
                    // updateMonthCalendar() === callback function for watch start_
                }
            },
            hoverDate(value) {
                if (this.inRange) {
                    if (value > this.start_) {
                        // 参见：https://github.com/ly525/blog/issues/254
                        this.hoverStart_ = this.start_.clone();
                        this.hoverEnd_ = value.clone();
                    } else {
                        this.hoverEnd_ = this.start_.clone();
                        this.hoverStart_ = value.clone();
                    }
                }
            },
            togglePicker() {
                // ---- fix #53 start ----
                let elm = this.$refs.defaultActivator && this.$refs.defaultActivator.$el;
                // fix #55: this.$slots.input[0] -> this.$slots.input[0].elm
                const slotActivator = this.$slots.input && this.$slots.input.length && this.$slots.input[0].elm;
                if (!elm && (slotActivator.querySelector('input') || slotActivator.querySelector('button'))) {
                    elm = slotActivator;
                }

                if (elm) {
                    // 1. dont return or do nothing here,
                    // because you need to show the picker panel if the picker panel is hidden(example: user click the activator first time)
                    // but `this.pickerVisible = !this.pickerVisible;` do the samething in this case.
                    // So why set pickerVisible always `true` if elm exist?
                    // 2. [interact]: because if the type of activator is input or button and the picker panel is already visible (pickerVisible === true),
                    // when the user click the activator, the picker panel should keep visible(can not fold the picker panel)

                    // Chinese：
                    // 1. 不能在这里 return 或 啥都不做
                    // 因为如果日期选择器是隐藏的，点击了 input 需要显示日期选择器。比如用户第一次点击日期选择器的时候
                    // 但是在这种情况下做，下面的 `this.pickerVisible = !this.pickerVisible;` 做了一样的事
                    // 那么为何需要在 elm 为 true 的时候，总是设置 pickerVisible 为 true 呢？
                    // 2. [交互] 我们约定，当 activator 的类型是 input 或 button，以及 选择器面板已经 打开的情况下，
                    // 当用户点击了 activator 的时候，不收起日期选择器面板

                    this.pickerVisible = true;
                } else {
                    this.pickerVisible = !this.pickerVisible;
                }
                // ---- fix #53 end ----
            },
            pickerStyles() {
                return {
                    'show-calendar': this.pickerVisible,
                    'opens-arrow-pos-right': this.opens === 'right',
                    'opens-arrow-pos-left': this.opens === 'left',
                    'opens-arrow-pos-center': this.opens === 'center',
                };
            },
            closeRangePicker() {
                if (this.pickerVisible)
                    this.pickerVisible = false;
            },
            clearSelection(event) {
                event.stopPropagation();

                if (this.pickerVisible)
                    this.pickerVisible = false;

                this.cloneStart = undefined;
                this.cloneEnd = undefined;

                this.updateTextField();
                this.emitChange();

                this.isDirty = false;
            },
            clickApply() {
                this.pickerVisible = false;

                // fix #14
                // if the use only click the picker only one time,
                // then close the picker directly(by clickoutside or click the activator)
                if (this.inRange) {
                    this.inRange = false;

                    if (this.cloneStart && this.cloneEnd) {
                        this.start_ = this.cloneStart.clone();
                        this.end_ = this.cloneEnd.clone();
                    }
                    else {
                        return;
                    }
                }

                if (this.isDirty) {
                    this.updateTextField();
                    this.cloneForCancelUsage();
                    this.emitChange();
                }
            },
            clickPreset(preset) {
                if (preset.label === this.locale.customRangeLabel) return;
                const [start, end] = preset.range;
                this.start_ = moment(start);
                this.end_ = moment(end);
                // fix #47
                this.leftCalendarMonth_ = moment(start);
                // TODO 需要想一下，联动情况下，快捷日期，选择范围如果超过两个月，该如何显示？
                // TODO if linkedCalendar, what should the UI show if end - start > 60 days?

                //dirty only when the user touched
                this.isDirty = true;

                // feature #49
                if (this.autoApply) {
                    this.clickApply();
                }
            },
            /**
             *
            */
            updateTextField() {
                // do not update the input slot provided content by the parent
                if (this.$slots.input) return;

                if (this.cloneStart && this.cloneEnd) {
                    this.startText = this.dateFormat ? this.start_.format(this.dateFormat) : this.start_.format(this.locale.format);
                    this.endText = this.dateFormat ? this.end_.format(this.dateFormat) : this.end_.format(this.locale.format);
                }
                else {
                    this.startText = "";
                    this.endText = "";
                }
            },
            /**
             * fix #14
             * clone start and end for the following scenes, mainly for reseting the selected date to origin state:
             * 1. (autoApply: false) click start [or both start and end], but click the cancel button
             * 2. (autoApply: true) just click one time, and then click outside
             *
             * TODO (need discussion) maybe we can do this action in watch pickerVisible (from hidden to visible)
             * DONE we also need to do clone start and end in the watcher of ther related prop
             *
             */
            cloneForCancelUsage() {
                if (this.start_ && this.end_) {
                    this.cloneStart = this.start_.clone();
                    this.cloneEnd = this.end_.clone();
                }
            },
            emitChange() {
                // TODO if developer do not set the event listener for @change or @input, we may need change the startText and endText in the component,
                // TODO support v-model
                if (this.cloneStart && this.cloneEnd) {
                    const start = this.start_.clone();
                    const end = this.end_.clone();
                    this.$emit('change', [start, end], [start.format('YYYY-MM-DD'), end.format('YYYY-MM-DD')]);
                    this.showClearIcon = true;
                }
                else {
                    this.$emit('change', [undefined, undefined], [undefined, undefined]);
                    this.showClearIcon = false;
                }
            },
            clickOutside() {
                if (!this.pickerVisible) return;
                this.clickApply();
            }
        },
        watch: {
            /**
             * 有两个地方：
             * 1. 点击左侧快捷键(clickPreset)，确认 start_
             * 2.
             *
             * 如果使用 计算属性，则 clickPrevMonth 和 clickNextMonth 的时候，需要设置计算属性的 setter，但这时候 setter 就不知道写什么了
             * TODO 值变化的时候，什么时候执行 watch 呢？ nextTick 吗？
             */
            start_(value) {
                if (value) {
                    this.hoverStart_ = value.clone();
                    // inspired by https://github.com/dangrossman/daterangepicker/blob/master/daterangepicker.js#L554
                    // fix #43
                    if (typeof this.end_.month == "function") {
                        if (value.month() === this.end_.month()) return;
                        this.leftCalendarMonth_ = value.clone();
                    }
                }

            },
            end_(value) {
                if (value) {
                    this.hoverEnd_ = value.clone();
                }
            },
            leftCalendarMonth_: {
                handler(leftMonth) {
                    this.rightCalendarMonth_ = leftMonth.clone().add(1, 'month');
                },
                immediate: true,
            },
            startDate(value) {
                if (value) {
                    this.start_ = moment(value);
                    this.startText = this.dateFormat ? moment(value).format(this.dateFormat) : moment(value).format(this.locale.format);
                    this.cloneStart = moment(value); // fix #14
                } else {
                    this.start_ = moment();
                    this.startText = "";
                    this.cloneStart = undefined;
                }
            },
            endDate(value) {
                if (value) {
                    this.end_ = moment(value);
                    this.endText = this.dateFormat ? moment(value).format(this.dateFormat) : moment(value).format(this.locale.format);
                    this.cloneEnd = moment(value); // fix #14
                    // TODO not linked calendar
                } else {
                    this.end_ = moment();
                    this.endText = "";
                    this.cloneEnd = undefined;
                }
            }
        },
    };
</script>
<style lang="scss" scoped>
    @import '../styles/components/picker.scss';
</style>

<style scoped>
    .clear {
        position: absolute;
        right: -22px;
        top: 10px;
        cursor: pointer;
        color: #2f3941;
    }

        .clear:hover {
            color: black;
        }
</style>
