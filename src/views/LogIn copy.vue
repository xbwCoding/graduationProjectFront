<template>
    <div>
        <div id="particles"></div>
        <div class="login-wrap">
            <h1 class="t-c">体育馆场地预约</h1>
            <el-form
                label-position="left"
                :model="ruleForm"
                :rules="rules"
                ref="ruleForm"
                label-width="100px"
                class="demo-ruleForm"
            >
                <el-form-item label="账户" prop="userName">
                    <el-input v-model="ruleForm.userName"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="pwd">
                    <el-input v-model="ruleForm.pwd" type="password"></el-input>
                </el-form-item>
                <el-form-item label="类型" prop="loginType">
                    <el-radio-group v-model="ruleForm.loginType">
                        <el-radio label="1">提供者</el-radio>
                        <el-radio label="0">用户</el-radio>
                    </el-radio-group>
                </el-form-item>
                
                <el-row v-if="ruleForm.loginType=='1'">
                    <el-col :span="10">
                        <el-form-item label="支付宝收款码">
                            <el-upload
                                :on-remove="removezhifubao"
                                :limit="1"
                                class="avatar-uploader"
                                :action="BaseUrl+'provider/activityUpload'"
                                :on-success="handleZFBSuccess"
                                :before-upload="beforeZFBUpload">
                                <img v-if="zhifubaoImage" :src="zhifubaoImage" class="avatar">
                                <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                            </el-upload>
                        </el-form-item>
                    </el-col>
                    <el-col :span="10">
                        <el-form-item label="微信收款码">
                            <el-upload
                                :on-remove="removeWx"
                                :limit="1"
                                class="avatar-uploader"
                                :action="BaseUrl+'provider/activityUpload'"
                                :on-success="handleWXSuccess"
                                :before-upload="beforeWXUpload">
                                <img v-if="wxImage"  :src="wxImage" class="avatar">
                                <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                            </el-upload>
                        </el-form-item>
                    </el-col>
                </el-row>

                <el-form-item>
                    <el-button type="primary" @click="login('ruleForm')"
                        >登录</el-button
                    >
                    <el-button type="primary" @click="register('ruleForm')"
                        >注册</el-button
                    >
                    <el-button @click="resetForm('ruleForm')">重置</el-button>
                </el-form-item>
            </el-form>
        </div>
    </div>
</template>

<script lang="ts">
import particlesJS from 'particles.js'
console.log("particlesJS",particlesJS)

import { registerHttp, loginHttp } from "@/axios/api";
import { ElForm } from "node_modules/element-ui/types/form";
import { Component, Vue, Ref } from "vue-property-decorator";
import { BaseUrl} from "@/axios/api";

interface loginInterface {
    userName: string;
    pwd: string;
    loginType: string;
    zhifubaoImage:string;
    wxImage:string
}

@Component
export default class LogIn extends Vue {
    BaseUrl = BaseUrl
    zhifubaoImage = ""
    wxImage = ""
    ruleForm: loginInterface = {
        userName: "",
        pwd: "",
        loginType: "",
        zhifubaoImage:"",
        wxImage:""
    };
    rules = {
        userName: [
            { required: true, message: "请输入用户名", trigger: "blur" },
            
        ],
        pwd: [{ required: true, message: "请输入用户名", trigger: "blur" },
            {
                min: 6,
                message: "密码至少六位",
                trigger: "blur",
            },],
        loginType: [
            {
                type: "string",
                required: true,
                message: "请至少选择一种身份",
                trigger: "change",
            },
        ],
    };

    beforeZFBUpload(){

    }

    beforeWXUpload(){

    }

    handleZFBSuccess(res:any, file:any) {
        this.zhifubaoImage = URL.createObjectURL(file.raw)
        console.log("res  res 上传成功", res)
        this.ruleForm.zhifubaoImage= res.data
    }

    handleWXSuccess(res:any, file:any) {
        this.wxImage = URL.createObjectURL(file.raw);
        this.ruleForm.wxImage= res.data
    }

    removezhifubao(){
        this.zhifubaoImage=""
    }

    removeWx(){
        this.wxImage=""
    }

    @Ref("ruleForm") readonly FormRef!: ElForm;
    login() {
        this.FormRef.validate((valid: boolean) => {
            if (valid) {
                console.log("this.ruleForm", this.ruleForm);
                loginHttp(this.ruleForm)
                    .then((res: any) => {
                        localStorage.loginId = res.data.loginId;
                        localStorage.loginType = this.ruleForm.loginType;
                        console.log("登录成功 res", res);
                        if (res.success !== 0 &&this.ruleForm.loginType == "1"){
                            this.$router.push({name: 'AddStadium'});
                        }

                        if (res.success !== 0 &&this.ruleForm.loginType == "0") {
                            this.$router.push({name: 'Subscribe'});
                        }
                    })
                    .catch((e: any) => {
                        console.log("登录失败");
                        throw new Error("登录失败");
                    });
            } else {
                console.log("error submit!!");
                return false;
            }
        });
    }

    register() {
        this.FormRef.validate((valid: boolean) => {
            if (valid) {
                console.log("this.ruleForm", this.ruleForm);
                registerHttp(this.ruleForm)
                    .then((res: any) => {
                        if (res.success !== 0 &&this.ruleForm.loginType == "1") {
                            localStorage.loginId = res.data.loginId 
                            this.$router.push("/provider");
                        }

                        if (res.success !== 0 &&this.ruleForm.loginType == "0") {
                            localStorage.loginId = res.data.loginId 
                            this.$router.push("/user");
                        }
                    })
                    .catch((e: any) => {
                        console.log("登录失败");
                        throw new Error("登录失败");
                    });
            } else {
                console.log("error submit!!");
                return false;
            }
        });
    }

    resetForm() {
        this.FormRef.resetFields();
    }

    mounted(){
        particlesJS('particles', {
        'particles': {
            'number': {
            'value': 10, 'density': {
                'enable': true, 'value_area': 800
            }
            }, 'color': {
            'value': '#000000'
            }, 'shape': {
            'type': 'circle', 'stroke': {
                'width': 0, 'color': '#000000'
            }, 'polygon': {
                'nb_sides': 5
            }, 'image': {
                'src': 'img/github.svg', 'width': 100, 'height': 100
            }
            }, 'opacity': {
            'value': 0.1, 'random': false, 'anim': {
                'enable': false, 'speed': 1, 'opacity_min': 0.1, 'sync': false
            }
            }, 'size': {
            'value': 20, 'random': true, 'anim': {
                'enable': false, 'speed': 20, 'size_min': 0.1, 'sync': false
            }
            }, 'line_linked': {
            'enable': true, 'distance': 1000, 'color': '#000000', 'opacity': 0.2, 'width': 1
            }, 'move': {
            'enable': true,
            'speed': 4,
            'direction': 'none',
            'random': false,
            'straight': false,
            'out_mode': 'out',
            'bounce': false,
            'attract': {
                'enable': false, 'rotateX': 600, 'rotateY': 1200
            }
            }
        }, 'interactivity': {
            'detect_on': 'canvas', 'events': {
            'onhover': {
                'enable': false, 'mode': 'grab'
            }, 'onclick': {
                'enable': false, 'mode': 'push'
            }, 'resize': true
            }, 'modes': {
            'grab': {
                'distance': 140, 'line_linked': {
                'opacity': 1
                }
            }, 'bubble': {
                'distance': 400, 'size': 40, 'duration': 2, 'opacity': 8, 'speed': 3
            }, 'repulse': {
                'distance': 200, 'duration': 0.4
            }, 'push': {
                'particles_nb': 4
            }, 'remove': {
                'particles_nb': 2
            }
            }
        }, 'retina_detect': true
        })
    }
}
</script>

<style scoped>
#particles {
        position: absolute;
        width: 100%;
        height: 100%;
    }

.login-wrap {
    margin: auto;
    margin-top: 100px;
    width: 30%;
    background-color:#fff;
}

  .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }

  .t-c{
      text-align: center;
  }
</style>

