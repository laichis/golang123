<template>
    <Row type="flex" align="middle" justify="center" class="golang-signin-container">
        <Col :xs="24" :lg="6" :md="14">
            <a class="golang-signin-title" href="/">GOLANG123</a>
            <p class="golang-signin-desc">和地鼠们分享你的知识、经验和见解</p>
        	<Form ref="formCustom" :model="formCustom" :rules="ruleCustom" class="signup-form">
        		<Form-item prop="username">
                    <i-input
                        size="large"
                        v-model="formCustom.username"
                        @on-blur="blur('formCustom.username')"
                        placeholder="用户名 / 邮箱"></i-input>
                </Form-item>
                <Form-item prop="passwd">
                    <i-input size="large" type="password" v-model="formCustom.passwd" placeholder="密码" @keydown.native="handleKeyUp"></i-input>
                </Form-item>
                <p style="text-align: right;padding-right: 2px">
                    <a href="/signup" class="golang-common-link" style="margin-right: 12px;">立即注册</a>
                    <a href="/ac/pwdReset" class="golang-common-link">忘记密码</a>
                </p>
                <Form-item style="margin-top: 10px">
                    <i-button size="large" type="primary" @click="handleSubmit('formCustom')" style="width: 100%">登&nbsp;&nbsp;录</i-button>
                </Form-item>
            </Form>
        </Col>
        <script type="text/javascript" color="51,133,255" opacity='0.7' zIndex="1" count="80" src="/javascripts/canvasnest/canvas-nest.min.js"></script>
    </Row>
</template>

<script>
    import ErrorCode from '~/constant/ErrorCode'
    import request from '~/net/request'
    import url from 'url'
    import {trim, trimBlur} from '~/utils/tool'
    import '~/utils/bd'

    export default {
        data () {
            return {
                loading: false,
                formCustom: {
                    passwd: '',
                    username: ''
                },
                success: false,
                ruleCustom: {
                    passwd: [
                        { required: true, message: '请输入密码', trigger: 'blur' }
                    ],
                    username: [
                        { required: true, message: '请输入用户名', trigger: 'blur' }
                    ]
                }
            }
        },
        asyncData (context) {
            let user = context.user
            let redirectURL

            let myURL = url.parse(context.req.url, true)
            if (myURL.query && myURL.query.ref) {
                redirectURL = decodeURIComponent(myURL.query.ref)
                let redirectObj = url.parse(redirectURL, true)
                let pathname = redirectObj.pathname
                // 由重置密码或激活账号跳过来的，登录后直接跳到首页
                if (pathname.match(/\/reset\/.+/) || pathname.match(/\/active\/.+/)) {
                    redirectURL = '/'
                }
            } else {
                redirectURL = '/'
            }
            if (user) {
                context.redirect(redirectURL)
                return
            }
            return {
                user: user,
                ref: myURL.query.ref,
                redirectURL: redirectURL
            }
        },
        layout: 'signin',
        middleware: 'userInfo',
        head () {
            return {
                title: '登录'
            }
        },
        methods: {
            handleSubmit (name) {
                this.$refs[name].validate((valid) => {
                    if (valid) {
                        if (this.loading) {
                            return
                        }
                        this.loading = true
                        request.signin({
                            body: {
                                signinInput: trim(this.formCustom.username),
                                password: trim(this.formCustom.passwd)
                            }
                        }).then(res => {
                            this.loading = false
                            if (res.errNo === ErrorCode.SUCCESS) {
                                window.location.href = this.redirectURL
                            } else if (res.errNo === ErrorCode.IN_ACTIVE) {
                                window.location.href = '/verify/mail?e=' + encodeURIComponent(res.data.email)
                            } else {
                                this.$Message.error(res.msg)
                            }
                        }).catch(err => {
                            this.loading = false
                            this.$Message.error(err.message)
                        })
                    }
                })
            },
            handleKeyUp (e) {
                if (e.keyCode === 13) {
                    return this.handleSubmit('formCustom')
                }
            },
            blur (name) {
                trimBlur(name, this)
            }
        }
    }
</script>

<style>
    @import '../assets/styles/signin.css'
</style>
