<template>
   <el-dialog
      :visible.sync="verifyDialogVisible_"
      width="30%"
      :close-on-click-modal="false"
      center>
      <el-form size="medium" :model="vertifyForm" :rules="vertifyRules" ref="vertifyForm">
        <!-- <el-form-item prop="phone_num">
          <el-select style="width:100%" v-model="vertifyForm.phone_num" filterable clearable placeholder="请选择手机号码">
            <template slot="prefix">
              <span class="iconPrend"><i class="rx-icon-mobile-phone"></i></span>
            </template>
            <el-option
              v-for="item in adminOptions"
              :key="item.phone"
              :label="item.phone"
              :value="item.phone">
               <span style="float: left">{{ item.userName }}</span>
               <span style="float: right; color: #8492a6; font-size: 13px">{{ item.phone }}</span>
            </el-option>
          </el-select>
        </el-form-item> -->
        <el-form-item prop="code_num">
          <el-input v-model="vertifyForm.code_num" placeholder="请输入验证码" clearable>
            <template slot="prepend"><i class="rx-icon-chat"></i></template>
            <template slot="append">
              <el-link
                type="primary"
                :underline="false"
                :disabled="forbid_login_code_btn"
                @click="sendCode">{{ vertifyForm.code_msg }}</el-link>
            </template>
          </el-input>
        </el-form-item>
      </el-form>
      <span slot="title" class="title">
        {{title === 'DJSZ' ? '定价设置验证开启-获取验证码' : '信用设置验证开启-获取验证码'}}
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="handleCancel">取 消</el-button>
        <el-button type="primary" @click="handleVertify">确 定</el-button>
      </span>
    </el-dialog>
</template>

<script>
import * as API_Login from '@/api/login'
import { RegExp } from '~/ui-utils'
export default {
  name: 'VerifyPhonePop',
  props: {
    verifyDialogVisible: {
      type: Boolean,
      default: false
    },
    title: {
      type: String,
      default: '定价设置验证开启-验证码获取'
    },
    unitId: {
      type: Number,
      default:0
    }
  },
  mounted() {
    this.getAdminInfo() //获取管理员手机
  },
  created() {
     this.getAdminInfo() //获取管理员手机
  },
  computed: {
    verifyDialogVisible_: {
       get() {
         return this.verifyDialogVisible
       },
       set(v) {
         this.$emit('changeVerifyDialog',v)
       }
    },
    forbid_login_code_btn() {
      return this.vertifyForm.sending_code;
    },
  },
  data() {
    return{
      adminOptions:[],
      vertifyForm:{
        phone_num:'',
        code_num: '',
        code_msg: '获取验证码',
        sending_code: false, // 正在发送验证码
      },
      vertifyRules: {
        // phone_num: [this.MixinRequired('手机号码不能为空！'),
        //   { validator: (rule, value, callback) => {
        //     if (!RegExp.mobile.test(value)) {
        //       callback(new Error('手机号码格式不正确！'))
        //     } else {
        //       callback()
        //     }
        //   } }
        // ],
      },
    }
  },
  methods: {
    /* 获取所有后台管理员 */
    getAdminInfo() {
      API_Login.getAdminInfo().then(res=>{
        if(res.code === 200) {
          this.adminOptions = res.data
        } else {
          this.$message.error(res.msg || '出错啦，请联系后台管理员')
        }
      })
    },
    /* 发送验证码 */
    sendCode() {
      this.$refs['vertifyForm'].validate((valid, error) => {
        if(valid) {
          const params = {
            phoneNum: this.vertifyForm.phone_num,
            type: this.title
          }
          API_Login.sendMSG(params).then(response => {
            if (response.code && response.code === 200) {
              this.$set(this.vertifyForm, 'sending_code', true)
              this.countDownTime(60)
            } else {
              this.$message.error(response.msg || '系统错误，请联系客服')
            }
          }).catch((e)=>{
            console.log(e)
          })
        } else {
          const key = Object.keys(error)[0]
          this.$message.error(error[key][0].message)
        }
      })
    },
    /* 手机号 验证码 验证 */
    handleVertify() {
      this.$refs['vertifyForm'].validate((valid,error) => {
        if(valid) {
          const params = {
            code: this.vertifyForm.code_num,
            phoneNum: this.vertifyForm.phone_num,
            type: this.title,
            unitId: this.unitId
          }
          API_Login.checkCode(params).then(res=>{
            if(res.code===200) {
              this.$message.success('验证成功')
              this.verifyDialogVisible_ = false
            } else {
              this.$message.error(res.msg || '系统错误，请联系客服')
            }
          })
        }
      })
    },
    /* 获取验证码倒计时 */
    countDownTime(seconds) {
      const that = this;
      seconds -= 1;
      if (seconds < 0 || !that.vertifyForm.sending_code) {
        that.$set(that.vertifyForm, 'sending_code', false)
        that.$set(that.vertifyForm, 'code_msg', '获取验证码')
        return false;
      }
      that.$set(that.vertifyForm, 'code_msg', `发送成功(${seconds})`)
      setTimeout(() => {
        that.countDownTime(seconds);
      }, 1000);
    },

    /* 取消 */
    handleCancel() {
      this.verifyDialogVisible_ = false
    }
  }
}
</script>

<style lang="scss" scoped>
.iconPrend{
  width: 71px;
  height: 30px;
  display: inline-block;
  background-color: 245, 247, 250;
  padding-top: 2px;
}
.rx-icon-mobile-phone{
  display: inline-block;
  width: 30px;
  height: 30px;
  background: url(../../assets/rx-icon-mobile-phone.png) no-repeat;
  background-size: 100% 100%;
  background-position: center center;
}
.rx-icon-chat{
  display: inline-block;
  width: 30px;
  height: 30px;
  background: url(../../assets/rx-icon-chat.png) no-repeat;
  background-size: 100% 100%;
  background-position: center center;
}

.title {
  font-size: 20px;
  font-weight: 500;
}

/deep/ .el-input--prefix .el-input__inner {
  padding-left: 87px;
  // padding-right: 160px;
  // border: 0.5px solid #dcdfe6;
}

/deep/ .el-input__prefix {
  left: 0;
  border: 0.5px solid #dcdfe6;
  background-color: #f5f7fa;
  border-radius: 5px 0px 0px 5px;
}
</style>