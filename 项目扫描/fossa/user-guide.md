原网页地址： https://github.com/fossas/fossa-cli/blob/master/docs/user-guide.md 

## 用户指南
Fossa最常用于分析项目并提取其完整依赖关系图，然后可以使用API密钥将其上传到fossa.com。 本页说明如何配置此工作流程以及FOSSA CLI的其他功能。 如果您想看看demo，请参阅 https://github.com/fossas/fossa-cli/blob/master/docs/how-it-works.md#how-it-works

### 1. 安装
以下命令将执行脚本，以在相应的操作系统上获取并安装最新的GitHub版本。


#### 1.1 MacOS (Darwin) or Linux amd64:
```
curl -H 'Cache-Control: no-cache' https://raw.githubusercontent.com/fossas/fossa-cli/master/install.sh | bash
```

#### 1.2 Windows with Powershell:
```
Set-ExecutionPolicy Bypass -Scope Process -Force; iex  ((New-Object System.Net.WebClient).DownloadString('https://raw.githubusercontent.com/fossas/fossa-cli/master/install.ps1'))
```
通过修改profile.ps1文件来将C:\ProgramData\fossa-cli添加到环境路径中， 或者可以执行下面命令临时添加到环境路径中：
```
$env:Path += ";C:\ProgramData\fossa-cli"
```

### 2. 配置项目

可以通过配置文件fossa.yml（config-file.md＃fossayml）或直接使用fossa命令的参数和标志来实现配置。 FOSSA CLI的构建旨在通过运行fossa init创建准确的配置文件，并且仅需手动配置即可进行复杂的构建或调整个人喜好。

#### 2.1 配置文件
通过在要分析的项目的根目录下运行 **fossa init** 来创建配置文件。 cli将在文件树中向下移动并搜索所有相关模块。
> 注意：默认情况下，Fossa将排除已确定为测试，开发或依赖项模块的模块。 添加 --include-all 以保留所有模块。
```
version: 1

cli:
  server: https://app.fossa.com
  fetcher: custom
  project: git+github.com/fossas/fossa-cli
analyze:
  modules:
    - name: fossa-cli
      type: go
      target: github.com/fossas/fossa-cli/cmd/fossa
      path: cmd/fossa
```

有关每个字段和自定义的信息，请参见.fossa.yml (https://github.com/fossas/fossa-cli/blob/master/docs/config-file.md#fossayml) 文档。

#### 2.2 参数配置
可以通过包含诸如 **fossa analysis <module_type>:\<target>** 之类的参数来提供模块，该参数确定分析的类型，然后提供构建目标。 更多示例位于下面。
> 注意：参数和标志配置优先于配置文件。

Examples(https://github.com/fossas/fossa-cli/blob/master/docs/user-guide.md#examples)
1. nodejs:.,rubygem:./docs
2. go:./cmd/fossa

### 3. 分析项目

1. 通过运行f **ossa analyst -o** 正确无误地验证分析是否成功。 此命令会将分析结果进行标准输出而不是上传。
2. 获取 FOSSA_API_KEY。 有关说明，请参阅FOSSA.com手册。
3. 运行 **export FOSSA_API_KEY = <your-api-key>** 来设置环境变量。
4. 运行fossa分析。 默认情况下，这将分析您的项目并将结果上传到指定的服务器app.fossa.com。

> 注意：可以使用命令FOSSA_API_KEY = <your-key> 内联添加 FOSSA_API_KEY fossa analysis 

根据您所处的环境，分析可能会有很大的不同。有关更多信息和可用选项，请参考受支持的各个环境页面(https://github.com/fossas/fossa-cli/blob/master/README.md/#supported-environments)。

#### 上传自定义构建项目
如果您的项目太复杂或您的项目系统高度定制，则fossa可能无法正确分析您的项目。 在这种情况下，您仍然可以使用fossa upload将项目构建信息上传到fossa.com进行构建分析和分类。

##### 数据格式
自定义构建以以下格式上传：
```
// You can upload builds for many modules at a time.
type UploadData = Module[];

type Module = {
  Name: string, // The name of the module/entry-point. This is currently ignored.
  Type: string, // The type of the entry point.
  Manifest: string, // The path of the project manifest. This is currently ignored.
  Build: {
    Artifact: string, // The build artifact name. This is currently ignored.
    Context: any, // Metadata for the build. This is currently ignored.
    Succeeded: boolean, // Generally, this is `true`.
    Error?: string, // An optional error string for failed builds.
    Dependencies: Dependency[]
  }
};

type Dependency = {
  locator: string, // See below.
  data?: any // Optional metadata for the dependency. This is currently ignored.
}
```


##### 定位器
FOSSA中的项目和软件包由其定位器标识，该定位器是生态系统，软件包名称和软件包修订版的组合。
生态系统(ecosystem)是以下之一：

> bower: Bower dependencies
>
> comp: Composer dependencies
>
> gem: RubyGems dependencies
>
> git: git submodules
>
> go: go get dependencies
>
> hackage: Haskell dependencies
>
> mvn: Maven dependencies
>
> npm: NPM dependencies

包名称(package)是生态系统中的包的名称（例如: express）。
修订版(revision)是生态系统中软件包的修订版标识符（例如3.0.0）。
他们组合为：
ecosystem+package$revision

Example:
```
npm+express$3.0.0
go+github.com/golang/dep$06d527172446499363c465968a132d7aa528e550
mvn+org.apache.hadoop:hadoop-core$2.6.0-mr1-cdh5.5.0
```


### 4. 命令行参考
所有标志都应传递给调用的子命令。 当前不支持全局标志。

| 命令           | 描述                           |
| -----------   | -----------                    |
| fossa         |  生成配置文件 & 分析当前配置       |
| fossa init    |  生成配置文件                    |
| fossa analyze |  分析当前配置                    |
| fossa test    |  在最新的fossa扫描上失败CI作业     |
| fossa upload  |  上传自定义构建信息               |
| fossa report  |  获取有关最新的fossa扫描的信息     |
| fossa update  |  更新当前的命令行版本              |

#### fossa
是 fossa init 和 fossa analyze 的合并

all-in-one 命令示例
```
# 创建配置文件 & 运行分析 & 上传结果
FOSSA_API_KEY=YOUR_API_KEY fossa
```

#### fossa init
尽最大努力从当前系统状态推断正确的配置。 如果成功，它将配置写入新的或现有的配置文件 (默认是 .fossa.yml)
> Note: 如果配置文件已经存在， 该命令不会覆盖它

如果配置中未定义任何模块，它将扫描工作目录中的代码模块以进行分析。

默认情况下，**fossa init** 会过滤掉可能是开发，测试或依赖项的模块。 过滤器将删除文件路径中具有以下任何内容的所有模块：**docs**, **test**, **examples**, **third-party**, **vendor**, **tmp**, **node_modules**, **.srclib-cache**, **spec**, **Godeps**, **.git**, **bower_components**, **Carthage**, and **Checkouts**。 可以通过传递--include-all标志来禁用过滤。

##### Example
```
# 在当前文件目录，创建 .fossa.yml 文件
fossa init
```

| Flag          | Short        | 描述           |
| -----------   | -----------  |-------------- |
| --include-all |              | 包含所有模块     |
| --debug       |              | 打印debug信息   |
| --help        |      -h      | 打印help信息    |


#### fossa analyze
分析项目的依赖项列表，可以选择将结果上传到FOSSA。 如果分析失败，请首先查看受支持的环境页面中的文档(https://github.com/fossas/fossa-cli/blob/master/README.md/#supported-environments)，以获取特定于您的环境的信息，以及可以设置为对您的设置进行条件化的标志。

> 注意：除非设置了--output，否则分析需要API密钥。


##### Example
```
# 使用.fossa.yml运行分析并上传到服务器端点。
FOSSA_API_KEY=YOUR_API_KEY fossa analyze
```

| Flag          | Short        | 描述           |
| -----------   | -----------  |--------------  |
| --config      |      -c      |  配置文件路径    |
| --project     |      -p      |  项目           |
| --revision    |      -r      |  版本号         |
| --endpoint    |      -e      |  endpoint      |
| --output      |      -o      |  分析结果输出方式                              |
| --server-scan |              |  在原始模块上运行服务器端依赖项扫描                |
| --dev         |              |  包括开发依赖项。 注意：仅对nodejs项目有效         |
| --debug       |              |  打印debug信息                                |
| --help        |      -h      |  帮助                                         |
| --team        |      -T      |  将项目与FOSSA中的指定团队联系起来。 仅适用于新项目  |
| --policy      |              |  将项目与FOSSA中的指定策略连接。 仅适用于新项目     |
| --title       |      -t      |  设置显示在FOSSA UI中的标题。 仅适用于新项目       |

> 注意：标题，策略和团队标志仅会在项目第一次上传时对其产生影响。 后续运行fossa analysis的所有标志都将被忽略。 创建此功能的目的是允许用户在首次上传时将项目与其团队相关联，但会阻止在UI中对CLI所做的更改。
