<!-- 审核不通过- -->
<template>
  <div style="height:100%;">
    <div id="areaContHead">
      <div class="cont-title">认证审核</div>
      <!--tab 审核不通过 -->
      <div class="tab-box">
        <ul>
          <li v-for="(item, index) in sidebar.thrLev" :key="index" :label="item.name" 
            @click="goPage(item)"
            :class="{'active':item.path==$route.path}">
            {{item.name}}
          </li>
        </ul>
      </div>
      <!-- 搜索 -->
      <el-row class="sizer searchBox">
        <el-col class="mgr search-date">
          <span class="demonstration"></span>
          <el-date-picker :editable="false" v-model="timeSlot" value-format="yyyy-MM-dd" type="daterange" range-separator="至" start-placeholder="开始日期" end-placeholder="结束日期" size="medium ">
          </el-date-picker>
        </el-col>
        <el-col :span="2" class="mgr">
          <el-select v-model="formData.type" size="medium " placeholder="选择类型">
            <el-option v-for="item in typeOption" 
            :key="item.type" 
            :label="item.label" 
            :value="item.type"></el-option>
          </el-select>
        </el-col>
        <el-col :span="3" class="mgr">
          <el-input
            placeholder="请输入内容"
            size="medium "
            v-model="formData.keyword">
            <i slot="suffix" class="el-input__icon el-icon-search cur-pot" @click="searchHandle"></i>
          </el-input>
        </el-col>
      </el-row>
    </div>
    
    <div>
      <!-- 列表 -->
      <el-table :data="dataList"  size="medium" stripe style="width: 100%" :height="calcTableHeight">
        <!-- <el-table-column
          type="selection"
          width="55">
        </el-table-column> -->
        <el-table-column prop="check_time" label="审核时间" ></el-table-column> 
        <el-table-column prop="submit_time" label="提交时间" ></el-table-column>
        <el-table-column prop="company_name" label="企业名称" ></el-table-column>
        <el-table-column prop="taxpayer" label="纳税识别号" ></el-table-column>
        <el-table-column prop="city" label="所在城市" ></el-table-column>
        <el-table-column prop="operation_name" label="运营者姓名" ></el-table-column>
        <el-table-column prop="operation_phone" label="运营者手机号码" ></el-table-column>
        <el-table-column label="营业执照">
          <template slot-scope="scope" >
            <i class="iconfont icon-test16" @click="open('busLisens', null, scope.row)"></i>
          </template>
        </el-table-column>
        <el-table-column prop="remark" label="审核备注" ></el-table-column>
      </el-table>
      <!-- cont 底部 -->
      <div id="areaContFooter" class="bot-cont" v-if="dataList.length">

        <div class="paging">
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="formData.page"
            :page-sizes="sidebar.pageSizeArr"
            :page-size="formData.pageSize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="totalCount">
          </el-pagination>
        </div>
      </div>
    </div>
    <bus-lisens 
      :show.sync="dialogShow.busLisens"
      :single-item='selectSingleItem'
      :call-back='fetchList'>
    </bus-lisens>
  </div>
  
</template>

<script>
import { mapGetters } from 'vuex'
import * as App from '@/utils/index'
import { getList } from '@/api/customer/certiaudit'
import busLisens from '@/dialog/customer/certiaudit/waitaudit/busLisens'

/*  */
export default {
  components: {
    busLisens,
  },
  data () {
    return {
      formData:{
        is_pass: 0,// 审核是否通过(0不通过 1通过 3待审核)
        page: 1,// 当前所在页
        pageSize: 10,// 每页大小
        start_time: null,// 开始时间
        end_time: null,// 结束时间
        type: null,// 1企业名称 2纳税识别号
        keyword: null,// 具体内容
      },
      dialogShow:{  //弹窗显示flag
        busLisens:false    
      },
      selectSingleItem: null,// table选中一项操作
      // 下拉框选项
      typeOption:[
        {
          type: '',
          label: '选择类型'
        }, {
          type: '1',
          label: '企业名称'
        }, {
          type: '2',
          label: '纳税识别号'
        }
      ],
      timeSlot:[],  //时间段
      totalCount:0, //总条数
      dataList: [],
      areaHeadH:0,
      areaFootH:0
    };
  },
  watch: {
    timeSlot: function (val, oldVal) {
      console.log('new:', val, oldVal)
      if (val){
        this.formData.start_time = val[0]
        this.formData.end_time = val[1]
      }
    },
  },
  computed: {
    // 获取state
    ...mapGetters([
      'sidebar',
    ]),
    // 计算table列表高度
    calcTableHeight () {
      let tableHeight = (this.sidebar.nowWinH-this.areaHeadH-this.areaFootH-this.sidebar.layoutH)
      return tableHeight
    }
  },
  
  created () {
    this.init(this.$route.path)
    
  },
  mounted () {
    this.$nextTick(function () {
      // 获取到内容区域头部和底部的高度--为了计算table的高度
      let areaContHead = document.getElementById('areaContHead')
      let areaContFooter = document.getElementById('areaContFooter')
      this.areaHeadH = areaContHead && areaContHead.offsetHeight
      this.areaFootH = areaContFooter && areaContFooter.offsetHeight
    })
  },
  methods: {
    init(){
      console.log('mapGetters-navbar', this.sidebar)
      const that = this
      this.fetchList()
    },

    goPage(item) {
      this.$router.push({path: item.path})
    },
    /** 
     *  获取列表数据
     */
    fetchList(){
      // this.formData
      getList(this.formData).then(response => {
        let _data = response
        console.log('getLoginCode:', _data)
        this.dataList = _data.list
        this.totalCount = _data.totalCount
      }).catch(error => {
        console.error('fetchList-error:', error)
      })
    },
    /** 
     *  搜索
     */
    searchHandle(){
      if (this.formData.keyword && !this.formData.type) {
        this.$message.warning('请选择类型')
        return
      }
      // 初始化参数
      this.formData.page = 1
      this.fetchList()
    },
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`)
      this.formData.pageSize = val
      this.formData.page = 1 //清空页数
      this.fetchList()
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`)
      this.formData.page = val
      this.fetchList()
    },
    /**
     * 打开弹窗
     * @augments par 弹窗名称
     * @augments batch 是否是批量操作
     */
    open (par, batch, item) {
      batch && (this.batchOper = batch) 
      item && (this.selectSingleItem = item)
      if (par){
        this.dialogShow[par] = true        
      }else{
        this.$message.error('打开弹出错误！');
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>

</style>
