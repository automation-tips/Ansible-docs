# 【備忘録】Ansible② Playbookとインベトリの書き方

### 環境

・インスタンスタイプ ： T2.micro
・OS ： Amazon Linux 2 



### インストール

 Amazon Linux2 環境ではExtras LibraryにAnsibleパッケージが含まれているので、

以下のコマンドですぐにインストールできる。（Pythonパッケージでのインストールもできる）



```shell
$ sudo amazon-linux-extras install -y ansible2
$ ansible --version
ansible 2.x.x
```



### Ansibleの実行に必要な要素

Ansibleを実行するには、以下のインプットが必要となる。



| 要素           | 備考                                                         |
| -------------- | ------------------------------------------------------------ |
| Playbook       | 実行する内容を記述したファイル                               |
| インベントリ   | Playbookの実行対象のホスト情報を記述した設定ファイル         |
| クレデンシャル | 実行対象に接続するための認証情報（ログインに使用する）       |
| 変数           | Playbook内で変数化された部分に値を設定（使用しない場合あり） |



### インベントリの設定

インベントリファイルは /etc/ansible/ansible.cfg にインベントリファイルの参照先を追記する。（例では /etc/ansible/hosts をインベントリファイルとする設定）

```
[defaults]

# some basic default values...

↓ この行がコメントアウトされているので#を削除する
inventory      = /etc/ansible/hosts
#library        = /usr/share/my_modules/
#module_utils   = /usr/share/my_module_utils/
#remote_tmp     = ~/.ansible/tmp
#local_tmp      = ~/.ansible/tmp
#forks          = 5
#poll_interval  = 15
#sudo_user      = root
#ask_sudo_pass = True
#ask_pass      = True
#transport      = smart
#remote_port    = 22
```



インベントリファイルはすでに存在するが、中身を消して下記の通り設定。



[文字列] は各ホストが属するグループを指定する際に使用する。例えば、webサーバー → [web]、アプリサーバー → [app]、DBサーバー → [db] となる。

各グループに複数の接続先ホストを記述することができる。



ansible_connection は接続の方法を指定する際に使用する。

指定しない場合、sshが選択される。sshの場合は実行時にssh接続と認証が行われる。



ansible_host は接続先ホストのIPアドレスと指定するオプションで、接続先ホストに別名をつけたい時に使用する（したの例では node-1 、node-2 という別名をつけている）。



```
[group1]
127.0.0.1 ansible_connection=local

[group2]
127.0.0.1 ansible_connection=local

[nodes]
node-1 ansible_host=127.0.0.1 ansible_connection=local
node-2 ansible_host=127.0.0.1 ansible_connection=local
```



## Playbookを変数と接続先ホストを指定して実行

ここでは、インベントリファイルで指定したグループで変数（実行時引数）に指定した文字列を表示するPlaybookを作成する。



Playbookの作成

```
vi ~/sample.yml
```



sample.ymlの内容

```
- hosts: group1  # ここでグループを指定
  vars:
    sample_args: "書き換えられる文字列"
  tasks:                        # tasks で定義された内容が実行される
    - debug:
        msg: "{{ sample_args }}"  # 変数を展開する場合 "{{}}" を使用する
  
```



作成後、以下のコマンドでPlaybookを実行する。

-e '変数名=設定したい値' でPlaybook内で宣言した変数に値を渡すことができる。



```
$ ansible-playbook sample.yml -e 'sample_args=上書き後の文字 列'

PLAY [group1] ***************************************************************************

TASK [Gathering Facts] ******************************************************************
ok: [127.0.0.1]

TASK [debug] ****************************************************************************
ok: [127.0.0.1] => {
    "msg": "上書き後の文字列"
}

PLAY RECAP ******************************************************************************
127.0.0.1                  : ok=2    changed=0    unreachable=0    failed=0 
```



変数を渡さなかった場合、Playbook内で宣言された値が表示される。



```
ansible-playbook sample.yml

PLAY [group1] ***************************************************************************

TASK [Gathering Facts] ******************************************************************
ok: [127.0.0.1]

TASK [debug] ****************************************************************************
ok: [127.0.0.1] => {
    "msg": "書き換えられる文字列"
}

PLAY RECAP ******************************************************************************
127.0.0.1                  : ok=2    changed=0    unreachable=0    failed=0 
```

