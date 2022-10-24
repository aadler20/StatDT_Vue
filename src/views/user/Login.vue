<template>
  <div class="main">
    <a-form
      class="user-layout-login"
      :form="form"
      @submit="handleSubmit"
    >
      <a-alert v-if="state.isLoginError" type="error" showIcon style="margin-bottom: 24px;" message="Please check your login info." />
      <a-form-item>
        <a-input
          size="large"
          type="text"
          :placeholder="i18nRender('user.login.username.placeholder')"
          v-decorator="customRoles.username"
        >
          <a-icon slot="prefix" type="user" :style="{ color: 'rgba(0,0,0,.25)' }"/>
        </a-input>
      </a-form-item>

      <a-form-item>
        <a-input-password
          size="large"
          :placeholder="i18nRender('user.login.password.placeholder')"
          v-decorator="customRoles.password"
        >
          <a-icon slot="prefix" type="lock" :style="{ color: 'rgba(0,0,0,.25)' }"/>
        </a-input-password>
      </a-form-item>

      <a-form-item style="margin-top:24px">
        <a-button
          size="large"
          type="primary"
          htmlType="submit"
          class="login-button"
          :loading="state.disabled"
          :disabled="state.disabled"
        >{{ i18nRender('user.login.login') }}</a-button>
        <!--<router-link class="forgetPassword" :to="{ name: 'forgetPassword' }">{{ i18nRender('user.login.forgot-password') }}</router-link>-->
        <router-link class="register" :to="{ name: 'register' }">{{ i18nRender('user.login.signup') }}</router-link>
      </a-form-item>
    </a-form>
  </div>
</template>

<script>
import { mapActions } from 'vuex'
import { timeFix } from '@/utils/util'
import { i18nRender } from '@/locales'

export default {
  data () {
    return {
      // 表单对象
      form: this.$form.createForm(this),
      // 状态
      state: {
        isLoginError: false,
        disabled: false
      },
      // 表单项规则
      customRoles: {
        username: [
          'username',
          {
            rules: [
              { required: true, message: '请输入邮箱地址' },
              { validator: this.handleUsernameOrEmail }
            ],
            validateTrigger: 'change'
          }
        ],
        password: [
          'password',
          {
            rules: [
              { required: true, message: '请输入密码' }
            ],
            validateTrigger: 'blur'
          }
        ]
      }
    }
  },
  methods: {
    i18nRender,
    ...mapActions(['Login', 'Logout']),
    // handler
    handleUsernameOrEmail (rule, value, callback) {
      const regex = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+((\.[a-zA-Z0-9_-]{2,3}){1,2})$/
      regex.test(value)
      callback()
    },
    handleSubmit (e) {
      e.preventDefault()
      const {
        form: { validateFields },
        state,
        Login
      } = this
      state.disabled = true
      validateFields({ force: true }, (err, values) => {
        if (!err) {
          const loginParams = { ...values }
          console.info('loginParams', loginParams)
          Login(loginParams)
            .then((res) => this.loginSuccess(res))
            .catch(err => this.requestFailed(err))
            .finally(() => {
              state.disabled = false
            })
        } else {
          setTimeout(() => {
            state.disabled = false
          }, 600)
        }
      })
    },
    /**
     * 登陆成功
     * 1、跳转首页
     * 2、显示欢迎语
     * @param res
     */
    loginSuccess (res) {
      this.state.isLoginError = false
      this.$router.push({ path: '/' })
      // 延迟 1 秒显示欢迎信息
      setTimeout(() => {
        this.$notification.success({
          message: '欢迎',
          description: `${timeFix()}，欢迎回来`
        })
      }, 1000)
    },
    requestFailed (err) {
      this.state.isLoginError = true
      this.$notification['error']({
        message: '错误',
        description: ((err.response || {}).data || {}).message || '请求出现错误，请稍后再试',
        duration: 4
      })
    }
  }
}
</script>

<style lang="less" scoped>
.user-layout-login {
  label {
    font-size: 14px;
  }

  .getCaptcha {
    display: block;
    width: 100%;
    height: 40px;
  }

  .forge-password {
    font-size: 14px;
  }

  button.login-button {
    padding: 0 15px;
    font-size: 16px;
    height: 40px;
    width: 100%;
  }

  .user-login-other {
    text-align: left;
    margin-top: 24px;
    line-height: 22px;

    .item-icon {
      font-size: 24px;
      color: rgba(0, 0, 0, 0.2);
      margin-left: 16px;
      vertical-align: middle;
      cursor: pointer;
      transition: color 0.3s;

      &:hover {
        color: #1890ff;
      }
    }

    .register {
      float: right;
    }
  }
}
</style>
