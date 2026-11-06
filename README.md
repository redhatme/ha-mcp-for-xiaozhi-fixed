# MCP Server for Xiaozhi（HA 2026.11 修复版）

把小智（xiaozhi.me）接入 Home Assistant 的 MCP 服务端集成，支持语音控制 HA 设备。

本仓库基于上游 [c1pher-cn/ha-mcp-for-xiaozhi](https://github.com/c1pher-cn/ha-mcp-for-xiaozhi) **v0.2.1**，
补上了 **Home Assistant 2026.11** 的兼容性修复。

---

## 为什么需要这个修复版

HA 2026.11 把容器内置的 `mcp` 库升级到了 **2.x**，删除了 1.x 的装饰器 API。
上游 v0.2.1 仍按 1.x 编写，升级 HA 后表现为：

- 小智接入点页面显示「未连接」
- 语音提示「找不到设备」
- 日志每 20 秒重试一次，报错：

```
AttributeError: 'Server' object has no attribute 'list_prompts'
AttributeError: 'ConfigEntry' object has no attribute 'runtime_data'
ValidationError: LLMContext ... requires_admin
```

## 本版改动

| 文件 | 改动 |
|---|---|
| `server.py` | 按 HA core 自带 `mcp_server` 组件（已适配 2.x）重写工具/资源注册，改用 `ServerRequestContext` 风格 |
| `websocket_transport.py` | JSON-RPC 用 `TypeAdapter[JSONRPCMessage].validate_python()` 解析；不再依赖 `entry.runtime_data`，改从 `hass.data` 取 session manager；`LLMContext` 传入带管理员权限的真实 `Context` 对象 |
| `manifest.json` | 版本 `0.2.2`，`requirements` 收紧为 `mcp>=2.0.0` |

功能行为与上游 v0.2.1 一致，仅做 API 适配，未改变业务逻辑。

## 安装

**要求：Home Assistant 2026.11.0 或更高。**

1. HACS → 右上角三个点 → **自定义存储库**
2. 填入本仓库地址：

   ```
   https://github.com/redhatme/ha-mcp-for-xiaozhi-fixed
   ```

3. 类别选择 **Integration**，点击添加
4. 在 HACS 里搜索 `Xiaozhi` → 下载 → **重启 Home Assistant**
5. 设置 → 设备与服务 → 添加集成 → 搜索 **MCP Server for Xiaozhi** → 填入小智接入点提供的 WebSocket 地址与令牌

## 装完后如果语音还是「找不到设备」

集成连通 ≠ 设备可用。HA 里必须把实体**公开给语音助手**：

设置 → 语音助手 → 公开 → 勾选允许语音控制的设备（灯、开关、空调、窗帘等）。

建议**不要**公开门锁、摄像头、报警器等敏感设备。

## 注意

- **不要同时安装上游 v0.2.1**。两者 `domain` 都是 `ws_mcp_server`，同时存在会互相覆盖、行为异常。
- 本仓库不会自动跟随上游更新。上游若发布官方 2026.11 修复版，建议切换回上游。
- 与 [mac8005/xiaozhi-mcp-ha](https://github.com/mac8005/xiaozhi-mcp-ha) 是两套不同实现，`domain` 不同（`xiaozhi_mcp`），可以共存但只需装一个。

## 许可

继承上游 [MIT License](LICENSE)，原始版权归 c1pher 所有。
