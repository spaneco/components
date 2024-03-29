<script>
import Prism from "prismjs";
import "prismjs/themes/prism.css";

import { getModuleNotFoundMsg } from "@/utils/helpers";

import CopyButton from "@/components/app/CopyButton.vue";
import Viewport from "@/components/app/Viewport.vue";

export default {
  name: "Component",
  components: {
    CopyButton,
    Viewport,
  },
  data() {
    return {
      contentHtml: "",
      contenCss: "",
      contentJs: "",
      iFrameSrc: "",
      isViewHidden: false,
      isMarkupHidden: true,
      dynamicElementClassname: "js-dynamically-created",
    };
  },
  computed: {
    isMarkupPanelVisible() {
      return this.$store.state.isMarkupPanelVisible;
    },
    isIframeView() {},
  },
  methods: {
    async loadData(componentPath) {
      try {
        const promiseHtml = await import(
          `!raw-loader!@/components/development/${componentPath}/c.html`
        );
        this.contentHtml = promiseHtml.default;
      } catch (e) {
        this.contentHtml = getModuleNotFoundMsg("HTML");
        console.warn(e);
      }

      try {
        const promiseCss = await import(
          `!raw-loader!@/components/development/${componentPath}/c.css`
        );
        this.contenCss = promiseCss.default;

        const style = document.createElement("style");
        style.innerHTML = this.contenCss;
        style.classList.add(`${this.dynamicElementClassname}`);
        document.head.appendChild(style);
      } catch (e) {
        this.contenCss = getModuleNotFoundMsg("CSS");
        console.warn(e);
      }

      try {
        const promiseJs = await import(
          `!raw-loader!@/components/development/${componentPath}/c.js`
        );
        this.contentJs = promiseJs.default;

        const script = document.createElement("script");
        script.innerHTML = this.contentJs;
        script.classList.add(`${this.dynamicElementClassname}`);
        document.body.appendChild(script);
      } catch (e) {
        this.contentJs = getModuleNotFoundMsg("JS");
        console.warn(e);
      }

      setTimeout(() => {
        Prism.highlightAll();
      });

      this.iFrameSrc = `/views/${componentPath}`;
    },
  },
  created() {
    this.$watch(
      () => this.$route.params,
      (toParams, previousParams) => {
        document
          .querySelectorAll(`.${this.dynamicElementClassname}`)
          .forEach((item) => item.remove());
        if (this.$route.params.id) {
          try {
            const componentPath = this.$route.params.id.join("/");
            this.loadData(componentPath);
          } catch (e) {
            console.log(e);
          }
        }
      }
    );
  },
  mounted() {
    this.isViewHidden = this.$route.meta.isViewHidden;
    const params = this.$route.params;
    try {
      const componentPath = params.id.join("/");
      this.loadData(componentPath);
    } catch (e) {
      console.log(e);
    }
  },
};
</script>

<template>
  <div class="component js-component" v-html="contentHtml"></div>

  <Viewport :iFrameSrc="iFrameSrc" v-if="!isViewHidden"/>

  <transition name="slide" v-if="!isViewHidden">
    <div
      :class="['markups', { slide: isMarkupPanelVisible }]"
      v-show="isMarkupPanelVisible"
    >
      <div class="markup__col">
        <div class="markup__header">
          <div class="markup__bookmark">
            <svg viewBox="0 0 15 15" class="markup__icon">
              <rect fill="#FF3C41" width="15" height="15" rx="4"></rect>
              <path
                d="M10.97 2.29a.563.563 0 0 0-.495-.29.572.572 0 0 0-.488.277l-5.905 9.86a.565.565 0 0 0-.007.574c.102.18.287.289.495.289a.572.572 0 0 0 .488-.277l5.905-9.86a.565.565 0 0 0 .007-.574"
                fill="#28282B"
              ></path>
            </svg>
            <span class="markup__title">HTML</span>
          </div>
          <CopyButton :copy-data="contentHtml" />
        </div>
        <pre
          class="js-language-html markup__language language-html"
        ><code>{{ contentHtml }}</code></pre>
      </div>

      <div class="markup__col">
        <div class="markup__header">
          <div class="markup__bookmark">
            <svg viewBox="0 0 15 15" class="markup__icon">
              <rect fill="#FCD000" width="15" height="15" rx="4"></rect>
              <path
                d="M6.554 3.705c0 .267-.19.496-.452.543-1.2.217-2.12 1.61-2.12 3.275 0 1.665.92 3.057 2.12 3.274a.554.554 0 0 1-.205 1.087c-1.733-.322-3.022-2.175-3.022-4.361 0-2.187 1.289-4.04 3.022-4.362a.554.554 0 0 1 .657.544zm1.892 0c0-.347.316-.607.657-.544 1.733.322 3.022 2.175 3.022 4.362 0 2.186-1.289 4.04-3.022 4.361a.554.554 0 0 1-.205-1.087c1.2-.217 2.12-1.61 2.12-3.274 0-1.665-.92-3.058-2.12-3.275a.551.551 0 0 1-.452-.543z"
                fill="#282828"
              ></path>
            </svg>
            <span class="markup__title">JS</span>
          </div>

          <CopyButton :copy-data="contentJs" />
        </div>
        <pre
          class="markup__language language-javascript"
        ><code>{{ contentJs }}</code></pre>
      </div>

      <div class="markup__col">
        <div class="markup__header">
          <div class="markup__bookmark">
            <svg viewBox="0 0 15 15" class="markup__icon">
              <rect fill="#0EBEFF" width="15" height="15" rx="4"></rect>
              <path
                d="M8 8.366l1.845 1.065a.507.507 0 0 0 .686-.181.507.507 0 0 0-.186-.685L8.5 7.5l1.845-1.065a.507.507 0 0 0 .186-.685.507.507 0 0 0-.686-.181L8 6.634v-2.13A.507.507 0 0 0 7.5 4c-.268 0-.5.225-.5.503v2.131L5.155 5.569a.507.507 0 0 0-.686.181.507.507 0 0 0 .186.685L6.5 7.5 4.655 8.565a.507.507 0 0 0-.186.685c.134.232.445.32.686.181L7 8.366v2.13c0 .271.224.504.5.504.268 0 .5-.225.5-.503V8.366z"
                fill="#282828"
              ></path>
            </svg>
            <span class="markup__title">CSS</span>
          </div>
          <CopyButton :copy-data="contenCss" />
        </div>
        <pre
          class="markup__language language-css"
        ><code>{{ contenCss }}</code></pre>
      </div>
    </div>
  </transition>
</template>

<style scoped>
.component {
}

.slide-enter {
  transform: translateY(0);
}
.slide-enter-active {
  transform: translateY(100%);
}

.slide-leave {
  transform: translateY(0);
}
.slide-leave-active {
  transform: translateY(100%);
}

.slide-enter-active,
.slide-leave-active {
  transition: all 750ms ease-in-out;
}

.markups {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  width: 100%;
  position: relative;
  bottom: 0;
  display: flex;
  flex-wrap: wrap;
  position: fixed;
  border-top: 1px solid #e1e1e1;
  transition: all 300ms ease;
}

.markup__col {
  display: flex;
  flex-flow: column;
  flex: 1;
  max-height: min(250px, 50vh);
  position: relative;
}

.markup__col::after {
  content: "";
  position: absolute;
  right: 100%;
  top: 0;
  width: var(--app-markup-col-border-width);
  height: 100%;
  background-color: var(--c-app-markup-col-bg);
  opacity: 0.1;
}

.markup__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  color: var(--c-app-markup-heading-fc);
  background-color: var(--c-app-markup-col-bg);
  margin-right: var(--app-markup-col-border-width);
}

.markup__bookmark {
  display: flex;
  padding: 9px 12px;
  gap: 8px;
  align-items: center;
  font-weight: 700;
}

.markup__icon {
  width: 17px;
}

.markup__language {
  margin: 0;
  flex: 1;
  overflow: auto;
  font-size: 13px;
}

.viewport {
  width: 400px;
}
</style>