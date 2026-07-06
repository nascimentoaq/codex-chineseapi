# codex接入国内第三方中转站配置方法

## 项目简介
在国内使用 Codex、OpenAI API 或 VS Code Codex 插件时，很多开发者会遇到网络连接失败、请求超时、账号风控、插件无法正常调用模型等问题。
通过配置第三方 API 中转站，可以让 Codex 请求走国内更稳定的代理接口，从而提升开发体验。

本文将以 **Codex 接入国内第三方中转站** 为核心，完整演示：

* 如何修改 Codex 的 `config.toml` 配置文件
* 如何配置第三方 API 中转站地址
* 如何设置系统环境变量
* 如何解决 VS Code Codex 插件提示缺少环境变量的问题
* 如何验证 Codex 是否成功接入中转站

如果你还没有中转站账号，可以先通过下面的链接注册：

👉 [点击注册 Codex 国内中转站](https://codex.dakeai.cc/register?promo=DKAI)

中转站的详细配置教程也可以参考：

👉 [国内中转站配置教程](https://dk82.com/646.html)

VS Code Codex 插件安装与配置教程可以参考：

👉 [VS Code Codex 插件安装配置教程](https://zhuanlan.zhihu.com/p/2035752399952950892)

---

## 一、为什么 Codex 需要接入国内第三方中转站？

由于 OpenAI 官方接口在国内网络环境下访问并不稳定，直接使用 Codex 时可能会出现以下问题：

* API 请求超时
* Codex 无法连接 OpenAI
* VS Code 插件频繁报错
* 科学上网环境不稳定
* 请求失败导致开发流程中断
* 账号异常或风控风险增加

因此，对于国内开发者来说，将 Codex 配置到稳定的第三方 API 中转站，是一种更适合日常开发的方案。

第三方中转站的作用可以简单理解为：

> Codex 不再直接访问 OpenAI 官方接口，而是通过国内可访问的中转 API 地址完成模型请求。

---

## 二、Codex 接入中转站的核心思路

Codex 接入第三方 API 中转站的核心逻辑并不复杂，主要分为两步：

1. 在 Codex 配置文件中新增一个自定义 `model_provider`
2. 将默认模型请求指向这个自定义中转站接口

实际需要修改的内容主要包括：

* Codex 本地配置文件：`config.toml`
* 系统环境变量：用于保存 API Key

为了安全，不建议直接把 API Key 明文写入配置文件，而是通过环境变量读取。

---

## 三、Codex 配置文件路径

Codex 的配置文件通常位于用户目录下的 `.codex` 文件夹中。

### macOS / Linux 路径

```bash
~/.codex/config.toml
```

### Windows 路径

```bash
C:\Users\你的用户名\.codex\config.toml
```

如果 `.codex` 文件夹中没有 `config.toml` 文件，可以手动新建一个。

---

## 四、修改 config.toml 全局配置

打开 `config.toml` 文件，在顶部加入或修改以下配置：

```toml
# 修改为中转站实际支持的模型名称
model = "gpt-5.5"

# 指定 Codex 默认使用自定义中转站
model_provider = "my_proxy"

# 推理强度，可根据需求调整
model_reasoning_effort = "high"

[windows]
sandbox = "elevated"
```

这里最关键的是：

```toml
model_provider = "my_proxy"
```

它表示 Codex 默认不再走官方接口，而是使用我们后面定义的 `my_proxy` 中转站服务商。

如果你原来的配置文件中已经存在 `projects`、`plugins` 等配置，不需要删除，保留即可。

---

## 五、添加第三方中转站 API 配置

在 `config.toml` 文件底部新增以下内容：

```toml
[model_providers.my_proxy]
name = "DKAI Proxy API"
base_url = "https://codex.dakeai.cc/v1"
env_key = "MY_PROXY_API_KEY"
wire_api = "responses"
requires_openai_auth = false
```

### 参数说明

| 参数                     | 说明                                             |
| ---------------------- | ---------------------------------------------- |
| `name`                 | 自定义服务商名称，可以随意命名                                |
| `base_url`             | 第三方中转站提供的 API 接口地址，通常以 `/v1` 结尾                |
| `env_key`              | Codex 从系统环境变量中读取的 API Key 名称                   |
| `wire_api`             | 接口类型，兼容新版 OpenAI Responses API 可填写 `responses` |
| `requires_openai_auth` | 第三方中转站一般设置为 `false`                            |

需要注意的是，示例中的：

```toml
base_url = "https://codex.dakeai.cc/v1"
```

需要替换为你在中转站后台获取到的真实 API 地址。

中转站注册地址：

👉 https://codex.dakeai.cc/register?promo=DKAI

配置教程参考：

👉 https://dk82.com/646.html

---

## 六、配置系统环境变量

在上面的配置中，我们写了：

```toml
env_key = "MY_PROXY_API_KEY"
```

这表示 Codex 启动时会自动从系统环境变量中读取名为 `MY_PROXY_API_KEY` 的变量，并将它作为 API Key 使用。

这样做的好处是：

* 避免 API Key 明文暴露在配置文件中
* 更安全
* 方便后续更换密钥
* 适合 VS Code、终端、Codex 等多个工具统一调用

---

## 七、Windows 设置环境变量方法

如果你使用的是 Windows，可以按照下面步骤设置：

1. 按下 `Win + S`
2. 搜索并打开 **编辑系统环境变量**
3. 点击右下角的 **环境变量**
4. 在上半部分的 **用户变量** 区域点击 **新建**
5. 变量名填写：

```bash
MY_PROXY_API_KEY
```

6. 变量值填写你在中转站后台获取到的 API Key，例如：

```bash
sk-xxxxxxxxxxxxxxxx
```

7. 点击 **确定** 保存所有窗口

注意：

变量名必须和 `config.toml` 中的 `env_key` 完全一致，不能多空格，也不能大小写写错。

---

## 八、macOS / Linux 设置环境变量方法

如果你使用的是 macOS 或 Linux，可以在终端中执行：

```bash
export MY_PROXY_API_KEY="你的中转站API_KEY"
```

如果希望长期生效，可以写入 Shell 配置文件。

### zsh 用户

```bash
echo 'export MY_PROXY_API_KEY="你的中转站API_KEY"' >> ~/.zshrc
source ~/.zshrc
```

### bash 用户

```bash
echo 'export MY_PROXY_API_KEY="你的中转站API_KEY"' >> ~/.bashrc
source ~/.bashrc
```

---

## 九、验证 Codex 是否配置成功

配置完成后，需要关闭当前所有终端窗口，然后重新打开一个新的终端。

原因是：
终端只会在启动时读取系统环境变量，如果你不重新打开终端，新设置的 API Key 可能不会生效。

在新终端中运行：

```bash
codex exec "Reply OK"
```

如果返回类似：

```bash
OK
```

说明 Codex 已经成功读取环境变量，并且能够通过第三方中转站完成请求。

---

## 十、VS Code Codex 插件仍然报错怎么办？

有些用户会遇到一种情况：

终端里运行 Codex 正常，Codex 独立环境也能正常使用，但是 VS Code 里的 Codex 插件仍然报错：

```bash
Missing environment variable: MY_PROXY_API_KEY
```

这个问题通常不是配置文件错误，而是 VS Code 没有重新读取系统环境变量。

---

## 十一、VS Code 插件报错原因

VS Code 在启动时会读取一次系统环境变量。

如果你是在 VS Code 已经打开的情况下新增了 `MY_PROXY_API_KEY`，那么当前 VS Code 进程并不知道这个变量已经存在。

即使你点击右上角关闭窗口，VS Code 也可能仍然有后台进程残留，导致插件继续读取旧环境变量。

---

## 十二、解决 VS Code 缺少环境变量的方法

可以按照下面步骤彻底重启 VS Code：

1. 关闭所有 VS Code 窗口
2. 检查系统托盘是否还有 VS Code 图标
3. 如果有，右键选择退出
4. 打开任务管理器
5. 搜索是否还有 `Code.exe` 进程
6. 如果存在，全部结束
7. 重新打开 VS Code
8. 再次启动 Codex 插件

重新启动后，VS Code 会重新读取系统环境变量，`MY_PROXY_API_KEY` 就可以正常注入到 Codex 插件中。

如果你还没有安装 VS Code Codex 插件，可以参考这篇教程：

👉 [VS Code Codex 插件安装配置教程](https://zhuanlan.zhihu.com/p/2035752399952950892)

---

## 十三、完整 config.toml 示例

下面是一个较完整的配置示例，你可以根据自己的中转站信息进行修改：

```toml
model = "gpt-5.5"
model_provider = "my_proxy"
model_reasoning_effort = "high"

[windows]
sandbox = "elevated"

[model_providers.my_proxy]
name = "DKAI Proxy API"
base_url = "https://codex.dakeai.cc/v1"
env_key = "MY_PROXY_API_KEY"
wire_api = "responses"
requires_openai_auth = false
```

请重点修改以下内容：

```toml
model = "gpt-5.5"
base_url = "https://codex.dakeai.cc/v1"
env_key = "MY_PROXY_API_KEY"
```

其中：

* `model` 要填写中转站实际支持的模型名称
* `base_url` 要填写中转站后台提供的 API 地址
* `env_key` 要和系统环境变量名称保持一致

---

## 十四、常见问题 FAQ

### 1. config.toml 文件不存在怎么办？

可以手动创建。

路径一般是：

```bash
~/.codex/config.toml
```

或：

```bash
C:\Users\你的用户名\.codex\config.toml
```

如果 `.codex` 文件夹也不存在，可以一起新建。

---

### 2. API Key 可以直接写在 config.toml 里吗？

不建议。

更推荐使用环境变量，例如：

```toml
env_key = "MY_PROXY_API_KEY"
```

这样可以避免密钥泄露，也更适合多工具共用。

---

### 3. 为什么终端可以用，VS Code 插件不能用？

通常是 VS Code 没有彻底重启。

解决办法是关闭所有 VS Code 窗口，并确认任务管理器中没有残留的 `Code.exe` 进程，然后重新打开 VS Code。

---

### 4. wire_api 应该填写 responses 还是 chat？

如果你的中转站兼容 OpenAI 新版 Responses API，可以填写：

```toml
wire_api = "responses"
```

如果中转站只兼容传统 Chat Completions 接口，可以尝试：

```toml
wire_api = "chat"
```

具体以中转站后台说明为准。

---

### 5. 模型名称应该怎么填？

模型名称需要填写中转站实际支持的模型。

例如：

```toml
model = "gpt-5.5"
```

如果请求失败，可以检查中转站后台支持的模型列表，确认模型名称是否填写正确。

---

## 十五、总结

通过以上配置，就可以让 Codex 接入国内第三方 API 中转站，从而解决国内网络环境下访问 OpenAI 不稳定、VS Code Codex 插件无法正常调用、环境变量读取失败等问题。

完整流程可以概括为：

1. 注册并获取第三方中转站 API Key
2. 修改 Codex 的 `config.toml`
3. 添加自定义 `model_provider`
4. 配置系统环境变量
5. 重启终端和 VS Code
6. 使用 `codex exec "Reply OK"` 验证配置

推荐入口：

* [Codex 国内中转站注册](https://codex.dakeai.cc/register?promo=DKAI)
* [国内中转站配置教程](https://dk82.com/646.html)
* [VS Code Codex 插件安装配置教程](https://zhuanlan.zhihu.com/p/2035752399952950892)

如果你正在国内环境中使用 Codex、OpenAI API 或 VS Code Codex 插件，建议优先完成中转站配置，可以明显减少请求失败和插件报错问题。
