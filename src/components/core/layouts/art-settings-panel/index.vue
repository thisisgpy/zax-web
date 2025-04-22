<template>
  <div class="layout-settings">
    <el-drawer
      size="300px"
      v-model="showDrawer"
      :lock-scroll="false"
      :with-header="false"
      :before-close="closeDrawer"
      @open="toggleDrawer(true)"
      @close="toggleDrawer(false)"
      modal-class="setting-modal"
    >
      <div class="drawer-con">
        <div class="close-wrap">
          <i class="iconfont-sys" @click="closeDrawer">&#xe7dc;</i>
        </div>

        <!-- 主题风格 -->
        <p class="title">{{ $t('setting.theme.title') }}</p>
        <div class="theme-styles">
          <div
            class="style-item"
            v-for="(item, index) in settingThemeList"
            :key="item.theme"
            @click="switchThemeStyles(item.theme)"
          >
            <div class="box" :class="{ 'is-active': item.theme === systemThemeMode }">
              <img :src="item.img" />
            </div>
            <p class="name">{{ $t(`setting.theme.list[${index}]`) }}</p>
            <div class="active" v-show="item.theme === systemThemeMode"></div>
          </div>
        </div>
      </div>
    </el-drawer>
  </div>
</template>

<script setup lang="ts">
  import { useSettingStore } from '@/store/modules/setting'
  import AppConfig from '@/config'
  import { SystemThemeEnum, MenuThemeEnum, MenuTypeEnum } from '@/enums/appEnum'
  import mittBus from '@/utils/mittBus'
  import { useTheme } from '@/composables/useTheme'
  import { useCeremony } from '@/composables/useCeremony'
  import { useI18n } from 'vue-i18n'
  const { t } = useI18n()

  const { openFestival, cleanup } = useCeremony()
  const { setSystemTheme, setSystemAutoTheme, switchThemeStyles } = useTheme()

  const settingStore = useSettingStore()

  const {
    systemThemeType,
    systemThemeMode,
    menuType,
    menuOpenWidth,
  } = storeToRefs(settingStore)

  const props = defineProps(['open'])

  const showDrawer = ref(false)

  const { width } = useWindowSize()

  // 记录窗口宽度变化前的菜单类型
  const beforeMenuType = ref<MenuTypeEnum>()
  const hasChangedMenu = ref(false) // 添加标记来跟踪是否已经改变过菜单

  watch(width, (newWidth: number) => {
    if (newWidth < 1000) {
      if (!hasChangedMenu.value) {
        beforeMenuType.value = menuType.value
        switchMenuLayouts(MenuTypeEnum.LEFT)
        settingStore.setMenuOpen(false)
        hasChangedMenu.value = true
      }
    } else {
      if (hasChangedMenu.value && beforeMenuType.value) {
        switchMenuLayouts(beforeMenuType.value)
        settingStore.setMenuOpen(true)
        hasChangedMenu.value = false
      }
    }
  })

  watch(
    () => props.open,
    (val: boolean) => (showDrawer.value = val)
  )

  const settingThemeList = AppConfig.settingThemeList
  const mainColor = AppConfig.systemMainColor

  const systemThemeColor = computed(
    () => settingStore.systemThemeColor as (typeof mainColor)[number]
  )

  const uniqueOpened = ref(true)
  const showMenuButton = ref(true)
  const autoClose = ref(true)
  const showRefreshButton = ref(true)
  const showCrumbs = ref(true)
  const showWorkTab = ref(true)
  const showLanguage = ref(true)
  const showNprogress = ref(true)
  const colorWeak = ref(false)

  watch(
    () => settingStore.showWorkTab,
    (e: boolean) => {
      showWorkTab.value = e
    }
  )

  onMounted(() => {
    mittBus.on('openSetting', openSetting)

    initSystemColor()
    listenerSystemTheme()
    initUserSetting()
    initSystemTheme()
    openFestival()
  })

  onUnmounted(() => {
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)')
    mediaQuery.removeEventListener('change', initSystemTheme)
    cleanup()
  })

  //  如果主题色不在列表中，则设置为列表中的第一个元素
  const initSystemColor = () => {
    if (!AppConfig.systemMainColor.includes(systemThemeColor.value)) {
      setElementTheme(AppConfig.systemMainColor[0])
    }
  }

  // 初始化用户设置
  const initUserSetting = () => {
    const mapping = {
      uniqueOpened,
      showMenuButton,
      autoClose,
      showRefreshButton,
      showCrumbs,
      showWorkTab,
      showLanguage,
      showNprogress,
      colorWeak
    }

    Object.entries(mapping).forEach(([key, refVal]) => {
      refVal.value = (settingStore as any)[key]
    })

    initColorWeak()
    setBoxMode(true, settingStore.boxBorderMode ? 'border-mode' : 'shadow-mode')
  }

  // 监听系统主题变化
  const listenerSystemTheme = () => {
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)')
    mediaQuery.addEventListener('change', initSystemTheme)
  }

  // 初始化系统主题
  const initSystemTheme = () => {
    if (systemThemeMode.value === SystemThemeEnum.AUTO) {
      setSystemAutoTheme()
    } else {
      setSystemTheme(systemThemeType.value)
    }
  }

  const switchMenuLayouts = (type: MenuTypeEnum) => {
    if (type === MenuTypeEnum.LEFT || type === MenuTypeEnum.TOP_LEFT) settingStore.setMenuOpen(true)
    settingStore.switchMenuLayouts(type)
    if (type === MenuTypeEnum.DUAL_MENU) {
      settingStore.switchMenuStyles(MenuThemeEnum.DESIGN)
      settingStore.setMenuOpen(true)
    }
    isAutoClose()
  }

  // 自动关闭
  const isAutoClose = () => {
    if (autoClose.value) {
      closeDrawer()
    }
  }

  // 打开或关闭抽屉
  const toggleDrawer = (open: boolean) => {
    let el = document.getElementsByTagName('body')[0]
    if (open) {
      setTimeout(() => {
        el.setAttribute('class', 'theme-change')
      }, 500)
    } else {
      el.removeAttribute('class')
    }
  }

  const openSetting = () => (showDrawer.value = true)

  const closeDrawer = () => (showDrawer.value = false)

  // 设置盒子边框 ｜ 阴影 样式
  const setBoxMode = (isInit: boolean = false, type: string) => {
    setTimeout(() => {
      const el = document.documentElement
      el.setAttribute('data-box-mode', type)

      if (!isInit) {
        settingStore.setBorderMode()
      }
    }, 50)
  }

  // 高阶函数：封装 store 方法调用后自动关闭抽屉
  const autoCloseHandler = (
    storeMethod: (...args: any[]) => void,
    needReload: boolean = false,
    ...args: any[]
  ) => {
    storeMethod(...args)
    if (needReload) {
      settingStore.reload()
    }
    isAutoClose()
  }

  const setElementTheme = (theme: string) =>
    autoCloseHandler(settingStore.setElementTheme, true, theme)

  const initColorWeak = () => {
    if (colorWeak.value) {
      let el = document.getElementsByTagName('html')[0]
      setTimeout(() => {
        el.setAttribute('class', 'color-weak')
      }, 100)
    }
  }

  watch(
    () => settingStore.menuOpenWidth,
    (newVal) => {
      menuOpenWidth.value = newVal
    }
  )
</script>

<style lang="scss">
  .setting-modal {
    background: transparent !important;

    .el-drawer {
      // 背景滤镜
      background: rgba($color: #fff, $alpha: 50%) !important;
      box-shadow: 0 0 30px rgb(0 0 0 / 10%) !important;

      @include backdropBlur();
    }
  }

  .dark {
    .setting-modal {
      .el-drawer {
        background: rgba($color: #000, $alpha: 50%) !important;
      }
    }
  }

  // 去除滚动条
  .el-drawer__body::-webkit-scrollbar {
    width: 0 !important;
  }
</style>

<style lang="scss" scoped>
  @use './style';
</style>
