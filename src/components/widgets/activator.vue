<template>
    <div class="activator-wrapper">
        <input v-bind="$attrs" class="text-field dateRangeActivatorInput" :class="inputClass" type="text" :value="value">
        <span v-if="picker.showActivatorBar" class="bar"></span>
        <label v-if="picker.showActivatorLabel">Select Date</label>
        <div class="activatorSelectBox" @click="onSelectBoxClicked($event)"></div>
    </div>
</template>

<script>
    export default {
        inheritAttrs: false,
        inject: ['picker'],
        props: ['value', 'isPickerActive'],
        data() {
            return {
                selectBoxElement: "",
                datePickerPanel: ""
            }
        },
        watch: {
            isPickerActive: function (isPickerActive) {
                if (isPickerActive) {
                    this.selectBoxElement.classList.add("active");
                }
                else {
                    this.selectBoxElement.classList.remove("active");
                }
            }
        },
        computed: {
            inputClass() {
                return {
                    'text-field__filled': this.value.trim().length > 0,
                };
            },
        },
        methods: {
            onSelectBoxClicked(event) {
                if (this.isPickerActive) {
                    event.stopPropagation();
                    this.$parent.closeRangePicker();
                }
            }
        },
        mounted() {
            this.selectBoxElement = document.querySelector(".activatorSelectBox");
        }
    };
</script>


<style lang="scss">
    @import '../../styles/components/activator.scss';
</style>

<style>

    .activator-wrapper {
        min-height: 38px;
        display: flex;
        flex-grow: 1;
    }

        .activator-wrapper .text-field.dateRangeActivatorInput {
            display: block;
            padding: 9px 16px;
            width: 100%;
            border: 1px solid #C2C8CC;
            box-shadow: none;
            background-color: white;
            color: #2f3941;
            outline: none;
            font-size: 13px;
            letter-spacing: 0.2px;
        }

    .activatorSelectBox {
        position: absolute;
        width: 26px;
        height: 26px;
        right: 7px;
        top: 5px;
        text-align: center;
        transition: transform .2s ease;
        cursor: pointer;
    }

        .activatorSelectBox.active {
            transform: rotate(180deg);
        }

        .activatorSelectBox:before {
            position: relative;
            right: 0;
            top: 65%;
            color: #999;
            margin-top: 4px;
            border-color: #999 transparent transparent;
            border-style: solid;
            border-width: 5px 5px 0;
            content: "";
        }
</style>
