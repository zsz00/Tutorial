
https://my.oschina.net/jxcdwangtao/blog/1585363  TensorFlow Serving在Kubernetes中的实践


# Tensorflow serving

粗略测试了下Tensorflow serving  

* 支持模型版本管理和热更新
* 支持batch模式
* 使用grpc作为rpc协议，速度比thrift快

使用中发现了一些问题

* 模型需要进行一次转换后才能使用，转换代码需要根据网络模型作调整
* batching 相关的参数调整对性能影响有限，并发数增大后（100左右），经常会达到预设的超时（10s）而失去响应
* batching设计比较复杂，实现大体上也是用的queue加超时机制控制
* 示例为python调用，golang调用需要通过参照原始protobuf文件编写（参照github上一个deep-recommend-system的项目）

---


## ShentuBackend using Tensorflow

为ShentuBackend(公司sdk)增加了Tensorflow支持

主要使用到了

* 加载 freezed graph with weights
  - python脚本训练的checkpoint，可以通过官方的graph_util 将variable转换constant，并将graph meta和权重合并为一个pb文件
  - 但是graph_util转换函数发现对BatchNorm转换有bug（转换人脸模型时发现），需要手动修正
  - slim 的模型转换时需要加上scope，否则会找不到对应node
* opencv mat 转换为 tensor
	- C++ API 文档内未找到，发现底层也是float* copy下就行了
* 模型预测
   - 比较奇特的问题是输入层的shape即使有指定，保存再读取后就无法取得了，所以要判断输入是否正确只能靠外界指定shape了

---

nhwc
