<template>
        <div class="count-bg">
            <el-row class="sizeden">
                <el-col :span="2">
                    <el-dropdown trigger="click" @command="handleCommand">
                            <el-button size="medium">
                                选择操作<i class="el-icon-arrow-down el-icon--right"></i>
                            </el-button>
                            <el-dropdown-menu slot="dropdown">
                                <el-dropdown-item :command="recently_day"
                                 >最近{{recently_day}}天</el-dropdown-item>
                            </el-dropdown-menu>
                        </el-dropdown>
                    </el-col>
                <el-col :span="6">
                    <div class="block">
                        <span class="demonstration"></span>
                        <el-date-picker :editable="false"  value-format="yyyy-MM-dd" v-model="timeSlot" type="daterange" range-separator="至" start-placeholder="开始日期" end-placeholder="结束日期" size="medium">
                        </el-date-picker>
                    </div>
                </el-col>
            </el-row>
            <el-row>
                <div class="newcount">
                    <div class="title-comon">
                        趋势图
                    </div>
                    <el-row class="newcount borline">
                        <el-col :span="8" class="county">
                            <p>平均金额（笔）</p>
                            <span>{{countList.avg_money}}</span>
                            
                        </el-col>
                        <el-col :span="8" class="county">
                            <p>最近30天交易笔数</p>
                            <span>{{countList.deal_count}}</span>
                            
                        </el-col>
                        <el-col :span="8" class="county">
                            <p>最近30天交易笔数（元）</p>
                            <span>{{countList.moneys}}</span>
                            
                        </el-col>
                    </el-row>
                    <el-row>
                            <el-col><div id="proLinea"></div></el-col>
                    </el-row>
                    
                </div>
            </el-row>  
        </div>
       
            
    
</template>
<style lang="scss" scoped>
    #proLinea{
      width: 100%;
      height: 405px;
      margin:0 auto;
    }
    
    .count-bg{background-color: $bg-default;margin-bottom: 20px;}
    .sizeden{padding:5px;}
    .newcount{
        background-color: $bg-fff;
        border: 1px solid $bg-default;
        padding: 20px;
        .borline{border:none;}
    }
         .county{
        border-right: 1px solid $border-default;
        text-align: center;
        p{
            font-size: 14px;
            color: $font-default-color;
            margin-bottom: 14px;
        }
        p+span{
            font-size: 24px;
            font-weight: bold;
            color:  $color-info;
        }
        li{
            line-height: 18px;
        }
    }
    .county:last-child{
        border-right: 0px;
    }
</style>

<script>
    import { Row, Col, Dropdown, DropdownMenu, DropdownItem } from 'element-ui';
    // 引入 ECharts 主模块
    const echarts = require('echarts/lib/echarts');
    // 引入提示框和标题组件
    require('echarts/lib/component/tooltip');
    require('echarts/lib/component/title');
    //  引入折线图
    require('echarts/lib/chart/line');
    require('echarts/lib/chart/lines');
    import { mapGetters } from 'vuex'
    import * as App from '@/utils/index'
    import { areaAnaly } from '@/api/database/monAna'


    export default {
        
        data () {
            return {
                prolines:'',
                recently_day:'30',
                formData:{
                    start_time: null,// 开始时间
                    end_time: null,// 结束时间
                    recently_day:null,
                },
                timeSlot: [],  //时间段
                countList:[],//新增客户
                countListOne:[],//累计客户数量
                countListTwo:[],//消费客户
                countListTh:[],//创建应用客户
            }
        },
        created () {
            this.init()
        },
        computed:{
            // 获取state
            ...mapGetters([
            'sidebar',
            ]),
            
            /**
             * 获取下拉操作项
             */
            dropOperArr(){
                let dropOper = []
                console.log('dropOperArr')
            if (this.sidebar.thrLev ){
                this.sidebar.thrLev.forEach(ele => {
                if(ele.path === this.$route.path){
                    dropOper = ele.children
                }
                })
            }
                return dropOper
            }
        },
        methods: {
             /**
             * 初始化
             */
            init(){
            //console.log( this.$router,this.$route)
                this.areaAnalyData()
            },
            handleCommand(command) {
                 console.log('现在点击的是',command)
                 this.formData.recently_day=command
                 this.areaAnalyData()
                 
            },
            // goPage(item) {
            //     this.$router.push({path: item.path})
            // },
               /**
     * 昨日关键数据
     */
    // yesterdayData(){
    //     yesterdayTarget(this.formData).then(response => {
    //         let _data = response
    //         this.countList = _data.data[0]
    //         this.countListOne = _data.data[1]
    //         this.countListTwo = _data.data[2]
    //         this.countListTh = _data.data[3]
    //         console.log('数字获取',this.countList)
    //     }).catch(error => {
    //         console.error('yesterdayTarget啊啊啊啊-error:', error)
    //     })
    // },
          
            /**
             * 趋势图
             */
            areaAnalyData(){
                areaAnaly(this.formData).then(response => {
                    const _data = response
                    this.formData=_data.data.fin
                    this.countList = _data.data.avg[0]
                    this.countListOne = _data.data.avg[1]
                    this.countListTwo = _data.data.avg[2]
                    this.countListTh = _data.data.avg[3]
                    this.countList.avg_money = this.countList.avg_money.toFixed(2)
                    this.countList.deal_count = this.countList.deal_count.toFixed(2)
                    this.countList.moneys = this.countList.moneys.toFixed(2)
                    //日期
                    const dateMetas = this.formData.map(ele=>ele.datelist)
                    //交易笔数折线经过点
                    const deal_count = this.formData.map(ele=>ele.deal_count)
                   
                   
                    //var   gettype=Object.prototype.toString
                    console.log('啊啊啊啊啊啊啊啊啊啊啊啊啊啊aaaaaaaaaaaa啊啊',this.countList.avg_money)                  
                     
                this.prolines = echarts.init(document.getElementById('proLinea'));
                this.prolines.setOption({
                    tooltip: {
                        trigger: 'axis'
                    },
                    legend: {
                        data:['交易笔数'],
                        bottom: '1%',
                    },
                    grid: {
                        left: '3%',
                        right: '4%',
                        bottom: '20%',
                        containLabel: true
                    },
                    // 是否需要下载图片按钮
                    // toolbox: {
                    //     feature: {
                    //         saveAsImage: {}
                    //     }
                    // },
                    xAxis: {
                        type: 'category',
                        boundaryGap: false,
                        data: dateMetas
                    },
                    yAxis: {
                        type: 'value'
                    },
                    series: [                     
                        {
                            name:'交易笔数',
                            type:'line',
                            stack: '总量',
                            data:deal_count,
                            color:['#d1ebc9']
                        }
                    ]
                })
   
                }).catch(error => {
                    console.error('areaAnaly-error:', error)
                })
            },
          

        },
        watch:{
            timeSlot: function (val, oldVal) {
                console.log('new-趋势图:', val, oldVal)
                if (val){
                    this.formData.start_time = val[0]
                    this.formData.end_time = val[1]
                     this.areaAnalyData()
                }
            },
        },
        mounted() {
            this.$nextTick(function () {
                this.areaAnalyData();
            })
        }
    }
</script>

