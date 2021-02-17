

# 相关文档

[Unity ML-Agents Toolkit Documentation](https://github.com/Unity-Technologies/ml-agents/blob/release_12_docs/docs/Readme.md)

[Installation](https://github.com/Unity-Technologies/ml-agents/blob/release_12_docs/docs/Installation.md)

[Getting Started Guide](https://github.com/Unity-Technologies/ml-agents/blob/release_12_docs/docs/Getting-Started.md)

[Making a New Learning Environment](https://github.com/Unity-Technologies/ml-agents/blob/release_12_docs/docs/Learning-Environment-Create-New.md)

# 配置环境

1. 安装[anaconda](https://www.anaconda.com/products/individual#Downloads)
2. 创建一个环境（需要python3.6.1或3.7，官网下的是3.8）用`conda create -n ml-agents python=3.7`的方式创建。
3. 用`conda env list`来看所有的环境，也可以用界面
4. `activte <环境名>`来激活环境，也可以用界面
5. `pip3 install torch==1.7.0 -f https://download.pytorch.org/whl/torch_stable.html`，失败的话可以试多几次，实在不行用镜像
6. `pip3 install mlagents`
7. 下载ml-agents，可以用git，也可以直接下

# 官方示例

1. 查看官方示例
2. 试着训练官方示例
3. cd到当前目录
4. `mlagents-learn config/ppo/3DBall.yaml --run-id=first3DBallRun` 来开始训练，`mlagents-learn`相当于是指令的名称，后面是参数，`config/ppo/3DBall.yaml`是配置文件的路径，`--run-id=<名称>`，名称是本次训练的ID
5. `Ctrl+C`马上终止
6. 结果在Result中，onnx后缀的就是
7. 去项目中替换掉即可运行

# 做一个简单的Demo

可以直接在官方的项目中做，也可以新建一个。

==PackageManager== 中下载的ML-Agents应该是版本有点旧或者什么别的原因，东西不太对。所以可以直接复制案例中下来的那个package。

1. 搭建场景（小球、目标、地板）
2. 给Agent添加脚本

```csharp
// 所需的命名空间
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;
```

类改成继承自`Agent`


```csharp
// 每一次开始的时候调用此函数
public override void OnEpisodeBegin() {}
    
// 收集输入数据的时候调用此函数
public override void CollectObservations(VectorSensor sensor) {}

// 输出计算结果的时候调用此函数
public override void OnActionReceived(ActionBuffers actionBuffers) {}
```
**定义需要的一些变量**

```csharp
// 刚体
private Rigidbody _rBody;
// 目标方块的Transform
public Transform target;
// 移动时候力度的大小
public float forceMultiplier = 10;

void Start () {
    // 获取刚体组件
    _rBody = GetComponent<Rigidbody>();
}
```

**重写开始的函数**

```csharp
public override void OnEpisodeBegin()
{
    // 如果小球的高度低于0（既掉下地面），则将小球的位置重置到中心，速度清零
    if (transform.localPosition.y < 0)
    {
        // 角速度设置为0（零向量）
        _rBody.angularVelocity = Vector3.zero;
        // 速度设置为0（零向量）
        _rBody.velocity = Vector3.zero;
        // 位置为中心点上方一点点（是localPosition）
        transform.localPosition = new Vector3( 0, 0.5f, 0);
    }

    // 随机更改目标方块的位置
    target.localPosition = new Vector3(
        Random.value * 8 - 4,
        0.5f,
        Random.value * 8 - 4);
    /*Random.value生成一个0到1的随机数，Random.value * 8 - 4相当于生成一个-4到4的随机数*/
    /*也可以Random.Range(-4f,4f)*/
}
```

**重写收集输入向量的函数**

```csharp
public override void CollectObservations(VectorSensor sensor)
    {
        // 添加目标方块的位置
        sensor.AddObservation(target.localPosition);
        // 添加小球自己的位置
        sensor.AddObservation(transform.localPosition);
            
        // 添加小球水平两个方向的速度
        sensor.AddObservation(_rBody.velocity.x);
        sensor.AddObservation(_rBody.velocity.z);
    }
```

**重写接收输出向量的函数**

```csharp
public override void OnActionReceived(ActionBuffers actionBuffers)
{
    // Actions, size = 2
    // 获取返回的两个数值，乘上力度的大小然后给到小球上
    Vector3 controlSignal = Vector3.zero;
    controlSignal.x = actionBuffers.ContinuousActions[0];
    controlSignal.z = actionBuffers.ContinuousActions[1];
    _rBody.AddForce(controlSignal * forceMultiplier);

    // 计算小球和目标方块的距离
    float distanceToTarget = Vector3.Distance(this.transform.localPosition, target.localPosition);
    // 如果这个距离小于1.42f就当作是碰到了
    if (distanceToTarget < 1.42f)
    {
        // 给奖励1分
        SetReward(1.0f);
        // 结束这一回合
        EndEpisode();
    }
    // 如果小球掉落了平面
    else if (this.transform.localPosition.y < 0)
    {
        // 结束这一回合
        EndEpisode();
    }
}
```

**加一个手动控制测试**

```csharp
public override void Heuristic(in ActionBuffers actionsOut)
{
    var continuousActionsOut = actionsOut.ContinuousActions;
    continuousActionsOut[0] = Input.GetAxis("Horizontal");
    continuousActionsOut[1] = Input.GetAxis("Vertical");
}
```

**给Agent添加上需要的组件**

`DecisionRequester`和`BehaviorParameters`

![image-20210121204549115](C:\Users\Ben\AppData\Roaming\Typora\typora-user-images\image-20210121204549115.png)

**创建YAML配置文件**

==注意：==下面第二行的名字要和Behavior Name保持一致

```
behaviors:
  RollerBall:
    trainer_type: ppo
    hyperparameters:
      batch_size: 10
      buffer_size: 100
      learning_rate: 3.0e-4
      beta: 5.0e-4
      epsilon: 0.2
      lambd: 0.99
      num_epoch: 3
      learning_rate_schedule: linear
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    max_steps: 500000
    time_horizon: 64
    summary_freq: 10000
```

开始训练

`mlagents-learn config/MyRollerBall.yaml --run-id=RollerBall`