# AGENTS.md

## 项目概览
C 语言项目（3proxy 0.9.6），轻量级代理服务器。无单元测试，CI 仅验证编译通过。

## 构建

### Linux Makefile（推荐，支持 chroot）
```bash
ln -s Makefile.Linux Makefile
make                # 需 gcc/make，pthread
make install        # 需 sudo
make clean
make STATIC=true    # 静态链接
```

### CMake（跨平台，无 chroot）
```bash
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -D3PROXY_USE_OPENSSL=ON -D3PROXY_USE_PCRE2=ON
cmake --build .
sudo cmake --install .
```

### 编译依赖（Linux）
`libssl-dev` `libpcre2-dev` `libpam-dev`（可选，按需启用特性）

## 目录边界
- `src/` — 核心逻辑：`conf.c`（配置解析，最大文件）、`proxy.c`、`ssl.c`、`proxymain.c`
- `src/plugins/` — 可加载插件，各自带独立 Makefile
- `cmake/` — CMake 模块，插件定义在此
- `cfg/` — 示例配置
- `scripts/` — 安装/init/Docker 模板
- `bin/` — 构建输出（gitignore，不要提交）

## 重要约束
- 无自动化测试套件，验证手段是 `make` / `cmake --build` 后手动检查
- CMake 构建**不使用 chroot**；Makefile.Linux 才是 chroot 部署方式
- 二进制及插件都以 `3proxy_` 前缀命名
- 代码使用 GNU 扩展（`-D_GNU_SOURCE`），不是严格标准 C
- 修改 `conf.c`（最大文件）时应格外谨慎
