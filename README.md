# Auto Campus · 校园网自动登录

> 开机自动认证校园网，掉线自动重连。不用再每次开机手动打开认证网页。

Windows 小工具，单文件免安装，支持**深澜 Srun / 锐捷 ePortal / 城市热点 Dr.COM** 及通用表单自动识别。

---

## 解决的痛点

- 每次开机都要重新打开校园网认证页面、手动点登录
- 认证页面显示"未连接"，但网其实已经能用（页面误报）
- 半夜掉线了没人重新认证，第二天起来没网

## 功能

| 功能 | 说明 |
|---|---|
| 开机自动认证 | 开机后台启动，检测到没网就自动登录 |
| 掉线自动重连 | 定时检测，断网自动重新认证 |
| 真实联网判定 | 直接请求真实外网（`msftconnecttest` / `generate_204` / 百度）判断，只有真能上网才显示"已连接"，**不会被认证页面的假状态骗到** |
| 多协议自动识别 | 深澜 Srun（完整协议实现）、锐捷 ePortal（新版 POST + 旧版 dr1003 双接口）、城市热点 Dr.COM、通用表单填表 |
| 日间 / 夜间主题 | 一键切换，选择会记住 |
| 状态横幅 | 顶部整条色带：绿=已连接 / 黄=正在认证 / 红=失败原因 |
| 开机自启 | 写当前用户注册表，免管理员权限 |

## 下载使用

1. 在本仓库点击 **Code → Download ZIP**，解压
2. 双击 `Auto Campus.exe`
   （首次运行 Windows 若提示"已保护你的电脑" → 更多信息 → 仍要运行）
3. 填写 **账号** 和 **密码** → 勾选"开机自动启动并后台认证" → 点 **保存并启用**

之后开机什么都不用做，等几秒网就通了。详细说明见包内 `使用说明.txt`。

## 常见问题

**认证失败怎么办？**
先到「运行日志」标签页看服务器返回的内容；再去「高级设置」手动指定认证方式试试。

**提示"获取 token 失败"？**
在「基础设置」的"登录页地址"里填入认证服务器 IP（例如 `172.16.20.130`）。

**会不会把账号锁了？**
不会。只有检测到**真的没网**才会发起认证，且每次间隔一个检测周期（默认 30 秒）。

**密码存在哪？**
`%APPDATA%\CampusNetLogin\config.json`，经 XOR + Base64 混淆存储（非明文，但也不等同于加密保险箱），**不会**联网上传到任何服务器。

## 技术实现

- Python 3 + tkinter（自绘圆角按钮与卡片式 UI），PyInstaller 打包为单文件 exe
- 深澜 xEncode 加密算法（XXTEA 变体 + 自定义字母表 Base64）已与公开参考实现逐字节比对一致，见 `源码/tests/test_crypto.py`
- 源码结构：

```
源码/
├─ main.py              入口（--check / --login / --minimized）
├─ src/app.py           界面 + 后台监控线程
├─ src/portal.py        各认证协议实现
├─ src/crypto_srun.py   深澜加密算法
├─ src/netcheck.py      联网判定与门户探测
├─ src/config.py        配置读写
└─ src/autostart.py     开机自启
```

自行构建：

```bash
python -m venv .venv
.venv/Scripts/pip install requests pillow pyinstaller
.venv/Scripts/pyinstaller --noconfirm --onefile --windowed --name AutoCampus \
    --icon assets/wifi.ico --add-data "assets/wifi.ico;." main.py
```

## 兼容性

仅在 Windows 10 / 11 上测试。各校认证系统参数存在差异，如遇某个学校不通过，
把「运行日志」里服务器返回的内容提 Issue，我按实际返回适配。

## 许可

仅供学习和个人使用。
