# 🔥試玩Ansible

## 1.1 Hello World

### 建立專案目錄

```bash
  $ mkdir ansible_lab && cd ansible_lab
```

### 安裝Ansible

**使用Python安裝**

```Python
  # 最好建立一個虛擬環境或--user方式安裝
  $ python3 -m vnen .venv
  $ source .venv/bin/activate
  $ pip install ansible-core
  or
  $ pip install --user ansible-core
```

**使用OS套件管理程式**

```bash
  $ dnf install ansible-core
```

### ansible.cfg設定檔

```bash
  # 使用ansible-config導出一個sample file
  $ ansible-config init --disabled > ansible.cfg

  # 建立相關目錄
  $ mkdir /home/edwin/ansible_lab/roles
  $ mkdir /home/edwin/ansible_lab/mycollection

  # 修改設定檔
  [defaults]
  inventory=/home/edwin/ansible_lab/inventory
  remote_user=edwin
  host_key_checking=False
  roles_path=/home/edwin/ansible_lab/roles:/usr/share/ansible/roles:/etc/ansible/roles
  collections_path=/home/edwin/ansible_lab/mycollection:/usr/share/ansible/collections
  interpreter_python=auto_silent

  [privilege_escalation]
  become=True
```

### 建立inventory

#### Inventory可將受管理節點以組織或類別等方式分開管理。

#### inventory可以為ini或yaml格式

ini格式適合少量受管理節點, 較為簡單
```ini
  [myhosts]
  10.0.1.4
  10.0.1.2
  10.0.2.10
```

在具有大量受管理節點時, 使用yaml格式更為理想
```yaml
myhosts:
  hosts:
    my_host_01:
      ansible_host: 10.0.1.4
    my_host_02:
      ansible_host: 10.0.1.2
    my_host_03:
      ansible_host: 10.0.2.10
```

### SSH Passwordless Authentication

```shell
  $ ssh-keygen
  Enter file in which to save the key (/home/edwin/.ssh/id_ed25519):
  Enter passphrase for "/home/edwin/.ssh/id_ed25519" (empty for no passphrase):
  Enter same passphrase again:
  Your identification has been saved in /home/edwin/.ssh/id_ed25519
  Your public key has been saved in /home/edwin/.ssh/id_ed25519.pub
  The key fingerprint is:
  SHA256:7Kmq42p+3ZbdDRFjw8WDLxFnEHfWYDKO8AN8eL5/oNA edwin@rocky-test
  The key's randomart image is:
  +--[ED25519 256]--+
  |       .o..+X=o+.|
  |        o+o@+*o .|
  |         += B .  |
  |       .  .+ .   |
  |        S. .o    |
  |       ...E..    |
  |    . . =..oo.   |
  | ... . = ......  |
  |+++o..o      .   |
  +----[SHA256]-----+

  $ ssh-copy-id edwin@10.0.1.2    #將公鑰複製給各台被管理節點
```


### 檢驗inventory

```shell
  $ ansible-inventory -i inventory --list
  {
    "_meta": {
        "hostvars": {},
        "profile": "inventory_legacy"
    },
    "all": {
        "children": [
            "ungrouped",
            "myhosts"
        ]
    },
    "myhosts": {
        "hosts": [
            "10.0.1.4",
            "10.0.1.2",
            "10.0.2.10"
        ]
    }
  }
```

### 使用inventory執行ping指令

```shell
  $ ansible myhosts -m ping -i inventory.yaml
  my_host_03 | SUCCESS => {
      "ansible_facts": {
          "discovered_interpreter_python": "/usr/bin/python3.12"
      },
      "changed": false,
      "ping": "pong"
  }
  my_host_02 | SUCCESS => {
      "ansible_facts": {
          "discovered_interpreter_python": "/usr/bin/python3.12"
      },
      "changed": false,
      "ping": "pong"
  }
  my_host_01 | SUCCESS => {
      "ansible_facts": {
          "discovered_interpreter_python": "/usr/bin/python3.12"
      },
      "changed": false,
      "ping": "pong"
  }
```

## 1.2 ad hoc commands

### 如果只有單一任務且很少會重覆時, ad hoc就很適合這樣的類型任務, 一個ad hoc command會像這樣:

```shell
  $ ansible [pattern] -m [module] -a "[module options]"
```

### 例如要重啟server, 可以一台或多台同時重啟

```shell
  $ ansible myhosts -a "/sbin/reboot"
```

### Ansible預設只使用五個同時進行的程序, 如果你的主機數量超過分支數量設定的值, 可能會增加Ansible與主機溝通的時間。
### 要重啟擁有 10 個平行分支的 [myhosts] 伺服器：

```shell
  $ ansible myhosts -a "/sbin/reboot" -f 10
```

### 預設使用的模組是ansible.builtin.command, 可以依需求使用其它模組
### ansible.builtin.shell

```shell
  $ ansible myhosts -m ansible.builtin.shell -a 'echo $TERM'
```

### 檔案管理模組
### ansible.builtin.copy

```shell
  $ ansible myhosts -m ansible.builtin.copy -a "src=/etc/hosts dest=/tmp/hosts"
```

### Ansible有很多模組可用, 以ansible-doc來查看有什麼模組可用

```shell
  $ ansible-doc -l
  amazon.aws.autoscaling_group                                                                                        
  amazon.aws.autoscaling_group_info                                                                                   
  amazon.aws.autoscaling_instance                                                                                     
  amazon.aws.autoscaling_instance_info                                                                                
  amazon.aws.autoscaling_instance_refresh                                                                             
  amazon.aws.autoscaling_instance_refresh_info                                                                        
  amazon.aws.aws_az_info                                                                                              
  amazon.aws.aws_caller_info
  ...................
```

### 查詢特定模組的說明

```shell
  $ ansible-doc copy
```

## 1.3 Playbook

playbook.yaml
```yaml
 - name: My first play
  hosts: myhosts
  tasks:
   - name: Ping my hosts
     ansible.builtin.ping:

   - name: Print message
     ansible.builtin.debug:
       msg: Hello world
```

playbook語法檢查

```shell
  $ ansible-playbook --syntax-check playbook.yaml
  [ERROR]: YAML parsing failed: Mapping values are not allowed in this context.
  Origin: /home/edwin/ansible_lab/playbook.yaml:2:9
  
  1 - name: My first play
  2    hosts: myhosts
            ^ column 9

```

執行playbook

```shell
  $ ansible-playbook playbook.yaml
  PLAY [My first play] ************************************************************************************************

  TASK [Gathering Facts] **********************************************************************************************
  ok: [10.0.2.10]
  ok: [10.0.1.2]
  ok: [10.0.1.4]
  
  TASK [Ping my hosts] ************************************************************************************************
  ok: [10.0.2.10]
  ok: [10.0.1.4]
  ok: [10.0.1.2]
  
  TASK [Print message] ************************************************************************************************
  ok: [10.0.1.4] => {
      "msg": "Hello world"
  }
  ok: [10.0.1.2] => {
      "msg": "Hello world"
  }
  ok: [10.0.2.10] => {
      "msg": "Hello world"
  }
  
  PLAY RECAP **********************************************************************************************************
  10.0.1.2                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
  10.0.1.4                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
  10.0.2.10                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

