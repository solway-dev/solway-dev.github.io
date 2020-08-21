---
title: vue项目主题切换
date: 2020-08-07 11:49:28
tags:
---

### 使用方法


#### 在路由中配置当前页面支持的主题
``` js
[{
    path: '/dataReport',
    name: 'dataReport',
    component: () => import('@/pages/dataReport'),
    meta: {
        title: '数据报表',
        icon: 'iconfont icon-baogao',
        // 在路由中配置当前页面支持的主题
        pageTheme: ['light', 'dark']
    }
}]
```
#### scss 中使用写
``` scss
// 定义主题列表，覆盖全局定义的主题列表，也就是当前页面支持的主题列表，
// 一般多定义几个，不用的主题随便写个颜色，以后好改好加
$pageTheme: darkBlue light dark;

// 定义当前页面内使用的scss变量, 颜色与主题色一一对应
$fontColor: #fff #333 rgba(47, 191, 204, 1);
$bgColor: #fff #fff #18305B;
$bdColor: #fff #fff #18305B;

div {
    @include page_c(color, $fontColor);
    @include page_c(background-color, $bgColor);

    // 复合属性，需要单独设置颜色。把设置颜色的属性拆出来，覆盖
    border: 1px solid;
    @include page_c(border-color, $bdColor);

    // 背景图，两个参数为前缀和后缀。  中间会自动拼接主题名
    @include bgi('~./img/', '/a.jpg');
    // 生成结果如下，所以需要在对应目录放图片
    // background-image: url('~./img/darkBlue/a.jpg');
    // background-image: url('~./img/light/a.jpg');
    // background-image: url('~./img/dark/a.jpg');
}
```


#### 写一个vue mixin组件
``` js
export default {
    data () {
        this._darkBlue = {
            $axisTextColor: 'rgba(26,127,161,1)',
            $chartTextColor: 'rgba(45,183,195,1)'
        }
        this._light = {
            $axisTextColor: 'rgba(110, 110,110,0.5)',
            $chartTextColor: 'rgba(110, 110,110,0.5)'
        }
        this._dark = {
            $axisTextColor: 'rgba(26,127,161,1)',
            $chartTextColor: '#BDC0C5'
        }
        return {}
    },
    computed: {
        theme () {
            return this['_' + this.themeName] || {}
        },
        themeName () {
            if (this.$route.meta?.pageTheme?.some(v => v === this.$store.getters.global_theme)) return this.$store.getters.global_theme
            return this.$route.meta?.pageTheme?.[0] || this.$store.getters.global_theme
        }
    }
}

```

#### js 中使用
``` js
import theme from './theme';
export default {
    mixins: [theme],
    // ...
    created () {
        this.extend.textStyle = {color: this.theme.$chartTextColor}
        this.extend.legend.textStyle = {color: this.theme.$chartTextColor}
        this.$nextTick(() => this.setChart())
    },
    // ...
}
```


### 实现原理

#### 定义scss函数
``` scss
@charset "utf-8";
$pageTheme: darkBlue;

// 这个是页面内使用，可以传入主题
@mixin page_c($attr, $color-list) {
    @for $i from 1 to length($pageTheme)+1 {
        [data-pagetheme="#{nth($pageTheme, $i)}"] & {
            #{$attr}: nth($color-list, $i);
        }
    }
}


// 背景图
@mixin page_bgi($frist, $last) {
    @for $i from 1 to length($pageTheme)+1 {
        [data-pagetheme="#{nth($pageTheme, $i)}"] & {
            background-image: url("#{$frist}#{nth($pageTheme, $i)}#{$last}");
        }
    }
}
```

#### webpack 配置如下，全局注入scss
``` js
[
    {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {
            loaders: {
                {
                    ...
                    scss: generateLoaders('sass').concat(
                        {
                            loader: 'sass-resources-loader',
                            options: {
                                // 需要全局引入的sass文件，这里引入了的scss文件，在所有的.vue文件都可以用到这份css样式，
                                // 下面的resources接受一个数组，可以添加多个scss文件
                                resources: [
                                    path.resolve(__dirname, `../src/${multiConfig.process.name}/assets/themes/index.scss`)
                                ]
                            }
                        }
                    ),
                    ...
                }
            },
            ...
        }
    }
]
```


#### 根组件修改
``` js
export default {
    name: 'App',
    created () {
        // 初始化选中主题
        document.documentElement.dataset.pagetheme = this.pagetheme
    },
    computed: {
        // 计算当前应该应用哪个主题，即使全局主题已切换，若当前页面不支持全局主题，就用默认第一个支持的主题。
        pagetheme () {
            if (this.$route.meta?.pageTheme?.some(v => v === this.$store.getters.global_theme)) return this.$store.getters.global_theme
            return this.$route.meta?.pageTheme?.[0] || this.$store.getters.global_theme
        }
    },
    watch: {
        pagetheme (val) {
            // 监听主题切换，根元素设置自定义属性为 当前主题名
            document.documentElement.dataset.pagetheme = val
        }
    }
}
```

全局主题可以记录到store


******************************************


完
