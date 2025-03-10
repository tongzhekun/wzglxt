<script setup lang="tsx">
import { reactive, ref, watch, onMounted, unref } from 'vue'
// 引入表单和表单验证
import { Form, FormSchema } from '@/components/Form'
import { ElMessage } from 'element-plus'
import { useI18n } from '@/hooks/web/useI18n'
import { ElCheckbox, ElLink } from 'element-plus'
import { useForm } from '@/hooks/web/useForm'
import { loginApi, getTestRoleApi, getAdminRoleApi } from '@/api/login'
import { useAppStore } from '@/store/modules/app'
import { usePermissionStore } from '@/store/modules/permission'
import { useRouter } from 'vue-router'
import type { RouteLocationNormalizedLoaded, RouteRecordRaw } from 'vue-router'
import { UserType } from '@/api/login/types'
import { useValidator } from '@/hooks/web/useValidator'
// import { Icon } from '@/components/Icon'
import { useUserStore } from '@/store/modules/user'
import { BaseButton } from '@/components/Button'
// import { changeGlobalNodesTarget } from 'element-plus/es/utils'

const { required } = useValidator()

const emit = defineEmits(['to-register'])

const appStore = useAppStore()

const userStore = useUserStore()

const permissionStore = usePermissionStore()

const { currentRoute, addRoute, push } = useRouter()

const { t } = useI18n()

// 验证规则
const rules = {
  userId: [required()],
  // username: [required()],
  password: [required()]
}
// 页面要素
const schema = reactive<FormSchema[]>([
  // 登录
  {
    field: 'title',
    colProps: {
      span: 24
    },
    formItemProps: {
      slots: {
        default: () => {
          return <h2 class="text-2xl font-bold text-center w-[100%]">{t('login.login')}</h2>
        }
      }
    }
  },
  // 用户名
  {
    field: 'userId',
    label: t('login.userId'),
    // value: 'admin',
    component: 'Input',
    colProps: {
      span: 24
    },
    componentProps: {
      placeholder: t('login.userId')
    }
  },
  // 密码
  {
    field: 'password',
    label: t('login.password'),
    // value: 'admin',
    component: 'InputPassword',
    colProps: {
      span: 24
    },
    componentProps: {
      style: {
        width: '100%'
      },
      placeholder: 'admin or test'
    }
  },
  {
    field: 'tool',
    colProps: {
      span: 24
    },
    formItemProps: {
      slots: {
        default: () => {
          return (
            <>
              <div class="flex justify-between items-center w-[100%]">
                <ElCheckbox v-model={remember.value} label={t('login.remember')} size="small" />
                <ElLink type="primary" underline={false} onClick={forgetSecret}>
                  {t('login.forgetPassword')}
                </ElLink>
              </div>
            </>
          )
        }
      }
    }
  },
  {
    field: 'login',
    colProps: {
      span: 24
    },
    formItemProps: {
      slots: {
        default: () => {
          return (
            <>
              <div class="w-[100%]">
                <BaseButton
                  loading={loading.value}
                  type="primary"
                  class="w-[100%]"
                  onClick={signIn}
                >
                  {t('login.login')}
                </BaseButton>
              </div>
              <div class="w-[100%] mt-15px">
                <BaseButton class="w-[100%]" onClick={toRegister}>
                  {t('login.register')}
                </BaseButton>
              </div>
            </>
          )
        }
      }
    }
  }
  // {
  //   field: 'other',
  //   component: 'Divider',
  //   label: t('login.otherLogin'),
  //   componentProps: {
  //     contentPosition: 'center'
  //   }
  // },
  // {
  //   field: 'otherIcon',
  //   colProps: {
  //     span: 24
  //   },
  //   formItemProps: {
  //     slots: {
  //       default: () => {
  //         return (
  //           <>
  //             <div class="flex justify-between w-[100%]">
  //               <Icon
  //                 icon="vi-ant-design:github-filled"
  //                 size={iconSize}
  //                 class="cursor-pointer ant-icon"
  //                 color={iconColor}
  //                 hoverColor={hoverColor}
  //               />
  //               <Icon
  //                 icon="vi-ant-design:wechat-filled"
  //                 size={iconSize}
  //                 class="cursor-pointer ant-icon"
  //                 color={iconColor}
  //                 hoverColor={hoverColor}
  //               />
  //               <Icon
  //                 icon="vi-ant-design:alipay-circle-filled"
  //                 size={iconSize}
  //                 color={iconColor}
  //                 hoverColor={hoverColor}
  //                 class="cursor-pointer ant-icon"
  //               />
  //               <Icon
  //                 icon="vi-ant-design:weibo-circle-filled"
  //                 size={iconSize}
  //                 color={iconColor}
  //                 hoverColor={hoverColor}
  //                 class="cursor-pointer ant-icon"
  //               />
  //             </div>
  //           </>
  //         )
  //       }
  //     }
  //   }
  // }
])

// const iconSize = 30
// 默认true
const remember = ref(userStore.getRememberMe)

const initLoginInfo = () => {
  const loginInfo = userStore.getLoginInfo
  if (loginInfo) {
    const { userId, password } = loginInfo
    setValues({ userId, password })
  }
}
onMounted(() => {
  initLoginInfo()
})

const { formRegister, formMethods } = useForm()
const { getFormData, getElFormExpose, setValues } = formMethods
//登录是按钮前面的加载
const loading = ref(false)
// const iconColor = '#999'

// const hoverColor = 'var(--el-color-primary)'

const redirect = ref<string>('')

watch(
  () => currentRoute.value,
  (route: RouteLocationNormalizedLoaded) => {
    redirect.value = route?.query?.redirect as string
  },
  {
    immediate: true
  }
)

// 登录
const signIn = async () => {
  //getElFormExpose方法是验证表单是否填写完整
  const formRef = await getElFormExpose()
  await formRef?.validate(async (isValid) => {
    if (isValid) {
      loading.value = true
      //getFormData得到表单里面的元素
      const formData = await getFormData<UserType>()
      try {
        // 请求接口register
        const res = await loginApi(formData)
        if (res.data.code === 200) {
          // 是否记住我
          // if (unref(remember)) {
          userStore.setLoginInfo({
            userId: formData.userId,
            username: formData.username,
            password: ''
          })
          // } else {
          //   userStore.setLoginInfo(undefined)
          // }
          userStore.setRememberMe(unref(remember))
          userStore.setUserInfo(res.data.data)
          // 是否使用动态路由
          if (appStore.getDynamicRouter) {
            getRole()
          } else {
            await permissionStore.generateRoutes('static').catch(() => {})
            permissionStore.getAddRouters.forEach((route) => {
              addRoute(route as RouteRecordRaw) // 动态添加可访问路由表
            })
            permissionStore.setIsAddRouters(true)
            push({ path: redirect.value || permissionStore.addRouters[0].path })
          }
        } else {
          ElMessage.error(res.data.data.message || '登录失败')
        }
      } catch (error) {
        ElMessage.error('请求失败')
        // 这里可以更新 UI，显示错误信息
      } finally {
        loading.value = false
      }
    }
  })
}

// 获取角色信息
const getRole = async () => {
  const formData = await getFormData<UserType>()
  const params = {
    roleName: formData.userId
  }
  const res = await getAdminRoleApi(params)
  appStore.getDynamicRouter && appStore.getServerDynamicRouter
    ? await getAdminRoleApi(params)
    : await getTestRoleApi(params)
  if (res) {
    const routers = res.data || []
    userStore.setRoleRouters(routers)
    appStore.getDynamicRouter && appStore.getServerDynamicRouter
      ? await permissionStore.generateRoutes('server', routers).catch(() => {})
      : await permissionStore.generateRoutes('frontEnd', routers).catch(() => {})

    permissionStore.getAddRouters.forEach((route) => {
      addRoute(route as RouteRecordRaw) // 动态添加可访问路由表
    })
    permissionStore.setIsAddRouters(true)
    push({ path: redirect.value || permissionStore.addRouters[0].path })
  }
}

// 去注册页面
const toRegister = () => {
  emit('to-register')
}
// 忘记密码
const forgetSecret = () => {
  ElMessage.warning('请联系管理员修改密码')
}
</script>

<template>
  <Form
    :schema="schema"
    :rules="rules"
    label-position="top"
    hide-required-asterisk
    size="large"
    class="dark:(border-1 border-[var(--el-border-color)] border-solid)"
    @register="formRegister"
  />
</template>
