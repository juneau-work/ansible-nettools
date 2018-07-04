-------------------------------------------------------------------------------
## 部署前 ##
1. 己安装好centos 7.3/7.4操作系统
2. 准备ansible环境  
说明: ansible和离线源需要一台额外的主机, 安装完成后即可回收主机

``` shell
# 安装pip
yum -y install python-setuptools
easy_install pip
# 安装ansible
pip install ansible
```

3. 下载dce_2.10 ansible playbook
``` shell
git clone https://github.com/juneau-work/dce_2.10
cd dce_2.10
```
> 以下操作都以dce_2.10为basedir

4. 配置认证信息
	- i. 复制以下内容生成vault.sh脚本
	``` shell
	cat <<'EOF' > vault.sh
	VAULT_ID='myVAULT@2018'
	echo $VAULT_ID > ~/.vault_pass.txt

	ANSIBLE_USER='root' # ssh用户名
	ANSIBLE_PASSWORD='root' # ssh用户密码
	DCE_USER='admin' # 具有admin权限的dce认证用户
	DCE_PASSWORD='admin' # 具有admin权限的dce认证用户密码

	ansible-vault encrypt_string --vault-id ~/.vault_pass.txt $ANSIBLE_USER --name 'vault_ansible_user' | tee dev/group_vars/vault
	ansible-vault encrypt_string --vault-id ~/.vault_pass.txt $ANSIBLE_PASSWORD --name 'vault_ansible_password' | tee -a dev/group_vars/vault
	ansible-vault encrypt_string --vault-id ~/.vault_pass.txt $DCE_USER --name 'vault_dce_user' | tee -a dev/group_vars/vault
	ansible-vault encrypt_string --vault-id ~/.vault_pass.txt $DCE_PASSWORD --name 'vault_dce_password' | tee -a dev/group_vars/vault
	EOF
	```
	**注意:** 请务必修改脚本中的ssh用户名密码及dce认证用户名密码与实际环境匹配
	- ii. 执行脚本
	``` shell
	bash vault.sh
	```
	**提示**: 后期新增集群节点时，请通过本步骤更新用户认证信息
   
   
   
   
   
-------------------------------------------------------------------------------
## ethernet ##
```shell
ansible-playbook -i dev/hosts --vault-password-file ~/.vault_pass.txt ethernet.yml
```


   
   
   
   
   
-------------------------------------------------------------------------------
## 错误解决 ##
#### 1. The following packages have pending transactions ####
报错:
```txt
failed: [192.168.130.11] (item=[u'kernel-devel', u'kernel-headers']) => {"changed": false, "item": ["kernel-devel", "kernel-headers"], "msg": "The following packages have pending transactions: kernel-headers-x86_64", "rc": 125, "results": ["kernel-devel-3.10.0-693.el7.x86_64 providing kernel-devel is already installed"]}  
```
原因:  网络不稳定或人为等原因造成yum安装包的事务没走完，需要清理掉未完成安装的事务   
解决: 
> ansible -i dev/hosts all --vault-password-file ~/.vault_pass.txt -e @dev/group_vars/vault -m shell -a 'yum -y install yum-utils && yum-complete-transaction --cleanup-only'
