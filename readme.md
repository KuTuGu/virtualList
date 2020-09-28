## 列表优化

- 无限滚动（Dom累计增加）  √
- 虚拟列表（Dom维持少量）  √
- 懒加载（请求优化）
  - 分页                √
  - 逐行（图片）

## 代码实现

- scroll监听，大量同步计算（考虑到IE降级处理）
- IntersectionObserver 异步监听视图交叉
- ResizeObserver 监听内容大小变化（懒加载）

## issues

- 当滑动框滑动太快时，IntersectionObserver不能精确触发问题：
  - IntersectionObserver是异步操作，基于requestIdleCallback实现，优先级较低。
    当执行用户高优先级事件交互（如滑动）时，有可能不触发requestIdleCallback执行，导致结果下方的内容完全空白。   
  - 解决思路：除IntersectionObserver外，执行另一个监听，当占位符完全进入视图后，强制触发更新