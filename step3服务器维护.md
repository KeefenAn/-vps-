# Step 3：服务器维护

## 为什么要做这一步

这类 NAT 小鸡的硬盘普遍很小（根分区可能只有 1 GB 左右）。而 3x-ui 和 Xray 的日志、系统日志、各种缓存会一天天往上涨，慢慢把存储占满。

**一旦存储写满，后果很直接：面板可能打不开、Xray 起不来、更没法再新增入站。** 所以给日志设个上限，再配上自动清理，是这类小硬盘服务器的必修课。

本篇的思路是三层防护：

1. **限制系统日志（journald）的大小**：最多约 30 MB，只保留最近 2 天；
2. **每日自动清理**：每天凌晨 4 点清一次缓存和旧日志；
3. **磁盘保护**：每小时检查一次，根分区用到 80% 以上才启动加强清理。

---

## 一、先看现状（只看不删）

动手之前，先确认磁盘用了多少、日志占了多少、已有的定时任务还在不在：

```bash
# 看根分区使用率
df -h /

# 看系统日志占了多少
journalctl --disk-usage

# 看现有的定时任务
crontab -l

# 看清理脚本是否已经存在
ls -l /usr/local/bin/vps-auto-clean.sh /usr/local/bin/disk-guard.sh
```

## 二、限制 journald 日志大小

先看看当前配置：

```bash
grep -E '^(SystemMaxUse|MaxRetentionSec)=' /etc/systemd/journald.conf
```

编辑配置文件：

```bash
nano /etc/systemd/journald.conf
```

确认有下面两行（文件里已有同名项就改那一行，不要重复添加）：

```ini
SystemMaxUse=30M
MaxRetentionSec=2day
```

保存后重启 journald 使设置生效，再确认一次占用：

```bash
systemctl restart systemd-journald
journalctl --disk-usage
```

## 三、每日自动清理脚本

新建脚本 `/usr/local/bin/vps-auto-clean.sh`（如果服务器上已有这个文件，先 `cat` 看一眼再决定要不要替换）：

```bash
#!/usr/bin/env bash
set -u

# 压缩/清理 systemd journal：最多保留 2 天，限制在约 30 MB
journalctl --vacuum-time=2d || true
journalctl --vacuum-size=30M || true

# 清理 APT 下载缓存
if command -v apt-get >/dev/null 2>&1; then
  apt-get clean
  apt-get autoclean
fi

# 清理超过 7 天的轮转日志压缩文件
find /var/log -type f \( -name "*.gz" -o -name "*.old" \) -mtime +7 -print -delete 2>/dev/null || true

# 清理 /tmp 中超过 7 天未访问的普通文件
find /tmp -xdev -type f -atime +7 -print -delete 2>/dev/null || true

echo "$(date '+%F %T') daily cleanup finished" >> /var/log/vps-auto-clean.log
```

设置执行权限，并加入 root 的定时任务，每天凌晨 4 点运行：

```bash
chmod 750 /usr/local/bin/vps-auto-clean.sh
crontab -e
```

```cron
0 4 * * * /usr/local/bin/vps-auto-clean.sh
```

## 四、每小时磁盘保护脚本

再建一个"守门员"脚本 `/usr/local/bin/disk-guard.sh`：每小时检查一次根分区，**只有使用率达到 80% 才动手加强清理**，平时直接退出，不折腾：

```bash
#!/usr/bin/env bash
set -u

THRESHOLD=80
USAGE=$(df -P / | awk 'NR==2 {gsub("%","",$5); print $5}')

# 读不到使用率时不删任何东西，直接报错退出
if ! [[ "$USAGE" =~ ^[0-9]+$ ]]; then
  logger -t disk-guard "Unable to read root filesystem usage; skipped"
  exit 1
fi

# 没到阈值就直接退出
if [ "$USAGE" -lt "$THRESHOLD" ]; then
  exit 0
fi

logger -t disk-guard "Root filesystem usage is ${USAGE}%; running cleanup"

# 加强清理 journald：目标约 20 MB、最近 1 天
journalctl --vacuum-time=1d || true
journalctl --vacuum-size=20M || true

# 清理 APT 缓存
if command -v apt-get >/dev/null 2>&1; then
  apt-get clean
  apt-get autoclean
fi

# 清理超过 3 天的轮转日志压缩文件
find /var/log -type f \( -name "*.gz" -o -name "*.old" \) -mtime +3 -print -delete 2>/dev/null || true

# 清理 /tmp 中超过 3 天未访问的普通文件
find /tmp -xdev -type f -atime +3 -print -delete 2>/dev/null || true

logger -t disk-guard "Cleanup completed; root usage was ${USAGE}% at start"
```

设置权限，并加入定时任务（每小时第 5 分钟检查一次；crontab 里已有这条就别重复加）：

```bash
chmod 750 /usr/local/bin/disk-guard.sh
crontab -e
```

```cron
5 * * * * /usr/local/bin/disk-guard.sh
```

## 五、验证是否生效

```bash
# 手动跑一次每日清理（会真的执行清理）
/usr/local/bin/vps-auto-clean.sh

# 手动跑一次磁盘保护（低于 80% 时应该检查完就退出）
/usr/local/bin/disk-guard.sh

# 再看一眼磁盘和日志占用
df -h /
journalctl --disk-usage

# 看磁盘保护脚本最近 24 小时的运行记录
journalctl -t disk-guard --since "24 hours ago"

# 看每日清理的日志
tail -n 50 /var/log/vps-auto-clean.log
```

对照检查一遍：

- 根分区使用率回到合理水平；
- journald 占用在 30 MB 左右或更低；
- crontab 里有每天 04:00 和每小时第 5 分钟两条任务；
- 低于 80% 时 disk-guard 不执行加强清理。

## 六、注意事项

- **改之前先看**：服务器上已有的脚本和 crontab 先看清楚，再决定改不改，不要盲目覆盖。
- 脚本里的 `find ... -delete` 是真删文件。不放心的话，先把 `-delete` 去掉只跑一遍，看打印出来的命中列表对不对，再加回去。
- **不要删**整个 `/var/log` 目录，更不要动 3x-ui / Xray 正在用的数据库、配置文件和证书。
- 清理日志文件 `/var/log/vps-auto-clean.log` 本身也会慢慢变大，长期用记得给它配 logrotate 或限制大小。
- 根分区只有 1 GB 左右的话，清理只能缓解；如果空间还是反复被吃满，要进一步查日志增长源头、面板数据库、系统更新缓存等大头。
- 非 Debian / Ubuntu 系统没有 apt-get，脚本里已经做了判断，会自动跳过那一段。

---

**上一篇：** [Step 2：故障排查](step2故障排查.md) ｜ **回到目录：** [README](README.md)
