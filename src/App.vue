<template>
  <v-app>
    <v-navigation-drawer
      v-model="drawer"
      :rail="rail"
      permanent
    >
      <div :class="['logo', { 'logo-rail': rail }]">
        <img src="@/assets/logo.svg" alt="UPMS"/>
        <span style="margin-left: 2em" v-show="!rail">UPMS</span>
      </div>

      <v-divider></v-divider>

      <v-list
        density="compact"
        nav
        v-model:opened="openGroups"
        @click:open="handleListOpen"
      >
        <v-list-item prepend-icon="mdi-home" title="首页" to="/"/>

        <v-list-group value="auth" active="false">
          <template v-slot:activator="{ props }">
            <!-- 添加高亮和图标切换的逻辑 -->
            <v-list-item
              v-bind="props"
              prepend-icon="mdi-shield"
              title="权限管理"
            >
            </v-list-item>
          </template>

          <v-list-item
            prepend-icon="mdi-account-multiple"
            title="用户管理"
            to="/users"
          />
          <v-list-item
            prepend-icon="mdi-account-group"
            title="角色管理"
            to="/roles"
          />
          <v-list-item
            prepend-icon="mdi-shield-key"
            title="权限管理"
            to="/permissions"
          />
          <v-list-item
            prepend-icon="mdi-view-list"
            title="资源列表"
            to="/resources"
          />
        </v-list-group>
        <v-list-item prepend-icon="mdi-information" title="关于" to="/about"/>
      </v-list>

      <v-list density="compact" nav class="collapse-btn">
        <v-list-item
          @click="toggleRailAndMenu"
          @contextmenu.prevent="toggleOpenGroups"
          :prepend-icon="rail ? 'mdi-chevron-right' : 'mdi-chevron-left'"
          class="collapse-item"
        />
      </v-list>
    </v-navigation-drawer>

    <v-app-bar>
      <page-breadcrumb/>
    </v-app-bar>

    <v-main>
      <router-view></router-view>
    </v-main>
  </v-app>
</template>

<script setup>
import {ref, watch} from 'vue'
import PageBreadcrumb from '@/components/common/PageBreadcrumb.vue'

const drawer = ref(true)
const rail = ref(false)
const openGroups = ref([])  // 初始状态，auth组是打开的
const previousOpenState = ref([])  // 存储之前的打开状态

watch(rail, (newValue) => {
  localStorage.setItem('rail', JSON.stringify(newValue))
})

watch(openGroups, (newValue) => {
  localStorage.setItem('openGroups', JSON.stringify(newValue))
})

// 初始化状态
if (localStorage.getItem('rail')) {
  rail.value = JSON.parse(localStorage.getItem('rail'))
}

if (localStorage.getItem('openGroups')) {
  openGroups.value = JSON.parse(localStorage.getItem('openGroups'))
}

if (localStorage.getItem('previousOpenState')) {
  previousOpenState.value = JSON.parse(localStorage.getItem('previousOpenState'))
}
const handleListOpen = (event) => {
  // 只有在 rail 为 true 且点击的是 group 时才处理
  if (rail.value && event.id) {
    rail.value = false
    setTimeout(() => {
      openGroups.value = [event.id]
    }, 300)
  } else if (event.id) {
    // 如果点击的是 group，清空其他组的激活状态
    openGroups.value = [event.id]
  }
}
const toggleOpenGroups = () => {
  if (openGroups.value.length === 0 && previousOpenState.value.length > 0) {
    // 如果当前全部折叠且有之前的状态，恢复之前的状态
    openGroups.value = [...previousOpenState.value]
  } else {
    // 保存当前状态并全部折叠
    previousOpenState.value = [...openGroups.value]
    localStorage.setItem('previousOpenState', JSON.stringify(previousOpenState.value))
    openGroups.value = []
  }
}
// 在 script setup 中添加
const toggleRailAndMenu = () => {

  // 如果要收缩侧边栏
  if (!rail.value) {
    // 先保存当前菜单展开状态
    previousOpenState.value = [...openGroups.value]
    // 收起所有菜单
    openGroups.value = []

  } else {
    // 如果要展开侧边栏，恢复之前的菜单状态
    console.log([...previousOpenState.value])
    if (previousOpenState.value.length > 0) {
      openGroups.value = [...previousOpenState.value]
    }
  }
  // 切换侧边栏状态
  rail.value = !rail.value
}
</script>
<style scoped>

.active-item {
  background-color: #e3f2fd; /* 添加一个高亮背景色 */
  color: #1976d2; /* 字体颜色 */
}

.logo {
  height: 60px;
  display: flex;
  align-items: center;
  padding: 0 20px;
  transition: transform 0.3s;
}

.logo-rail {
  transform: translateX(-0.5em);
}

.logo img {
  height: 32px;
  margin-right: 12px;
}

.logo span {
  font-size: 18px;
  font-weight: 600;
  white-space: nowrap;
}

.collapse-btn {
  position: absolute;
  bottom: 0;
  width: 100%;
  border-top: thin solid rgba(0, 0, 0, 0.12);
}

.collapse-item {
  display: flex;
  justify-content: center;
  padding-left: 2.1em;
}

:deep(.collapse-item .v-list-item__prepend) {
  margin-inline-end: 0;
}
</style>


<!--<template>-->
<!--  <v-app>-->
<!--    <v-navigation-drawer>-->
<!--      <v-list>-->
<!--        &lt;!&ndash; 父级菜单 &ndash;&gt;-->
<!--        <v-list-group>-->
<!--          &lt;!&ndash; 自定义 activator 插槽 &ndash;&gt;-->
<!--          <template v-slot:activator>-->
<!--            <v-list-item-->
<!--              @click="toggleMenu"-->
<!--              :class="{ 'active-item': isOpen }"-->
<!--              prepend-icon="mdi-shield"-->
<!--            >-->
<!--              <v-icon>{{ isOpen ? 'mdi-chevron-down' : 'mdi-chevron-right' }}</v-icon>-->
<!--              <span>权限管理</span>-->

<!--              &lt;!&ndash; 手动控制收缩/展开的按钮 &ndash;&gt;-->
<!--              <v-btn-->
<!--                icon-->
<!--                @click.stop="manualCollapse"-->
<!--                variant="text"-->
<!--              >-->
<!--                <v-icon>{{ isCollapsed ? 'mdi-plus' : 'mdi-minus' }}</v-icon>-->
<!--              </v-btn>-->
<!--            </v-list-item>-->
<!--          </template>-->

<!--          &lt;!&ndash; 子菜单 &ndash;&gt;-->
<!--          <v-expand-transition>-->
<!--            <div v-show="!isCollapsed && isOpen">-->
<!--              <v-list-item title="用户管理" to="/users" />-->
<!--              <v-list-item title="角色管理" to="/roles" />-->
<!--              <v-list-item title="资源列表" to="/resources" />-->
<!--            </div>-->
<!--          </v-expand-transition>-->
<!--        </v-list-group>-->
<!--      </v-list>-->
<!--    </v-navigation-drawer>-->
<!--  </v-app>-->
<!--</template>-->

<!--<script setup>-->
<!--import { ref } from 'vue'-->

<!--// 自定义状态控制-->
<!--const isOpen = ref(false) // 控制默认的展开/收缩状态-->
<!--const isCollapsed = ref(false) // 手动按钮控制收缩-->

<!--// 手动切换菜单展开/关闭-->
<!--const toggleMenu = () => {-->
<!--  isOpen.value = !isOpen.value-->
<!--}-->

<!--// 手动按钮控制收缩/展开状态-->
<!--const manualCollapse = () => {-->
<!--  isCollapsed.value = !isCollapsed.value-->
<!--}-->
<!--</script>-->

<!--<style scoped>-->
<!--.active-item {-->
<!--  background-color: #e3f2fd; /* 高亮背景色 */-->
<!--  color: #1976d2; /* 字体颜色 */-->
<!--  cursor: pointer;-->
<!--}-->
<!--</style>-->
