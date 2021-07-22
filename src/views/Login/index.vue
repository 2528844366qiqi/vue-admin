<template>
    <div id="login">
        <div class="login-wrap">
            <ul class="menu-tab">
                <li :class="{'current':item.current}" v-for="item in menuTab" :key="item.id" @click="toggleMneu(item)">{{item.txt}}</li>
            </ul>

            <!-- 表单 -->
            <el-form :model="ruleForm" status-icon :rules="rules" ref="ruleForm" class="login-form" size="medium">
                <el-form-item prop="username" class="item-from">
                    <label for="username">邮箱</label>
                    <el-input id="username" type="text" v-model="ruleForm.username" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item prop="password" class="item-from">
                    <label for="">密码</label>
                    <el-input type="password" v-model="ruleForm.password" autocomplete="off" minlength="6" maxlength="20"></el-input>
                </el-form-item>
                <el-form-item prop="passwords" class="item-from" v-show="model === 'register'">
                    <label for="">重置密码</label>
                    <el-input type="password" v-model="ruleForm.passwords" autocomplete="off" minlength="6" maxlength="20"></el-input>
                </el-form-item>
                <el-form-item prop="code" class="item-from">
                    <label for="">验证码</label>
                    <el-row :gutter="10">
                        <el-col :span="15">
                            <el-input v-model.number="ruleForm.code" minlength="6" maxlength="6"></el-input>
                        </el-col>
                        <el-col :span="9">
                            <el-button type="success" class="block" @click="getSms()">获取验证码</el-button>
                        </el-col>
                    </el-row>
                </el-form-item>
                <el-form-item>
                    <el-button type="danger" @click="submitForm('ruleForm')" class="login-btn block" :disabled="loginButtonStatus">{{model === 'login' ? '登录' : '注册'}}</el-button>
                </el-form-item>
            </el-form>
        </div>
    </div>
</template>
<script>
import {GetSms} from '@/api/login'
import {onMounted, reactive, ref} from '@vue/composition-api';
import { stripscript } from '@/utils/validate.js';
import axios from 'axios';
export default {
    name:"login",
    setup(props, {refs, root}){
        // 验证用户名s
      let validateUsername = (rule, value, callback) => {
        let reg = /^([a-zA-Z]|[0-9])(\w|\-)+@[a-zA-Z0-9]+\.([a-zA-Z]{2,4})$/;
        if (value === '') {
          callback(new Error('请输入用户名'));
        } else if(!reg.test(value)){
          callback(new Error('用户名格式有误'));
        }else{
          callback();
        }
      };
      // 验证密码
      let validatePassword = (rule, value, callback) => {
        let reg = /^(?!\D+$)(?![^a-zA-Z]+$)\S{6,20}$/
        if (value === '') {
          callback(new Error('请输入密码'));
        } else if (!reg.test(value)) {
          callback(new Error('密码位6至20加字母'));
        } else {
          callback();
        }
      };
      // 验证重置密码
      let validatePasswords = (rule, value, callback) => {
        if(model.value == 'login'){
            callback();
        }
        let reg = /^(?!\D+$)(?![^a-zA-Z]+$)\S{6,20}$/
        if (value === '') {
          callback(new Error('请再次输入密码'));
        } else if (value != ruleForm.password) {
          callback(new Error('两次密码不正确'));
        } else {
          callback();
        }
      };
      // 验证验证码
      let validateCode = (rule, value, callback) => {
        let reg = /^[a-z0-9]{6}$/
        if (value === '') {
          return callback(new Error('请输入验证码'));
        }else if(!reg.test(value)){
          return callback(new Error('验证码不正确'));
        }else{
          callback();
        }
      };
        //这里面放置data数据、生命周期、自定义的函数
        const menuTab = reactive([
                {txt:'登录', current:true, type:'login'},
                {txt:'注册', current:false, type:'register'}
            ])
          
          const loginButtonStatus = ref(false)
           const ruleForm = reactive({
                username: '',
                password: '',
                passwords: '',
                code: ''
            })

            const rules = reactive({
                username: [
                    { validator: validateUsername, trigger: 'blur' }
                ],
                password: [
                    { validator: validatePassword, trigger: 'blur' }
                ],
                passwords:[
                    { validator: validatePasswords, trigger: 'blur' }
                ],
                code: [
                    { validator: validateCode, trigger: 'blur' }
                ]
            })
        // 声明函数
        const toggleMneu = (data => {
            console.log(data)
            menuTab.forEach((elem,index) => {
                elem.current = false
            });
            data.current = true
            model.value = data.type
        })
        // 获取验证码
        const getSms = (() => {
          // 进行提示
          if(ruleForm.username == ''){
            root.$message.error('邮箱不能为空！！');
            return false
          }

          GetSms({username:ruleForm.username, module:'login'}).then(response => {
            console.log(response)
          }).catch(error => {
            console.log(error)
          })
        })

        const model = ref('login')
        const submitForm = (formName => {
            refs[formName].validate((valid) => {
          if (valid) {
            alert('submit!');
          } else {
            console.log('error submit!!');
            return false;
          }
            })
        })

        // 生命周期挂载完成
        onMounted(() => {
          // GetSms()
        })
        return {
            menuTab,
            model,
            toggleMneu,
            submitForm,
            getSms,
            ruleForm,
            rules,
            loginButtonStatus
        }
    },
    created(){},
    mounted(){},
    // 写函数的地方
    methods:{
    //   resetForm(formName) {
    //     this.$refs[formName].resetFields();
    //   }
    }
}
</script>
<style lang="scss" scoped>
    #login{
        height: 100vh;
        background-color: #344a5f;
    }
    .login-wrap{
        width: 330px;
        margin: auto;
    }
    .menu-tab{
        text-align: center;
        li{
            display: inline-block;
            width: 88px;
            line-height: 36px;
            font-size: 14px;
            color: #fff;
            border-radius: 2px;
            cursor: pointer;
        }
        .current{
            background-color: rgba(0,0,0, .1);
        }
    }
    .login-form{
        margin-top: 29px;
        label{
            display: block;
            margin-bottom: 3px;
            font-size: 14px;
            color: #fff;
        }
        .item-from{
            margin-bottom: 13px;
        }
        .block{
            display: block;
            width: 100%;
        }
        .login-btn{
            margin-top: 19px;
        }
    }
</style>