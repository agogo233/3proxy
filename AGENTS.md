# AGENTS.md

## 项目概览
C 语言项目（3proxy 1.0.1），轻量级代理服务器。上游自带 `tests/` 回归套件，本地 CI 仅验证编译通过。

## 构建

### Linux Makefile（推荐，支持 chroot）
```bash
ln -s Makefile.Linux Makefile
make                # 需 gcc/make，pthread
make install        # 需 sudo
make clean
make STATIC=true    # 静态链接
make MAILPROXY=true # 构建 pop3p/imapp/smtpp（1.0.1 起默认不构建）
make FTP=true       # 构建 ftppr（1.0.1 起默认不构建）
```

splice() 1.0.1 起默认编译（`-DWITHSPLICE`），仅对带 `-s` 的服务生效。

### CMake（跨平台，无 chroot）
```bash
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -D3PROXY_USE_OPENSSL=ON -D3PROXY_USE_PCRE2=ON
cmake --build .
sudo cmake --install .
```

邮件代理与 FTP 需显式开启，否则对应二进制不生成：
`-D3PROXY_USE_MAILPROXY=ON -D3PROXY_USE_FTP=ON`

### 编译依赖（Linux）
`libssl-dev` `libpcre2-dev` `libpam-dev`（可选，按需启用特性）

### 测试（上游自带，1.0.1 起）
```bash
python3 tests/run.py              # 全部用例，需已构建的 bin/3proxy
python3 tests/run.py ipv6         # 只跑名字匹配的用例
python3 tests/run.py -v           # 打印每项断言
```

## 目录边界
- `src/` — 核心逻辑：`conf.c`（配置解析，最大文件）、`proxy.c`、`ssl.c`、`proxymain.c`
- `src/plugins/` — 可加载插件，各自带独立 Makefile
- `cmake/` — CMake 模块，插件定义在此
- `cfg/` — 示例配置
- `scripts/` — 安装/init/Docker 模板
- `tests/` — 上游回归套件（Python，`tests/run.py`）
- `bin/` — 构建输出（gitignore，不要提交）

## 重要约束
- 验证手段：`make` / `cmake --build` + `python3 tests/run.py`（未装 libssl-dev 时 ssl/tlspr 类用例会失败）
- CMake 构建**不使用 chroot**；Makefile.Linux 才是 chroot 部署方式
- 二进制及插件都以 `3proxy_` 前缀命名
- 代码使用 GNU 扩展（`-D_GNU_SOURCE`），不是严格标准 C
- 修改 `conf.c`（最大文件）时应格外谨慎
