


#### 一问：Vue3 比 Vue2 好在哪，为什么要用 Vue3？
答：Vue3 相较于 Vue2 进行了许多改进和增强，提供了更高效、更现代化的开发体验。以下是 Vue3 相较于 Vue2 的一些主要优点和使用 Vue3 的必要性：
1.性能提升
Vue3 的性能相比 Vue2 有显著提升，主要体现在以下几个方面：
- 虚拟 DOM 优化：Vue3 对虚拟 DOM 做了更精细的优化，减少了 diff 算法的复杂度，提高了渲染性能。
- Tree-shaking: Vue3 对代码进行了优化，支持 tree-shaking（摇树优化）。这意味着如果你没有使用某些功能，打包时这些未使用的代码就不会被包含进最终的包中，从而减少了最终打包文件的大小。
- 更快的响应式系统：Vue3 引入了 Proxy 来替代 Vue2 中的 Object.defineProperty，使得 Vue3 的响应式系统更高效、更灵活，减少了性能开销。

2.Composition API(组合式API)
Vue3 引入了 Composition API，这是一个全新的方式来组织和复用代码。与 Vue2 中的 Options API（选项式API）相比，Composition API提供了更多的灵活性和可维护性，特别是在处理大型应用时。其主要优势包括：
- 更好的代码组织：Composition API 允许你在同一个组件内将逻辑分散在多个地方，而不是仅依赖于 data、methods、computed、watchers等选项。这样可以根据功能进行逻辑组织，避免了多层嵌套和重复代码。
- 增强的类型推导：Composition API 与 Typescript 的集成更加紧密，类型推导更加准确，开发者可以更容易地使用 Typescript。
- 更好的逻辑复用：使用 Composition API,你可以通过自定义的 Hooks（组合函数）将逻辑抽象出独立的函数，复用起来更加方便，减少代码的冗余。

3.更好的 Typescript 支持
Vue3 从一开始就进行了更深入的 Typescript 集成，相较于 Vue2 中的 Typescript 支持，Vue3 在类型推导、类型声明和类型检查等方面都得到了显著改善。开发者可以更轻松地利用 Typescript 进行开发，并且能够享受到更强大的类型支持和更精确的类型推导。

4.更简洁的生命周期钩子
Vue3 引入了新的生命周期钩子，优化了生命周期的命名，使得代码更加简洁和易于理解。例如，beforeDestroy和destroyed被统一为 beforeUnmount 和 unmounted，这些新名称更加符合浏览器的标准生命周期命名。

5.更好的开发工具
Vue3 提供了更强大的开发工具支持，包括 Vue Devtools 2.x（专门针对Vue3进行优化）。Vue Devtools 使得开发者能够更直观地查看应用的状态、组件树、响应式数据等，大大提高了调试和开发的效率。

6.Fragment、Teleport 和 Suspense
Vue3新增了一些非常实用的功能：
- Fragment: Vue3 支持组件返回多个根节点(Fragment)。这意味着你可以不再强制要求组件只能有一个根元素，使得组件结构更灵活。
- Teleport：Teleport 是一个新的内置组件，可以让你将子组件的 DOM 移动到页面的其他位置，通常用于模态框、通知等需要插入到页面其他位置的场景。
- Suspense: Suspense 是一种新的异步渲染机制，可以更优雅地处理异步数据加载，让开发者可以指定在数据加载完成前如何渲染组件，提升用户体验。

7.更好的响应式系统
Vue3 的响应式系统做了彻底的重构，基于Proxy实现，而不是 Vue2 中的Object.defineProperty。这使得Vue3在性能、灵活性和功能上都优于Vue2：
- Proxy 可以直接拦截对象的操作，允许更精细的控制，支持更多的操作，例如添加和删除对象属性，不再有‘数据劫持’的限制：Vue2 中，某些对象和数组的变更可能无法触发视图更新，而 Vue3 可以更好地处理这些场景。

8.更小的体积
Vue3 在很多地方做了优化，减少了代码的冗余，因此 Vue3 的包体积比 Vue2 更小。通过引入更先进的构建工具和方法，Vue3 可以在不牺牲功能的前提下，降低应用的体积。

9.支持更广泛的平台
Vue3不仅支持浏览器端开发，还支持服务器端渲染（SSR）、原生应用开发（通过 Vue Native）等多种平台。这样可以更方便地进行跨平台开发，尤其对于一些全栈开发者或者需要支持多种设备和平台的项目来说非常有用。

10.更长的支持和未来发展。
Vue3 是 Vue 官方未来发展的重点版本，Vue2 虽然仍然会获得维护，但未来的新特性和优化将主要集中在 Vue3 上。如果你选择Vue2，未来可能会错过很多新特性、工具和社区支持。

##### 总结：使用 Vue3 的必要性
- 性能优化：Vue3 的响应式系统、虚拟 DOM、Tree-shaking等多项性能提升，使得开发大型应用时更加高效。
- 更灵活的开发体验：Composition API 为开发者提供了更清晰和可维护的代码结构，特别适合大型和复杂的应用。
- 更好的 Typescript 支持：Vue3 与 Typescript 的集成更加完善，提升了类型安全和开发效率。
- 现代化的特性：如 Fragment、Teleport、Suspense等新特性提升了开发效率，并使得组件的功能更加强大。
- 长期维护和发展：Vue3 是未来 Vue 的发展方向，支持更长时间，能够享受更多社区贡献和工具支持。

因此，如果正在开发新的项目或者有计划迁移现有项目，选择 Vue3 会是更明智的选择。


#### 二问：Vue 中 nextTick的作用
在 Vue 中，nextTick是一个非常重要的工具，它允许我们在 DOM 更新后执行某些操作。简单来说，nextTick 让我们在 Vue 完成 DOM 更新后执行代码。它通常用于获取更新后的 DOM 状态或执行需要 DOM 更新后的逻辑。

###### nextTick 的作用
Vue 是一个响应式的框架，当响应式数据发生变化时，Vue 会异步地更新 DOM。这是为了提高性能，避免频繁的 DOM 操作导致不必要的渲染。由于这种一步更新的特性，某些时候我们需要确保在 DOM 更新后执行特定的代码，比如获取更新后的 DOM 元素、操作 DOM、执行某些与视图相关的任务等。

这时，nextTick 就派上了用场。它会将一个回调函数推入到下一个事件循环队列（即DOM更新后执行），确保代码在 DOM 更新后执行。

###### nextTick 的使用场景
1.获取更新后的 DOM 状态
如果在某个数据变化后需要获取更新后的 DOM 状态，nextTick 就可以确保获取到的是最新的 DOM。
2.执行依赖于 DOM 更新的操作
如果某些操作（如动画、滚动、焦点等）依赖于 DOM 的渲染结果，你需要等到 DOM 完全更新后再执行。这时，nextTick 就能确保代码在合适的时机执行。 例如某个输入框组件需要在某个DOM更新后进行聚焦。
3.与外部库配合使用
当与一些第三方库（例如JQuery、D3.js等）结合使用时，这些库通常依赖于 DOM 元素的存在或状态。因为 Vue 更新 DOM 是异步的额，如果在 Vue 更新 DOM 之前执行这些库的操作，可能会遇到 DOM 还没更新的情况。使用 nextTick 可以确保第三方库的操作发生在 Vue 更新之后，避免 DOM 操作出错。

##### nextTick 的工作原理
Vue的DOM更新是异步的，Vue会在事件循环的“微任务”队列中等待DOM更新完成后再执行回调。因此，nextTick会将你提供的回调推入到这个队列，并确保它在 Vue 完成 DOM 更新之后执行。

执行顺序：
1.Vue响应式数据发生变化，触发DOM更新。
2.Vue将更新操作加入到微任务队列。
3.微任务队列中的回调函数执行（虚拟DOM更新、差异计算等）（修改真实DOM的属性和获取，但是还没渲染到页面上）。
4.微任务执行完毕后，浏览器进行渲染，显示更新的DOM。

（常见理解误区，啊，数据修改触发的回调更新视图是放在微任务里的，nextTick的回调函数也是同一个微任务队列里的，nextTick怎么可以拿到渲染后DOM的属性的，其实DOM还没渲染，只是真实DOM树发生了修改）（同时，手动通过$ref去修改DOM属性的不会去放到Vue里面的异步更新DOM里面，因为异步更新DOM只针对由响应式数据修改后引发的视图修改，也就是patch diff）