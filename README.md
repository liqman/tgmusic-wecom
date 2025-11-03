# 项目介绍
  - 本项目依赖于TGBot（@music_v1bot），使用该项目前，请关注该机器人；
  - 本项目通过企业微信与 Telegram 音乐下载机器人交互，实现搜索、翻页与下载；并在首次运行时通过企业微信完成 Telegram 登录（手机号、验证码、两步验证密码）。

## 1. 目录与持久化
建议的宿主机目录结构：

- 项目根目录
  - `config.ini`（配置文件，必须存在）
  - `data/`（保存 Telegram 会话等数据）
  - `downloads/`（下载歌曲保存目录，可自定义）

注意：
- `downloads/` 与 `data/` 将通过卷挂载到容器内，确保升级/重启后数据不丢失。
- 为了避免路径问题，建议在 `config.ini` 中将 `[telegram]` 下的 `session_name` 设置为 `data/tg_music_cli`，这样会话文件会保存在 `/app/data` 中（映射到宿主机 `./data`）。

## 2. 准备 config.ini
详见 config.ini

## 3. 配置企业微信回调地址

将应用的回调 URL 指向：

- `http://<你的公网IP或域名>:18000/wecom/callback`

其中 `18000` 为 docker-compose 默认映射端口（可自行修改）。`token` 与 `encoding_aes_key` 须与 `config.ini` 一致。

## 4. 构建并启动

方式一：docker-compose（推荐）

```
docker compose build
docker compose up -d
```

方式二：Docker 命令行

```
docker run -d --name wecom-bridge \
  -p 8000:8000 \
  -e TZ=Asia/Shanghai \
  -v $(pwd)/config.ini:/app/config.ini:ro \
  -v $(pwd)/music:/music \
  -v $(pwd)/data:/app/data \
  --restart unless-stopped \
  liqman/tgmusic-wecom
```

## 5. 首次登录（通过企业微信）

容器启动后：
- 在企业微信中向你的自建应用发送任意消息。
- 系统会引导你：
  1) 输入手机号（国际格式，如 `+86138xxxxxxx`）
  2) 输入验证码（收到 Telegram 发送的短信或 App 内验证码）
  3) 如账号开启两步验证，再输入密码
- 登录成功后，直接发送歌曲名即可搜索；返回列表后发送数字选择可下载；支持 `n/p` 翻页、`kg/kw/wyy/qq` 切源，以及 `mv/rn/rm` 文件操作命令。

## 6. 其他功能介绍
- 直接发送歌曲名进行搜索
- 翻页：n 下一页，p 上一页
- 选择：发送数字编号（例如：1）
- 切换音源：/kg /kw /wyy /qq（若列表支持）
- 取消本次搜索：/c
- 移动已下载歌曲：/mv <文件名.后缀> <目标分类>
- 删除已下载歌曲：/rm <文件名.后缀>
- 重命名已下载歌曲：/rn <旧文件名.旧后缀> <新文件名.新后缀>
