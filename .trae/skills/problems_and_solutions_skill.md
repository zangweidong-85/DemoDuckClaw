---
name: "problems_and_solutions"
description: "记录和查询开发过程中遇到的环境配置、编译报错等问题及其解决方法。Invoke when encountering bugs, compile errors, or environmental issues, or when a new solution needs to be recorded."
---

# Problems and Solutions (经验沉淀)

此技能用于总结与沉淀项目开发（特别是编译、环境配置）中遇到的各类问题与解决方案，避免后续重复踩坑。每次解决新问题后，都必须将问题现象与解决方法更新到此文件中。

## 资源清理与状态同步规范
每次问题排查完毕或结束任务后：
1. **必须**清理并结束掉不再使用的 Docker 容器，避免占用系统内存。
2. **必须**在回复中给出正在使用的终端链接，方便开发者随时点击查看调试环境。

## 一键构建命令 (综合修复)
如果您在 macOS 上全新构建或遇到了环境及缓存问题，可以在项目根目录下执行以下组合命令，它将一次性解决 Python 环境、头文件缺失、CMakeCache 路径冲突以及 Armino 分区表丢失问题，并执行构建：
```bash
mkdir -p .bin && ln -sf $(which python3) .bin/python && export PATH=$PWD/.bin:$PATH && cp ./TuyaOpen/apps/tuya.ai/ai_components/assets/include/lang_config.h ./ai_components/assets/include/ && rm -rf .build/CMakeCache.txt TuyaOpen/platform/T5AI/t5_os/build && ./TuyaOpen/tos.py build
```

## 已沉淀问题与解决方案

### 1. tos.py 子脚本调用的 Python 环境问题 (macOS)
- **问题现象**：在执行 `./TuyaOpen/tos.py build` 编译时，抛出 `sh: python: command not found` 的错误，导致 `Prepare platform error`。
- **问题原因**：`tos.py` 的子脚本（如 `platform_prepare.py`）硬编码调用了 `python` 命令，而在 macOS 环境下默认仅提供 `python3` 命令。
- **解决方法**：在项目临时目录中创建 `python` 指向 `python3` 的软链接，并将其注入到 `PATH` 环境变量中，以满足脚本的调用需求。
  ```bash
  mkdir -p .bin && ln -s $(which python3) .bin/python && export PATH=$PWD/.bin:$PATH
  ```



### 2. CMakeCache 路径冲突导致 Configure 失败
- **问题现象**：执行编译时，抛出 CMake Error: `The current CMakeCache.txt directory ... is different than the directory ... where CMakeCache.txt was created.` 导致配置失败。
- **问题原因**：项目的路径发生了移动或目录结构变更，导致 CMake 之前生成的缓存文件(`.build/CMakeCache.txt`)中保存的绝对路径与当前真实路径不匹配。
- **解决方法**：删除 `.build` 目录下的缓存文件或直接清理缓存环境后再重新执行构建即可。
  ```bash
  rm -rf .build/CMakeCache.txt
  ```

### 3. Armino (T5_OS) 增量编译导致分区表缺失错误 (primary_tuyaboot is not exists)
- **问题现象**：在进行底层 `make bk7258` 构建时，如果存在旧缓存，可能会抛出 `RuntimeError: primary_tuyaboot is not exists` 错误，导致打包失败。
- **问题原因**：Armino SDK 内部自动分区的脚本（`bk_build_auto_partition.py`）由于项目名（如 `tuya_app` vs `app`）变更或增量编译的残留文件干扰，导致找不到对应的分区配置。
- **解决方法**：进入 `TuyaOpen/platform/T5AI/t5_os` 目录执行 `make bk7258 fullclean` 清理残留的构建配置，或者更彻底地删除底层 build 目录 `rm -rf TuyaOpen/platform/T5AI/t5_os/build`。之后再执行 `tos.py build` 即可重新进行全量正确的构建。
  ```bash
  rm -rf TuyaOpen/platform/T5AI/t5_os/build
  ```