# 有限会社くりぼう ホームページ｜自動デプロイ設定

main ブランチに push すると、`public/` の中身がさくらのレンタルサーバへ自動でアップロードされます。

```
kuribou-site/
├─ .github/workflows/deploy.yml   ← 自動デプロイの設定（さくらには上がりません）
├─ public/                        ← この中だけがサーバーへ上がります
│   ├─ index.html
│   └─ assets/
│       ├─ bg-hero.jpg
│       ├─ bg-feature.jpg
│       └─ bg-contact.jpg
└─ README.md
```

---

## セットアップ（初回のみ・所要15分ほど）

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
