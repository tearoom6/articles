# 手順

たまにしかやらないとよく忘れるので、手順メモです。

## ACM で証明書発行

最大の注意点ですが、 CloudFront で証明書を使うために `us-east-1` (N. Virginia) で証明書を発行する必要があります。
別のリージョンに既存の証明書があったとしても、これを `us-east-1` リージョンに移行したり、併用したりすることはできません。
新規で `us-east-1` リージョンで証明書を発行してください。 (同一のカスタムドメインに対する証明書を複数リージョンで作成することは可能です)

1. AWS Web Console で [Certificate Manager] を開く
2. 右上のリージョンで `US East (N. Virginia)` を選択
3. [Request a certificate] を実行
4. [Request a public certificate] を選択
   - Step 1: Add domain names
      - Domain name: `*.example.com` のようにすれば、サブドメインも対象にできる
   - Step 2: Select validation method
      - `DNS validation` を選択
   - Step 3: Review
   - Step 4: Validation
      - [Create record in Route 53] を押下して、ダイアログ内の [Create] を実行


## S3 で静的ホスティングの設定

S3 はどのリージョンを対象としても構いません。 (バケットごとにリージョンを選択する形になります)

1. AWS Web Console で [Amazon S3] を開く
2. [Create bucket] を押下
   - Step 1: Name and region
      - Name: Global で一意になるので、注意
   - Step 2: Set properties
   - Step 3: Set permissions
      - Manage public permissions: `Grant public read access to this bucket`
   - Step 4: Review
3. 作成された bucket の画面を開き、 [Properties] タブを開く
   - Static website hosting: `Use this bucket to host a website`
4. さらに [Permissions] タブを開く
   - Bucket Policy: 以下の内容を入力 (`BUCKET_NAME` には作成したバケット名を入力)

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::BUCKET_NAME/*"
        }
    ]
}
```


## CloudFront で Distribution の作成

CloudFront はリージョンの区別は無いので、 Global リージョンでの作業になります。

1. AWS Web Console で [CloudFront] を開く
2. [Create Distribution] を押下
   - Step 1: Select delivery method
      - Web: [Get Started]
   - Step 2: Create Distribution
      - Origin Settings
         - Origin Domain Name: S3 bucket の URL (バケット名を入れるとサジェストされる)
      - Default Cache Behavior Settings
         - Viewer Protocol Policy: Redirect HTTP to HTTPS
         - Allowed HTTP Methods: GET, HEAD
         - Object Caching: Use Origin Cache Headers
      - Distribution Settings
         - Price Class: Use U.S., Canada, Europe, Asia and Africa
         - Alternate Domain Names (CNAMEs): カスタムドメイン名をサブドメイン込みで入力
         - SSL Certificate:
            - Custom SSL Certificate (example.com): 先程発行した証明書を選択 (`US East (N. Virginia)` 以外のリージョンで作成した場合は選択できない)

Status: Deployed になるのを待ちます。 (結構かかります)


## Route 53 の設定

Route 53 もリージョンの区別は無いので、 Global リージョンでの設定になります。

1. AWS Web Console で [Route 53] を開く
2. [Hosted zones] 画面で、対象のドメインを選択
3. [Create Record Set]
   - Type: `A - IPv4 address`
   - Alias: `Yes`
   - Alias Target: 先程作成した CloudFront の Distribution を選択 (サジェストされる)

数分後に HTTPS でアクセスできるかチェックします。

> References

- [チュートリアル例 – Amazon S3 でのウェブサイトのホスティング - Amazon Simple Storage Service](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/hosting-websites-on-s3-examples.html)
- [amazon web services - Unable to select Custom SSL Certificate (stored in AWS IAM) - Stack Overflow](https://stackoverflow.com/questions/28609262/)
- [Migrate SSL Certificates to the US East (N.Virginia) Region](https://aws.amazon.com/premiumsupport/knowledge-center/migrate-ssl-cert-us-east/)
