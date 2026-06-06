# 抖音下载器 (Douyin Downloader) V2.0

实用的抖音批量下载工具，支持视频、图文笔记、合集、音乐、收藏夹及作者主页批量下载，去水印、自动选最高画质。

## 功能特性

- **多种下载类型**：单个视频、图文笔记、合集、音乐、短链接解析
- **作者主页批量下载**：支持作品(post)、喜欢(like)、合集(mix)、音乐(music) 四种模式
- **登录收藏夹下载**：支持收藏视频(collect)和收藏合集(collectmix)
- **去水印 + 最高画质**：自动选择无水印源，选取最高比特率
- **直播录制**：支持 FLV/HLS 直播流录制（实验性）
- **评论采集**：按作品采集评论及回复，保存为 JSON
- **热搜/关键词搜索**：导出热搜榜和关键词搜索结果到 JSONL
- **REST API 服务**：`--serve` 模式，支持 FastAPI 后端
- **通知推送**：Bark / Telegram / Webhook 下载完成通知
- **视频转写**：可选 OpenAI Transcriptions API 转录
- **并发下载 + 重试**：可配置并发数，指数退避重试
- **SQLite 去重**：数据库 + 本地文件双重去重，支持增量下载
- **浏览器兜底**：分页受限时启动浏览器，支持手动验证
- **Docker 部署**：提供 Dockerfile

## 技术栈

- **语言**：Python 3.9+
- **异步 I/O**：aiohttp、aiofiles、aiosqlite
- **终端 UI**：Rich（进度条、表格）
- **配置**：YAML + 环境变量覆盖
- **加密**：gmssl（SM3/SM4 反爬签名）
- **浏览器自动化**：Playwright（可选，Cookie 获取）
- **API 服务**：FastAPI + Uvicorn（可选）

## 快速开始

### 安装

```bash
git clone https://github.com/jiji262/douyin-downloader.git
cd douyin-downloader
pip install -r requirements.txt

# 可选：浏览器兜底和自动 Cookie 获取
pip install playwright
python -m playwright install chromium
```

### 配置

```bash
cp config.example.yml config.yml
```

编辑 `config.yml`，填入目标链接和 Cookie。最小配置：

```yaml
link:
  - https://www.douyin.com/user/MS4wLjABAAAAxxxx
path: ./Downloaded/
mode:
  - post
number:
  post: 0          # 0 = 无限制
thread: 5
database: true
cookies:
  ttwid: YOUR_TTWID
  odin_tt: YOUR_ODIN_TT
  passport_csrf_token: YOUR_CSRF_TOKEN
```

### 获取 Cookie

```bash
python -m tools.cookie_fetcher --config config.yml
```

登录抖音后回到终端按回车，Cookie 自动写入配置。

### 运行

```bash
# 使用配置文件
python run.py -c config.yml

# 追加 CLI 参数
python run.py -c config.yml -u "https://www.douyin.com/video/xxxx" -t 8

# 热搜榜
python run.py --hot-board 30 -p ./Downloaded

# 关键词搜索
python run.py --search "猫咪" --search-max 100 -p ./Downloaded

# REST API 服务
pip install fastapi uvicorn
python run.py --serve --serve-port 8000
```

### Docker 部署

```bash
docker build -t douyin-downloader .
docker run -v $(pwd)/config.yml:/app/config.yml -v $(pwd)/Downloaded:/app/Downloaded douyin-downloader
```

## 项目结构

```
douyin-downloader/
├── run.py                    # 入口脚本
├── cli/                      # CLI 参数解析、进度展示
│   └── main.py               # 主异步循环
├── core/                     # 业务逻辑
│   ├── api_client.py         # API 请求客户端
│   ├── url_parser.py         # URL 解析
│   ├── video_downloader.py   # 视频下载器
│   ├── music_downloader.py   # 音乐下载器
│   ├── mix_downloader.py     # 合集下载器
│   ├── live_downloader.py    # 直播录制
│   ├── comments_collector.py # 评论采集
│   ├── discovery.py          # 热搜/搜索
│   ├── downloader_factory.py # 工厂模式创建下载器
│   └── user_modes/           # 用户下载策略（post/like/mix/music/collect）
├── auth/                     # Cookie 和 Token 管理
├── config/                   # YAML 配置加载
├── control/                  # 并发控制（限速、重试、队列）
├── storage/                  # SQLite 存储、文件管理、元数据
├── server/                   # REST API 服务（FastAPI）
├── tests/                    # 测试用例
├── config.example.yml        # 配置示例
├── requirements.txt          # 依赖清单
├── Dockerfile                # 容器构建
└── pyproject.toml            # 项目配置
```

## 许可证

MIT
