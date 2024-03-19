# 参考
- [[Github Action]]
- https://www.youtube.com/watch?v=XcKT_0kFqBw&ab_channel=DevOpsw%2FGeorge
- https://medium.com/@CORDEA/distribute-apps-to-firebase-app-distribution-with-github-actions-using-workload-identity-federation-d064eb22b18a



# 三个部分

1. Action 执行推送上来的 gradle 脚本
2. gradle 执行打包脚本，打好包
3. gradle 和 Firebase distribution 通信，把 APK push 到 Firebase 上



# 最简单的例子

取材于：[Firebase App Distribution Github Action](https://github.com/marketplace/actions/firebase-app-distribution)

```yaml
name: Build & upload to Firebase App Distribution 

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v1
    - name: set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    - name: build release 
      run: ./gradlew assembleRelease
    - name: upload artifact to Firebase App Distribution
      uses: wzieba/Firebase-Distribution-Github-Action@v1
      with:
        appId: ${{secrets.FIREBASE_APP_ID}}
        serviceCredentialsFileContent: ${{ secrets.CREDENTIAL_FILE_CONTENT }}
        groups: testers
        file: app/build/outputs/apk/release/app-release-unsigned.apk
```

>触发条件是在发生 push 的时候
>
>定义一个 build 的 job
>
>定于运行环境 ubuntu-latest
>
>执行下列 steps
>
>选择执行 checkout@v4 这个 action 检出当前分支，v4 是版本号
>
>定义步骤中显示的名称：set up JDK 1.8
>
>选择执行 setup-java@v1，并用 with 带上参数 java-version: 1.8
>
>名称：build release，并使用系统的 shell 执行命令程序 ./gradlew assembleRelease
>
>名称：upload artifact to Firebase App Distribution，选择执行 wzieba/Firebase-Distribution-Github-Action@v1
>
>并带上appId，serviceCredentialsFileContent，groups，file 参数



# Firebase App_id 和 serviceCredentialsFileContent

## 获取 Firebase App_id

打开 Firebase Project，然后选择 settings，找到对应的 app，会有一个 app_id，类似于

```
1:902386383957:android:07d83ac30c12219ca0e8dc
```

利用下面 "获取 serviceCredentialsFileContent" 的 6-7 的步骤把 Firebase App_id 也添加到 Github Repo 里面



## 获取 serviceCredentialsFileContent

1. 打开 Google Cloud Console，选择对应的项目

2. 点击创建服务账号 ServiceAccount

3. 创建并继续

4. 添加 **Firebase App Distribution Admin** 角色，点击完成

5. 回到列表后，点击更多（...），创建秘钥（KEYS），创建一个 JSON 的私钥，会自动下载

   > 请务必将此文件保存在安全的位置，因为它会向管理员授予对您 Firebase 项目中 App Distribution 的访问权限。

6. 打开 Github，Repo，Settings，Secrets and variables，Actions

7. 创建一个新的 Repository Secret，取名， eg. `CREDENTIAL_FILE_CONTENT`，并把 JSON 文件内容复制进去

8. 创建好后，在 Action yaml 文件里就可以用 `{{ secrets.CREDENTIAL_FILE_CONTENT }}` 访问了



#  完善简单例子

```yaml
name: Build & upload to Firebase App Distribution

on: [push]

jobs:
  build:

    name: Building and distributing app
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: set up JAVA 17
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Setup Gradle
        uses: gradle/gradle-build-action@v2

      - name: Make gradlew executable
        run: chmod +x ./gradlew

      - name: Build Demo Debug APK
        run: ./gradlew assembleDemoDebug

      - name: upload artifact to Firebase App Distribution
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{secrets.FIREBASE_APP_ID}}
          serviceCredentialsFileContent: ${{ secrets.CREDENTIAL_FILE_CONTENT }}
          groups: testers
          file: app/build/outputs/apk/demo/debug/app-demo-debug.apk
```

上面为打了个 debug 包的完整例子



# 多 Flavors 打包发布 Firebase App Distribution

多个 Flavors

- 可以直接在打包命令里面去串行执行
- 可以设置多个 job 去并行执行
- 可以使用 jobs.<job_id>.strategy 并行执行

这里为了记录，使用了最简单的 `com.google.firebase.appdistribution` 插件实现



## build.gradle

分别在项目级别和 app 级别的 build.gradle 加入配置：(注意不要使用 kotlin 的 build.gradle，有 bug，使用 groovy)

```groovy
// project level
id("com.google.firebase.appdistribution") version "4.0.1" apply false

// app level
id 'com.google.firebase.appdistribution'

signingConfigs {
    release {
        storeFile file("../kakuyasu.keystore")
        storePassword "${System.getenv("KAK_STOREPASSWORD")}"
        keyAlias = 'kak'
        keyPassword "${System.getenv("KAK_KEYPASSWORD")}"
    }
}

productFlavors {
			create('demo') {
            dimension 'version'
            applicationId 'jp.co.kakuyasu.app.dev'
            firebaseAppDistribution {
                artifactType = "APK"
                groups = "testers"
                releaseNotesFile = "ci/release-log.txt"
                serviceCredentialsFile = "${project.rootDir}/ci/demo-credential-file.json"
            }
        }
        create('demo2') {
            dimension 'version'
            applicationId 'jp.co.kakuyasu.app.stg'
            firebaseAppDistribution {
                artifactType = "APK"
                groups = "testers"
                releaseNotesFile = "ci/release-log.txt"
                serviceCredentialsFile = "${project.rootDir}/ci/demo2-firebase-credential-file.json"
            }
        }
}
```



## yaml

```yaml
name: Build & upload to Firebase App Distribution

on: [push]

jobs:
  build:

    strategy:
      matrix:
        flavor: ["demo", "demo2"]

    name: Building and distributing app
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: set up JAVA 17
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Prepare Keystore
        run: |
          echo ${{ secrets.KEYSTOREBASE64 }} | base64 -d > $(pwd)/kakuyasu.keystore
        env:
          KEYSTORE: ${{ secrets.KEYSTOREBASE64 }}

      - name: Setup Gradle
        uses: gradle/gradle-build-action@v2

      - name: Make gradlew executable
        run: chmod +x ./gradlew

      - name: Build Release APK And Push To Firebase
        run: |
          flavorName="${{ matrix.flavor }}"
          capitalizedFlavorName="$(echo $flavorName | awk '{print toupper(substr($0,1,1)) tolower(substr($0,2))}')"
          ./gradlew --no-daemon --stacktrace --build-cache assemble${capitalizedFlavorName}Release appDistributionUpload${capitalizedFlavorName}Release
        env:
          KAK_STOREPASSWORD: ${{ secrets.STOREPASSWORD }}
          KAK_KEYPASSWORD: ${{ secrets.KEYPASSWORD }}
```

> 技术点：
>
> strategy.matrix.flavor 使用
>
> 
>
> Keystore 的准备：将保持在 github secrets 里进过 base64 的 keystore 重新 decode 成 keystore 文件
>
> $(pwd)/kakuyasu.keystore 对应 build.gradle 里 storeFile file("../kakuyasu.keystore")
>
> 
>
> 在执行打包命令的时候，设置 STOREPASSWORD 和 KEYPASSWORD 到环境变量里
>
> env:
>           KAK_STOREPASSWORD: ${{ secrets.STOREPASSWORD }}
>           KAK_KEYPASSWORD: ${{ secrets.KEYPASSWORD }}
>
> 对应：
>
> storePassword "${System.getenv("KAK_STOREPASSWORD")}"
>
> keyPassword "${System.getenv("KAK_KEYPASSWORD")}"



# 使用 Workload Identify Federation

什么是 WIF，简单说就是 Service  Account 的无钥匙认证，不用管理 SA 的 Keys



## 好处

在第三方上部署应用或工作，如果需要访问到 GCP，将不在需要管理任何 GCP 的 Token 或者 Key

这也意味着不必持有一个长期的 key，也就减少了很多安全问题



## 可以在哪里使用

任何的支持 OpenId connect 的 provider（OIDC），比如 AWS，MS，Github



## 原理

- 第三方支持 OIDC 的 provider 去 WIF（GC） 换取 federated token

- 再用 federated token 去 WIF（GC） 换取 short-lived IAM Token（Identity & Access Managerment）
- 使用 short-lived token 去访问 GCP 的的资源



## 创建流程

- Go into GCP, and click `Workload Identity Federation` create a workload Identity pool. eg.

  - Name: github actions pool
  - Desc: github action workload identity pool

- Add a provider to pool

  - Choose `OpenID connect(OIDC)`
  - Provider details:
    - Name: actions-provider
    - Provider id: actions-provider
    - Issuer: https://token.actions.githubusercontent.com
      - From the GitHub OIDC document
    - Default audience

- Configure provider attributes

  ```
  google.subject -> assertion.subject
  attribute.repository -> assertion.repository
  attribute.actor -> assertion.actor
  ```

  And no `Attribute Condition`

- Save

- `Grant Access` in Workload Identity Pool

  - If no `Service Account`, Create `Service Account`
  - Service Account details
    - name: app-distributor
    - Id: app-distributor
    - desc: ""
  - Create and continue
  - Grant this service account access to project
    - Select a role: Firebase App Distribution Admin
    - Select a role: Workload Identity User
  - Grant users access to this service account
    - None
  - Done
  - Back to Workload Identity Pool

- Click `Grant Access`

- Select service account

  - Select service account: `app-distributor`
  - Select principals (identities that can access the service account)
    - select Attribute name: repository
    - attribute value: $repositoryName (eg. JohnZh/Demo)

  

> Pool 组织和管理着外部的 Identities，在 pool 里 IAM 会让你授权给这个 Identities 是否有权限访问 GCP
>
> 可以添加 Attribute Conditions
>
> 1. Restrict authentication to a subset of identities. Eg. `assertion.repository == 'LukaFontanilla/pubsub-cloudrun'`



## Action yml 代码

```yaml
  - id: 'auth'
    name: 'Authenticate to GCP'
    uses: 'google-github-actions/auth@v1'
    with:
      create_credentials_file: 'true'
      workload_identity_provider: 'projects/902386383957/locations/global/workloadIdentityPools/github-actions-pool/providers/actions-provider'
      service_account: 'app-distributor@firebasedemo-411613.iam.gserviceaccount.com'
```

> 注意 workload_identity_provider 格式，/providers/$PROVIDER_ID



# 补充学习

在 GitHub Actions 中，`run: |` 表示使用多行字符串（YAML 多行折叠块）来定义一个包含多条命令的脚本块。

`|` 符号用于保留字符串的换行符，并将其作为多行字符串处理。

```yaml
jobs:
  example-job:
    runs-on: ubuntu-latest
    steps:
      - name: Run multiple commands
        run: |
          echo "Hello, world!"
          echo "This is a multiline script."
          # You can include more commands here
```

