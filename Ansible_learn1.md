# 【備忘録】Ansible① 概要

## Ansibleとは

サーバー構築を自動化するツール

主に構成管理やアプリケーションのデプロイ、継続的デリバリーを自動で実行するためのツール（他の使い方もできる）

Playbookと呼ばれるPlaybookと呼ばれるファイルに設定したい情報を記載するだけ上述した自動化を実行できる。



Ansibleで操作できる対象

| 対象             | 操作                                                         |
| :--------------- | ------------------------------------------------------------ |
| 物理サーバー     | パワーON/OFF                                                 |
| OS               | パッケージインストール、設定ファイルの配布、設定値の変更、再起動 |
| ネットワーク機器 | インターフェース設定、VLAN設定など各種コンフィグの投入（やったことない） |
| ストレージ       | ボリュームの作成、編集、削除                                 |
| 仮装基盤         | 仮想マシン、仮装ネットワークの作成、編集、削除               |
| クラウド         | インスタンス、ネットワーク、セキュリティグループの作成、編集、削除 |

　  

ファイルの記述自体はとても可読性が高く、組織間での共有がしやすいかつ学習コストが低い。

Ansibleは、設定先のサーバーにエージェントのインストールが不要ないわゆるエージェントレスで利用できる

自動化ツール（ssh等で接続するための認証の設定は必要）



例）Apacheのインストール、httpd設定ファイル、Apacheの起動を自動化する場合

PlaybookはYAML形式で記述する。拡張子は yml

```YAML
- name: インストール、セットアップ、Apache起動
  hosts: localhost  # 対象サーバーを指定 ログインが必要なサーバーの場合は別途 ssh の設定が必要
  becode: yes  # 管理者権限で実行する場合
  vars:
    http_port: 80  # ポート80を指定
  tasks:

  - name: Apache（httpd）のインストール  # 実行内容を記載
    yum:
      name: httpd  # yum install -y httpd と同義
      state: latest
    become: True

  - name: Apache（httpd）の自動起動設定
    systemd:
      name: httpd
      state: started
      enabled: True
    become: True
 
  - name: html ファイルの配置
    copy:
      src: files/index.html
      dest: /var/www/html/
  
  - name: Apache（httpd）の起動
    service:
      name: httpd
      state: started
    become: True

```



Playbookの管理、監視ツールとして Ansible Tower（OSS版はAWX）がある。

