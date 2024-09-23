---
title: \@vue/shared 共享工具函数
---

<script setup lang="ts">
import * as shared from '@vue/shared'

console.log("shared=", shared)
</script>

# @vue/shared 共享工具函数

::: info 介绍
`@vue/shared`中的所有函数都是一些短小且在其它包中共享使用的函数。在普通用户工程中，尤其是组件库开发中，就容易会用到其中某些函数，比如转换驼峰。 与其自己项目中重新编写这些小函数，不如花点时间熟悉`@vue/shared`包，引用其中的函数，这样质量更好更可靠。
:::

## **4 个常量值** {#CONST}

- `EMPTY_OBJ` = Object.freeze({})
- `EMPTY_ARR` = Object.freeze([])
- `NOOP` = () => {}
- `NO` = () => false

`EMPTY_OBJ,EMPTY_ARR` 通常用于参数解构时，充当相应属性不存在时的默认值。
`NOOP，NO` 通常也是赋值一个默认空函数。

```javascript
// 使用常量
function foo(list = EMPTY_ARR, option = EMPTY_OBJ) {
  // 下面2行,控制台会显示 error
  list.push(1)
  option.add = 'add'

  // 精细判断 list是入参还是默认值
  if (list === EMPTY_ARR) {
    // 用户未传入参
  }
}

foo()

// 不使用常量时
function bar(list = [], option = {}) {
  // 正常执行
  list.push(1)
  option.add = 'add'

  // 无法判断调用时的入参是否为默认值
  if (list === []) {
    // ......
  }
}
bar()
```

## **makeMap** {#makeMap}

通过传入`,`分隔的字符串，返回一个测试函数用于测试 val 是否在传入字符串中。

```javascript
// 是否为自闭合元素
const isSelfCloseTag = makeMap('img,br,hr,input,link')

isSelfCloseTag('div') // false
isSelfCloseTag('input') // true
```

## **PatchFlags**枚举值 {#PatchFlags}

`Patch flags`是编译器生成的优化提示信息。 当一个具有动态子元素的`block`进行 diff 时，如果有该标记信息，表明该 render 函数是编译生成的，diff 算法就可以按需更新这些标记。

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
  NEED_PATCH = 1 << 9, // 需要非props的patch, 比如： ref, onVNodeXXX hook变化时。
  DYNAMIC_SLOTS = 1 << 10, // 动态的slot，比如v-for 或 slot name是动态变量时。
  DEV_ROOT_FRAGMENT = 1 << 11, // 在dev模式时，用户在template的根下有注释时，才有该标记

  CACHED = -1, // 标记为：已被缓存静态的vnode
  BAIL = -2 // 退出优化模式。 比如在renderSlot() 创建的vnode 是非编译的，此时通常需要完整的diff。
}
```

## **PatchFlagNames** {#PatchFlagNames}

`DEV` 模式下才有的变量。一个数字为键值的对象，所有值都基本等同于上面的`PatchFlag`, 除了 `CACHED`。

```typescript
  {
    1: "TEXT",
    2: "CLASS",
    ...

    -1: 'HOISTED' // PatchFlag中的值为： CACHED
  }
```

## **ShapeFlags** {#ShapeFlags}

枚举值：节点的 Shape。当使用`createVNode`等函数创建了一个有效的`VNode`对象时，它的`shapeFlag属性`记录了该节点的类型，一个节点可以同时有多个 shape 值，比如 `ShapeFlags.ELEMENT | ShapeFlags.TEXT_CHILDREN`,表示当前节点是一个元素，且拥有一个文字节点。

```typescript
enum ShapeFlags {
  ELEMENT = 1,
  FUNCTIONAL_COMPONENT = 2,
  STATEFUL_COMPONENT = 4,
  TEXT_CHILDREN = 8,
  ARRAY_CHILDREN = 16,
  SLOTS_CHILDREN = 32,
  TELEPORT = 64,
  SUSPENSE = 128,
  COMPONENT_SHOULD_KEEP_ALIVE = 256,
  COMPONENT_KEPT_ALIVE = 512,
  COMPONENT = 6
}
```

## **SlotFlags** {#SlotFlags}

枚举值：插槽的类型。 todo: 补充该值的位置 \_, 用途。

```typescript
enum SlotFlags {
  STABLE = 1,
  DYNAMIC = 2,
  FORWARDED = 3
}
```

## **camelize** {#camelize}

将带 `-` 的字符串转小驼峰。内部通过正则替换，且自带缓存。

```typescript
const eventName = camelize('on-click') // onClick
```

## **def** {#def}

`Object.defineProperty` 函数的简写, 为对象添加不可枚举的属性。

```typescript
const obj = {}

def(obj, 'a', 42) // obj.a 不可枚举，不可变
def(obj, 'b', 64, true) // obj.b 不可枚举，可变
def(obj, Symbol('c'), 64, true) // obj.c symbol类型键值
```

## **escapeHtml** {#escapeHtml}

将` "'&<> ` 这5个特殊符号，简单的替换为对应的HTML转义后编码，防止跨站脚本攻击(XSS)。

```typescript
const ret1= escapeHtml(`a && b`) // a &amp;&amp; b
const ret2= escapeHtml(`<div>`) // &lt;div&gt;
```

## **escapeHtmlComment** {#escapeHtmlComment}

将HTML中的注释标记移除，只保留注释中间的内容。

```typescript
const ret= escapeHtmlComment('<!-- Comment 1 -->') // Comment 1

```

## **extend** {#extend}

`Object.assign`的简写。

```typescript

```

## **genPropsAccessExp** {#genPropsAccessExp}

在 Vue3.5 版本中，支持`响应式 Props 解构`语法糖的一个内部方法。在`@vue/compiler-core`中，用于生成解构后变量所对应的 Props 属性的引用。

```typescript
const { name } = defineProps(["name"]); // 编译后，所有name的地方替换为 "__props.name",不丢失响应性。

const name=genPropsAccessExp('name') // __props.name

```

## **generateCodeFrame** {#generateCodeFrame}

使在一些编译链工具，遇到语法错误时，经常会看到终端打印报错的代码行，并且在下方标 ^^^^ 等指示，这种功能为`CodeFrame`。本函数是就一个简易版的标记`CodeFrame`的方法。

```typescript
const source = 'const a b = 12'

const frame = generateCodeFrame(source, 6,9) // frame输出如下：
// 1  |  const a b = 12
//    |        ^^^
```

## **getEscapedCssVarName** {#getEscapedCssVarName}

xxx

```typescript

```

## **getGlobalThis** {#getGlobalThis}

xxx

```typescript

```

## **hasChanged** {#hasChanged}

xxx

```typescript

```

## **hasOwn** {#hasOwn}

xxx

```typescript

```

## **hyphenate** {#hyphenate}

xxx

```typescript

```

## **includeBooleanAttr** {#includeBooleanAttr}

xxx

```typescript

```

## **invokeArrayFns** {#invokeArrayFns}

xxx

```typescript

```

## **isArray** {#isArray}

xxx

```typescript

```

## **isBooleanAttr** {#isBooleanAttr}

xxx

```typescript

```

## **isBuiltInDirective** {#isBuiltInDirectivev}

xxx

```typescript

```

## **isDate** {#isDate}

xxx

```typescript

```

## **isFunction** {#isFunction}

xxx

```typescript

```

## **isGloballyAllowed** {#isGloballyAllowed}

xxx

```typescript

```

## **isGloballyWhitelisted** {#isGloballyWhitelisted}

xxx

```typescript

```

## **isHTMLTag** {#isHTMLTag}

xxx

```typescript

```

## **isIntegerKey** {#isIntegerKey}

xxx

```typescript

```

## **isKnownHtmlAttr** {#isKnownHtmlAttr}

xxx

```typescript

```

## **isKnownMathMLAttr** {#isKnownMathMLAttr}

xxx

```typescript

```

## **isKnownSvgAttr** {#isKnownSvgAttr}

xxx

```typescript

```

## **isMap** {#isMap}

xxx

```typescript

```

## **isMathMLTag** {#isMathMLTag}

xxx

```typescript

```

## **isModelListener** {#isModelListener}

xxx

```typescript

```

## **isObject** {#isObject}

xxx

```typescript

```

## **isOn** {#isOn}

xxx

```typescript

```

## **isPlainObject** {#isPlainObject}

xxx

```typescript

```

## **isPromise** {#isPromise}

xxx

```typescript

```

## **isRegExp** {#isRegExp}

xxx

```typescript

```

## **isRenderableAttrValue** {#isRenderableAttrValue}

xxx

```typescript

```

## **isReservedProp** {#isReservedProp}

xxx

```typescript

```

## **isSSRSafeAttrName** {#isSSRSafeAttrName}

xxx

```typescript

```

## **isSVGTag** {#isSVGTag}

xxx

```typescript

```

## **isSet** {#isSet}

xxx

```typescript

```

## **isSpecialBooleanAttr** {#isSpecialBooleanAttr}

xxx

```typescript

```

## **isString** {#isString}

xxx

```typescript

```

## **isSymbol** {#isSymbol}

xxx

```typescript

```

## **isVoidTag** {#isVoidTag}

xxx

```typescript

```

## **looseEqual** {#looseEqual}

xxx

```typescript

```

## **looseIndexOf** {#looseIndexOf}

xxx

```typescript

```

## **looseToNumber** {#looseToNumber}

xxx

```typescript

```

## **normalizeClass** {#normalizeClass}

xxx

```typescript

```

## **normalizeProps** {#normalizeProps}

xxx

```typescript

```

## **normalizeStyle** {#normalizeStyle}

xxx

```typescript

```

## **objectToString** {#objectToString}

xxx

```typescript

```

## **parseStringStyle** {#parseStringStyle}

xxx

```typescript

```

## **propsToAttrMap** {#propsToAttrMap}

xxx

```typescript

```

## **remove** {#remove}

xxx

```typescript

```

## **slotFlagsText** {#slotFlagsText}

xxx

```typescript

```

## **stringifyStyle** {#stringifyStyle}

xxx

```typescript

```

## **toDisplayString** {#toDisplayString}

xxx

```typescript

```

## **toHandlerKey** {#toHandlerKey}

xxx

```typescript

```

## **toNumber** {#toNumber}

xxx

```typescript

```

## **toRawType** {#toRawType}

xxx

```typescript

```

## **toTypeString** {#toTypeString}

xxx

```typescript

```
