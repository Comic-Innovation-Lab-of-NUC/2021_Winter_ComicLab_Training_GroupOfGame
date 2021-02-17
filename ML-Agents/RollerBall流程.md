

# ����ĵ�

[Unity ML-Agents Toolkit Documentation](https://github.com/Unity-Technologies/ml-agents/blob/release_12_docs/docs/Readme.md)

[Installation](https://github.com/Unity-Technologies/ml-agents/blob/release_12_docs/docs/Installation.md)

[Getting Started Guide](https://github.com/Unity-Technologies/ml-agents/blob/release_12_docs/docs/Getting-Started.md)

[Making a New Learning Environment](https://github.com/Unity-Technologies/ml-agents/blob/release_12_docs/docs/Learning-Environment-Create-New.md)

# ���û���

1. ��װ[anaconda](https://www.anaconda.com/products/individual#Downloads)
2. ����һ����������Ҫpython3.6.1��3.7�������µ���3.8����`conda create -n ml-agents python=3.7`�ķ�ʽ������
3. ��`conda env list`�������еĻ�����Ҳ�����ý���
4. `activte <������>`���������Ҳ�����ý���
5. `pip3 install torch==1.7.0 -f https://download.pytorch.org/whl/torch_stable.html`��ʧ�ܵĻ������Զ༸�Σ�ʵ�ڲ����þ���
6. `pip3 install mlagents`
7. ����ml-agents��������git��Ҳ����ֱ����

# �ٷ�ʾ��

1. �鿴�ٷ�ʾ��
2. ����ѵ���ٷ�ʾ��
3. cd����ǰĿ¼
4. `mlagents-learn config/ppo/3DBall.yaml --run-id=first3DBallRun` ����ʼѵ����`mlagents-learn`�൱����ָ������ƣ������ǲ�����`config/ppo/3DBall.yaml`�������ļ���·����`--run-id=<����>`�������Ǳ���ѵ����ID
5. `Ctrl+C`������ֹ
6. �����Result�У�onnx��׺�ľ���
7. ȥ��Ŀ���滻����������

# ��һ���򵥵�Demo

����ֱ���ڹٷ�����Ŀ������Ҳ�����½�һ����

==PackageManager== �����ص�ML-AgentsӦ���ǰ汾�е�ɻ���ʲô���ԭ�򣬶�����̫�ԡ����Կ���ֱ�Ӹ��ư������������Ǹ�package��

1. �������С��Ŀ�ꡢ�ذ壩
2. ��Agent��ӽű�

```csharp
// ����������ռ�
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;
```

��ĳɼ̳���`Agent`


```csharp
// ÿһ�ο�ʼ��ʱ����ô˺���
public override void OnEpisodeBegin() {}
    
// �ռ��������ݵ�ʱ����ô˺���
public override void CollectObservations(VectorSensor sensor) {}

// �����������ʱ����ô˺���
public override void OnActionReceived(ActionBuffers actionBuffers) {}
```
**������Ҫ��һЩ����**

```csharp
// ����
private Rigidbody _rBody;
// Ŀ�귽���Transform
public Transform target;
// �ƶ�ʱ�����ȵĴ�С
public float forceMultiplier = 10;

void Start () {
    // ��ȡ�������
    _rBody = GetComponent<Rigidbody>();
}
```

**��д��ʼ�ĺ���**

```csharp
public override void OnEpisodeBegin()
{
    // ���С��ĸ߶ȵ���0���ȵ��µ��棩����С���λ�����õ����ģ��ٶ�����
    if (transform.localPosition.y < 0)
    {
        // ���ٶ�����Ϊ0����������
        _rBody.angularVelocity = Vector3.zero;
        // �ٶ�����Ϊ0����������
        _rBody.velocity = Vector3.zero;
        // λ��Ϊ���ĵ��Ϸ�һ��㣨��localPosition��
        transform.localPosition = new Vector3( 0, 0.5f, 0);
    }

    // �������Ŀ�귽���λ��
    target.localPosition = new Vector3(
        Random.value * 8 - 4,
        0.5f,
        Random.value * 8 - 4);
    /*Random.value����һ��0��1���������Random.value * 8 - 4�൱������һ��-4��4�������*/
    /*Ҳ����Random.Range(-4f,4f)*/
}
```

**��д�ռ����������ĺ���**

```csharp
public override void CollectObservations(VectorSensor sensor)
    {
        // ���Ŀ�귽���λ��
        sensor.AddObservation(target.localPosition);
        // ���С���Լ���λ��
        sensor.AddObservation(transform.localPosition);
            
        // ���С��ˮƽ����������ٶ�
        sensor.AddObservation(_rBody.velocity.x);
        sensor.AddObservation(_rBody.velocity.z);
    }
```

**��д������������ĺ���**

```csharp
public override void OnActionReceived(ActionBuffers actionBuffers)
{
    // Actions, size = 2
    // ��ȡ���ص�������ֵ���������ȵĴ�СȻ�����С����
    Vector3 controlSignal = Vector3.zero;
    controlSignal.x = actionBuffers.ContinuousActions[0];
    controlSignal.z = actionBuffers.ContinuousActions[1];
    _rBody.AddForce(controlSignal * forceMultiplier);

    // ����С���Ŀ�귽��ľ���
    float distanceToTarget = Vector3.Distance(this.transform.localPosition, target.localPosition);
    // ����������С��1.42f�͵�����������
    if (distanceToTarget < 1.42f)
    {
        // ������1��
        SetReward(1.0f);
        // ������һ�غ�
        EndEpisode();
    }
    // ���С�������ƽ��
    else if (this.transform.localPosition.y < 0)
    {
        // ������һ�غ�
        EndEpisode();
    }
}
```

**��һ���ֶ����Ʋ���**

```csharp
public override void Heuristic(in ActionBuffers actionsOut)
{
    var continuousActionsOut = actionsOut.ContinuousActions;
    continuousActionsOut[0] = Input.GetAxis("Horizontal");
    continuousActionsOut[1] = Input.GetAxis("Vertical");
}
```

**��Agent�������Ҫ�����**

`DecisionRequester`��`BehaviorParameters`

![image-20210121204549115](C:\Users\Ben\AppData\Roaming\Typora\typora-user-images\image-20210121204549115.png)

**����YAML�����ļ�**

==ע�⣺==����ڶ��е�����Ҫ��Behavior Name����һ��

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

��ʼѵ��

`mlagents-learn config/MyRollerBall.yaml --run-id=RollerBall`