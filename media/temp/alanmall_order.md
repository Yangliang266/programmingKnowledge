```
context
AbsTransHandlerContext
CreateOrderContext
TransHandlerContext


conveter
CreateOrderConveter
TransConvert


handler
AbstactTransHandler
ValidateHandler
ClearCartItemHandler
InitOrderHandler
LogisticalHandler
SubStockHandler
TransHandler
DefaultTransPipeline
TransHandlerNode
TransOutboundInvoker
TransPipeline

factory
AbstractTransPipelineFactory
OrderProcessTransPipeline
TransPipelineFactory
```





factory.build(request);

工厂

- build 方法

关联

- pipeline
  - head
  - tail
  - add方法
- transnode
  - next 链式组件
  - exec 开始组件
    - handler 开始组件
  - 停止组件





invoke.start()

执行单位开始

- 各个执行单位

关联器开始

- 依赖各个执行单位

总开关开始

- 依赖关联器