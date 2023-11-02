
### 1. 基本概念
step ：相当于 container，可能有多个
task：相当于 pod，可能有多个
pipeline：接收定义的参数，通过 params 指定

results 可共享，临时小量数据，类似变量存储
workspace 工作空间，可持久化

属于定制的 k8s resource
kind 有
- Task
- TaskRun
- TaskLoop
- Run
- Pipeline  定义
- PipelineRun   执行 Pipeline
- TriggerBinding  传递参数给 TriggerTemplate 创建的 PipelineRun
- TriggerTemplate 配置 PipelineRun
- EventListener 监听事件发生
- ClusterInterceptor 可选，校验事件数据


```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: project-pipeline
spec:
  params:
    - name: api-url
    - name: cloud-region
  tasks:
    - name: clone
      taskRef:
        name: git-clone
    - name: build
      taskRef: 
        name: build
      runAfter:
        - clone
    - name: deploy
      taskRef:
        name: deploy
      runAfter:
        - build
```

### 2. Task
创建一个 Task，名字为 testtask.yml，创建个pod，kubectl apply -f testtask,yml
```yml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: testtask   # 任务名称
spec:
  params:
    - name: test-type
      type: string
  steps:
    - name: run-test
      image: xxx
      args: ['$(params.test-type)']
  steps:
    - name: echo
      image: alpine
      scripts: |
        #/bin/sh
        echo "Hello World"
```

创建一个 TaskLoop，对 test-type 这个参数轮询
```yml
apiVersion: tekton.dev/v1beta1
kind: TaskLoop
metadata:
  name: testloop
spec:
  taskRef:
    name: testtask
  iterateParam: test-type
```
创建一个 Run，运行测试任务
```yml
apiVersion: tekton.dev/v1alpha1
kind: Run
metadata:
  generateName: testloop-run-
spec:
  params:
    - name: test-type
      value:
        - codeanalysis
        - unittests
        - e2etests
  ref:
    apiVersion: custom.tekton.dev/v1alpha1
    kind: TaskLoop
    name: testloop
```

### 3. Pipelines 实践

安装 Tekton Pipelines
```
kubectl apply --filename \
https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

创建多个 Task， Pipeline 里通过 taskRef 引入 Task ，会为每个 Task，自动创建 TaskRun，参数可以传递

创建一个 hello-goodbye.yml，定义配置信息，kubectl apply -f pipeline.yml
```yml 
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: hello-goodbye
spec:
  params:
  - name: username
    type: string
  tasks:
    - name: hello 
      taskRef:
        name: hello # hello 任务，需要事先创建
    - name: goodbye
      taskRef:
        name: goodbye
      runAfter:
        - hello
      params:
        - name： username
          value: $(params.username)
```

创建一个 hello-goodbye-run.yml，执行一个 pipeline，在这里指定参数
```yml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: hello-goodbye-run
spec:
  pipelineRef:
    name: hello-goodbye  # 上面 metadata 里 声明的名字
  params:
  - name: username
    value: "Tekton"
```

开始执行pipeline，kubectl apply -f hello-goodbye-run.yml
查看日志：tkn pipelinerun logs hello-goodbye-run -f -n default

### 4. Triggers

安装
```
kubectl apply --filename \
https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml
kubectl apply --filename \
https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml
```

查看安装进度
kubectl get pods --namespace tekton-pipelines --watch

创建 TriggerTemplate，trigger-template.yml
```
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerTemplate
metadata:
  name: hello-template
spec:
  params:
  - name: username
    defaults: "Kubernetes"  # 设置了一个默认值
  resourcetemplates:       # 设置 PipelineRun 的模板
  - apiVersion: tekton.dev/v1beta1
    kind: PipelineRun
    metadata:
      generateName: hello-goodbye-run-
    spec:
      pipelineRef:
        name: hello-goodbye
      params:
      - name: username
        value: $(tt.params.username)
```
把模板应用到集群中
kubectl apply -f trigger-template.yml

创建一个 TriggerBinding
```
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerBinding
metadata:
  name: hello-binding
spec:
  params:
  - name: username
    value: $(body.username)
```
绑定了触发器模板，将值传递给模板