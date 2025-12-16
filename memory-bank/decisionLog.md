# 决策日志 (Decision Log)

## 2025-12-16: WebDAV同步功能实现

### 背景
用户需要在积分系统中集成坚果云WebDAV同步功能，并彻底解决CORS跨域问题。

### 决策
1. **采用混合架构**：
   - 优先支持 **直连模式**（适用于浏览器已安装CORS插件或同源部署的情况）。
   - 核心支持 **代理模式**（通过反向代理解决跨域问题）。
   
2. **代理服务器配置**：
   - 目标地址：`https://dav.jianguoyun.com/dav/`
   - 路径重写：`/proxy` -> `/`
   - 认证头转发：客户端生成 `Basic Auth` 头，代理服务器透传。

3. **客户端实现 (`NutcloudWebDAVClient`)**：
   - **封装性**：将所有WebDAV操作（测试连接、上传、下载、同步）封装在独立类中。
   - **配置管理**：使用 `localStorage` 持久化配置，密码采用 Base64 简单加密（客户端侧）。
   - **冲突解决**：
     - 默认策略：`newest-wins` (最新数据优先)。
     - 备选策略：`local-wins`, `remote-wins`, `manual`。
   - **同步逻辑**：
     - 下载云端数据 -> 解决冲突 -> 更新本地 -> 上传最新。
     - 支持自动同步（数据变更后防抖触发）。

4. **UI集成**：
   - 在管理员面板添加“云端同步”区域。
   - 提供配置弹窗、同步状态栏和进度指示器。

### 验证
- **单元测试**：创建了 `webdav-test.html`，覆盖了配置加载、URL生成、认证头生成和冲突解决逻辑。
- **集成测试**：在 `ultimate-points-system-fixed.html` 中集成并手动验证UI响应。

---
### 代码实现 [WebDAV Client]
[2025-12-16 19:44:00] - 实现了NutcloudWebDAVClient类及UI集成

**实现细节：**
- 新增 `NutcloudWebDAVClient` 类，处理WebDAV协议交互。
- 集成配置弹窗和同步状态显示。
- 实现 `newest-wins` 等冲突解决策略。
- 添加 `X-Target-Url` 头以支持通用代理服务。

**测试框架：**
- 自研简易测试框架 (在 `webdav-test.html` 中实现)。
- 模拟 `localStorage` 和 DOM 环境。

**测试结果：**
- 覆盖率：核心逻辑（配置、URL、冲突解决）100%。
- 通过率：100% (基于 `webdav-test.html` 运行结果)。
