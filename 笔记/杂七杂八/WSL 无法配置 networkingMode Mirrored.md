---
aliases: [WSL无法配置networkingMode Mirrored，报错0x8007054f解决方案]
tags: [WSL]
creat: 2026-03-15T23:23:22+08:00
mod: 2026-03-16T01:21:24+08:00
publish: "true"
---

## 原因
根据 [Error code: CreateInstance/CreateVm/ConfigureNetworking/0x8007054f · Issue #12351 · microsoft/WSL](https://github.com/microsoft/WSL/issues/12351)，有三种可能：
1. Windows 与 Linux 端口冲突
2. WSL/HNS 网络配置错乱
3. swap.vhdx 状态异常
## 解决
可以直接按这个方式全试一遍，说不定就能解决了。
### 查端口
一般的 Linux 发行版，在默认情况下可能会占用的主要是 **53** 端口，在 Linux 上可以使用 `ss tuln` 查询：
![Pasted image 20260316005956](https://nbb-1313023833.cos.ap-chengdu.myqcloud.com/Pasted%20image%2020260316005956.png)
我使用的发行版占用了 53 和 5355 两个端口，查询到 Windows 只占用了53，处理53就好。
在Windows上使用 `netstat -ano | findstr 53` 查询进程。
排查结论是：
![Pasted image 20260316010423](https://nbb-1313023833.cos.ap-chengdu.myqcloud.com/Pasted%20image%2020260316010423.png)
### 重置网络设置
直接在 powershell 敲：
```powershell
wsl --shutdown
# restart HNS services
net stop hns
net start hns

# reset WSL network configurations
netsh winsock reset
netsh int ip reset
```
记得用管理员终端
### 修复 swap.vhdx
我们需要清理 `%TEMP%\<GUID>\swap.vhdx`。
**在 WSL 启动的情况下**，文件资源管理器访问：
![Pasted image 20260316010933](https://nbb-1313023833.cos.ap-chengdu.myqcloud.com/Pasted%20image%2020260316010933.png)
找到类似 `2FF0006A-6F05-45AD-86A2-2AB40F74ED43` 这样的一个\<GUID\>文件夹，如果发现里面只有一个 `swap.vhdx` 就对了。
然后操作：
1. 尝试删除父`<GUID>`文件夹`swap.vhdx`。如果提示占用，**请不要点击“重试”**。
2. Powershell 执行 `wsl --shutdown`
3. 在执行完毕的那一刻点重试，顺利删除就成功了
不过我没成功过，每次都被WSL自己删了，但可以试试。

最后重启电脑，再次打开 WSL 测试，大概率没问题了。