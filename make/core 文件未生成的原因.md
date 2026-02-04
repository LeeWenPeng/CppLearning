当 `core file size` 已设置为 `unlimited` 但依然无法生成 core 文件时，可能是以下原因导致。请按以下步骤排查：

---

## 1  系统 core 文件数量限制为 0

查看系统生成 core 文件的限制

```bash
ulimit -a

ulimit -c
```

修改系统生成 core 文件的限制

```bash
ulimit -c ulimited
```

- **注意**：某些场景（如通过 `sudo` 或 `systemd` 启动的服务）可能需要单独配置。

---

## 2  检查系统全局 core dump 配置

- **查看内核参数 `core_pattern`**：

  ```bash
  cat /proc/sys/kernel/core_pattern
  ```

  - 如果输出是 `|/path/to/program`（如 `|/usr/share/apport/apport`），表示 core 文件被重定向到某个程序处理（如 Ubuntu 的 Apport），不会生成传统 core 文件。
  - **解决方法**：修改 `core_pattern` 为文件路径（需 root 权限）：

	```bash
    echo "core.%p.%e" | sudo tee /proc/sys/kernel/core_pattern
    # 或永久生效（编辑 /etc/sysctl.conf）：
    sudo sysctl -w kernel.core_pattern=core.%p.%e
    ```

- **检查 `suid_dumpable` 设置**（针对 SUID 程序）：

  ```bash
  cat /proc/sys/fs/suid_dumpable
  ```

  - 如果值为 `0`，SUID 程序不会生成 core。设为 `2`：

	```bash
    sudo sysctl -w fs.suid_dumpable=2
    ```

---

## 3  检查文件系统和目录权限

- **存储路径权限**：core 文件默认生成在进程的工作目录，确保该目录有写入权限。
- **磁盘空间**：确认磁盘未满。
- **文件系统挂载选项**：检查是否挂载时使用了 `noexec` 或 `nodev`，临时关闭测试：

  ```bash
  mount -o remount,exec,dev /path/to/directory
  ```

---

## 4  系统级限制（systemd 或 limits.conf）

- **针对 systemd 服务**：修改 `/etc/systemd/system.conf`，添加：

```ini
  DefaultLimitCORE=infinity


```

  重启 systemd：

  ```bash
  sudo systemctl daemon-reload
  ```

- **/etc/security/limits.conf**：确保未限制 core 文件大小：

  ```conf
  * soft core unlimited
  * hard core unlimited
  ```

---

## 5  第三方工具拦截（如 Apport/ABRT）

- **Ubuntu 的 Apport**：禁用 Apport（编辑 `/etc/default/apport`）：

  ```conf
  enabled=0
  ```

  重启服务：

  ```bash
  sudo systemctl restart apport
  ```

- **RHEL/CentOS 的 ABRT**：停止服务：

  ```bash
  sudo systemctl stop abrtd
  ```

---

## 6  安全策略限制（SELinux/AppArmor）

- **临时禁用 SELinux**：

  ```bash
  sudo setenforce 0
  ```

- **检查 AppArmor 规则**：

  ```bash
  sudo aa-status
  ```

  如果进程受 AppArmor 限制，修改规则或设为 `complain` 模式。

---

## 7  验证程序是否真的触发了 Core Dump

- 编写测试程序 `test.c`：

  ```c
  #include <stdio.h>
  int main() { *(int*)0 = 0; return 0; } // 触发段错误
  ```

- 编译并运行：

  ```bash
  gcc -o test test.c
  ./test
  ```

- 检查是否生成 core 文件。

---

## 8  检查程序自身行为

- 程序是否捕获信号（如 `SIGSEGV`）并忽略？
- 是否在代码中调用了 `setrlimit(RLIMIT_CORE, …)` 限制 core 文件？

---

## 9  总结命令

```bash
# 1. 设置 ulimit
ulimit -c unlimited

# 2. 检查 core_pattern 和 suid_dumpable
sudo sysctl -w kernel.core_pattern=core.%p.%e
sudo sysctl -w fs.suid_dumpable=2

# 3. 验证测试程序
echo -e '#include <stdio.h>\nint main() { *(int*)0=0; }' | gcc -x c - && ./a.out
ls | grep core
```

根据以上步骤排查，通常可以解决 core 文件未生成的问题。
