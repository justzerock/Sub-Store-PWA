<template>
  <div class="nav-bar-wrapper">
    <nav>
      <nut-navbar
        :left-show="isNeedBack"
        @on-click-back="back"
        @on-click-right="showLangSwitchPopup = true"
        :title="currentTitle"
        :tit-icon="currentTitleWhetherAsk"

        @on-click-icon="onClickNavbarIcon"
      >
        <template #right>
          <font-awesome-icon
              v-if="isSimpleMode"
              @click.stop="setSimpleMode(false)"
              class="navBar-right-icon fa-toggle"
              icon="fa-solid fa-toggle-on "
            />
          <font-awesome-icon
              v-else
              @click.stop="setSimpleMode(true)"
              class="navBar-right-icon fa-toggle"
              icon="fa-solid fa-toggle-off "
            />
            
          <font-awesome-icon
            class="navBar-right-icon fa-lg"
            icon="fa-solid fa-language "
          />
        </template>
      </nut-navbar>
    </nav>
  </div>

  <nut-popup
    lock-scroll
    pop-class="nav-bar-lang-switch-popup"
    position="top"
    v-model:visible="showLangSwitchPopup"
    z-index="1000"
  >
  <!-- :title="$t(`navBar.langSwitcher.cellTitle`)" -->
    <nut-cell-group >
      <div style="color: var(--comment-text-color); padding: 10px 0 10px 15px; font-size: 14px;">
        {{$t(`navBar.langSwitcher.cellTitle`)}}
      </div>
      <nut-cell
        v-for="lang in langList"
        :title="$t(`navBar.langSwitcher.${lang}`)"
        @click="changeLang(lang)"
        :key="lang"
        :class="{ selected: lang === locale }"
      >
        <template v-slot:icon>
          <font-awesome-icon
            v-if="lang === locale"
            class="fa-lg"
            icon="fa-solid fa-check"
          />
        </template>
      </nut-cell>
    </nut-cell-group>
  </nut-popup>
</template>

<script lang="ts" setup>
  import { computed, ref } from 'vue';
  import { useI18n } from 'vue-i18n';
  import { useRoute, useRouter } from 'vue-router';
  import { useGlobalStore } from '@/store/global';
  import { storeToRefs } from 'pinia';
  import { Toast } from '@nutui/nutui';


  const { t, locale } = useI18n();
  const router = useRouter();
  const route = useRoute();
  const globalStore = useGlobalStore();
  const showLangSwitchPopup = ref(false);
  const langList = ['zh', 'en'];
  const { isSimpleMode } = storeToRefs(globalStore);

  const isNeedBack = computed(() => {
    return route.meta.needNavBack ?? false;
  });

  const currentTitle = computed(() => {
    const metaTitle = route.meta.title;
    return metaTitle ? t(`navBar.pagesTitle.${metaTitle}`) : undefined;
  });
  const currentTitleWhetherAsk = computed(() => {
    const ownAsk = ['sync','subEditor','moreSetting'];
    const metaTitle = route.meta.title;
    return ownAsk.includes(metaTitle) ? 'ask' : '';
  });
  const onClickNavbarIcon = () => {
    const metaTitle = route.meta.title;
    const toastContent =
      t(`navBar.pagesTitle.askWhat.${metaTitle}.content`) || '';
    const toastTitle = t(`navBar.pagesTitle.askWhat.${metaTitle}.title`) || '';
    Toast.text(toastContent, {
      title: toastTitle,
      duration: 0,
      cover: true,
      'close-on-click-overlay': true,
      'bg-color': 'rgba(0, 0, 0, 0.8)',
      'cover-color': 'rgba(0, 0, 0, 0.2)',
      'text-align-center': false,
    });
  };

  const navBarHeight = '56px';
  const changeLang = (type: string) => {
    locale.value = type;
    localStorage.setItem('locale', type);
    showLangSwitchPopup.value = false;
  };

  const back = () => {
    if (isNeedBack.value) {
      router.back();
    }
  };
  const setSimpleMode = (isSimpleMode: boolean) => {
    globalStore.setSimpleMode(isSimpleMode);
  };

</script>

<style lang="scss">
  .nav-bar-wrapper {
    position: fixed;
    width: 100%;
    top: 0;
    height: v-bind(navBarHeight);
    z-index: 20;

    nav {
      .nut-navbar {
        height: v-bind(navBarHeight);
        top: 0;
        box-shadow: none;
        backdrop-filter: blur(var(--nav-bar-blur));
        -webkit-backdrop-filter: blur(var(--nav-bar-blur));
        background: var(--nav-bar-color);
        border-bottom: var(--divider-color) solid 1px;

        .nut-navbar__title {
          min-width:53%;
          margin: 0 auto;
          text-align: center;
          display: flex;
          justify-content: center;
          align-items: center;
          .title {
            min-width: 20px;
            font-size: 18px;
            font-weight: 600;
            // line-height: 100%;
            color: var(--primary-text-color);
            display: nowrap;
            -webkit-box-orient: vertical;
            -webkit-line-clamp: 1;
            overflow: hidden;
          }
          .nutui-iconfont {
            margin-left: 5px;
          }
        }

        .navBar-right-icon {
          padding-top: 15px;
          padding-right:4px;
          padding-bottom: 15px;
          padding-left: 10px;
          color: var(--icon-nav-bar-right);
        }
        .fa-lg {
          position: absolute;
          right: 15px;
          top: 50%;
          transform: translateY(-50%);
        }
        .fa-toggle {
          position: absolute;
          right: 58px;
          top: 50%;
          transform: translateY(-50%);
        }
      }
    }
  }

  .nav-bar-lang-switch-popup > .nut-cell-group {
    width: 100%;

    background-color: var(--popup-color);
    > .nut-cell-group__title {
      color: var(--comment-text-color);
    }

    > .nut-cell-group__warp {
      background-color: var(--popup-color);

      > .nut-cell {
        background-color: var(--popup-color);

        &::after {
          border-color: var(--divider-color);
        }
      }

      > .nut-cell:not(.selected) {
        color: var(--primary-text-color);
      }
    }

    .selected.nut-cell {
      color: var(--primary-color);
      font-weight: bold;
      display: flex;
      align-items: center;
      flex-direction: row-reverse;
    }
  }

</style>
