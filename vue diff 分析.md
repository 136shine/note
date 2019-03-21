# vue Diff算法 
传统的diff算法，为了找到最小变化，需要逐层的去搜索（深度优先遍历）比较，时间复杂度将会达到 O(n^3)的级别，代价十分高，考虑到节点变化很少是跨层次的，vue2.0借鉴[snbbdom](https://github.com/snabbdom/snabbdom) (一个高效、简单的Virtual Dom library)，只比较同层节点；如果不同，那么即使该节点的子节点没变化，我们也不复用，直接将从父节点开始的子树全部删除，然后再重新创建节点添加到新的位置。如果父节点没变化，我们就比较所有同层的子节点，对这些子节点进行删除、创建、移位操作。

### 名词解释
 1. `虚拟DOM` （virtual DOM）就是在js中模拟DOM对象树来优化DOM操作的一种技术或思路。
 2. `DOM` (document object model) 文档对象模型，在浏览器中可通过 js 来操作DOM
 3. `VNode` 可以理解为vue框架的虚拟dom的基类，通过 new 实例化得到 VNode对象；VNode 对真实DOM的一层抽象，以JavaScript对象描述真实DOM

VNode对象 主要包括一下属性:
* `tag`: 当前节点的标签名
* `data`: 当前节点的数据对象，具体包含哪些字段可以参考vue源码types/vnode.d.ts中对VNodeData的定义
* `children`: 数组类型，包含了当前节点的子节点
* `text`: 当前节点的文本，一般文本节点或注释节点会有该属性
* `elm`: 当前虚拟节点对应的真实的dom节点
* `key`: 节点的key属性，用于作为节点的标识，有利于patch的优化
* `child`: 当前节点对应的组件实例
* `parent`: 组件的占位节点
* `isStatic`: 静态节点的标识
* `isRootInsert`: 是否作为根节点插入，被<transition>包裹的节点，该属性的值为false
* `isComment`: 当前节点是否是注释节点
具体可参考源码[vue 中的 Vnode](https://github.com/vuejs/vue/blob/dev/src/core/vdom/vnode.js) （ps: snabbdom 中的 Vnode 更简单，没有vue 中的那没多属性 https://github.com/snabbdom/snabbdom/blob/master/src/vnode.ts)


**主要过程：patch -> patchVnode -> updateChildren**
	
## 代码分析
patch 判断两vnode是否为同一节点，需要深层次比对，若是进行深度的比较，得出最小差异，否则直接删除旧有DOM节点，创建新的DOM节点
``` javascript
	patch(oldVnode, vnode) {
		// 只有oldVnode, 销毁老节点
		if (oldVnode && !vnode) {
			api.invokeDestroyHook(oldVnode)
		}

		// 只有vnode, 新建节点 （首次渲染）
		if (!oldVnode && vnode) {
			var oEl = oldVnode.el
			var parentEle = api.parentNode(oEl)
			createEle(vnode)
			if(parentEle !== null) {
				api.insertBefore(parentEle, vnode)
			}
		}
    // 同时存在
		if (oldVnode && vnode) {
			if(sameVnode(oldVnode, vnode)){
        // 两vnode 为同一节点，则进行patchVnode比较
				patchVnode(oldVnode, vnode)
			} else {
        // 不是一节点，则新建vnode并插入到父节点中，同时删除oldVnode
				var oEl = oldVnode.el
				var parentEle = api.parentNode(oEl)
				createEle(vnode)
				if(parentEle !== null) {
					api.insertBefore(parentEle, vnode, api.nextSibling(oEl))
					api.removeChild(parentEle, oldVnode)
					oldVnode = null
				}
			}
		}

		return vnode
	}

```

  vue 源码中的详细的判断两个`vnode`是不是同一节点，需要同时具备以下条件：
  1. 两个VNode的tag、key、isComment都相同
  2. 同时定义或未定义data的时候
  3. 若标签为input则type必须相同
  ``` javascript
	function sameVnode (a, b) {
	  return (
	    a.key === b.key && (
	      (
	        a.tag === b.tag &&
	        a.isComment === b.isComment &&
	        isDef(a.data) === isDef(b.data) &&
	        sameInputType(a, b)
	      ) || (
	        isTrue(a.isAsyncPlaceholder) &&
	        a.asyncFactory === b.asyncFactory &&
	        isUndef(b.asyncFactory.error)
	      )
	    )
	  )
	}
  
     function sameInputType (a, b) {
	  if (a.tag !== 'input') return true
	  let i
	  const typeA = isDef(i = a.data) && isDef(i = i.attrs) && i.type
	  const typeB = isDef(i = b.data) && isDef(i = i.attrs) && i.type
	  return typeA === typeB || isTextInputType(typeA) && isTextInputType(typeB)
     }
  ```
  
 【snabbdom】
`sameVnode` 比较两节点是否相同（相似）
``` javascript
  function sameVnode(oldVnode, vnode) {
	if (oldVnode.key === vnode.key && oldVnode.elm === vnode.elm) {
		return true
	}
  }
```

 `createKeyToOldIdx` 将oldvnode数组中位置对oldvnode.key的映射转换为oldvnode.key
对位置的映射（生成oldvnode.key对位置的映射）
``` javascript
	function createKeyToOldIdx(children, beginIdx, endIdx) {
	  var i, map = {}, key;
	  for (i = beginIdx; i <= endIdx; ++i) {
	    key = children[i].key;
	    if (isDef(key)) map[key] = i;
	  }
	  return map;
	}
 ```
  

两节点比较
patchVnode的规则:
1. 如果新旧VNode都是静态的，同时它们的key相同（代表同一节点），并且新的VNode是clone或者是标记了once（标记v-once属性，只渲染一次），那么只需要替换elm以及componentInstance即可。
2. 新老节点均有children子节点，则对子节点进行diff操作，调用updateChildren，这个updateChildren也是diff的核心。
3. 如果老节点没有子节点而新节点存在子节点，先清空老节点DOM的文本内容，然后为当前DOM节点加入子节点。
4. 当新节点没有子节点而老节点有子节点的时候，则移除该DOM节点的所有子节点。
5. 当新老节点都无子节点的时候，只是文本的替换。
``` javascript

     function patchVnode(oldVnode, vnode) {
	var oldCh = oldVnode.children
	var ch = vnode.children

	if (oldVnode === vnode) {
	    return
	}

	if (oldVnode.text !== null && vnode.text !== null && oldVnode.text !== vnode.text) {
        	// 两节点为文本节点，且不相同，则进行替换
	        api.setTextContent(el, vnode.text)
	  } else {
        	// 进一步判断子节点
	      	updateEle(el, vnode, oldVnode)
	    if (oldCh && ch && oldCh !== ch) {
          	// 两节点都有子节点，进行 updateChildren，判断子节点是否发生改变，找到两节点之间的差异
		updateChildren(el, oldCh, ch)
	    }else if (ch){
		// 只有vnode有子节点，新建该子节点
		createEle(vnode) //create el's children dom
	    }else if (oldCh){
		// 只有oldVnode有子节点，直接删除该子节点
		api.removeChildren(el)
	    }
	  }
	}
```

 ** 核心内容 updateChildren **  
 
 主要分为设置key和不设置key两种方式：
  1. 不设key，newCh和oldCh只会进行头尾两端的相互比较
  2. 设key，从用key生成的对象oldKeyToIdx中查找匹配的节点，通过查找、移动节点以更高效的利用dom，不会出现不必要的删除和新建节点  
  当头尾两端没查找到与`newStartIdx`相匹配的节点，则为oldCh节点设置key      

``` javascript
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
            if (oldStartVnode == null) {   //当为oldvnode设置key，对于vnode.key的比较，会把oldVnode = null，对应 此处 oldCh[idxInOld] = null
                oldStartVnode = oldCh[++oldStartIdx] 
            }else if (oldEndVnode == null) {
                oldEndVnode = oldCh[--oldEndIdx]
            }else if (newStartVnode == null) {
                newStartVnode = newCh[++newStartIdx]
            }else if (newEndVnode == null) {
                newEndVnode = newCh[--newEndIdx]
            }else if (sameVnode(oldStartVnode, newStartVnode)) {
            	// oldStartVnode & newStartVnode 为同一节点，可以patch
                patchVnode(oldStartVnode, newStartVnode)
                oldStartVnode = oldCh[++oldStartIdx]
                newStartVnode = newCh[++newStartIdx]
            }else if (sameVnode(oldEndVnode, newEndVnode)) {
            	// oldEndVnode & newEndVnode 为同一节点，可以patch
                patchVnode(oldEndVnode, newEndVnode)
                oldEndVnode = oldCh[--oldEndIdx]
                newEndVnode = newCh[--newEndIdx]
            }else if (sameVnode(oldStartVnode, newEndVnode)) {
            	// oldStartVnode & newEndVnode 为同一节点，可以patch，并把oldStartVnode移到oldEndVnode后面
                patchVnode(oldStartVnode, newEndVnode)
                api.insertBefore(parentElm, oldStartVnode.el, api.nextSibling(oldEndVnode.el))
                oldStartVnode = oldCh[++oldStartIdx]
                newEndVnode = newCh[--newEndIdx]
            }else if (sameVnode(oldEndVnode, newStartVnode)) {
            	// oldEndVnode & newStartVnode 为同一节点，可以patch，并把oldEndVnode移到oldStartVnode前面
                patchVnode(oldEndVnode, newStartVnode)
                api.insertBefore(parentElm, oldEndVnode.el, oldStartVnode.el)
                oldEndVnode = oldCh[--oldEndIdx]
                newStartVnode = newCh[++newStartIdx]
            }else {
               // 使用key时的比较
                if (oldKeyToIdx === undefined) {
                    oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx) // 有key生成index表 map {key [children.key]: index [该节点在children中的索引]}
                }
                idxInOld = oldKeyToIdx[newStartVnode.key] // 返回 newStartVnode 节点在 oldVnode.children 中的索引index
                if (!idxInOld) {
                	// 在 oldVnode 的 children 中没有找到与 newStartVnode 为同一节点的节点，新建 newStartVnode 节点并插入到 oldStartVnode 之前
                    api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el)
                    newStartVnode = newCh[++newStartIdx]
                }
                else {
                	// 在 oldVnode 的 children 中找到与 newStartVnode 为同一节点的节点
                    elmToMove = oldCh[idxInOld]
                    if (elmToMove.sel !== newStartVnode.sel) {
                        api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el)
                    }else {
                    	// 在 oldVnode 的 children 中找到与 newStartVnode 为同一节点的节点，并且节点选择器相同
                        patchVnode(elmToMove, newStartVnode)
                        oldCh[idxInOld] = null // 在 oldVnode.children 中存在与 newStartVnode 对应的，把对应的 oldVnode 设置为 null，防止，遍历完后被误删除
                        api.insertBefore(parentElm, elmToMove.el, oldStartVnode.el)
                    }
                    newStartVnode = newCh[++newStartIdx]
                }
            }
        }

        // 遍历完成
        if (oldStartIdx > oldEndIdx) {
        	// oldVnode.children 先遍历完成，添加多余的 newVnode.children 节点
            before = newCh[newEndIdx + 1] == null ? null : newCh[newEndIdx + 1].el // 插入的位置
            addVnodes(parentElm, before, newCh, newStartIdx, newEndIdx)
        }else if (newStartIdx > newEndIdx) {
        	// newVnode.children 先遍历完成，多余的 oldVnode.children 节点删除掉
            removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
        }
}
```
  
  一些主要的 DOM API 操作，参考aooy的代码，实际上是DOM API 的封装，具体可参考 `snabbdom/vue`中关于patch部分 dom 操作API
  ``` javascript
	api.createElement = function (tag) {
		return document.createElement(tag)
	} 
	api.createTextNode = function (txt) {
		return document.createTextNode(txt)
	}
	api.appendChild = function (parent, child) {
		return parent.appendChild(child)
	}
	api.parentNode = function (node) {
		return node.parentNode
	}
	api.insertBefore = function (parent, newNode, rf) {
		return parent.insertBefore(newNode, rf)
	}
	api.nextSibling = function (el) {
		return el.nextSibling
	}
	api.removeChild = function (parent, rc) {
		return parent.removeChild(rc)
	}
	api.setTextContent = function (ele, txt) {
		ele.textContent = txt
	}

	function createEle (vdom) {
		let i, e 
		if( !vdom.el && (i = vdom.text)) {
			e = vdom.el = api.createTextNode(i)
			return vdom
		} 
		if ( (i = vdom.tagName) && vdom.el === null) {
			e = vdom.el = api.createElement(i)
		}else if (vdom.el.nodeType === 1) {
			e = vdom.el
		}
		updateEle(e, vdom)
		return vdom
	}

	function updateEle (e ,vdom, oldVdom) {
		let i
		if( (i = vdom.className).length > 0 ) api.setClass(e, i)
		if( (i = vdom.data) !== null ) api.setAttrs(e, i)
		if( (i = vdom.id) !== null ) api.setId(e, i)
		if( (i = vdom.children) !== null && !oldVdom) api.appendChildren(e, i)
	}
  ```
  
## 总结
### 详细流程图（自己画的）
  ![diff-flow](/img/vue-diff.png)
  
### 整体流程概述
Vue通过数据绑定来修改视图，当某个数据被修改的时候，set方法会让闭包中的Dep调用notify通知所有订阅者Watcher，Watcher通过get方法执行`vm._update(vm._render(), hydrating)`来更新视图。  

`vm._render()`会生成新的VNode，在内部会将该VNode对象与之前旧的VNode对象进行`__patch__`

patch将新老VNode节点进行比对，然后将根据两者的比较结果进行最小单位地修改视图，而不是将整个视图根据新的VNode重绘。patch的核心在于diff算法，这套算法可以高效地比较virtual DOM的变更，得出变化以修改视图。  

#### patch 过程  
当oldVnode与vnode在sameVnode的时候才会进行patchVnode，也就是新旧VNode节点判定为同一节点的时候才会进行patchVnode这个过程，否则就是创建新的DOM，移除旧的DOM。 

#### patchVnode 过程
1. 如果新旧VNode都是静态的，同时它们的key相同（代表同一节点），并且新的VNode是clone或者是标记了once（标记v-once属性，只渲染一次），那么只需要替换elm以及componentInstance即可。
2. 新老节点均有children子节点，则对子节点进行diff操作，调用updateChildren，这个updateChildren也是diff的核心。
3. 如果老节点没有子节点而新节点存在子节点，先清空老节点DOM的文本内容，然后为当前DOM节点加入子节点。
4. 当新节点没有子节点而老节点有子节点的时候，则移除老节点（当前节点）的所有子节点。
5. 当新老节点都无子节点的时候，只是文本的替换。  

#### updateChildren 过程
**对子节点进行diff，分为设置key和不设置key两种情况**  

概括为：oldCh和newCh分别设置两个头尾的变量StartIdx和EndIdx，它们的2个变量相互比较，一共有2x2=4种比较方式。如果4种比较都没匹配，如果设置了key，就会用key进行比较，在比较的过程中，变量从两端向中间移动，一旦StartIdx>EndIdx表明oldCh和newCh至少有一个已经遍历完了，就会结束比较。  

具体比对有以下几种情况：
1. 若 sameVNode(newStartVnode, oldStartVnode)，patchVnode(newStartVnode, oldStartVnode)，同时newStartIdx, oldStartIdx 指针分别后移+1
2. 若 sameVNode(newEndVnode, oldEndVnode)，patchVnode(newEndVnode, oldEndVnode)，同时newEndIdx, oldEndIdx 指针分别前移-1
3. 若 sameVNode(newStartVnode, oldEndVnode)，patchVnode(newStartVnode, oldEndVnode)，同时oldEndVnode 移动到 oldStartVnode 之前，同时 oldEndIdx--，newStartIdx++
4. 若 sameVNode(newEndVnode, oldStartVnode)，patchVnode(newEndVnode, oldStartVnode)，同时oldStartVnode移动到 oldEndVnode 之后， oldStartIdx++，newEndIdx--
5. 以上都不符合，判断是否有为oldCh设置key，若没有，则设置，newStartIdx从中找出具有相同key的节点，找到后，进行patchVnode，否则生成新的节点


最后简单描述：
**************************
1. patch只需要对两个vnode进行判断是否相似，如果相似，则对他们进行
patchVnode操作，否则直接用vnode替换oldvnode；
2. patchVnode进一步比较两个vnode，如果两节点都有子节点，则对他们的子节点进行updateChildren操作，对这些子节点进行删除、创建、移位操作，否则新建新子节点，或者是删除已有的子节点；
3. updateChildren 通过创建四个新旧节点的头尾索引，从新旧子节点头尾两端向中间推进对比，若相同，则进行patchVnode
**************************
  
  
## 其他 （react & vue diff区别）
 react是批量更新Component，做完整个Diff之后再做DOM操作；  
 
 Vue是即时移动或操作DOM，需要两个数组维护startIndex 和 endIndex
 
 
 **参考：
 [https://github.com/aooy/blog/issues/2](https://github.com/aooy/blog/issues/2)  
 [VirtualDOM与diff(Vue实现).MarkDown](https://github.com/answershuto/learnVue/blob/master/docs/VirtualDOM%E4%B8%8Ediff(Vue%E5%AE%9E%E7%8E%B0).MarkDown)
 **
 
 **注意: 该文中的代码，不完全是vue 源码，是参考其他代码改动，可以和[源码](https://github.com/vuejs/vue/blob/dev/src/core/vdom/patch.js)一起看，文中注释是根据自己的理解添加的，如有不对，请指出，谢谢指教**
