# Core Dump 完整解决方案

## 问题：为什么没有生成 coredump 文件？

生成 coredump 需要**内核支持**，主要需要以下内核配置项：
- `CONFIG_ELF_CORE=y`
- `CONFIG_COREDUMP=y`

如果内核没有启用这些选项，即使代码正确，也**无法**生成 coredump 文件。

---

## 第一步：检查内核是否支持 Core Dump

运行检查脚本：

```bash
chmod +x check_coredump_support.sh
./check_coredump_support.sh
```

或者手动检查：

```bash
# 方法1: 检查 core_pattern 文件是否存在
ls -l /proc/sys/kernel/core_pattern

# 方法2: 尝试查看内核配置
zcat /proc/config.gz | grep CONFIG_ELF_CORE
# 或
cat /boot/config-$(uname -r) | grep CONFIG_ELF_CORE

# 方法3: 检查内核符号
cat /proc/kallsyms | grep core_dump
```

---

## 方案 A：内核支持 Core Dump

如果 `/proc/sys/kernel/core_pattern` 文件存在，说明内核支持 coredump。

### 1. 启用 Core Dump

```bash
# 设置 core 文件大小限制
ulimit -c unlimited

# 设置 core 文件保存路径和命名
echo "/tmp/core-%e-%p-%t-%s" > /proc/sys/kernel/core_pattern
# 或保存到 SD 卡
echo "/mnt/sd/core-%e-%p-%t-%s" > /proc/sys/kernel/core_pattern

# 确保目录可写
chmod 777 /tmp  # 或 /mnt/sd
```

### 2. 在启动脚本中添加

```bash
#!/bin/sh

# 启用 core dump
ulimit -c unlimited
echo "/tmp/core-%e-%p-%t-%s" > /proc/sys/kernel/core_pattern

# 运行程序
./your_program
```

### 3. 分析 Core Dump

```bash
# 传输 core 文件到开发机
scp root@192.168.1.100:/tmp/core-* ./

# 使用 GDB 分析
mipsel-linux-gdb /path/to/binary /path/to/core-file

# GDB 命令
(gdb) bt full              # 完整调用栈
(gdb) info threads         # 所有线程
(gdb) thread apply all bt  # 所有线程调用栈
```

---

## 方案 B：内核不支持 Core Dump（备用方案）

如果 `/proc/sys/kernel/core_pattern` 不存在，需要使用备用方案。

### 选项 1：重新编译内核（推荐，但需要时间）

```bash
# 1. 进入内核源码目录
cd /path/to/Ingenic-SDK-T41/opensource/kernel-4.4.94/

# 2. 配置内核
make menuconfig
# 导航到: General setup -> Configure standard kernel features
#         -> [*] Enable support for core dumps (CONFIG_ELF_CORE)

# 3. 编译内核
make uImage -j$(nproc)

# 4. 更新到设备
# 备份原有 uImage
cp /path/to/device/uImage /path/to/device/uImage.bak
# 复制新的 uImage
cp arch/mips/boot/uImage /path/to/device/
# 重启设备
```

参考脚本：
```bash
chmod +x kernel_coredump_config.sh
./kernel_coredump_config.sh
```

### 选项 2：使用手动崩溃信息保存（快速方案）

如果无法重新编译内核，可以修改代码手动保存崩溃信息。

#### 替换信号处理函数

将 `alternative_crash_handler.c` 中的代码替换到 `nest_ipc/main/main.c` 的 `SignalHandleKill` 函数。

这个备用方案会保存：
- ✅ 崩溃时间、信号类型、PID
- ✅ 进程命令行参数
- ✅ 内存映射 (/proc/self/maps)
- ✅ 调用栈 (backtrace，如果支持)
- ✅ 所有线程列表和名称
- ✅ 进程状态信息

#### 崩溃信息保存位置

崩溃时会生成文件：
- `/mnt/sd/crash-<PID>-<timestamp>.txt` (如果 SD 卡可写)
- `/tmp/crash-<PID>-<timestamp>.txt` (备选)

#### 分析崩溃信息

```bash
# 查看崩溃文件
cat /tmp/crash-*.txt

# 重点关注：
# 1. Backtrace - 调用栈
# 2. All Threads - 找出哪个线程崩溃
# 3. Memory Maps - 查看加载的库和地址
```

---

## 对比：两种方案优缺点

### Core Dump (方案 A)
✅ 优点：
- 完整的内存快照
- 可以用 GDB 交互式调试
- 可以查看所有变量值
- 工业标准，工具支持好

❌ 缺点：
- 需要内核支持
- 文件较大（几十到几百 MB）
- 需要传输到开发机分析

### 手动保存崩溃信息 (方案 B)
✅ 优点：
- 不需要内核支持
- 文件小（几 KB）
- 可以直接在设备上查看
- 实现简单

❌ 缺点：
- 信息不完整
- 无法查看变量值
- 需要根据调用栈手动分析
- 如果没有 backtrace 支持，信息更少

---

## 快速决策流程图

```
是否有内核源码访问权限？
├─ 有 → 重新编译内核启用 CONFIG_ELF_CORE (最佳方案)
└─ 没有
   └─ /proc/sys/kernel/core_pattern 是否存在？
      ├─ 存在 → 使用 Core Dump (方案 A)
      └─ 不存在 → 使用手动崩溃信息保存 (方案 B)
```

---

## T41 平台特殊说明

Ingenic T41 平台使用的是定制内核，默认配置可能不支持 core dump。

### 检查你的 T41 内核

```bash
uname -a
# 输出类似: Linux (none) 4.4.94 #1 PREEMPT ...

ls -l /proc/sys/kernel/core_pattern
# 如果文件存在 → 支持 coredump
# 如果文件不存在 → 不支持 coredump
```

### 获取支持 Core Dump 的内核

1. **联系 Ingenic 技术支持**
   - 要求提供启用了 `CONFIG_ELF_CORE` 的内核镜像
   
2. **使用 SDK 重新编译**
   - SDK 路径：`Ingenic-SDK-T41-1.2.5-20241024-zh/opensource/kernel-4.4.94/`
   - 按照上面的编译步骤操作

3. **使用备用方案**
   - 如果以上都不可行，使用 `alternative_crash_handler.c`

---

## 推荐配置

### 开发阶段（推荐方案 B）
- 使用手动崩溃信息保存
- 快速获取基本崩溃信息
- 不需要修改内核

### 生产调试阶段（推荐方案 A）
- 如果可能，启用 Core Dump
- 获取完整崩溃信息
- 更容易定位复杂问题

---

## 总结

1. **先运行** `check_coredump_support.sh` 检查内核支持
2. **如果支持**：使用 Core Dump（方案 A）
3. **如果不支持**：
   - 优先考虑重新编译内核
   - 快速方案：使用手动崩溃信息保存（方案 B）
4. 两种方案可以**并存**，代码会先尝试生成 core dump，失败后保存文本信息

---

## 相关文件

- `check_coredump_support.sh` - 检查内核是否支持 core dump
- `kernel_coredump_config.sh` - 内核配置指南
- `alternative_crash_handler.c` - 备用崩溃处理函数代码
- `COREDUMP_GUIDE.md` - Core dump 详细使用指南

## 技术支持

如有问题，请提供：
1. `uname -a` 输出
2. `ls -l /proc/sys/kernel/core_pattern` 结果
3. SDK 版本信息
4. 崩溃日志或崩溃信息文件
