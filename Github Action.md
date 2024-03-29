
# Action 仓库

每个 Action 就是一个独立脚本，因此可以做成一个代码仓库 repo，因此可以使用 userName/actionName 的方法引用
GitHub 官方的 actions 都放在 [github.com/actions](https://github.com/actions)

Actions 代码仓库版本的概念：[versioning-your-action](https://help.github.com/en/articles/about-actions#versioning-your-action)
```bash
actions/setup-node@74bc508 # 指向一个 commit
actions/setup-node@v1.0    # 指向一个标签
actions/setup-node@master  # 指向一个分支
```



# 基础概念

- workflow：一次运行过程
- job：一个 workflow 由一个或者多个 job 组成
- step：一个 job 由多个 step 构成
- action：每个 step 可以执行一个或者多个 action



# Workflow

- Action 配置文件叫做 workflow 文件，存放路径 `.github/workflows`
- 使用 YAML 格式，文件名可以任意取，统一后缀 `.yml`
- 一个 repository 可以有多个 workflow。Github 只要发现 `.github/workflows` 下有 `.yml` 文件，就会自动运行该文件
- workflow 文件的配置字段非常多：https://help.github.com/en/articles/workflow-syntax-for-github-actions



## 字段举例

### name：workflow 名称

省略默认使用文件名



### on：触发 workflow 的事件

-  `on: push` push 事件触发该 workflow
- 也可以是事件组：`on: [push, pull_request]`
- 完整事件文档：https://help.github.com/en/articles/events-that-trigger-workflows

`on.<push|pull_request>.<tags|branches>`  触发事件时，限定标签或者分支
``` shell
# 只有 master 分支发生 push 事件才会触发 workflow
on:
	push:
		branches:
			master
```



### jobs：workflow 文件的主体

表示要执行的一项或多项任务
`jobs.<job_id>.name`
```shell
jobs:
  my_first_job:
    name: My first job
  my_second_job:
    name: My second job
```



### jobs 执行顺序

`jobs<job_id>.needs`
指明了依赖关系，即运行顺序
```shell
jobs:
  job1:
  job2:
    needs: job1
  job3:
    needs: [job1, job2]
```



### jobs.<job_id>.runs-on 必填， 指定运行所需虚拟机环境

可用如下:
-   `ubuntu-latest`，`ubuntu-18.04` 或 `ubuntu-16.04`
-   `windows-latest`，`windows-2019` 或 `windows-2016`
-   `macOS-latest` 或 `macOS-10.14`
eg. `runs-on: ubuntu-18.04`



### jobs.<job_id>.steps 指定 Job 运行步骤，包含一个或多个步骤。

步骤可指定下 3 个字段：
- name 步骤名称
- run 运行的命令或者 action
- env 所需要的环境变量



## 完整案例

```shell
name: Greeting from Mona
on: push

jobs:
  my-job:
    name: My Job
    runs-on: ubuntu-latest
    steps:
    - name: Print a greeting
      env:
        MY_VAR: Hi there! My name is
        FIRST_NAME: Mona
        MIDDLE_NAME: The
        LAST_NAME: Octocat
      run: |
        echo $MY_VAR $FIRST_NAME $MIDDLE_NAME $LAST_NAME.
```

	steps 字段定义了 4 个环境变量然后执行了一条 bash 命令



# 其他语法

## jobs.<job_id>.steps[*].with

数据结构为 key/value 的 map 定义的 Action 的参数

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: actions/hello_world@main
        with:
          first_name: Mona
          middle_name: The
          last_name: Octocat
```



## workflow_dispatch

使用这个事件，可以选择性的定义传递到 workflow 的输入

### workflow_dispatch.inputs

```yaml
on:
  workflow_dispatch:
    inputs:
      logLevel:
        description: 'Log level'
        required: true
        default: 'warning'
        type: choice
        options:
          - info
          - warning
          - debug
      print_tags:
        description: 'True to print to STDOUT'
        required: true
        type: boolean
      tags:
        description: 'Test scenario tags'
        required: true
        type: string
      environment:
        description: 'Environment to run tests against'
        type: environment
        required: true

jobs:
  print-tag:
    runs-on: ubuntu-latest
    if:  ${{ inputs.print_tags }} 
    steps:
      - name: Print the input tag to STDOUT
        run: echo  The tags are ${{ inputs.tags }} 
```

> on.workflow_dispatch.inputs.<input_id>.required 指定该输入是否会被提供
>
> on.workflow_dispatch.inputs.<input_id>.type 参数的值用 string 得方式指明类型，必须是下列其中之一：
>
> `boolean`, `choice`, `number`, `environment` , `string`
>
> 解释一下：
>
> inputs 下面，每一个 field：logLevel，print_tags，tags，environment，都会是一个输入框（input）
>
> required 会指明这个 field 是否是必须的，必须的会带 * 号
>
> type 是指明输入类型
>
> 最后，这些输入的内容都会被带入 jobs 执行，可以通过 ${{ inputs.tags }} 这种形式访问



## jobs.<job_id>.env

变量的 map，可以应用到 job 里面所有的 step，所以是针对整个 workflow 的

## jobs.<job_id>.steps[*].env

对某一个 step 设置环境变量，如果设置和 jobs.<job_id>.env 一个名字的环境变量会覆盖之前的



## jobs.<job_id>.strategy & jobs.<job_id>.strategy.matrix

在一个 job 定义中，基于一个变量的组合，使用多个变量创建并运行多个 job

```yaml
jobs:
  example_matrix:
    strategy:
      matrix:
        version: [10, 12, 14]
        os: [ubuntu-latest, windows-latest]
```

根据上面定义的组合会创建出 3x2 个 job 来执行，依次：

```
version: 10, os: ubuntu-latest
version: 10, os: windows-latest
version: 12, os: ubuntu-latest
version: 12, os: windows-latest
version: 14, os: ubuntu-latest
version: 14, os: windows-latest
```

- 每个 workflow 可以创建 256 个 jobs
- 可以通过属性方式 matrix.version/matrix.os 来访问当前 job 里的变量



```
- name: demo
  appId: ${{ secrets.FIREBASE_APP_ID_DEV }}
  credentialsFile: ${{ secrets.CREDENTIAL_FILE_CONTENT }}
- name: demo2
  appId: ${{ secrets.FIREBASE_APP_ID_STG }}
  credentialsFile: ${{ secrets.CREDENTIAL_FILE_STG }}
```






# 语法文档

https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions