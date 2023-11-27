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
