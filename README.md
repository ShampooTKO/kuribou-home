# 有限会社くりぼう ホームページ

**公開URL:** https://shampootko.github.io/kuribou-home/

`main` ブランチに push すると、`public/` の中身が GitHub Pages として自動公開されます。

```
kuribou-home/
├─ .github/workflows/
│   ├─ pages.yml    ← GitHub Pages への自動公開（稼働中）
│   └─ deploy.yml   ← さくらへのFTP転送（手動実行のみ・予備）
├─ public/          ← この中だけが公開されます
│   ├─ index.html
│   └─ assets/
│       ├─ bg-hero.jpg
│       ├─ bg-feature.jpg
│       └─ bg-contact.jpg
└─ README.md
```

---

## 公開の仕組み

Pages の有効化はワークフロー内で自動的に行われるため、Settings での手動操作は不要です。

アップロードと公開は別ジョブに分けてあります。同一ジョブで続けて実行すると、
アップロード直後はまだアーティファクトの索引付けが終わっておらず、
公開側が「No artifacts named github-pages」で失敗することがあるためです。

**この失敗が出ても Actions の「Re-run」は押さないでください。**
アップロードがやり直され、同名のアーティファクトが2つになって必ず失敗するようになります。
公開ステップだけ1回リトライする設定を入れてあります。

---

## 更新のしかた

`public/index.html` を編集して push するだけです。

```bash
git add public
git commit -m "お問い合わせ先を更新"
git push
```

1〜2分で公開サイトに反映されます。

---

## 独自ドメイン（kuribow.com）で公開する手順

公開URLを `https://kuribow.com/` にします。HTTPS証明書は GitHub が自動発行します。
**メールはさくらのまま使えます。**

### 作業前に知っておくこと

- さくらのサブドメイン（`home.kuribow.com` / `info.kuribow.com`）は、
  それぞれ **別のゾーン** として管理されています。
  編集するゾーンを間違えると、意図しない側が壊れます。
- `info.kuribow.com` のゾーンは MX が `@`（自分自身）を指しており、
  **Aレコードを変えるとメールが止まります。触らないでください。**
- 触るのは `kuribow.com` のゾーンの **Aレコードだけ** です。

---

### STEP 0　現状を記録する

ゾーン編集画面を開いたら、まず画面全体をスクリーンショットで保存してください。
元に戻す必要が生じたときの唯一の手がかりになります。

---

### STEP 1　ドメインコントロールパネルを開く

`kuribow.com` はさくらで取得したドメインなので、会員メニュー側で編集します。
レンタルサーバへのドメイン追加は不要です。

1. https://secure.sakura.ad.jp/ にログイン（会員ID）
2. 「契約情報」→「ドメインの契約情報を確認」
3. `kuribow.com` の行の「ドメインコントロールパネル」（または「ゾーン編集」）

サーバーコントロールパネル側の「DNSレコード設定」には、レンタルサーバに
追加済みのドメインしか出てきません。`kuribow.com` はそちらには出ない想定です。

---

### STEP 2　今あるレコードを確認する

次の2点を確認してメモします。

| 確認する項目 | 見るポイント |
|---|---|
| Aレコード | さくらのIP（133.167.x.x など）が入っているか |
| MXレコード | **あるか、ないか** |

MXの有無で STEP 4 の対応が変わります。

---

### STEP 3　Aレコードを4つに差し替える

既存のAレコード（さくらのIP）を削除または上書きし、次の4つを登録します。
エントリー名は `@` または空欄です。

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

CNAMEではなくAレコードを使います。ゾーンのapexにはNSレコードがあるため、
apexにCNAMEは置けません。

---

### STEP 4　MXレコードの扱い

**MXがすでにある場合** → 何もしない。そのままで正しく動きます。

**MXがない場合** → `@kuribow.com` 宛のメールはAレコードに引きずられるため、
GitHub に飛んでしまいます。将来そのアドレスを使う予定があるなら、
次を追加してください。

```
10 kuribow.sakura.ne.jp.
```

末尾のドットまで必要です。値は独自ドメインではなく **初期ドメイン** を入れます。
初期ドメイン名はサーバーコントロールパネルの「サーバー情報」で確認できます。

使う予定がなければ、何も追加しなくて構いません。

---

### STEP 5　www も使えるようにする（任意）

| エントリー | 種別 | 値 |
|---|---|---|
| www | CNAME | `shampootko.github.io.` |

リポジトリ名は付けません。末尾のドットが必要です。

---

### STEP 6　保存する

画面下部の「設定する」「保存」をクリックします。押し忘れが最も多い失敗です。

---

### STEP 7　反映を待って確認する

数時間、長いと72時間かかります。**この間は GitHub 側を触らないでください。**

スマホからでも https://www.whatsmydns.net/ で確認できます。
`kuribow.com` を入力し、種別を `A` にして検索。
世界各地のチェック欄に `185.199.108.153` などが並べば反映済みです。

パソコンがあればこちらのほうが速いです。

```bash
dig kuribow.com +short
dig kuribow.com MX +short
```

---

### STEP 8　GitHub にドメインを登録する

**STEP 7 で反映を確認してから**行ってください。

1. https://github.com/ShampooTKO/kuribou-home/settings/pages を開く
2. Custom domain に `kuribow.com` を入力して Save
3. 「DNS check in progress」が「DNS check successful」に変わるまで待つ（数分）

---

### STEP 9　HTTPS を有効にする

証明書が発行されると「Enforce HTTPS」のチェックが押せるようになります。
発行まで数十分かかることがあります。押せるようになったらチェックを入れてください。

---

### STEP 10　CNAMEファイルとURLを更新する

Actionsでデプロイしているため、成果物に `CNAME` を含めないと
デプロイのたびに独自ドメインの設定が外れることがあります。

```bash
echo "kuribow.com" > public/CNAME
```

あわせて次の4か所を `https://kuribow.com/` に書き換えます。

- `public/index.html` の `canonical`
- `public/index.html` の `og:url`
- `public/index.html` の構造化データの `url`
- `public/sitemap.xml` の `loc`、`public/robots.txt` の `Sitemap`

---

### STEP 11　最終確認

- `https://kuribow.com/` が表示される
- 鍵マークが出る
- `http://kuribow.com/` が自動で https に変わる
- 画像3枚が表示される

---

### STEP 12　Google Search Console に登録する

ここまで終えてから登録してください。先に github.io で登録すると二度手間になります。

1. https://search.google.com/search-console
2. 「URLプレフィックス」で `https://kuribow.com/` を登録
3. 所有権の確認（HTMLファイル方式が確実。ファイル名を伝えればリポジトリに追加します）
4. 上部の検索窓にURLを入れて「インデックス登録をリクエスト」

---

### うまくいかないとき

| 症状 | 原因 |
|---|---|
| GitHub で「DNS check unsuccessful」 | 反映待ち。STEP 7 で確認してからやり直す |
| サイトが404 | Custom domain の綴り違い、またはAレコードの入力ミス |
| Enforce HTTPS が押せない | 証明書の発行待ち。最大1時間ほど |
| メールが届かなくなった | 編集するゾーンを間違えた可能性。STEP 0 の記録から戻す |
| デプロイ後にドメイン設定が外れた | `public/CNAME` が無い。STEP 10 |

---

## さくらへ切り替えたい場合

WordPressブログを同じドメインで運用するなど、さくらに戻す必要が出たときは、
`.github/workflows/deploy.yml` を使います。手順は以下のとおりです。

### さくらへ切り替える手順（所要15分ほど）

### STEP 1　さくら側でアップロード先のパスを確認する

1. サーバーコントロールパネル（`https://secure.sakura.ad.jp/rs/cp/`）に、**初期ドメインとサーバーパスワード**でログイン
2. 「ドメイン/SSL」→ 独自ドメインの「設定」→「基本設定」を開く
3. **Web公開フォルダー**の値を控える（例：`/kuribou`）

FTPでのアップロード先パスは、これを使って次のように決まります。

```
/home/<アカウント名>/www<Web公開フォルダー>/
```

例：アカウント名が `kuribow`、公開フォルダーが `/kuribou` なら

```
/home/kuribow/www/kuribou/
```

**末尾のスラッシュは必須です。** 付け忘れると正しくアップロードされません。

---

### STEP 2　さくら側で「国外IPアドレスフィルタ」を無効にする

GitHub Actions の実行サーバーは海外にあるため、初期設定のままだと**FTP接続が弾かれます**。

1. コントロールパネル →「セキュリティ」→「**国外IPアドレスフィルタ**」
2. 「有効（制限する）」→「**無効（制限しない）**」に変更
3. 「設定する」をクリック

#### これはセキュリティを下げる変更です

この設定を切ると、FTP・コントロールパネル・メールへ海外からもアクセスできるようになります。元々は不正アクセス対策の機能です。そのため、**切るなら必ず以下をセットで**行ってください。

- サーバーパスワード・FTPパスワードを、十分に長く推測されにくいものへ変更する
- コントロールパネルの「セキュリティ」→「サーバーログイン履歴」を時々確認する

セキュリティを下げたくない場合は、後述の「別案：SFTPでデプロイする」をご覧ください。

---

### STEP 3　GitHub にリポジトリを作る

1. GitHub で新しいリポジトリを作成（例：`kuribou-site`）
   - **Private** を推奨
   - README や .gitignore は追加しない（空で作る）
2. このフォルダの中身をそのまま push する

```bash
cd kuribou-deploy
git init
git add .
git commit -m "くりぼうホームページ 初回"
git branch -M main
git remote add origin https://github.com/<あなたのID>/kuribou-site.git
git push -u origin main
```

---

### STEP 4　GitHub に接続情報を登録する

リポジトリの **Settings → Secrets and variables → Actions → New repository secret** から、次の4つを登録します。

| Name | 値 | 例 |
|---|---|---|
| `FTP_SERVER` | 初期ドメイン | `kuribow.sakura.ne.jp` |
| `FTP_USERNAME` | アカウント名 | `kuribow` |
| `FTP_PASSWORD` | サーバーパスワード | （契約時に設定したもの） |
| `FTP_SERVER_DIR` | STEP 1 で確認したパス | `/home/kuribow/www/kuribou/` |

Secrets に入れた値は、登録後は本人にも表示されません。GitHub のログにも出力されない扱いになります。

---

### STEP 5　デプロイを実行する

STEP 3 の push の時点で、すでに1回目のデプロイが走っています。

**Actions** タブを開き、緑のチェックが付いていれば成功です。ブラウザで独自ドメインを開いて確認してください。

以降は `public/` の中身を変更して push するだけで、自動的にサーバーへ反映されます。手動で動かしたいときは Actions タブの「Run workflow」から実行できます。

---

## うまくいかないときの確認順序

| 症状 | 原因と対処 |
|---|---|
| `Server sent FIN packet unexpectedly` | 国外IPアドレスフィルタが有効のまま。STEP 2 を実施 |
| `425 Unable to build data connection` | 同上 |
| `530 Login incorrect` | `FTP_USERNAME` か `FTP_PASSWORD` の誤り。ユーザー名はアカウント名のみ（メールアドレスではない） |
| 成功しているのにサイトが変わらない | `FTP_SERVER_DIR` のパス違い。末尾の `/` が付いているかも確認 |
| 画像だけ表示されない | `public/assets/` の中にjpgが入っているか確認 |

---

## 別案：SFTPでデプロイする（国外IPフィルタを切りたくない場合）

SSH接続は国外IPアドレスフィルタの対象外という報告があります。確認できれば、フィルタを有効のままデプロイできます。

ただし公式ドキュメントで確証が取れていないため、**まずは手動でSSH接続できるか試してから**切り替えてください。スタンダードプラン以上でSSHが使えます。

### 手順の概要

1. 手元で鍵を作る

```bash
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/sakura_deploy
```

2. 公開鍵をさくらへ登録する（サーバーに `~/.ssh/authorized_keys` を作り、`sakura_deploy.pub` の中身を追記。パーミッションは `.ssh` が 700、`authorized_keys` が 600）

3. GitHub Secrets に `SSH_PRIVATE_KEY`（`sakura_deploy` の中身をそのまま）、`SSH_HOST`、`SSH_USER`、`SSH_TARGET_DIR` を登録

4. `.github/workflows/deploy.yml` のアップロード部分を差し替える

```yaml
      - name: SFTP でアップロード
        uses: wlixcc/SFTP-Deploy-Action@v1.2.4
        with:
          server: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          ssh_private_key: ${{ secrets.SSH_PRIVATE_KEY }}
          local_path: './public/*'
          remote_path: ${{ secrets.SSH_TARGET_DIR }}
          sftp_only: true
```

切り替えが成功したら、STEP 2 で無効にした国外IPアドレスフィルタを**有効に戻してください**。

---

## 手動でアップロードする場合

自動デプロイを使わず、今すぐ公開したいときは次の方法が最速です。

1. コントロールパネル →「Webサイト/データ」→「ファイルマネージャー」
2. `www` → 公開フォルダー（例：`kuribou`）へ移動
3. `public/index.html` をドラッグ＆ドロップ
4. `assets` フォルダを作り、その中に3枚のjpgをドラッグ＆ドロップ

---

## 内容を更新したいとき

`public/index.html` を編集して push するだけです。

```bash
git add public
git commit -m "お問い合わせ先を更新"
git push
```

数十秒で本番サイトに反映されます。
