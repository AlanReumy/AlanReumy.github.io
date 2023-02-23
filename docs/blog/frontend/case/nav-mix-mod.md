# 导航栏模式切换

需求：实现一个导航栏模式切换的功能，要求能够通过按钮切换不同模式

![20230223103841](https://codertzm.oss-cn-chengdu.aliyuncs.com/20230223103841.png)

三种模式：

- 开启左侧侧边栏模式：

  ![20230223104010](https://codertzm.oss-cn-chengdu.aliyuncs.com/20230223104010.png)

- 开启顶部菜单模式：

  ![20230223104036](https://codertzm.oss-cn-chengdu.aliyuncs.com/20230223104036.png)

- 开启混合模式加分割菜单：

  ![20230223103946](https://codertzm.oss-cn-chengdu.aliyuncs.com/20230223103946.png)

## 组件封装

封装一个 `ASideMenu` 组件，这里基于 `Naive UI` 提供的 `Menu` 组件进行二次封装

```html
<NMenu
  :options="menus"
  :inverted="inverted"
  :mode="mode"
  :collapsed="collapsed"
  :collapsed-width="64"
  :collapsed-icon-size="20"
  :indent="24"
  :expanded-keys="openKeys"
  :value="getSelectedKeys"
  @update:value="clickMenuItem"
  @update:expanded-keys="menuExpanded"
/>
```

其中，该组件主要接受三个 `props`：

- mode：菜单模式
- collapsed：侧边栏菜单是否收起
- location：位置

## 组件使用

布局 (简洁代码)：

```html
<layout>
  <n-layout-sider
    v-if="
      !isMobile && isMixMenuNoneSub && (navMode === 'vertical' || navMode === 'horizontal-mix')
    "
    @collapse="collapsed = true"
    :position="fixedMenu"
    @expand="collapsed = false"
    :collapsed="collapsed"
    :width="leftMenuWidth"
    :inverted="inverted"
    class="layout-sider"
  >
    <Logo :collapsed="collapsed" />
    <AsideMenu
      v-model:collapsed="collapsed"
      v-model:location="getMenuLocation"
    />
  </n-layout-sider>

  <n-layout :inverted="inverted">
    <n-layout-header :inverted="getHeaderInverted" :position="fixedHeader">
      <PageHeader v-model:collapsed="collapsed" :inverted="inverted" />
    </n-layout-header>
    // ...
  </n-layout>
</layout>
```

其中 `<PageHeader/>` 组件中包含了 `<AsideMenu>` 组件：

```html
<div
  class="layout-header-left"
  v-if="navMode === 'horizontal' || (navMode === 'horizontal-mix' && mixMenu)"
>
  <div class="logo" v-if="navMode === 'horizontal'">
    <img :src="websiteConfig.logo" alt="" />
    <h2 v-show="!collapsed" class="title">{{ websiteConfig.title }}</h2>
  </div>
  <AsideMenu
    v-model:collapsed="collapsed"
    v-model:location="getMenuLocation"
    :inverted="getInverted"
    mode="horizontal"
  />
</div>
// ...
```

## 逻辑梳理

我们可以将 `navMode` 导航栏模式、`menuSetting.mixMenu` 配置 存到 `store` 状态管理中

1. 控制显示隐藏：对不同的 `navMode` 进行不同的显示
2. 响应式更新 `menu` 配置：实现一个 `updateMenu` 方法。同时对 `mixMenu` 进行监听，该变量发生变化后，对 `updateMenu` 方法进行调用

   ```js
   function updateMenu() {
     if (!settingStore.menuSetting.mixMenu) {
       // 对非分割按钮开启的模式进行配置
       menus.value = generatorMenu(asyncRouteStore.getMenus);
     } else {
       //混合菜单
       const firstRouteName: string = (currentRoute.matched[0].name as string) || '';
       // 对分割按钮开启的模式进行配置
       menus.value = generatorMenuMix(asyncRouteStore.getMenus, firstRouteName, props.location);
       const activeMenu: string = currentRoute?.matched[0].meta?.activeMenu as string;
       headerMenuSelectKey.value = (activeMenu ? activeMenu : firstRouteName) || '';
     }
     updateSelectedKeys();
   }
   ```

3. 对不同位置的 `menu` 进行不同的配置

   ```js
   function generatorMenuMix(
     routerMap: Array<any>,
     routerName: string,
     location: string
   ) {
     const cloneRouterMap = cloneDeep(routerMap);
     const newRouter = filterRouter(cloneRouterMap);
     // 如果是header位置
     if (location === "header") {
       const firstRouter: any[] = [];
       newRouter.forEach((item) => {
         const isRoot = isRootRouter(item);
         const info = isRoot ? item.children[0] : item;
         info.children = undefined;
         const currentMenu = {
           ...info,
           ...info.meta,
           label: info.meta?.title,
           key: info.name,
         };
         firstRouter.push(currentMenu);
       });
       return firstRouter;
     } else {
       // left位置只对currentRouter的children进行返回
       return getChildrenRouter(
         newRouter.filter((item) => item.name === routerName)
       );
     }
   }
   ```

4. 对 `left` 位置的 `sidebar` 进行递归组装子菜单

  ```js
  /**
   * 递归组装子菜单
  * */
  function getChildrenRouter(routerMap: Array<any>) {
    return filterRouter(routerMap).map((item) => {
      const isRoot = isRootRouter(item);
      const info = isRoot ? item.children[0] : item;
      const currentMenu = {
        ...info,
        ...info.meta,
        label: info.meta?.title,
        key: info.name,
      };
      // 是否有子菜单，并递归处理
      if (info.children && info.children.length > 0) {
        // Recursion
        currentMenu.children = getChildrenRouter(info.children);
      }
      return currentMenu;
    });
  }
  ```
