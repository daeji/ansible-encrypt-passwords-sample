# Ansibleでパスワードを暗号化して管理する

Ansibleのplaybook内にsudoやsshパスワードを管理したい場合がよくありますが、プレーンテキストのままgitリポジトリに放り込むと非常に嫌な感じがします。暗号化した状態でもちょっと嫌ですが、生の状態よりマシかもしれません。
ホストグループ別にgroup_varsディレクトリにパスワードをyamlファイルを作成し、そのファイルを暗号化します。

## Ansible実行環境

| Item             | Value           |
|:-----------------|:----------------|
| OS               | Mac OS X 10.9.4 |
| Ansible          | 1.5.4           |

## 対象ホスト

| Item             | Value          |
|:-----------------|:---------------|
| 対象ホスト       | 192.168.240.34 |
| OS               | CentOS 7       |
| 認証方式         | パスワード認証 |
| sshユーザー      | ansible        |
| sshパスワード    | password       |
| sudoパスワード   | password       |
| 暗号化パスワード | vaultpass      |

## ansibleのファイル構成
```
.
├── hosts
├── group_vars
│   └── servers.yml
├── site.yml
├── roles
│   └── common
│       └── tasks
│           └── main.yml
└── vars
     └── main.yml
```

### 各ファイルの内容
* hosts

```
[servers]
192.168.240.34
```
* group_vars/servers.yml
    * hosts内に[]で囲って記述したグループ名.ymlをgroup_varsディレクトリに作成

```yaml
---
ansible_ssh_user: ansible
ansible_ssh_pass: password
ansible_sudo_pass: password
```
* site.yml
    * hosts
        * グループ名を指定
    * var_files
        * グローバルスコープの変数を記述したファイルパスを指定
    * roles
        * 実行するroleを指定

```yaml
---
- hosts: servers
  vars_files:
    - vars/main.yml
  roles:
      - common
```

* roles/common/tasks/main.yml

```yaml
- name: Install yum packages
  action: yum name={{ item }} state=installed
  with_items:
    - wget
    - tcpdump
    - bind-utils
    - telnet
```

* vars/main.yml

```yaml
---
sudo: yes
```

## 暗号化・復号

### group_vars/servers.ymlを暗号化してplaybookを実行

```
echo vaultpass > ~/.ansible_vault_pass # こいつはgitに格納しない
ansible-vault encrypt group_vars/servers.yml --vault-pass ~/.ansible_vault_pass
# --vault-pass を指定しない場合は、パスワードを聞いてくる
ansible-playbook site.yml -i hosts --vault-pass ~/.ansible_vault_pass
```

### group_vars/servers.yml を復号

```shell
ansible-vault decrypt group_vars/servers.yml --vault-pass ~/.ansible_vault_pass
```

### group_vars/servers.yml を暗号化したまま編集

```shell
ansible-vault edit group_vars/servers.yml --vault-pass ~/.ansible_vault_pass
```

