<template>
  <a-row id="globalHeader" align="center" :wrap="false">
    <a-col flex="auto">
      <a-menu
        mode="horizontal"
        :selected-keys="selectedKeys"
        @menu-item-click="doMenuClick"
      >
        <a-menu-item
          key="0"
          :style="{ padding: 0, marginRight: '38px' }"
          disabled
        >
          <div class="title-bar">
            <img class="logo" src="../assets/dxoj_logo.png" />
            <div class="title">DX OJ</div>
          </div>
        </a-menu-item>
        <a-menu-item v-for="item in visibleRoutes" :key="item.path">
          {{ item.name }}
        </a-menu-item>
      </a-menu>
    </a-col>
    <a-col flex="100px">
      <!--      <div>-->
      <!--        {{ store.state.user?.loginUser?.userName ?? "未登录" }}-->
      <!--      </div>-->
      <a-dropdown>
        <a-avatar @click="showDetail" :style="{ backgroundColor: '#14C9C9' }">
          <IconUser />
          <template #trigger-icon>
            <IconEdit />
          </template>
        </a-avatar>
        <template #content>
          <a-doption @click="doLogin">
            <template #icon>
              <icon-import />
            </template>
            <template #default>Log In</template>
          </a-doption>
          <a-doption @click="doLogout">
            <template #icon>
              <icon-export />
            </template>
            <template #default>Log Out</template>
          </a-doption>
        </template>
      </a-dropdown>
    </a-col>
  </a-row>
</template>

<script setup lang="ts">
import { routes } from "../router/routes";
import { useRouter } from "vue-router";
import { computed, onMounted, onUpdated, ref } from "vue";
import { useStore } from "vuex";
import checkAccess from "@/access/checkAccess";
import ACCESS_ENUM from "@/access/accessEnum";
import {
  IconEdit,
  IconImport,
  IconExport,
  IconUser,
} from "@arco-design/web-vue/es/icon";
import { UserControllerService } from "../../generated";
import message from "@arco-design/web-vue/es/message";

const router = useRouter();
const store = useStore();

const userName = ref("未登录");

// 展示在菜单的路由数组
const visibleRoutes = computed(() => {
  return routes.filter((item, index) => {
    if (item.meta?.hideInMenu) {
      return false;
    }
    // 根据权限过滤菜单
    if (
      !checkAccess(store.state.user.loginUser, item?.meta?.access as string)
    ) {
      return false;
    }
    return true;
  });
});

// 默认主页
const selectedKeys = ref(["/"]);

// 路由跳转后，更新选中的菜单项
router.afterEach((to, from, failure) => {
  selectedKeys.value = [to.path];
});

onMounted(() => {
  userName.value = store.state.user?.loginUser?.userName.slice(0, 10);
});
onUpdated(() => {
  userName.value = store.state.user?.loginUser?.userName.slice(0, 10);
});
console.log();

// setTimeout(() => {
//   store.dispatch("user/getLoginUser", {
//     userName: "管理员",
//     userRole: ACCESS_ENUM.ADMIN,
//   });
// }, 3000);

const doMenuClick = (key: string) => {
  router.push({
    path: key,
  });
};

const showDetail = () => {
  console.log("详情");
};

const doLogout = async () => {
  const res = await UserControllerService.userLogoutUsingPost();
  // 退出登录成功，跳转到主页
  if (res.code === 0) {
    message.success("已退出登录");
    store.dispatch("user/getLoginUser");
    userName.value = "未登录";
  } else {
    message.error("退出登录失败，" + res.message);
  }
};
const doLogin = () => {
  let loginUser = store.state.user.loginUser;
  if (
    !loginUser ||
    !loginUser.userRole ||
    loginUser.userRole === ACCESS_ENUM.NOT_LOGIN
  ) {
    router.push({
      path: "/user/login",
      replace: true,
    });
  } else {
    message.info("已登录啦！！请勿重复登录");
  }
};
</script>

<style scoped>
.title-bar {
  display: flex;
  align-items: center;
}

.title {
  color: #444;
  margin-left: 16px;
}

.logo {
  height: 48px;
}
</style>
