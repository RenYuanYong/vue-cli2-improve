<!-- 状态权限 -dialog -->
<template>
  <div class="dialog-container">
      <el-dialog
          title="状态权限"
          width="450px"
          :visible.sync="visible"
          :modal-append-to-body="false"
          :close-on-click-modal="false"
          :before-close="close"
          :show="show">
          <div >
            <div >
              <div class="dis-flex warn-hint" style="justify-content: start;margin-bottom:20px;">
                <div class="icon-fuhao iconfont "></div>
                <div>选择禁用后所属该组下的账号都没有权限！</div>
              </div>
            </div>
            <el-form :model="formData" label-position="left" status-icon :rules="rules2" ref="formData" label-width="100px" class="demo-ruleForm">
              <el-form-item label="变更状态" prop="state">
                <el-radio-group  v-model="formData.state">
                  <el-radio :label="1">正常</el-radio>
                  <el-radio :label="2">禁用</el-radio>
                </el-radio-group>
              </el-form-item>
            </el-form>
            
          </div>
          <span slot="footer" class="dialog-footer">
            <el-button type="success" @click="submitForm('formData')">确认提交</el-button>
            <el-button  @click="close">取消</el-button>
          </span>
      </el-dialog>
  </div>
  
</template>

<script>
import * as App from '@/utils/index'
import { updateRoleState } from '@/api/set/power'
/*  */
export default {
  components: {
    
  },
  data () {
    return {
      visible: this.show,
      formData:{
        id: null, // 
        state: '', //
      },
      start_time: '',
      rules2: {
        state: [
          { required: true, message: '请选择变更状态', trigger: 'blur,change' }
        ],
      },
      
    };
  },
  props: {
    show: {
      type: Boolean,
      default: false
    },
    singleItem:{
      type: Object,
      default: null
    },
    batchItem:{
      type: Array,
      default: []
    },
    batchOper:{
      type: Boolean,
      default: false
    },
    callBack:{
      type: Function,
      default: null
    },
  },
  watch: {
    show () {
      this.visible = this.show;
      this.show&&this.init()
    }
  },
  computed: {
  },
  created () {
 
  },
  mounted () {
    this.$nextTick(function () {
      console.log('callBack', this.callBack)
      // Code that will run only after the
      // entire view has been rendered
    })
  },
  methods: {
    init(){
      this.formData.state = this.singleItem.status
    },
    /**
     * 获取当前操作数据
     */
    getNowOperData(){
      if(this.batchOper){
        // 批量操作
        this.batchItem.forEach(element => {
          this.formData.id += element.id + ','
        });
        this.formData.id = this.formData.id.substr(0, this.formData.id.length - 1);  
      }else{
        // 非批量操作
        this.formData.id = this.singleItem.id
      }
    },
    /**
     * 确定提交
     */
    confirmBtn(){
      this.getNowOperData()

      updateRoleState(this.formData).then(response => {
        this.$message.success(response.errmsg)
        this.callBack && this.callBack()
        this.close()        
      }).catch(error => {
        console.error('noPass-error:', error)
      })
    },
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.confirmBtn()
        } else {
          console.log('error submit!!');
          return false;
        }
      });
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
    /**
     * 关闭弹窗
     */
    close(){
      let formDataInit = {
        id: null, // 
        state: '', // 
        remark: '', // 
      }
      this.$emit('update:show', false)
      this.resetForm('formData')
      this.formData = formDataInit
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>

</style>
