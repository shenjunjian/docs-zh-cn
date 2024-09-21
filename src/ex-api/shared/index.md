# @vue/shared 共享工具函数

::: info 介绍
`@vue/shared`中的所有函数都是一些短小且在其它包中共享使用的函数。在普通用户工程中，尤其是组件库开发中，就容易会用到其中某些函数，比如转换驼峰。 与其自己项目中重新编写这些小函数，不如花点时间熟悉`@vue/shared`包，引用其中的函数，这样质量更好更可靠。
:::

## **4个常量值** {#CONST}

- `EMPTY_OBJ` = Object.freeze({})
- `EMPTY_ARR` = Object.freeze([])
- `NOOP`      = () => {}
- `NO`        = () => false

`EMPTY_OBJ,EMPTY_ARR` 通常用于参数解构时，充当相应属性不存在时的默认值。
`NOOP，NO` 通常也是赋值一个默认空函数。

```javascript
// 使用常量
function foo(list = EMPTY_ARR, option = EMPTY_OBJ){
    // 下面2行,控制台会显示 error
    list.push(1)
    option.add='add'

    // 精细判断 list是入参还是默认值
    if(list===EMPTY_ARR){
        // 用户未传入参
    }
}

foo()

// 不使用常量时
function bar(list = [], option = {}){
    // 正常执行
    list.push(1)
    option.add='add'

    // 无法判断调用时的入参是否为默认值
    if(list===[]){
        // ......
    }
}
bar()
```

## **makeMap** {#makeMap}

通过传入`,`分隔的字符串，返回一个测试函数用于测试val是否在传入字符串中。

```javascript
// 是否为自闭合元素
const isSelfCloseTag = makeMap('img,br,hr,input,link')

isSelfCloseTag('div') // false
isSelfCloseTag('input') // true

```

## **PatchFlags**枚举值 {#PatchFlags}

`Patch flags`是编译器生成的优化提示信息。 当一个具有动态子元素的`block`进行diff时，如果有该标记信息，
表明该render函数是编译生成的，diff算法就可以按需更新这些标记。

阅读 `runtime-core/src/renderer.ts` 下的 `patchElement`函数， 了解如何使用`PatchFlags` 的。

```typescript
enum PatchFlags {
  TEXT = 1,
  CLASS = 1 << 1, // 动态class
  STYLE = 1 << 2, // 动态style
  PROPS = 1 << 3, // 非 class,style的其它prop。  vnode还支持动态prop，也会有该标记
  FULL_PROPS = 1 << 4, // 元素有 props,且有key。 此时则不再标记: CLASS/STYLE/PROPS
  NEED_HYDRATION = 1 << 5, // 元素需要 水合props。 大概是不立即patch props, 等水合时再绑定props,events
  STABLE_FRAGMENT = 1 << 6, // fragment的子元素顺序不会变
  KEYED_FRAGMENT = 1 << 7, // fragment的子元素中， 全部或部分有key
  UNKEYED_FRAGMENT = 1 << 8, // fragment的子元素中，全没有key
  NEED_PATCH = 1 << 9,  // 需要非props的patch, 比如： ref, onVNodeXXX hook变化时。
  DYNAMIC_SLOTS = 1 << 10, // 动态的slot，比如v-for 或 slot name是动态变量时。  
  DEV_ROOT_FRAGMENT = 1 << 11, // 在dev模式时，用户在template的根下有注释时，才有该标记
 
  CACHED = -1, // 标记为：已被缓存静态的vnode
  BAIL = -2,  // 退出优化模式。 比如在renderSlot() 创建的vnode 是非编译的，此时通常需要完整的diff。
}

```

## **PatchFlagNames** {#PatchFlagNames}

一个数字为键值的对象，所有值都基本等同于上面的`PatchFlag`, 除了 `CACHED`

```typescript
  {
    1: "TEXT",
    2: "CLASS",
    ...

    -1: 'HOISTED' // PatchFlag中的值为： CACHED
  }
```
