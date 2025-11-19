# AppHttpService 使用说明文档

## 概述

`AppHttpService` 是一个基于 Dio 封装的 HTTP 请求服务类，提供了统一的网络请求接口、错误处理机制、自动请求头管理和多种请求类型支持。该类主要用于 Flutter 应用中的网络请求处理。

## 主要特性

- ✅ 支持 GET、POST、PUT 请求类型
- ✅ 自动管理请求头（设备信息、用户 Token、加密标识等）
- ✅ 统一的响应数据模型（ResultModel）
- ✅ 完善的错误处理机制（网络错误、业务错误、解析错误）
- ✅ 自动登录失效处理
- ✅ 图片上传功能
- ✅ 图片验证码获取
- ✅ 自动加载指示器管理
- ✅ 详细的日志记录

## 目录结构

- [枚举类型](#枚举类型)
- [常量定义](#常量定义)
- [数据模型](#数据模型)
- [核心类](#核心类)
- [扩展方法](#扩展方法)
- [使用示例](#使用示例)
- [错误处理](#错误处理)
- [配置说明](#配置说明)

---

## 枚举类型

### AppHttpServiceType

请求类型枚举，支持三种 HTTP 方法：

```dart
enum AppHttpServiceType { 
  post,  // POST 请求
  get,   // GET 请求
  put    // PUT 请求
}
```

---

## 常量定义

### 业务状态码

| 常量名 | 值 | 说明 |
|--------|-----|------|
| `successfulCode` | 200 | 请求成功 |
| `verifyingCode` | 2002 | 认证中 |
| `verifySuccessCode` | 2003 | 认证通过 |
| `notLoginCode` | [800] | 登录失效（列表） |
| `otherDeviceEnterGameCode` | 8116 | 其他设备进入游戏 |

---

## 数据模型

### ResultModel

统一的响应数据模型，包含三个字段：

```dart
class ResultModel {
  int? code;      // 状态码
  String? msg;    // 消息提示
  dynamic data;   // 响应数据
}
```

**方法：**
- `ResultModel.fromJson(Map<String, dynamic> json)` - 从 JSON 创建实例
- `Map<String, dynamic> toJson()` - 转换为 JSON

---

## 核心类

### AppHttpService

HTTP 请求服务主类。

#### 静态属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `xrkimGlobal` | `String` | 全局 XRKIMG 值，用于后续请求携带 |
| `isXRKENC` | `bool` | 是否启用加密（影响 Content-Type） |

#### 实例属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `dio` | `Dio` | Dio 实例 |

#### 构造函数

```dart
AppHttpService()
```

自动设置默认请求头，并交由 `DebugManager` 附加调试功能。

#### 方法说明

##### 1. setGlobalXRKIMG

设置全局 XRKIMG 值，影响后续新建的 `AppHttpService` 实例。

```dart
static void setGlobalXRKIMG(String value)
```

**参数：**
- `value` - XRKIMG 值

**使用场景：** 通常在获取验证码后，将返回的 XRKIMG 设置为全局值。

---

##### 2. setHeaders

动态添加请求头。

```dart
void setHeaders(Map<String, dynamic> headers)
```

**参数：**
- `headers` - 要添加的请求头键值对

---

##### 3. requestApi

核心请求方法，支持 GET、POST、PUT 请求。

```dart
Future<ResultModel> requestApi({
  AppHttpServiceType type = AppHttpServiceType.get,
  required String url,
  Map<String, dynamic>? parameters,
  bool showToast = true,
  bool goToLogin = false,
})
```

**参数：**
- `type` - 请求类型（默认 GET）
- `url` - 请求地址（必需）
- `parameters` - 请求参数（可选）
- `showToast` - 是否显示错误提示（默认 true）
- `goToLogin` - 登录失效时是否跳转登录页（默认 false）

**返回值：** `Future<ResultModel>`

**特性：**
- 自动显示/隐藏加载指示器
- 15 秒后自动关闭加载指示器（防止卡死）
- 自动处理响应数据解析
- 自动处理业务错误码
- 自动处理网络异常

---

##### 4. uploadImages

上传多张图片。

```dart
Future<dynamic> uploadImages({
  required List<AssetEntity> entities,
  required String url,
})
```

**参数：**
- `entities` - 图片资源列表（AssetEntity）
- `url` - 上传接口地址

**返回值：** `Future<dynamic>` - 返回 `ResultModel` 或错误处理结果

**说明：**
- 使用 `FormData` 格式上传
- 字段名为 `images[]`
- 自动处理上传成功/失败

---

##### 5. httpUrl

使用 `http` 包进行简单的 GET 请求（不经过 Dio）。

```dart
Future<String> httpUrl(String url)
```

**参数：**
- `url` - 请求地址

**返回值：** `Future<String>` - 响应体字符串

**使用场景：** 适用于不需要复杂配置的简单请求。

---

##### 6. getCaptchaImage

获取图片验证码。

```dart
Future<Map<String, dynamic>> getCaptchaImage(String url)
```

**参数：**
- `url` - 验证码接口地址

**返回值：** `Map<String, dynamic>` 包含：
- `success` - 是否成功
- `imageBytes` - 图片字节数据
- `xrkim` - 响应头中的 XRKIMG 值
- `error` - 错误信息（失败时）

**说明：**
- 自动从响应头提取 XRKIMG
- 建议在获取成功后调用 `setGlobalXRKIMG` 设置全局值

---

### 默认请求头

`AppHttpService` 会自动设置以下请求头：

| 请求头 | 说明 | 来源 |
|--------|------|------|
| `Content-Type` | 内容类型 | `isXRKENC ? 'text/plain' : 'application/json'` |
| `devicetype` | 设备类型代码 | `DeviceInfoManager` |
| `appversion` | 应用版本号 | `DeviceInfoManager` |
| `devicebrand` | 设备品牌 | `DeviceInfoManager` |
| `devicemodel` | 设备型号 | `DeviceInfoManager` |
| `device` | 设备ID | `DeviceInfoManager` |
| `appsystem` | 操作系统版本 | `DeviceInfoManager` |
| `user-agent` | User-Agent | `DeviceInfoManager` |
| `buildnumber` | 构建号 | `DeviceInfoManager` |
| `XRKENC` | 加密标识 | 当 `isXRKENC = true` 时添加 |
| `XRKIMG` | 图片验证码标识 | 当 `xrkimGlobal` 不为空时添加 |
| `X-token` | 用户 Token | 当用户已登录时添加 |

**设备类型代码说明：**
- 1: PC
- 2: ANDROID_H5
- 3: IOS_H5
- 4: ANDROID_APP
- 5: IOS_APP

---

## 扩展方法

### AppHttpServiceExtension

通过扩展方法提供错误处理和提示功能。

#### 1. showToast

显示提示信息。

```dart
void showToast(String? message)
```

---

#### 2. handleBusinessError

处理业务逻辑错误。

```dart
ResultModel handleBusinessError({
  required ResultModel resultModel,
  required String url,
  required bool showToast,
  required bool goToLogin,
})
```

**功能：**
- 记录错误日志
- 显示错误提示（可选）
- 处理登录失效跳转（可选）

---

#### 3. handleParseError

处理数据解析错误。

```dart
ResultModel handleParseError({
  required String url,
  required dynamic responseData,
  required bool showToast,
})
```

**功能：**
- 记录解析错误日志
- 返回错误 ResultModel（code: 0, msg: "数据解析错误"）
- 显示错误提示（可选）

---

#### 4. handleNetworkError

处理网络错误。

```dart
ResultModel handleNetworkError({
  required DioException e,
  required String url,
  required bool showToast,
  required bool goToLogin,
})
```

**功能：**
- 延迟 1 秒后关闭加载指示器
- 尝试解析服务器错误响应
- 处理各种网络异常类型
- 显示友好的错误提示
- 处理登录失效跳转（可选）

---

#### 5. getNetworkErrorMessage

根据 DioException 类型返回友好的错误信息。

```dart
String getNetworkErrorMessage(DioException e)
```

**错误类型映射：**

| DioExceptionType | 错误信息 |
|------------------|----------|
| `connectionTimeout` | 连接超时，请检查网络 |
| `sendTimeout` | 发送超时，请重试 |
| `receiveTimeout` | 接收超时，请重试 |
| `badResponse` | 服务器响应错误 |
| `cancel` | 请求已取消 |
| `connectionError` | 网络连接失败，请检查网络 |
| `badCertificate` | 证书验证失败 |
| `unknown` | 未知网络错误 |

---

#### 6. getHttpStatusMessage

根据 HTTP 状态码返回友好的错误信息。

```dart
String getHttpStatusMessage(int statusCode)
```

**状态码映射：**

| 状态码 | 错误信息 |
|--------|----------|
| 400 | 请求参数错误 |
| 401 | 未授权，请重新登录 |
| 403 | 禁止访问 |
| 404 | 请求的资源不存在 |
| 405 | 请求方法不允许 |
| 408 | 请求超时 |
| 409 | 请求冲突 |
| 422 | 请求参数验证失败 |
| 429 | 请求过于频繁，请稍后再试 |
| 500 | 服务器内部错误，请稍后再试 |
| 502 | 网关错误 |
| 503 | 服务暂时不可用 |
| 504 | 网关超时 |
| 其他 | 服务器错误(状态码) |

---

## 使用示例

### 1. 基本 GET 请求

```dart
final httpService = AppHttpService();
final result = await httpService.requestApi(
  type: AppHttpServiceType.get,
  url: 'https://api.example.com/users',
  parameters: {'page': 1, 'limit': 10},
);

if (result.code == successfulCode) {
  print('请求成功: ${result.data}');
} else {
  print('请求失败: ${result.msg}');
}
```

### 2. POST 请求

```dart
final httpService = AppHttpService();
final result = await httpService.requestApi(
  type: AppHttpServiceType.post,
  url: 'https://api.example.com/login',
  parameters: {
    'username': 'user123',
    'password': 'pass123',
  },
  showToast: true,
  goToLogin: false,
);
```

### 3. 上传图片

```dart
final httpService = AppHttpService();
final result = await httpService.uploadImages(
  entities: selectedImages, // List<AssetEntity>
  url: 'https://api.example.com/upload',
);

if (result is ResultModel && result.code == successfulCode) {
  print('上传成功: ${result.data}');
}
```

### 4. 获取验证码

```dart
final httpService = AppHttpService();
final result = await httpService.getCaptchaImage(
  'https://api.example.com/captcha',
);

if (result['success'] == true) {
  final imageBytes = result['imageBytes'] as Uint8List;
  final xrkim = result['xrkim'] as String;
  
  // 设置全局 XRKIMG，后续请求会自动携带
  AppHttpService.setGlobalXRKIMG(xrkim);
  
  // 显示验证码图片
  // Image.memory(imageBytes)
}
```

### 5. 自定义请求头

```dart
final httpService = AppHttpService();
httpService.setHeaders({
  'Custom-Header': 'custom-value',
  'Authorization': 'Bearer token123',
});

final result = await httpService.requestApi(
  url: 'https://api.example.com/protected',
);
```

### 6. 不显示错误提示

```dart
final result = await httpService.requestApi(
  url: 'https://api.example.com/api',
  showToast: false, // 不显示 Toast 提示
);
```

### 7. 登录失效不跳转

```dart
final result = await httpService.requestApi(
  url: 'https://api.example.com/api',
  goToLogin: false, // 登录失效时不自动跳转登录页
);
```

---

## 错误处理

### 错误处理流程

1. **网络错误** → `handleNetworkError`
   - 连接超时、网络异常等
   - 自动显示友好提示
   - 延迟关闭加载指示器

2. **数据解析错误** → `handleParseError`
   - 响应数据格式不正确
   - 返回 code: 0, msg: "数据解析错误"

3. **业务逻辑错误** → `handleBusinessError`
   - 服务器返回的业务错误码（非 200）
   - 根据错误码决定是否跳转登录

### 登录失效处理

当服务器返回错误码 `800`（`notLoginCode`）且 `goToLogin = true` 时：
1. 调用 `UserManager().outLogin()` 退出登录
2. 延迟 150ms 后跳转到登录页

---

## 配置说明

### 启用加密模式

```dart
AppHttpService.isXRKENC = true;
```

启用后，`Content-Type` 将变为 `text/plain`，并自动添加 `XRKENC: 1` 请求头。

### 设置全局 XRKIMG

```dart
AppHttpService.setGlobalXRKIMG('your-xrkim-value');
```

设置后，所有新建的 `AppHttpService` 实例都会自动携带该请求头。

---

## 注意事项

1. **加载指示器管理**
   - 请求开始时会自动显示加载指示器
   - 请求成功或失败后会自动隐藏
   - 15 秒后会自动关闭（防止卡死）

2. **错误提示**
   - 默认会显示错误 Toast 提示
   - 可通过 `showToast: false` 关闭

3. **登录失效**
   - 默认不会自动跳转登录页
   - 需要设置 `goToLogin: true` 才会自动跳转

4. **验证码处理**
   - 获取验证码后建议立即调用 `setGlobalXRKIMG` 设置全局值
   - 后续请求会自动携带该值

5. **日志记录**
   - 所有请求和错误都会记录到日志
   - 使用 `Logger()` 进行日志输出

6. **Debug 模式**
   - 在 Debug 模式下，`DebugManager` 会自动附加调试功能
   - Release 模式下会自动跳过

---

## 依赖项

- `dio` - HTTP 客户端
- `fluttertoast` - Toast 提示
- `logger` - 日志记录
- `wechat_assets_picker` - 图片选择器
- `http` - 简单 HTTP 请求

---

## 相关文件

- `AppApiManage.dart` - API 地址管理
- `DeviceInfoManager.dart` - 设备信息管理
- `UserModeManage.dart` - 用户管理
- `DebugManager.dart` - 调试管理
- `loading_components.dart` - 加载指示器组件

---

## 更新日志

- 支持自动请求头管理
- 支持加密模式
- 支持图片验证码获取
- 完善的错误处理机制
- 自动登录失效处理

---

## 作者

本文档基于 `AppHttpService.dart` 代码自动生成。

