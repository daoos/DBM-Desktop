<template>
    <div class="filter-header">
        <Row>
            <Col span="20">
            <input class="filter-header-input" @blur="createFilter" v-number-only :disabled="isDisabled"
                   v-if="column.dataType == 'decimal'" @keyup="enter($event)" v-model="inputNum" v-focus/>
            <DatePicker
                    class="filter-header-input"
                    v-else-if="column.dataType == 'datetime'"
                    v-model="value"
                    :type="dateType"
                    transfer
                    confirm
                    :disabled="isDisabled"
                    @on-clear="clearFilter"
                    @on-ok="createFilter"></DatePicker>
            <input class="filter-header-input" v-else :disabled="isDisabled"
                   v-focus @keyup="enter($event)" @blur="createFilter" v-model="value"/>
            </Col>
            <Col span="4">
            <div class="filter-icon">
                <Dropdown trigger="click" transfer placement="left-start" @on-click="selectFilterType">
                    <a href="javascript:void(0)">
                        <Icon type="android-bar" size="20" color="rgba(43, 55, 52, 0.58)"></Icon>
                    </a>
                    <DropdownMenu slot="list">
                        <DropdownItem v-for="let (item,index) in filterType" :name="item.name" :selected="selectType==item.name">{{item.title}}</DropdownItem>
                    </DropdownMenu>
                </Dropdown>
            </div>
            </Col>
        </Row>
    </div>
</template>

<script>
    let num = 0;
    import dbmUtl from '../../../../../libs/dbmUtil';
    export default {
        name: '',
        components: {},
        data() {
            return {
                selectType:'',
                value:'',
                filterOpt:{}
            }
        },
        props: {
            column: {
                type: Object,
                default: function () {
                    return {
                        dataType: 'varchar'
                    }
                }
            },
            clearFilterFlag: {
                type: Boolean,
                default:false
            },
            dataFilterRules:{
                type: Object,
                default: function () {
                    return {}
                }
            }
        },
        computed: {
            filterType(){
                for(let key in this.dataFilterRules)
                    if(key == this.column.dataType)
                        return this.dataFilterRules[key];
                return this.dataFilterRules['varchar'];
            },
            inputNum:{
                get:function(){
                    return this.value;

                },
                set:function(newValue){
                    if(isNaN(newValue)&&newValue.length == 1)
                        newValue = num;
                    if(newValue.length == 0)
                        newValue = '';

                    this.value=newValue.replace(/[^\d.]/g,"");
                    num = this.value;
                }
            },
            isDisabled(){
                return this.selectType == 'IS NULL'||this.selectType == 'IS NOT NULL';
            },
            dateType(){
                let type = 'datetime';
                if(this.selectType == 'BETWEEN')
                    type =  'datetimerange';
                return type;
            }
        },
        directives: {
            focus: {
                inserted: function (el) {
                    el.focus()
                }
            },
            numberOnly: {
                bind: function(el) {
                    el.handler = function() {
                        var val = el.value;
                        el.value = val.replace(/[^\d.]/g,"");
                    }
                    el.addEventListener('input', el.handler)
                },
                unbind: function(el) {
                    el.removeEventListener('input', el.handler)
                }
            }
        },
        methods: {
            selectFilterType(name){
                this.selectType = name;
                this.createFilter();
            },
            enter(ev){
                if(ev.keyCode == 13)
                    this.createFilter();
            },
            createFilter(){
                if(this.value == ''&&!this.isDisabled){
                    this.$emit('on-filter-delete', this.column.field);
                    return;
                }
                if(this.isDisabled)
                    this.value = ''
                let filterOpt = {
                    type:this.selectType,
                    value:this.value
                }
                if(JSON.stringify(this.filterOpt)==JSON.stringify(filterOpt))
                    return;
                this.filterOpt = JSON.parse(JSON.stringify(filterOpt));
                filterOpt.columnName = this.column.field;
                this.$emit('on-filter-change', filterOpt);
            },
            clearFilter(){
                this.value = '';
                this.$emit('on-filter-delete', this.column.field);
            }
        },
        watch: {
            clearFilterFlag(val){
                this.clearFilter();
            }
        },
        created(){
            if(this.filterType.length > 0)
                this.selectType = this.filterType[0].name;
        }
    }
</script>
<style>
    .filter-header {
        background-color: #eeecec;
        width: 100%;
        display: inline-block;
        padding: 0 3px;
        vertical-align: middle;
        word-break: break-all;
        overflow: hidden;
        line-height: 1.2em;
    }
    .filter-header-input{
        text-align: center;
        padding-left:17%;
        left: 30px;
        width:100%;
        height: 32px;
        outline:none;
        margin: 3px;
        font-size: 18px;
        color: rgba(16, 11, 19, 0.93);
        border:1px solid rgba(221, 222, 225, 0.78);
    }
    .filter-icon{
        height: 100%;
        width: 100%;
        padding:8px 0;
    }
</style>