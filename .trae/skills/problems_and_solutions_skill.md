# DuckyClaw (TuyaOpen) 常见问题与解决方案&#x20;

> 本文档是一个全局的“排坑百科全书”，记录在 DuckyClaw 项目（TuyaOpen SDK）开发、本地调试、飞书联调、硬件烧录等**所有情况**下曾经遇到的问题及其正确的解决方案。

# Problems and Solutions (经验沉淀)

此技能用于总结与沉淀项目开发（特别是编译、环境配置）中遇到的各类问题与解决方案，避免后续重复踩坑。每次解决新问题后，都必须将问题现象与解决方法更新到此文件中。

## 资源清理与状态同步规范

每次问题排查完毕或结束任务后：

1. **必须**清理并结束掉不再使用的 Docker 容器，避免占用系统内存。
2. **必须**在回复中给出正在使用的终端链接，方便开发者随时点击查看调试环境。

## 已沉淀问题与解决方案

### 1. tos.py 子脚本调用的 Python 环境问题 (macOS)

- **问题现象**：在执行 `./TuyaOpen/tos.py build` 编译时，抛出 `sh: python: command not found` 的错误，导致 `Prepare platform error`。
- **问题原因**：`tos.py` 的子脚本（如 `platform_prepare.py`）硬编码调用了 `python` 命令，而在 macOS 环境下默认仅提供 `python3` 命令。
- **解决方法**：在项目临时目录中创建 `python` 指向 `python3` 的软链接，并将其注入到 `PATH` 环境变量中，以满足脚本的调用需求。
  ```bash
  mkdir -p .bin && ln -s $(which python3) .bin/python && export PATH=$PWD/.bin:$PATH
  ```

### 2. 缺失 lang\_config.h 头文件导致编译失败

- **问题现象**：编译 `app_im.c` 时抛出致命错误 `fatal error: lang_config.h: No such file or directory`，编译过程终止。
- **问题原因**：项目代码引用了 `lang_config.h` 头文件，但该文件并未包含在标准的头文件查找路径或项目的源文件目录中。
- **解决方法**：在 TuyaOpen 的示例组件目录中找到该文件（位于 `TuyaOpen/apps/tuya.ai/ai_components/assets/include/lang_config.h`），将其拷贝到项目本地的 `./ai_components/assets/include/` 目录下即可解决依赖缺失。
  ```bash
  cp ./TuyaOpen/apps/tuya.ai/ai_components/assets/include/lang_config.h ./ai_components/assets/include/
  ```

