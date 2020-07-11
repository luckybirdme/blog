---
title: 虚拟 DOM
date: 2020-06-27 11:11:52
tags: vuejs
---

> 虚拟 DOM

<!-- more -->


## 一. 虚拟 DOM

1. HTML 最终会解析成 DOM 树，当要修改 DOM 树的节点时，需要快速定位到节点，然后进行增删改的操作。
2. 操作节点少的 DOM 树时，上述过程依然能快速完成，但是成千上万个节点的 DOM 树，增删改的性能会下降，代价变高。
3. 虚拟 DOM 是将 DOM 树转化成 JS 对象，然后在 JS 中对 DOM 进行操作，由于 JS 计算相对便宜，所以更容易接受。
4. 另外，虚拟 DOM 利用 JS 进行 UI 编写，也可以利用 JS 实现跨平台交互，比如React Native开发的 UI 可兼容 Web 和 APP，包括 Android 和 IOS。

```html
<!-- 真实 DOM -->
<div>
    <p>123</p>
</div>

```

```js
// 虚拟 DOM
var Vnode = {
    tag: 'div',
    children: [
        { tag: 'p', text: '123' }
    ]
};

```


## 二. diff 算法

由于虚拟 DOM 通过 JS 完成 DOM 的渲染，那么判断 DOM 变化的 diff 算法显得尤为重要，是提升虚拟 DOM 的渲染性能的关键所在。
下面以 Vue 2 为例子，分析下 diff 算法，它是基于[snabbdom](https://www.npmjs.com/package/snabbdom)改编而成的。

1. diff 比较只是针对同层级节点，跨层级节点对比没有意思

![](/img/2020/vuejs_virtual_dom_1.png)


2. vue 的 diff 位于 patch.js 文件中，diff 的过程就是调用 patch 函数，就像打补丁一样修改真实 dom。

```js

function patch (oldVnode, vnode) {
	// 是否相同的节点结构
    if (sameVnode(oldVnode, vnode)) {
    	// 进行 diff 比较节点差异，然后进行打补丁
        patchVnode(oldVnode, vnode)
    } else {
        const oEl = oldVnode.el
        let parentEle = api.parentNode(oEl)
        // 创建新节点
        createEle(vnode)
        if (parentEle !== null) {
        	// 插入新节点
            api.insertBefore(parentEle, vnode.el, api.nextSibling(oEl))
            // 删除旧节点
            api.removeChild(parentEle, oldVnode.el)
            oldVnode = null
        }
    }
    return vnode
}
```

3. sameVnode 函数看这两个节点是否值得比较，两个vnode的key和sel相同才去比较它们，比如p和span，div.classA和div.classB都被认为是不同结构而不去比较它们。

```js
function sameVnode(oldVnode, vnode){
    return vnode.key === oldVnode.key && vnode.sel === oldVnode.sel
}
```

4. 对于不同结构的节点，那就不用 diff 了，直接创建新节点，然后插入到虚拟 DOM 中，最后再把旧节点删除

```js
const oEl = oldVnode.el
let parentEle = api.parentNode(oEl)
createEle(vnode)
if (parentEle !== null) {
    api.insertBefore(parentEle, vnode.el, api.nextSibling(oEl))
    api.removeChild(parentEle, oldVnode.el)
    oldVnode = null
}
```

5. patchVnode 会判断新节点和旧节点的子节点是否存在，如果新节点不存在子节点，那么删除旧节点的子节点；如果旧节点不存在子节点，那么新增子节点。

```js
function patchVnode (oldVnode, vnode) {
    const el = vnode.el = oldVnode.el
    let i, oldCh = oldVnode.children, ch = vnode.children
    if (oldVnode === vnode) return
    // 如果只是文本不一样
    if (oldVnode.text !== null && vnode.text !== null && oldVnode.text !== vnode.text) {
        api.setTextContent(el, vnode.text)
    }else {
    	// 更新节点的属性，比如 data class 之类的
        updateEle(el, vnode, oldVnode)
        // 如果新节点和旧节点都存在子节点，那么开始对比差异，并更新子节点 
        if (oldCh && ch && oldCh !== ch) {
            updateChildren(el, oldCh, ch)
        // 新节点存在子节点，但旧节点不存在子节点，所以需要增加新子节点
        }else if (ch){
            createEle(vnode) 
        // 旧节点存在子节点，但是新节点不存在子节点，所以需要删除旧子节点
        }else if (oldCh){
            api.removeChildren(el)
        }
    }
}
```

6. updateChildren 是 vue diff 算法的核心，也是最复杂的地方

```js
updateChildren (parentElm, oldCh, newCh) {
    let oldStartIdx = 0, newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    let oldStartVnode = oldCh[0]
    let oldEndVnode = oldCh[oldEndIdx]
    let newEndIdx = newCh.length - 1
    let newStartVnode = newCh[0]
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx
    let idxInOld
    let elmToMove
    let before
    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
        if (oldStartVnode == null) {   // 对于vnode.key的比较，会把oldVnode = null
            oldStartVnode = oldCh[++oldStartIdx] 
        }else if (oldEndVnode == null) {
            oldEndVnode = oldCh[--oldEndIdx]
        }else if (newStartVnode == null) {
            newStartVnode = newCh[++newStartIdx]
        }else if (newEndVnode == null) {
            newEndVnode = newCh[--newEndIdx]
        }else if (sameVnode(oldStartVnode, newStartVnode)) {
            patchVnode(oldStartVnode, newStartVnode)
            oldStartVnode = oldCh[++oldStartIdx]
            newStartVnode = newCh[++newStartIdx]
        }else if (sameVnode(oldEndVnode, newEndVnode)) {
            patchVnode(oldEndVnode, newEndVnode)
            oldEndVnode = oldCh[--oldEndIdx]
            newEndVnode = newCh[--newEndIdx]
        }else if (sameVnode(oldStartVnode, newEndVnode)) {
            patchVnode(oldStartVnode, newEndVnode)
            api.insertBefore(parentElm, oldStartVnode.el, api.nextSibling(oldEndVnode.el))
            oldStartVnode = oldCh[++oldStartIdx]
            newEndVnode = newCh[--newEndIdx]
        }else if (sameVnode(oldEndVnode, newStartVnode)) {
            patchVnode(oldEndVnode, newStartVnode)
            api.insertBefore(parentElm, oldEndVnode.el, oldStartVnode.el)
            oldEndVnode = oldCh[--oldEndIdx]
            newStartVnode = newCh[++newStartIdx]
        }else {
           // 使用key时的比较
            if (oldKeyToIdx === undefined) {
                oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx) // 有key生成index表
            }
            idxInOld = oldKeyToIdx[newStartVnode.key]
            if (!idxInOld) {
                api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el)
                newStartVnode = newCh[++newStartIdx]
            }
            else {
                elmToMove = oldCh[idxInOld]
                if (elmToMove.sel !== newStartVnode.sel) {
                    api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el)
                }else {
                    patchVnode(elmToMove, newStartVnode)
                    oldCh[idxInOld] = null
                    api.insertBefore(parentElm, elmToMove.el, oldStartVnode.el)
                }
                newStartVnode = newCh[++newStartIdx]
            }
        }
    }
    if (oldStartIdx > oldEndIdx) {
        before = newCh[newEndIdx + 1] == null ? null : newCh[newEndIdx + 1].el
        addVnodes(parentElm, before, newCh, newStartIdx, newEndIdx)
    }else if (newStartIdx > newEndIdx) {
        removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
    }
}

```

这里讲解下代码的主要逻辑：

- 为了优化 diff 过程，优先判断新节点和旧节点的头部和尾部，如果头部相同，则向中间移动，如果尾部相同，也向中间移动；所以最后剩下的就是中间不同的子节点。
- 对于中间不同的子节点，旧节点的头部 oldStartIdx 和新节点的尾部 newEndIdx 比较，如果相同，则把旧节点的头部节点 oldStartIdx 移到 oldEndIdx 后面；旧节点的尾部 oldEndIdx 和新节点的头部 newStartIdx 比较，如果相同，则把旧节点的尾部 oldEndIdx 移到 oldStartIdx 后面
- 以此类推，一直到新节点或旧节点中某一个为空，说明比较结束了，然后处理多余的子节点，如果是旧节点还有，则删除；如果新节点还有，则增加。



![](/img/2020/vuejs_virtual_dom_2.png)
