# vue2



* [https://v2.vuejs.org/v2/guide/components](https://v2.vuejs.org/v2/guide/components)
* [https://vueuse.org/](https://vueuse.org/)





#### array

```javascript
Vue.set(this.mValue, 1, null)
```

```html
<template>
<el-select v-model="platChalValue" v-bind="$attrs" placeholder="请选择平台渠道" clearable>
    <el-option-group
      v-for="group in options"
      :key="group.label"
      :label="group.label">
      <el-option
        v-for="item in group.options"
        :key="item.value"
        :label="item.label"
        :value="item.value"
        v-show="!item.isAllOp || showAllOp">
      </el-option>
    </el-option-group>
</el-select>
</template>

<script>
import Vue from 'vue'
import { getPlatformList } from '@/api/channel'
import { isEqual } from '@/utils/util';
export default {
    props: [ 'hideAll', 'value' ],
    watch: {
        hideAll(val) {
            if (val === undefined) {
                this.showAllOp = true
            } else {
                this.showAllOp = false
            }
        },
        platChalValue(val) {
            if (val === '') {
                Vue.set(this.mValue, 0, null)
                Vue.set(this.mValue, 1, null)
            } else {
                const [ platform_type, client_channel ]  = JSON.parse(val)
                Vue.set(this.mValue, 0, platform_type)
                Vue.set(this.mValue, 1, client_channel)
            }
            this.onChange(this.mValue)
        },
        value(val) {
            if (!isEqual(val, this.mValue)) {
                Vue.set(this.mValue, 0, val[0])
                Vue.set(this.mValue, 1, val[1])
            }
        },
        mValue(val) {
            const vstr = JSON.stringify(this.mValue)
            if (!isEqual(vstr, this.platChalValue)) {
                this.platChalValue = vstr.includes('null') ? '' : vstr
            }
        }
    },
    computed: {
    },
    data() {
        return {
            showAllOp: this.hideAll === undefined,
            options: [{
                label: '平台',
                options: [{
                    value: 0,
                    label: '渠道'
                }]
                }],
            platChalValue: '',
            mValue: [0,0]
        }
    },
    async mounted() {
        this.options = await this.fetchOptions()
        if (Array.isArray(this.value) && this.value[0] !== null) {
            this.platChalValue = JSON.stringify(this.value)
        }
    },
    methods: {
        onChange(val) {
            this.$emit('input', val)
        },
        fetchOptions() {
            return getPlatformList().then(({ data }) => {
                return data.platform.map(pl => {
                    const platform_type = pl.value
                    const chOps = data.channel
                        .filter(ch => ch.platform_type == platform_type && ch.client_channel > 0)
                        .sort((a,b) => {
                            return a.channel_name.localeCompare(b.channel_name)
                        })
                        .map(ch => {
                            return {
                                label: ch.channel_name,
                                value: JSON.stringify([ch.platform_type, ch.client_channel]),
                                isAllOp: false
                            }
                    })
                    const platOp = {
                        label: '全部 - ' + pl.label,
                        value: JSON.stringify([platform_type, null]),
                        isAllOp: true,
                    }
                    return {
                        label: pl.label,
                        options: [platOp, ...chOps]
                    }
                })
            })
        }
    }
}
</script>
```



### persistance in vuex&#x20;



when you wanna persistance vuex in vue2

* useStorage and toReactive
* ```
  import { useStorage, toReactive } from '@vueuse/core'
  ...
  const stateRef = useStorage('vuex:settings', {
    showSettings: showSettings,
    fixedHeader: fixedHeader,
    sidebarLogo: sidebarLogo,
    showBeta: showBeta
  })
  const state = toReactive(stateRef)
  ```

```javascript
import defaultSettings from '@/settings'
import { useStorage, toReactive } from '@vueuse/core'
const { showSettings, fixedHeader, sidebarLogo, showBeta } = defaultSettings

const stateRef = useStorage('vuex:settings', {
  showSettings: showSettings,
  fixedHeader: fixedHeader,
  sidebarLogo: sidebarLogo,
  showBeta: showBeta
})
const state = toReactive(stateRef)

const mutations = {
  CHANGE_SETTING: (state, { key, value }) => {
    // eslint-disable-next-line no-prototype-builtins
    if (state.hasOwnProperty(key)) {
      state[key] = value
    }
  }
}

const actions = {
  changeSetting({ commit }, data) {
    commit('CHANGE_SETTING', data)
  }
}

export default {
  namespaced: true,
  state,
  mutations,
  actions
}

```



### 操作引导

使用纯js的库[driver.js](https://driverjs.com/)，根据css选择器进行页面操作引导



#### floating  icon group



使用图标入口

````
```vue
    <div class="floating-group-container">
      <i class="el-icon-info" @click="startTour"></i>
      <!-- <MarkdownHelp title="数据说明" :markdownText="helpDialog.markdownText"/> -->
    </div>
```
````



悬浮图表组的全局样式

````
```scss
.floating-group-container {
  position: fixed;
  bottom: 40px;
  right: 20px;
  // background-color: #fff;
  color: #d0d0d0;
  width: 40px;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: 0;
  // box-shadow: 0 0 6px rgba(0,0,0,.12);
  border-radius: 5px;
  z-index: 1024;
  overflow: visible;

  i {
    margin: auto;
    width: 100%;
    font-size: 32px;
    text-align: center;
    &:hover {
      color: $--color-success;
      cursor: pointer;

      transform: scale(1.2);
      transition: all .1s;
    }
  }

}
```
````



#### 使用markdown弹窗帮助文档

````
```vue
<template>
<div>
  <el-dialog append-to-body v-bind="$attrs" :title="title" v-on="$listeners" @open="onOpen" @close="onClose">
    <div v-if="loading">Loading...</div>
    <div v-else v-html="markdownHtml"></div>
  </el-dialog>
</div>
</template>
  
  <script>
  import MarkdownIt from 'markdown-it';
  
  export default {
    props: {
      markdownText: {
        type: String,
        require: true
      },
      title: {
        type: String,
        require: true
      }
    },
    data() {
      return {
        markdownHtml: '',
        loading: true
      };
    },
    created() {
      this.md = new MarkdownIt()
    },
    watch: {
      markdownText(val) {
        this.mdRender(val)
      },
    },
    mounted() {
      this.mdRender(this.markdownText)
    },
    methods: {
      onOpen() {
      },
      onClose() {
      },
      close() {
      },
      mdRender(val) {
        this.loading = true
        this.markdownHtml = this.md.render(val || '');
        this.loading = false
      }
    },
  };
  </script>
  
  <style>
  /* Add your component-specific styles here */
  </style>
  
```
````



