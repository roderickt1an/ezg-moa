<template>
    <van-row style="overflow-x: hidden">
        <van-row>
            <div style="height: 150px;margin-top: 7rem">
                <center>
                    <img style="height: 80px" src="./logo.png"/>
                </center>
            </div>
            <div style="width:80%;margin:auto">
                <van-cell-group>
                        <van-field
                            v-model="username"
                            label="用户名"
                            placeholder="请输入用户名"
                        />
                        <van-field
                            v-model="password"
                            type="password"
                            label="密码"
                            placeholder="请输入密码"
                        />
                </van-cell-group>
            </div>
            <van-row style="width:80%;margin:auto;margin-top:60px">
                <van-button size="large" type="primary" @click="login">登 陆</van-button>
            </van-row>
        </van-row>
    </van-row>
</template>

<script>
import Cookies from 'js-cookie';

export default {
    data(){
        return{
            username:"",
            password:""
        }
    },
    methods:{
        login(){
            let _self = this
            let url = `api/user/login`
            let config = {
                username: _self.username,
                password: _self.password
            }
            this.$http.post(url, config).then(function(res){
                if(res.data.msgCode == "40000"){
                    Cookies.set('user', _self.username);
                    Cookies.set('password', _self.password);
                    localStorage.setItem('realname', res.data.data.user.realname)
                    localStorage.setItem('id', res.data.data.user.id)
                    //  获取权限菜单
                    _self.get_menu(res.data.data.user.id)
                    // _self.getRole(localStorage.getItem("id"))
                    _self.$router.push({
                       name: 'index'
                    })

                }else{
                    _self.$toast.fail(res.data.msg)
                }
            }).catch(function(err){
                _self.$toast.fail("网络异常！")
            })
        },
        // 获取当前用户角色
        getRole(e){
            let _self = this
            this.$http.get('/api/user/checkUserRoleByUserId?userId='+ e).then(function(res){
                if(res.data.msgCode == "40000"){
                    let temp = []
                    for(let i = 0;i<res.data.data.length;i++){
                        temp.push(res.data.data[i].rolecode)
                    }
                    let str = JSON.stringify(temp)
                    localStorage.setItem('role',str)
                }else{
                    _self.$toast.fail("系统错误！")
                }
            }).catch(function(err){
                _self.$toast.fail("网络异常！")
            })
        },

        autologin(code){
            let _self = this
            let url = `api/legwork/apiLoginByWechatCode`
            let formdata = new FormData()
            formdata.append("agentId","1000013")
            formdata.append("code",code)

            this.$http.post(url,formdata).then(function(res){
                if(res.data.msgCode == 40000){
                    localStorage.setItem('realname', res.data.data.user.realname)
                    localStorage.setItem('id', res.data.data.user.id)
                    //  获取权限菜单
                    _self.get_menu(res.data.data.user.id)
                    // _self.getRole(localStorage.getItem("id"))
                    _self.$router.push({
                       name: 'index'
                    })
                }else{
                    _self.$toast.fail("免登陆失败！请登陆！")
                }
            }).catch(function(err){
                _self.$toast.fail("免登陆失效，请登录！")
            })
        },

        loading(){
            let _self = this
            let str = location.href
            let temp = str.split("?")
            let temp2 = str.split("&")
            let params = temp2[0].split("=")
            console.log(params)
            if(params.length == "1"){
                _self.$toast.fail("免登陆失效，请登录！")
            }else{
                console.log(params[1])
                this.autologin(params[1])
            }
        },
        get_menu(e){
            let _self = this
            let url = `api/menu/getInterfaceItemByUserId`
            let config = {
                params: {
                    userId: e
                }
            }

            function success(res){
                // console.log(res.data.data.interfaces)
                Cookies.set('access', (res.data.data.interfaces).join());
                localStorage.setItem("access_array",JSON.stringify(res.data.data.interfaces))
            }

            this.$Get(url, config, success)
        }
    },
    created(){
        // this.autologin()
        this.loading()
    }
}
</script>
