## 插槽
		槽是组件的一块html模板，作为承载分发内容的出口。相当于把控制权交给了父组件，子组件该显示啥，在哪里显示，如何显示，都有父组件控制；组件与函数的功能类似，用于封装模块，以达到复用，及控制的作用。
		分为：默认（匿名）插槽、具名插槽、作用域插槽（slot\slot-scoped【2.6.0以后已废弃】，使用v-slot指令，但理念还是一样，哈哈～）。
		1. 具名插槽
		在子组件里面定义插槽加个标识 在父组件调用时候，自定义内容加上标识；这样就实现了在任意地方定义内容。父组件通过html模板上的slot属性关联具名插槽， 没有slot属性的htlm模板关联匿名插槽。
		2. 作用域插槽
		提供的组件带有一个可以从子组件获取数据的可复用的插槽，可以将所有和其上下文相关的数据传递给这个插槽

		#### 匿名插槽和具名插槽
    ```
    // 子组件 child-template
		<div class="container">
      <header>
        <slot name="header"></slot>
      </header>
      <main>
        <slot></slot>
      </main>
      <footer>
        <slot name="footer"></slot>
      </footer>
    </div>
    
    // 父组件
    <child-template>
		  <template slot="header">
		    I am header
		  </template>
      <span>匿名插槽</span>
      <template slot="footer">
		    I am footer
		  </template>
		</child-template>
    
    ** 2.6.0之后的写法，下面同是，把 `slot-scope` 改为 `v-slot` **
    <child-template>
		  <template v-slot:header>
		    I am header
		  </template>
      <span>匿名插槽</span>
      <template v-slot:footer>
		    I am footer
		  </template>
		</child-template>
    ```

		#### 作用域插槽
    ```
		// 子组件
		<ul>
		    <li
		        v-for="todo in todos"
		      	:key="todo.id"
		     >
		        <slot :todo="todo">
		        {{todo.txt}}
		        </slot>
		    </li>
		</ul>
    ```
    ```
		// 父组件
		<todo-list v-bind:todos="todos">
		  <!-- 将 `slotProps` 定义为插槽作用域的名字 -->
		  <template slot-scope="scope">
		    <!-- 为待办项自定义一个模板，-->
		    <!-- 通过 `slotProps` 定制每个待办项。-->
		    <span v-if="scope.todo.isComplete">✓</span>
		    {{ scope.todo.text }}
		  </template>
		</todo-list>

		// 或者使用解构
		<todo-list v-bind:todos="todos">
		  <!-- 将 `slotProps` 定义为插槽作用域的名字 -->
		  <template slot-scope="{todo}">
		    <!-- 为待办项自定义一个模板，-->
		    <!-- 通过 `slotProps` 定制每个待办项。-->
		    <span v-if="todo.isComplete">✓</span>
		    {{ todo.text }}
		  </template>
		</todo-list>
    ```
    详细内容可参考 官网：[slot](https://cn.vuejs.org/v2/guide/components-slots.html)
    以及之前的一篇blog [vue2.0 slot](https://www.cnblogs.com/136asdxxl/p/8337551.html)
