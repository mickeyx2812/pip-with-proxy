# Proxies を使用した pip の利用方法

[![Promo](https://github.com/bright-jp/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.jp/) 

このガイドでは、制限を回避し、セキュリティを向上させ、パッケージ管理を効率化するために、pip でプロキシを設定する方法を説明します。

- [Public vs Private Proxies](#public-vs-private-proxies)
- [Using Proxies with pip](#using-proxies-with-pip)
- [Configuring a pip Proxy with the Command Line](#configuring-a-pip-proxy-with-the-command-line)
- [Configuring a pip Proxy with the pip Config File](#configuring-a-pip-proxy-with-the-pip-config-file)
- [Configuring a pip Proxy with Environment Variables](#configuring-a-pip-proxy-with-environment-variables)
- [Testing the Configuration](#testing-the-configuration)
- [Troubleshooting pip Proxies](#troubleshooting-pip-proxies)
- [Using pip with Rotating Proxies](#using-pip-with-rotating-proxies)
- [Benefits of Using Proxies with pip](#benefits-of-using-proxies-with-pip)
- [Common Mistakes and Best Practices](#common-mistakes-and-best-practices)
- [Using Bright Data Proxies](#using-bright-data-proxies)
- [Conclusion](#conclusion)

## Public vs Private Proxies

### Public Proxies

Public proxy は誰でもアクセスでき、通常は認証を必要としません。代替 IPアドレス を素早く利用できる一方で、速度が遅い、接続が不安定、IP ban のリスクが高いなどの明確な欠点があります。無料で広く入手できることが多いため、プロキシローテーション、キャッシュ、アクセス制御といった主要機能が欠けていることが多く、本番環境での信頼性の高い利用には適していません。

public URL は次のような形式になる場合があります: `https://proxyserver:port`。

### Private Proxies

[Private proxies](https://brightdata.jp/solutions/private-proxies) は認証を必要とし、より高いセキュリティ、安定性、高度な機能を提供しますが、一般的に費用がかかります。高速で信頼性の高い接続を [dedicated IP address](https://brightdata.jp/solutions/dedicated-proxies) に対して提供し、パフォーマンスと制御を向上させるためのプロキシ認証やローテーションなどの機能を含みます。

通常、アクセスは認証によって制御され、次のようにプロキシ URL の接頭辞として username と password を含めることがよくあります: `https://username:password@proxyserver:port`。

## Using Proxies with pip

pip でプロキシを使用する前に、プロキシに関する関連情報を収集する必要があります。次の例では、以下の設定で public proxy を使用する方法を示します。

- プロキシサービスの **proxy address**
- 通信に必要なプロキシサービスの **port**

次の [`proxy-list` repo](https://github.com/clarketm/proxy-list) は、毎日テストされた public proxy address を提供しており、テストには便利ですが、本番環境では使用しないでください。

`proxy-list` repo 内で、動作する public proxy を見つけるには [`proxy-list-status.txt`](https://github.com/clarketm/proxy-list/blob/master/proxy-list-status.txt) ファイルを確認してください。ファイル内で、隣に `success` フラグが付いているアドレスを探すことで、動作していることを示すものを確認できます。

![Selecting a public proxy](https://github.com/bright-jp/pip-with-proxy/blob/main/Images/Selecting-a-public-proxy.png)

このチュートリアルでは、public proxy address として `45.185.162.203:999` を使用します。これはプロキシサーバーアドレスが `http://45.185.162.203:999` であることを意味します。

## Configuring a pip Proxy with the Command Line

pip プロキシを設定する最も速い方法は、`pip install` コマンドを呼び出す際に、`--proxy` コマンドラインオプションでアドレスを渡すことです。

public proxy 経由でのアクセスを確認し、パッケージ取得をテストするには、次のコマンドを実行してください。

```
# Public Proxy
pip install boto3 --proxy http://45.185.162.203:999
```

この方法は、プロキシを恒久的に設定する前に、素早くテストして検証するのに便利です。pip パッケージを公開する場合にも、別の IPアドレス からの可用性を確認するのに役立ちます。

## Configuring a pip Proxy with the pip Config File

pip プロキシを恒久的に設定するには、pip config file が簡単で宣言的なソリューションです。場所は OS に依存し、次のディレクトリで見つかります。

- **Global:** システム全体の設定ファイルで、ユーザー間で共有されます。
- **User:** pip プロセスを実行するユーザーごとの設定ファイルです。
- **Site:** Python virtual environments を使用する環境ごとの設定ファイルです。

これらの設定ファイルは、各システムで以下の場所に存在するか、作成できます。

### Linux/macOS

Linux と macOS では、pip config file は `pip.conf` と呼ばれ、以下の場所にあります。

- **Global:**
    - **Debian-based systems:** `/etc\` ディレクトリ内の `pip.conf in the` を編集または作成します。
    - **macOS-based systems:** `/Library/Application Support/pip/pip.conf` を編集または作成します。
- **User:**
    - **Debian-based systems:** `~/pip/pip.conf` ファイルを編集または作成します。
    - **macOS-based systems:** `~/.config/pip/pip.conf` config file を編集または作成します。
- **Site:** Python virtual environment で読み込まれる場合、`$VIRTUAL_ENV/pip.conf` にあります。

### Windows

Windows では、ファイルは `pip.ini` で、以下の場所にあります。

- **Global:** `C:\ProgramData\pip\pip.ini` ファイルを編集または作成します。このファイルは Windows ではデフォルトで非表示ですが、書き込み可能です。
- **User:** `%APPDATA%\pip\` に `pip.ini` を編集または作成します。
- **Site:** Python virtual environment で読み込まれる場合、`%VIRTUAL_ENV%\pip.ini` にある config file を編集または作成します。

### Config File Contents

この例では、Python virtual environment の pip config file を使用することを前提とします。有効化された virtual environment で、Debian-based systems では `$VIRTUAL_ENV/pip.conf` を、Windows では `%VIRTUAL_ENV%\pip.ini` を編集してください。

config file では、`proxy` キーを目的の HTTP または HTTPS プロキシに更新する必要があります。

```
[global]
proxy = http://45.185.162.203:999
```

ファイルを保存すると、プロキシはすべての pip コマンドに自動的に適用され、毎回プロキシフラグを指定する必要がなくなります。

```bash
(venv) $ pip install boto3
```

設定オプションの詳細については、[project’s documentation](https://pip.pypa.io/en/stable/topics/configuration/) を参照してください。

## Configuring a pip Proxy with Environment Variables

システム環境変数を設定すると、pip およびシステム上の他のすべての HTTP リクエストでプロキシが使用されるようになります。これは `HTTP_PROXY` と `HTTPS_PROXY` 変数を設定することで実現でき、pip を含む多くのアプリケーションが、HTTP リクエストを処理するためのシステム既定プロキシとしてこれらに依存しています。

### Linux/macOS

Linux OS を使用している場合は `/etc/environment` ファイルを更新するか、macOS の場合はホームディレクトリにある `.zshrc` ファイルを更新してください。その後、プロキシサーバーの新しいエントリを追加します。

```
HTTP_PROXY=https://proxyserver:port
HTTPS_PROXY=https://proxyserver:port
```

ターミナルセッションまたはシステムを再起動すると、環境変数が反映されます。

### Windows

Windows では、コマンドプロンプトで次のコマンドを使用して環境変数を設定できます。

```
setx HTTP_PROXY "https://proxyserver:port" /M
setx HTTPS_PROXY "https://proxyserver:port" /M
```

変更を反映するためにコマンドプロンプトを再起動してください。

## Testing the Configuration

pip config file または環境変数のいずれかでシステムレベルの設定を行った後、プロキシが正常に接続してデータを送受信できることを確認するためにテストしてください。

### Linux/macOS

Linux/macOS では、次のコマンドを使用します。

```bash
$ python -m venv venv
$ source venve/scripts/activate

# for pip config file or environment variables
(venv) $ pip install requests
```

特定のプロキシでこれらの設定を上書きしたい場合は、CLI フラグを使用できます。

```bash
# pip cli flag
(venv) $ pip install requests --proxy https://proxyserver:port
```

このコマンドでは、必ず `https://proxyserver:port` をご自身のプロキシに更新してください。

### Windows

Windows では、次のコマンドを使用します。

```powershell
> python -m venv venv
> .\venv\Scripts\Activate.bat
(venv) > pip install requests
```

これらの設定は、pip CLI フラグでいつでも上書きできます。

```bash
# pip cli flag
(venv) $ pip install requests --proxy https://proxyserver:port
```

### Troubleshooting pip Proxies

pip で HTTP または HTTPS プロキシに接続する際、特に高度な機能を持つ private proxy や HTTPS プロキシの利用を検討する場合、次の一般的な問題に遭遇することがあります。

#### Authentication Issues

認証の問題は、pip でプロキシに接続しようとした際に `407 Proxy Authentication Required` エラーとしてよく現れます。これは、プロキシが接続に username と password を必要としているか、プロキシに対して誤った認証情報を提供していることを示します。

#### Certificate Issues

HTTPS プロキシに接続する際、pip から `Certificate verify failed` エラーを受け取ることがあります。これは、プロキシサーバーが提供する証明書に問題があることを示します。

private proxy server が自己署名証明書を使用している場合、証明書が認証局によって検証されないためにエラーが発生することがあります。これを回避するには、特定ドメインへの接続時に自己署名証明書エラーを無視するため、`--trusted-host` CLI オプションの使用を検討してください。

## Using pip with Rotating Proxies

[Rotating proxies](https://brightdata.jp/solutions/rotating-proxies) は、リクエストごとに IPアドレス を自動的に切り替えることで IP ban を回避するのに役立ちます。これは複数ユーザーを模倣し、制限を回避します。

これは、リストからランダムにプロキシを選択することで実装できます。以下は、public proxy をローテーションしながら pip パッケージをインストールする簡単な bash script です。

次の bash script を `rotate-proxies.sh` という名前で作成してください。

```bash
proxy_list=(
  'http://45.185.162.203:999'
  'http://177.23.176.58:8080'
  'http://83.143.24.66:80'
)

pip_packages=(
  'requests'
  'numpy'
  'pandas'
)

# Loop through packages and install them
for package in "${pip_packages[@]}"
do
  # Randomly select a proxy from the list
  proxy=${proxy_list[$RANDOM % ${#proxy_list[@]}]}
  echo -e  "\nInstalling $package with proxy $proxy"
  pip install --proxy $proxy $package
done
```

ファイル作成後、実行すると、各 pip コマンドごとにランダムなプロキシをローテーションしながら pip パッケージをダウンロードできます。以下はスクリプト出力の要約です。

```
$ ./rotate-proxies.sh 

Installing requests with proxy http://177.23.176.58:8080
Collecting requests
  Downloading requests-2.32.3-py3-none-any.whl.metadata (4.6 kB)

….

Downloading urllib3-2.3.0-py3-none-any.whl (128 kB)
Installing collected packages: urllib3, idna, charset-normalizer, certifi, requests
Successfully installed certifi-2025.1.31 charset-normalizer-3.4.1 idna-3.10 requests-2.32.3 urllib3-2.3.0

Installing six with proxy http://45.185.162.203:999
Collecting numpy
 Downloading numpy-2.2.2-cp313-cp313-macosx_14_0_x86_64.whl.metadata (62 kB)

…

Installing collected packages: numpy
Successfully installed numpy-2.2.2

Installing pandas with proxy http://83.143.24.66:80
Collecting pandas
  Downloading pandas-2.2.3-cp313-cp313-macosx_10_13_x86_64.whl.metadata (89 kB)

….

Installing collected packages: pytz, tzdata, six, python-dateutil, pandas
Successfully installed pandas-2.2.3 python-dateutil-2.9.0.post0 pytz-2025.1 six-1.17.0 tzdata-2025.1
```

## Benefits of Using Proxies with pip

プロキシにより、開発者はネットワーク制限を回避し、ブロックされたリソースへアクセスし、パッケージのダウンロード速度を最適化できます。private proxy は、アイデンティティを隠すことでセキュリティを強化しつつ、キャッシュや高速接続も提供します。

すべてのインターネットトラフィックを暗号化してプライバシーを高める一方で遅延を生む可能性がある [VPNs](https://brightdata.jp/blog/proxy-101/vpn-vs-proxy) と異なり、プロキシは pip リクエストに対する軽量な代替手段として機能します。VPN のような性能オーバーヘッドなしに、依存関係をより速く効率的に管理できます。

## Common Mistakes and Best Practices

pip でプロキシを使用する際は、セキュリティ脆弱性につながり得る一般的なミスを避けることが重要です。誤ったプロキシ URL や、不適切に整形されたアドレス（HTTP/HTTPS protocol の欠落や誤りなど）は、パッケージリポジトリへの接続を妨げる可能性があります。

大きなセキュリティリスクの一つは、プロキシ認証情報（例: username や password）をソースコード、スクリプト、または CI/CD pipeline の設定にハードコードすることです。コードが共有または露出すると、プロキシへの不正アクセスが発生する可能性があります。認証情報が侵害されると、悪用につながり、コスト増加やサイバー攻撃による潜在的な悪用の原因になります。

これらのミスを避けるために、プロキシ認証情報はコードに直接書き込むのではなく、環境変数や暗号化された設定ファイルに保存して安全に管理することを推奨します。さらに、pip を使用する前にプロキシ接続性をテストして、適切なセットアップであることを確認し、実行時エラーを回避する必要があります。[curl](https://curl.se/) や [ping](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/ping) のようなツールを使うことで、サービス投入前にプロキシのパフォーマンスを検証できます。これにより、よりスムーズなパッケージ管理を実現できます。

## Using Bright Data Proxies

Bright Data は、レジデンシャル、データセンター、モバイルデバイスなどを含む幅広い [proxy services](https://brightdata.jp/proxy-types) を提供しています。プロジェクトのニーズに合わせて、プロキシを簡単に作成できます。ここでは、pip で別の IPアドレス 経由でパッケージへアクセスできるように、private [residential proxy](https://brightdata.jp/proxy-types/residential-proxies) を作成してみましょう。

まず、無料の Bright Data アカウントにサインアップしてください。次に、[user dashboard](https://brightdata.jp/cp/start/) に移動します。

サイドメニューで **Proxies & Scraping** をクリックします。

![Bright Data proxies](https://github.com/bright-jp/pip-with-proxy/blob/main/Images/Bright-Data-proxies-2048x1041.png)

フォームが読み込まれたら、新しい residential proxy を設定します。デフォルト設定を使用すると、複数の Bright Data ユーザーによって使用される共有 IPアドレス が割り当てられます。

![Creating a residential proxy](https://github.com/bright-jp/pip-with-proxy/blob/main/Images/Creating-a-residential-proxy-2048x1040.png)

特定の地域の IPアドレス が必要な場合は、セットアップ中に希望する国を選択できます。

プロキシが作成されると、エンドポイントと認証の詳細を表示するダッシュボードにリダイレクトされます。username、password、server address を必ず控えておいてください。

![Proxy dashboard](https://github.com/bright-jp/pip-with-proxy/blob/main/Images/Proxy-dashboard-2048x1040.png)

`--proxy` フラグを使用して、endpoint values の可用性をテストしてください。

```bash
$ pip install pandas \
    --trusted-host pypi.org \
    --trusted-host files.pythonhosted.org \
    --proxy https://username:[email protected]:33335
```

Bright Data のプロキシは自己署名証明書を使用しているため、`trusted-host` フラグを使用して `pypi.org` と `files.pythonhosted.org` を信頼済みドメインとしてホワイトリスト登録できます。

## Conclusion

pip 用のプロキシ設定は、CLI フラグ、pip config file、環境変数など複数の方法があり簡単です。ただし、public proxy には制限があり、大規模なワークロードや本番利用には理想的ではありません。より信頼性の高いソリューションとして、Bright Data はレジデンシャルおよび [datacenter IPs](https://brightdata.jp/proxy-types/datacenter-proxies) を提供し、高速で安定した接続に加えて、Webスクレイピング とデータ収集のための高度なツールも提供しています。無料でサインアップして開始してください。