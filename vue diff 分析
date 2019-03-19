# vue Diff算法 

### 名词解释
#### 虚拟DOM （virtual DOM）就是在js中模拟DOM对象树来优化DOM操作的一种技术或思路。
#### DOM (document object model) 文档对象模型，在浏览器中可通过 js 来操作DOM
#### VNode可以理解为vue框架的虚拟dom的基类，通过 new 实例化得到 VNode对象
```
VNode对象 主要包括一下属性
	tag: 当前节点的标签名
	data: 当前节点的数据对象，具体包含哪些字段可以参考vue源码types/vnode.d.ts中对VNodeData的定义
	children: 数组类型，包含了当前节点的子节点
	text: 当前节点的文本，一般文本节点或注释节点会有该属性
	elm: 当前虚拟节点对应的真实的dom节点
	key: 节点的key属性，用于作为节点的标识，有利于patch的优化
	child: 当前节点对应的组件实例
	parent: 组件的占位节点
	isStatic: 静态节点的标识
	isRootInsert: 是否作为根节点插入，被<transition>包裹的节点，该属性的值为false
	isComment: 当前节点是否是注释节点
```

## 主要过程：patch -> patchVnode -> updateChildren
	
## 代码
	patch(oldVnode, vnode) {
		// 只有oldVnode, 销毁老节点
		if (oldVnode && !vnode) {
			api.invokeDestroyHook(oldVnode)
		}

		// 只有vnode, 新建节点
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

	function sameVnode(oldVnode, vnode) {
		if (oldVnode.key === vnode.key && oldVnode.elm === vnode.elm) {
			return true
		}
	}

	// 两节点比较
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
  
  ** 核心内容 **
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


  vue 源码中的详细的判断两个`vnode`是不是同一节点，需要同时具备以下条件：
  1、两个VNode的tag、key、isComment都相同
  2、同时定义或未定义data的时候
  3、若标签为input则type必须相同
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
  
  一些主要的 DOM API 操作，参考aooy的代码，实际上是DOM API 的封装
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
  
## 其他 （react & vue diff区别）

	 react是批量更新Component，做完整个Diff之后再做DOM操作。
	 Vue是即时移动或操作DOM，需要两个数组维护startIndex 和 endIndex
 
 
 ** 参考：[https://github.com/aooy/blog/issues/2](https://github.com/aooy/blog/issues/2)**
 
 ** ps: 部分注释是根据自己的理解，如有不对，请指出，谢谢指教 **
