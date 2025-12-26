# 版本
本文档适用于 2FA 系统 1.0.x 版本

# 流程
1. 获取 pam_2fa-{version}-Linuxx.deb 包
	- 获取 deb 包的方法
		- [gitlab 2FA 仓库](http://172.16.1.220/yudt/2FA) 的 release 获取
		- 从 gitlab CI 中产生的 artifacts 中获取最新到未 release 的包
			-  [示例链接](http://172.16.1.220/yudt/2FA/-/jobs/295/artifacts/browse/dist/)
			- 示例链接并非一直是最新的, 请自行选择 最小的 job 中打包到 deb 
		- 自编译 2FA 仓库中到 pam 模块
2. 获取密钥 `pam_2fa.toml`
	- 管理员通过密钥生成器生成
3. 在目标机器上 使用 apt install deb 包
	- 注意这里使用 ./pam_2fa-{version}-Linux.deb, 否则 apt 会在远程仓库中寻找名为 pam_2fa-{version}-Linuxx.deb 的包
4. 将密钥拷贝到 目标机器上
	- 路径为 `/etc/pam_2fa.toml`
5. 确保 sshd service 启动
6. 测试  pam_2fa 模块是否成功加载 
	1. 输入 `ssh user@localhost`
	2. 输入 user 密码后查看是否有 如下截图显示提示根据 UUID 和挑战码来向管理员来获取口令
		- ![[prompt.png]]
	3. 输入口令查看是否能正确登录到机器

# 错误排查
## 未预期的情况检查的步骤
1. 查看密钥是否存在于 `/etc/pam_2fa.toml`
2. 查看 sshd 配置是否正确
	1. 查看 `/etc/ssh/sshd_config` 中是否 include 了 `99-pam_2fa.conf`
	2. 查看 `/etc/ssh/sshd_config.d/` 下是否存在 `99-pam_2fa.conf`
	3. 检查 `sshd_config` 中是否覆盖了`99-pam_2fa.conf` 中到设置

3. 查看 pam module sshd config 是否正确
	1. 检查 `/etc/pam.d/sshd` 中是否有 `auth required pam_2fa.so`
	2. 检查 `/usr/lib/x86_64-gnu-linux/security` 下是否存在 pam_2fa.so 
4. 检查sshd service日志
	1. `journalctl -u ssh`
	2. 检查日志中的最近一次错误
## 错误汇报
1. 未能排查到错误请发布问题以及上下文在 [2FA Issues](http://172.16.1.220/yudt/2FA/-/issues) 并通知开发者
	- 错误上下文
		- 错误的日志
		- PAM 模块的版本
			- 查看 `/usr/lib/x86_64-gnu-linux/security` 下的pam_2fa.so版本
		- `sshd_config` 和 `99-pam_2fa.conf`
		- `etc/pam.d/sshd`