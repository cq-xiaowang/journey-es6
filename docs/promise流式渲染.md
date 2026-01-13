:::mermaid
graph TD
    A[初始化] --> B[开始加载图片]
    B --> C{对于每一个图片URL}
    C -->|是| D[并行发起请求以加载图片]
    D --> E{请求是否成功?}
    E -->|否| F{达到最大重试次数?}
    F -->|是| G[处理失败情况]
    F -->|否| D
    E -->|是| H[生成Blob URL]
    H --> I[调用fillNextSlot函数]
    I --> J[更新nextSlotIndex]
    J --> K[增加completedCount计数]
    K --> L{所有图片都已处理完毕?}
    L -->|否| C
    L -->|是| M[停止监控]
    G --> K
:::
初始状态（预创建）：
```
<div class="img-container">
  <div class="img-slot"><div class="loading">等待图片...</div></div>
  <div class="img-slot"><div class="loading">等待图片...</div></div>
  <div class="img-slot"><div class="loading">等待图片...</div></div>
  <!-- ... 共 N 个 -->
</div>
```

当第一张图加载成功后（调用 fillNextSlot）：
```
<div class="img-container">
  <div class="img-slot"><img src="blob:..."></div>   
  <!-- ✅ 被替换 -->
  <div class="img-slot"><div class="loading">等待图片...</div></div>
  <div class="img-slot"><div class="loading">等待图片...</div></div>
  <!-- ... -->
</div>
```

怎么保证blob插入到正确位置呢?
| 机制         | 作用
|--------------|-------------
| 预创建`slots`数组 | 提供固定、有序的DOM容器
| 全局 `nextSlotIndex` | 标记下一个可用位置
| JS单线程执行         | 保证 `index`读写原子性，无竞争
| `replaceChildren`    | 只更新内容，不改变容器位置
| 越界检查              | 防止异常情况
