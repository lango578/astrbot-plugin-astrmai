# astrmai（AstrBot 插件版）

把 **[MaiBot 麦麦](https://github.com/MaiM-with-u/MaiBot)** 的群聊人格引擎嵌进 **[AstrBot](https://github.com/AstrBotDevs/AstrBot)** 的同一个进程：
一个 asyncio 事件循环、一条 NapCat 反向 WebSocket、逐条消息裁决归属。同一条消息不会被答两遍。

![astrmai](assets/astrmai-icon.svg)

## 装之前要知道的三件事

1. **本插件不打包麦麦本体**。官方市场 zip 上限 16MB，而麦麦是独立的 GPL 项目 —— 请自己 `git clone -b v1.3.18 https://github.com/MaiM-with-u/MaiBot`，
   放到 `data/plugin_data/astrmai/MaiBot`，或在插件配置 `mai_root` 里填你已有副本的路径。
2. **本插件不带 `requirements.txt`，也不会替你 pip**。AstrBot 加载插件时会把 requirements 静默装进你的 Python 环境，
   而麦麦要的是 faiss / sentence-transformers 这一类重包 —— 装坏你环境这件事我们不代做。
   缺依赖时 astrmai 会明确告诉你缺什么、装哪几个，你自己决定。
3. **合并作品按 AGPL-3.0-or-later 分发**（GPL-3.0 的麦麦可以与 AGPL 的 AstrBot 合并，反向不行）。
   完整对应源码在本仓库；如果你改了再分发，请连改动一起给源码。

## 装完之后

```
# 1) 放麦麦
git clone -b v1.3.18 https://github.com/MaiM-with-u/MaiBot <AstrBot>/data/plugin_data/astrmai/MaiBot
# 2) 在麦麦目录里装它自己的依赖（建议先备份你的 venv，或用 AstrBot 的 venv 并接受风险）
cd <上面那个 MaiBot 目录> && pip install -r requirements.txt
# 3) 在 AstrBot 面板里把本插件的 mai_root 指过去（留空就是第 1 步那个路径），打开 panel_enable，重启 AstrBot
```

控制台：`http://127.0.0.1:18124/astrmai/`，初始口令 `astrmai-init`，**登录即强制改密**，该口令只接受服务器本机登录。
QQ 里：`/mai status`、`/mai panel`、`/mai 许可证`、`/mai market astrbot <关键词>`、`/mai install astrbot <id>`。
`/mai` 只对管理员开放（管理员 = `allowed_users`，两者都空时一律拒绝）。

## 它会往你机器上写什么

| 东西 | 位置 |
|---|---|
| 控制台凭据（只有 scrypt 哈希，无明文） | `data/plugin_data/astrmai/credentials.json` |
| 面板改过的闸门/自主性设置 | `data/plugin_data/astrmai/settings.json` |
| 市场目录缓存（30 分钟 TTL） | `data/plugin_data/astrmai/cache/` |
| 你放的麦麦本体与其运行数据 | `data/plugin_data/astrmai/MaiBot/` |

状态一律放在 `plugin_data` 而不是插件目录里：**AstrBot 更新插件会把插件目录整个删掉重建**。

## 出网清单

| 目标 | 什么时候 | 能不能关 |
|---|---|---|
| 你的模型端点 | 每次回复 | 你配的 |
| NapCat / QQ 服务器 | 收发消息 | 你配的 |
| AstrBot 官方插件目录（api.soulter.top → cloud.astrbot.app、codeload.github.com） | 你主动搜/装市场时 | `ASTRMAI_MARKET_NETWORK=0`（两条链路一起关，缓存也不能当后门） |
| 麦麦官方市场 `hyybuth.xyz:10059` | 同上 | 同上（该主机与麦麦遥测同域） |
| 上游遥测 | **从不**（默认掐断） | 需要显式 `ASTRMAI_TELEMETRY=on` 才放行 |

## 已知边界

- 单管理员控制台，没有多用户/权限分级；登录进去能看见完整对话历史（只读，本插件没做删除入口）+ 200 条裁决摘要 + 400 条日志环。
- 管不住麦麦的自主发言循环（她 own 的主动聊天），裁决层只管"单条消息谁答"。
- 端到端取证用本地桩模型；接真模型请自己先看几轮。
