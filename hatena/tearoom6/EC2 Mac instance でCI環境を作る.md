re:Invent 冒頭で、いきなりビッグニュースが飛び込んで来ましたね、 AWS EC2 の Mac instance!
事前観測も自分の知る限り見なかったので、これは驚いた人が多かったのではないでしょうか!?

用途としては、まずは真っ先に iOS 向けの CI 環境構築が視野に入ってくるかと思います。
(価格的には、現時点ではなかなか視野には入ってこないですけど)

そこで、お遊びですが、実際に CI で使うことをイメージして Xcode の Simulator が動くかどうかを試してみたいと思います。

## まずは触ってみる

今日の時点で、 Mac instance が利用できるのは以下のリージョンとなっています。

- US East (N. Virginia)
- US East (Ohio)
- US West (Oregon)
- EU (Ireland)
- Asia-Pacific (Singapore)

早速 EC2 dashboard から作成してみます。

なお、後述しますが、はじめ試しに Singapore region で作ろうとしたんですが、リリース直後で人気があるためなのか、ホストを取得できなかったので、やむなく取得可能だった Ohio region で作成しました。

[Incetances] > [Launch instances] ボタンを押すと、早速 macOS が選べるようになっています。
Catalina or Mojave が選べるようですね。 (Big Sur は comming soon とブログには書かれています)

- macOS Catalina 10.15.7 - ami-00692c69a6f9c6ea1
- macOS Mojave 10.14.6 - ami-0f9804951520118c4

AMI ID は Ohio の場合です。

![ec2_macos_instance_choose_ami.png](https://files.tearoom6.biz/078e898f-853f-4b67-94e6-dc982bc712e3.png)

Catalina を選んでみました。

次は instance type 選択画面ですが、こちらは `mac1.metal` しか選べません。
bare metal instance とは、非仮想化で動く、ハード専有インスタンスのことです。
ざっくりといえば、クラウド上でレンタルサーバを動かしているようなイメージですかね。。
料金そこそこするので、起動しっぱなしにしないように注意です。
ブログによると、裏では Mac mini が動いているようです。

12 vCPUs, Memory 32GiB となっています。

![ec2_macos_instance_choose_instance_type.png](https://files.tearoom6.biz/1d82e155-8977-4790-ae74-458778fea70b.png)

次は instance detail 設定画面です。
こちら、下の方にスクロールしていくと、 [Host] の設定が求められているので、 [Allocate a new host] をクリックして、専有ホストを割り当てます。

![ec2_macos_instance_configure_instance_detail.png](https://files.tearoom6.biz/c61ea997-3566-46ad-b3ff-12fef556150b.png)

クリックすると [Allocate Dedicated Host] という画面が出てくるので、例えば、以下のような項目を入力して [Allocate] ボタンを押します。
(CLI コマンドがコピーできるようになっているので、後のためにコピーしておきましょう)

- Name tag: `tearoom6-test`
- Instance family: `mac1`
- Instance type: `mac1.metal`
- Availability Zone: `us-east-2b`
- Instance auto-placement: Enable
- Quantity: `1`

先ほど書いたように、はじめは Singapore region で取ろうとしたんですが、 `Insufficient capacity.` というエラーが出て、取れなかったので、上記のように Ohio region で取得しました。 (スクショと異なるのはそのためです・・・)

また、一度 allocate すると、少なくとも 24 時間は return できない点にも注意です。
dedicated host の単価は Ohio region で `$1.083` / hr となっていますので、特に割引オプションを使わなければ 24 時間で 3000 円近くかかるわけですね、たぶん。

![ec2_macos_instance_allocate_dedicated_host.png](https://files.tearoom6.biz/cf0077bf-844b-48c9-a764-187041350bef.png)

さて、 Host を取得した後、 instance 作成画面に戻ると、先ほどの [Host] の箇所が選べるようになっているので、取得できた Host を割り当てます。

![ec2_macos_instance_configure_instance_detail_host.png](https://files.tearoom6.biz/880707c1-a285-4df6-adbd-df84142b9afe.png)

EBS の設定画面では、デフォルトの 30 GiB となっていますが、それとは別に 200 GiB のディスクを `/dev/sdb` に付けておきました。
ここがよく分かっていないのですが、デフォルトの方をディスク容量を大きくしても、起動直後は 30 GiB しか容量が確保されていない状態になるのです...。
きっともっといい方法はある気がする。

Security Group の設定画面では SSH (TCP 22) と VNC (TCP 5900) を開けておきました。
(でも、今回のように VNC を SSH port forwarding で行う場合は、 SSH のみで十分ですし、その方が推奨されます)

![ec2_macos_instance_configure_security_group.png](https://files.tearoom6.biz/84a12c88-7b84-46f8-98f1-3f7b1a08a07d.png)

[Launch] ボタンを押すと、 Linux 系の instance を立ち上げた際と同様 SSH key を指定できるダイアログが出てくるので、新規作成します。

![ec2_macos_instance_ssh_key.png](https://files.tearoom6.biz/74f6ab6d-8232-4c72-bfcd-63d37dcfa23a.png)

Launch 後、 instance が使えるようになるまでは、結構時間がかかるようです。
Status checks のところが System status checks が passed になるのは比較的早いんですが、 Instance status checks の方がかなり時間がかかります。しかも、途中 `Instance reachability check failed` みたいなエラーが表示されるので、ハラハラします。

ちなみに instance を何らかの事情で一度 terminate した後、別の instance で Dedicated Host を使えるようになるまでも結構小一時間くらい待たされます。

無事 instance の Status checks に pass したら、まず SSH access してみます。
ユーザ名は macOS の場合も `ec2-user` となっています。

```sh
# @macOS@localhost
chmod 400 tearoom6-ec2-mac.pem
ssh -i tearoom6-ec2-mac.pem ec2-user@ec2-18-217-163-168.us-east-2.compute.amazonaws.com
#
#              .:'
#          __ :'__       __|  __|_  )
#       .'`  `-'  ``.    _|  (     /
#      :          .-'   ___|\___|___|
#      :         :
#       :         `-;   Amazon EC2
#        `.__.-.__.'    macOS Catalina 10.15.7
#
# ec2-user@ip-172-31-24-151 ~ %
```

ちょっとこのリンゴが出てきた時は嬉しいです! macOS から macOS に繋ぐのは斬新な感じですね・・・!

さらに、クラスメソッドさんの記事の追体験に過ぎないのですが、 VNC でも繋いでみます。

SSH access したまま、接続用のユーザを生成します。
`<password>` のところには、設定したいパスワードをセットしてください。

```sh
# @macOS@ec2
sudo dscl . -create /Users/tearoom6
sudo dscl . -create /Users/tearoom6 UserShell /bin/bash
sudo dscl . -create /Users/tearoom6 RealName "Tomohiro Murota"
sudo dscl . -create /Users/tearoom6 UniqueID "1010"
sudo dscl . -create /Users/tearoom6 PrimaryGroupID 80
sudo dscl . -create /Users/tearoom6 NFSHomeDirectory /Users/tearoom6

sudo dscl . -passwd /Users/tearoom6 <password>
sudo dscl . -append /Groups/admin GroupMembership tearoom6
```

続いて、コマンドで VNC サーバを起動します。
`<password>` のところに、先程と同じ値をセットするのをお忘れなく。

```sh
# @macOS@ec2
sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart \
  -activate -configure -access -on \
  -clientopts -setvnclegacy -vnclegacy yes \
  -clientopts -setvncpw -vncpw <password> \
  -restart -agent -privs -all
```

VNC は直接繋ぐと、セキュリティ上のリスクがあるため、 SSH tunnel を通して行います。
localhost の空 port `5901` を用いています。

```sh
# @macOS@localhost
ssh -i tearoom6-ec2-mac.pem -L 5901:localhost:5900 ec2-user@ec2-18-217-163-168.us-east-2.compute.amazonaws.com
```

この状態で Finder の [Go] > [Connect to Server] で `vnc://localhost:5901` に接続します。

ユーザ名とパスワード入力ダイアログが出てくるので、先ほど設定したものを入力します。

![ec2_macos_instance_vnc_sign_in.png](https://files.tearoom6.biz/dfe3d339-1284-4bb2-bf38-873a4c6a8ce9.png)

すると、 macOS の GUI が現れます。
macOS 上のログイン画面が表示されますが、私の場合、この時点では `ec2-user` しかリストに出てきませんでした。
そのため、 VNC 画面上で macOS の [Restart] を実行しました。すると、 EC2 上の Status check が再度落ちてしまいますが、 5-10 分くらい待っていると、また正常に戻ります。
その後、同じ手順で VNC アクセスをすると、今度は、先ほど作成したユーザがリストに出てきます。

![ec2_macos_instance_vnc_mac_login.png](https://files.tearoom6.biz/7200c486-3373-46a1-810f-769b4121bac9.png)

もう、この後は普通に macOS 触っているのと同様の感覚です。
でも、ローカルか EC2 か、どっちを操作しているのか、注意しないと危ないですね。

いきなり Big Sur へのアップデートを勧められたんだけど、これ押したらどうなるんだろう・・・

![ec2_macos_instance_vnc_software_update.png](https://files.tearoom6.biz/21fb05a3-ab49-48dd-b489-a0dae15568c4.png)

なお、 `aws` コマンドで EC2 起動までを上記と同様に実行するには、例えば、以下のようにします。 (Ohio region の場合)
AWS のブログの記載の通りにやると `HostId` の指定が足りなくてエラーになります。 (auto assign はされません)

```sh
aws ec2 allocate-hosts \
  --instance-type mac1.metal \
  --region us-east-2 \
  --availability-zone us-east-2b \
  --auto-placement on \
  --quantity 1

aws ec2 create-key-pair \
  --key-name tearoom6-ec2-mac \
  --region us-east-2

aws ec2 create-security-group \
  --group-name macos-instance \
  --description "Security Group for macOS instance" \
  --vpc-id vpc-a976bfc0 \
  --region us-east-2

aws ec2 authorize-security-group-ingress \
  --group-id sg-024ad680e59b1113c \
  --protocol tcp \
  --port 22 \
  --cidr 113.xxx.xxx.xxx/32 \
  --region us-east-2

aws ec2 authorize-security-group-ingress \
  --group-id sg-024ad680e59b1113c \
  --protocol tcp \
  --port 5900 \
  --cidr 113.xxx.xxx.xxx/32 \
  --region us-east-2

aws ec2 run-instances \
  --region us-east-2 \
  --instance-type mac1.metal \
  --image-id ami-00692c69a6f9c6ea1 \
  --key-name tearoom6-ec2-mac \
  --security-group-ids sg-024ad680e59b1113c \
  --block-device-mappings "[\
    {\"DeviceName\": \"/dev/sda1\", \"Ebs\": {\"VolumeType\": \"gp2\", \"VolumeSize\": 30, \"DeleteOnTermination\": true}},\
    {\"DeviceName\": \"/dev/sdb\", \"Ebs\": {\"VolumeType\": \"gp2\", \"VolumeSize\": 200, \"DeleteOnTermination\": true}}\
  ]" \
  --placement "{\"Tenancy\": \"host\", \"HostId\": \"h-0e807d0d4457f6c1e\"}" \
  --associate-public-ip-address
```

> References

- [Amazon EC2 Mac Instances - Amazon Web Services](https://aws.amazon.com/ec2/instance-types/mac/)
- [Announcing Amazon EC2 Mac instances for macOS](https://aws.amazon.com/about-aws/whats-new/2020/11/announcing-amazon-ec2-mac-instances-for-macos/)
- [New – Use Amazon EC2 Mac Instances to Build & Test macOS, iOS, ipadOS, tvOS, and watchOS Apps | AWS News Blog](https://aws.amazon.com/blogs/aws/new-use-mac-instances-to-build-test-macos-ios-ipados-tvos-and-watchos-apps/)
- [Amazon EC2 Dedicated Host Pricing](https://aws.amazon.com/ec2/dedicated-hosts/pricing/)
- [Amazon EC2 Mac Instance上のmacOSのデスクトップにVNCでログインしてみた #reinvent | Developers.IO](https://dev.classmethod.jp/articles/reinvent2020-mac-instance-vnc/)
- [mac EC2にVNCからつないでみた。 #reinvent | Developers.IO](https://dev.classmethod.jp/articles/access-to-mac-ec2-via-vnc/)
- [【速報】EC2がMac対応！ Amazon EC2 Mac Instancesがリリースされたので触ってみた #reinvent | Developers.IO](https://dev.classmethod.jp/articles/amazon-ec2-mac-instance/)
- [ベアメタルとは？-「ベアメタルサーバ」と「ベアメタルクラウド」のそれぞれの意味と違いとは – | 物理サーバ愛が止まらないホスティング事業者のブログ | ベアメタルブログ](https://baremetal.jp/blog/2017/05/15/272/)
- [新しい Amazon EC2 ベアメタルインスタンスのご紹介](https://aws.amazon.com/jp/about-aws/whats-new/2019/02/introducing-five-new-amazon-ec2-bare-metal-instances/)
- [Amazon EC2 Dedicated Hosts を使ってみた | Developers.IO](https://dev.classmethod.jp/articles/using-ec2-dedicated-hosts/)
- [ec2 — AWS CLI 1.18.187 Command Reference](https://docs.aws.amazon.com/cli/latest/reference/ec2/)
- [Creating, configuring, and deleting security groups for Amazon EC2 - AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-ec2-sg.html)
- [C.4. Linux におけるデバイス名](https://www.debian.org/releases/stable/i386/apcs04.ja.html)


## 追加のディスクのフォーマット

追加の EBS ボリュームを付けた場合、起動直後に以下のようなダイアログが出てきます。
[Initialize] を押しても、何も変わりませんでした。

![ec2_macos_instance_vnc_disk_warning.png](https://files.tearoom6.biz/bad84c61-8c42-4972-b263-fbab4d32c7d2.png)

ディスクの状況を確認すると、以下のようになっていました。

```sh
# @macOS@ec2
$ df -h
Filesystem      Size   Used  Avail Capacity iused     ifree %iused  Mounted on
/dev/disk3s5    30Gi   10Gi   11Gi    49%  488252 312036148    0%   /
devfs          187Ki  187Ki    0Bi   100%     648         0  100%   /dev
/dev/disk3s1    30Gi  5.8Gi   11Gi    35%  154986 312369414    0%   /System/Volumes/Data
/dev/disk3s4    30Gi  2.0Gi   11Gi    16%       1 312524399    0%   /private/var/vm
map auto_home    0Bi    0Bi    0Bi   100%       0         0  100%   /System/Volumes/Data/home

$ diskutil list
/dev/disk0 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                                                   *214.7 GB   disk0

/dev/disk1 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *32.2 GB    disk1
   1:                        EFI EFI                     209.7 MB   disk1s1
   2:                 Apple_APFS Container disk3         32.0 GB    disk1s2

/dev/disk2 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *121.3 GB   disk2

/dev/disk3 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +32.0 GB    disk3
                                 Physical Store disk1s2
   1:                APFS Volume Macintosh HD - Data     6.3 GB     disk3s1
   2:                APFS Volume Preboot                 80.1 MB    disk3s2
   3:                APFS Volume Recovery                529.0 MB   disk3s3
   4:                APFS Volume VM                      2.1 GB     disk3s4
   5:                APFS Volume Macintosh HD            11.0 GB    disk3s5
```

そこで、 Launcher を起動して、 "Disk Utility" app を開き、以下の操作を行うことで、追加ディスクのフォーマットを行います。

1. [View] > [All Show Devices] を選ぶ
2. 200 GiB 近くある "Amazon Elastic Block Store Media" を選択し、 [Erase] を実行
3. ダイアログの中の各フィールドを入力し、 [Erase] ボタンを押す
   - Name: `Amazon Elastic Block Store Media Extended` (任意)
   - Format: `APFS`
   - Scheme: `GUID Partition Map`

![ec2_macos_instance_vnc_disk_utility_001.png](https://files.tearoom6.biz/e576e73b-8830-47d4-95cd-0f7af3dc2a1d.png)
![ec2_macos_instance_vnc_disk_utility_002.png](https://files.tearoom6.biz/076cba59-e558-47f7-8ca9-fbfa2ef6728f.png)

この処理の後は、以下のように `/dev/disk4s1` がマウントされました。

```sh
$ df -h
Filesystem      Size   Used  Avail Capacity iused      ifree %iused  Mounted on
/dev/disk3s5    30Gi   10Gi   11Gi    49%  488252  312036148    0%   /
devfs          192Ki  192Ki    0Bi   100%     664          0  100%   /dev
/dev/disk3s1    30Gi  5.8Gi   11Gi    35%  155833  312368567    0%   /System/Volumes/Data
/dev/disk3s4    30Gi  2.0Gi   11Gi    16%       1  312524399    0%   /private/var/vm
map auto_home    0Bi    0Bi    0Bi   100%       0          0  100%   /System/Volumes/Data/home
/dev/disk4s1   200Gi  708Ki  200Gi     1%      70 2095103530    0%   /Volumes/Amazon Elastic Block Store Media Extended

$ diskutil list
/dev/disk0 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *214.7 GB   disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:                 Apple_APFS Container disk4         214.5 GB   disk0s2

/dev/disk1 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *32.2 GB    disk1
   1:                        EFI EFI                     209.7 MB   disk1s1
   2:                 Apple_APFS Container disk3         32.0 GB    disk1s2

/dev/disk2 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *121.3 GB   disk2

/dev/disk3 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +32.0 GB    disk3
                                 Physical Store disk1s2
   1:                APFS Volume Macintosh HD - Data     6.3 GB     disk3s1
   2:                APFS Volume Preboot                 80.1 MB    disk3s2
   3:                APFS Volume Recovery                529.0 MB   disk3s3
   4:                APFS Volume VM                      2.1 GB     disk3s4
   5:                APFS Volume Macintosh HD            11.0 GB    disk3s5

/dev/disk4 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +214.5 GB   disk4
                                 Physical Store disk0s2
   1:                APFS Volume Amazon Elastic Block... 708.6 KB   disk4s1
```

> References

- [How to erase your Intel-based Mac - Apple Support](https://support.apple.com/en-us/HT208496)
- [外付けハードディスク/SSDのフォーマット手順（Mac） | バッファロー](https://www.buffalo.jp/support/faq/detail/1199.html)
- [Mac - 外部ディスクをマウント/アンマウント（マウント解除） - PC設定のカルマ](https://pc-karuma.net/mac-mount-disk/)


## Homebrew をインストールしてみる

まずは Homebrew をインストールしてみます。

```sh
# @macOS@ec2
sudo chown -R $(whoami) /usr/local/*
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

1 行目は、以下のようなエラーが出るのを防ぐために行っています。

```
error: could not lock config file /usr/local/Homebrew/.git/config: Permission denied
fatal: could not set 'core.repositoryformatversion' to '0'
Failed during: git init -q
```

[mas-cli/mas: Mac App Store command line interface](https://github.com/mas-cli/mas) をインストールしておきます。

```sh
# @macOS@ec2
brew install mas
```

> References

- [Homebrew with Catalina - Homebrew - Homebrew](https://discourse.brew.sh/t/homebrew-with-catalina/6602)
