

# Android App Bundle常用命令行

## 快捷使用

#### [BundleTools下载地址](https://github.com/google/bundletool/releases)

#### 从app bundle生成一组apk

> 如果是mac 需要将 bundletools 改为 java -jar bundletool.jar
>
> 或者 使用 brew 安装，之后就可以直接使用 bundletools 命令
>
> **brew install bundletool**

```sh
bundletool build-apks --bundle=/MyApp/my_app.aab --output=/MyApp/my_app.apks
```

#### 直接安装到已连接的设备

如果没有指定签名信息，那么会使用寻找默认的Android 签名并使用

```shell
bundletool install-apks --apks=/MyApp/my_app.apks
```

**（如果连接了多个设备，请添加 `--device-id=serial-id` 标记来指定目标设备。）**

```sh
bundletool install-apks --apks=/MyApp/my_app.apks --device-id=101
```

---



### App Bundle 简介

> Android App Bundle 是一种发布格式，其中包含您应用的所有经过编译的代码和资源，它会将 APK 生成及签名交由 Google Play 来完成。
>
> Google Play 会使用您的 App Bundle 针对每种设备配置生成并提供经过优化的 APK，因此只会下载特定设备所需的代码和资源来运行您的应用。您不必再构建、签署和管理多个 APK 来优化对不同设备的支持，而用户也可以获得更小且更优化的下载文件包。

### 如何测试App Bundle

- [使用 bundletool 在本地测试 Android App Bundle](https://developer.android.com/studio/command-line/bundletool)，此测试会根据您的 App Bundle 生成 APK 并将其部署到连接的设备上。
- [通过网址分享应用](https://support.google.com/googleplay/android-developer/answer/9303479)。通过这种方式，您能够以最快的速度上传 app bundle 并通过 Google Play 商店中的链接将应用分享给受信任的测试人员。此外，这也是测试自定义分发选项（如按需下载功能）的最快方式。
- [设置开放式测试、封闭式测试或内部测试](https://support.google.com/googleplay/android-developer/answer/3131213)。这是测试自定义分发选项（如按需下载功能）的另一种方法。

第一种基于本地开发测试。

第二种属于让别人或者说测试同学自己安装测试。(前提是已安装google play全家桶)

第三种未尝试。



### 1. 从 app bundle 生成一组 APK

默认通过bundletool build-apks 生成的apks没有包含签名信息，如果要进行安装到设备，必须为应用添加签名信息。

```shell
bundletool build-apks --bundle=/MyApp/my_app.aab --output=/MyApp/my_app.apks
--ks=/MyApp/keystore.jks
--ks-pass=file:/MyApp/keystore.pwd
--ks-key-alias=MyKeyAlias
--key-pass=file:/MyApp/key.pwd
```

#### bundleTools支持的命令如下：

下表更详细地介绍了使用 `bundletool build-apks` 命令时可以设置的各种标记和选项。只有 `--bundle` 和 `--output` 是必需的，所有其他标记都是可选的。

| 标记                                                         | 说明                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `--bundle=path`                                              | **（必需）**指定您使用 Android Studio 构建的 App Bundle 的路径。如需了解详情，请参阅[构建您的项目](https://developer.android.com/studio/run#reference)。 |
| `--output=path`                                              | **（必需）**指定“.apks”输出文件的名称，该文件中包含了应用的所有 APK 工件。如需了解如何在设备上测试此文件中的工件，请转至介绍如何[将 APK 部署到已连接设备](https://developer.android.com/studio/command-line/bundletool#deploy_with_bundletool)的部分。 |
| `--overwrite`                                                | 如果您要覆盖您使用 `--output` 选项指定的路径下的现有输出文件，请添加此标记。如果已存在输出文件，而您又未添加此标记，会发生构建错误。 |
| `--aapt2=path`                                               | 指定 AAPT2 的自定义路径。 默认情况下，`bundletool` 包含自己的 AAPT2 版本。 |
| `--ks=path`                                                  | 指定用于为 APK 签名的部署密钥库的路径。此标记是可选的。如果您不添加此标记，`bundletool` 会尝试使用调试签名密钥为您的 APK 签名。 |
| `--ks-pass=pass:password` 或 `--ks-pass=file:/path/to/file`  | 指定密钥库的密码。如果您指定纯文本格式的密码，请使用 `pass:` 限定该密码。如果您要传递包含该密码的文件的路径，请使用 `file:` 限定该路径。如果您使用 `--ks` 标记指定密钥库，而未指定 `--ks-pass`，那么 `bundletool` 会提示您从命令行输入密码。 |
| `--ks-key-alias=alias`                                       | 指定要使用的签名密钥的别名。                                 |
| `--key-pass=pass:password` 或 `--key-pass=file:/path/to/file` | 指定签名密钥的密码。如果您指定纯文本格式的密码，请使用 `pass:` 限定该密码。如果您要传递包含该密码的文件的路径，请使用 `file:` 限定该路径。如果此密码与密钥库的密码相同，您可以省略此标记。 |
| `--connected-device`                                         | 指示 `bundletool` 针对已连接设备的配置生成 APK。如果您不添加此标记，`bundletool` 会为您的应用支持的所有设备配置生成 APK。 |
| `--device-id=serial-number`                                  | 如果您有多个已连接的设备，请使用此标记指定要部署应用的设备的序列 ID。 |
| `--device-spec=spec_json`                                    | 使用此标记提供 `.json` 文件的路径，该文件指定了您要针对其生成 APK 的设备配置。如需了解详情，请转至介绍如何[创建和使用设备规范 JSON 文件](https://developer.android.com/studio/command-line/bundletool#create_use_json)的部分。 |
| `--mode=universal`                                           | 如果您希望 `bundletool` 只构建一个包含应用的所有代码和资源的 APK，以使该 APK 与应用支持的所有设备配置兼容，请将模式设置为 `universal`。**注意**：`bundletool` 仅包含功能模块，这些模块在通用 APK 中的对应清单中指定 `<dist:fusing dist:include="true"/>`。如需了解详情，请参阅[功能模块清单](https://developer.android.com/studio/projects/dynamic-delivery#dynamic_feature_manifest)。请注意，这些 APK 要比针对特定设备配置优化过的 APK 更大。但是，这些 APK 更便于与内部测试人员共享，例如想在多种设备配置上测试您的应用的测试人员。 |
| `--local-testing`                                            | 使用此标志启用 app bundle 进行本地测试。 在本地测试时，由于无需上传到 Google Play 服务器，因此能够实现快速的迭代测试周期。 有关如何使用 `--local-testing` 标记测试模块安装的示例，请参阅[在本地测试模块的安装](https://developer.android.com/guide/app-bundle/test/testing-fakesplitinstallmanager)。 |



### 2. 生成设备专用的apk集

如果您不想针对应用支持的所有设备配置生成一组 APK，则可以使用 `--connected-device` 选项，仅针对已连接设备的配置生成 APK，如下所示。（如果连接了多个设备，请添加 `--device-id=serial-id` 标记来指定目标设备。）

```
bundletool build-apks --connected-device
--bundle=/MyApp/my_app.aab --output=/MyApp/my_app.apks
```

#### 生成并使用设备规范 JSON 文件

`bundletool` 能够针对 JSON 文件指定的设备配置生成一组 APK。如需首先为已连接的设备生成 JSON 文件，请运行以下命令：

```
bundletool get-device-spec --output=/tmp/device-spec.json
```

`bundletool` 会在该工具所在的目录中为您的设备创建一个 JSON 文件。然后，您可以将该文件传递给 `bundletool`，以仅针对该 JSON 文件中描述的配置生成一组 APK，如下所示：

```
bundletool build-apks --device-spec=/MyApp/pixel2.json
--bundle=/MyApp/my_app.aab --output=/MyApp/my_app.apks
```

#### 手动创建设备规范 JSON

如果您无法访问要针对其构建目标 APK 集的设备（例如，朋友想通过您手头没有的设备试用您的应用），则可以使用以下格式手动创建 JSON 文件：

```json
{
  "supportedAbis": ["arm64-v8a", "armeabi-v7a"],
  "supportedLocales": ["en", "fr"],
  "screenDensity": 640,
  "sdkVersion": 27
}
```

然后，您可以将此 JSON 传递给 `bundle extract-apks` 命令，如上一部分中所述。

#### 从现有的 APK 集中提取设备专用 APK

如果您已有一个 APK 集，并且想要从中提取出一部分针对特定设备配置的 APK，则可以使用 `extract-apks` 命令并指定设备规范 JSON，如下所示：

```
bundletool extract-apks
--apks=/MyApp/my_existing_APK_set.apks
--output-dir=/MyApp/my_pixel2_APK_set.apks
--device-spec=/MyApp/bundletool/pixel2.json
```



#### 估算 APK 集中的 APK 的下载大小

APK 集内的 APK 将在压缩后通过网络传送。如需估算这些 APK 的下载大小，可使用 `get-size total` 命令：

```
bundletool get-size total --apks=/MyApp/my_app.apks
```

您可以使用以下标记修改 `get-size total` 命令的行为：

| 标记                      | 说明                                                         |
| :------------------------ | :----------------------------------------------------------- |
| `--apks=path`             | **（必需）**指定要估算下载大小的现有 APK 集文件的路径。      |
| `--device-spec=path`      | 指定用于匹配的设备规范文件（通过 `get-device-spec` 获取或手动构建）的路径。您可以通过指定部分路径来估算一组配置。 |
| `--dimensions=dimensions` | 指定估算大小时使用的维度。接受以逗号分隔的 `SDK`、`ABI`、`SCREEN_DENSITY` 和 `LANGUAGE` 列表。如需使用所有维度进行估算，请指定 `ALL`。 |
| `--instant`               | 估算支持免安装体验的 APK（而不是安装版 APK）的下载大小。默认情况下，`bundletool` 会估算安装版 APK 的下载大小。 |
| `--modules=modules`       | 指定要纳入估算范围的 APK 集中的模块，以逗号分隔列表的形式指定。`bundletool` 命令会自动添加指定集的所有相关模块。默认情况下，该命令会估算首次下载时安装的所有模块的下载大小。 |



### Codelab

- [您的首个 Android App Bundle](https://codelabs.developers.google.com/codelabs/your-first-dynamic-app/index.html)：一个探索 Android App Bundle 基本原理的 Codelab，向您展示了如何使用 Android Studio 快速开始构建您自己的 App Bundle。此 Codelab 还探讨了如何[使用 `bundletool`](https://developer.android.com/studio/command-line/bundletool) 测试 app bundle。

