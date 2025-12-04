# Project Portal Template Repository

## 1. ポータル用テンプレート（`temp_knowledge_portal`）

親リポジトリのひな形として、プロジェクトポータルのテンプレートを提供します。  
このテンプレートは、プロジェクトポータルの基本的な構造と設定を含んでおり、新しいプロジェクトを迅速に開始するための基盤となります。

### ディレクトリ構成：

```tree
/
├── .github/
│   └── workflows/
│       └── handle_app_update.yml  # [重要] 子からの更新を受け取るCI
├── apps/                          # 各アプリのサブツリー配置場所
├── docs/
│   ├── index.md                   # ポータル表紙
│   └── overrides/                 # (任意) カスタムCSS等
├── .gitignore
├── amplify.yml                    # Amplifyビルド設定
├── mkdocs.yml                     # MkDocs設定
├── README.md                      # このファイル
└── requirements.txt               # 依存ライブラリ
```

#### 主要ファイルの中身

`.github/workflows/handle_app_update.yml`

```yaml
name: Handle App Update
on:
  repository_dispatch:
    types: [app-update]
jobs:
  sync-pr:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - name: Configure Git
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
      - name: Sync Subtree
        env:
          APP_NAME: ${{ github.event.client_payload.app_name }}
          REMOTE_URL: ${{ github.event.client_payload.remote_url }}
        run: |
          git remote add $APP_NAME $REMOTE_URL
          git checkout -b update/$APP_NAME-${{ github.run_id }}
          git subtree pull --prefix=apps/$APP_NAME $APP_NAME main --squash -m "Update $APP_NAME"
          git push origin update/$APP_NAME-${{ github.run_id }}
      - name: Create PR
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          APP_NAME: ${{ github.event.client_payload.app_name }}
        run: |
          gh pr create --title "Update $APP_NAME" --body "Auto-sync from $APP_NAME" --base main --head update/$APP_NAME-${{ github.run_id }}
```

`mkdocs.yml`

```yaml
site_name: Knowledge Portal (Project Name)
theme:
  name: material
  language: ja
nav:
  - Home: index.md
  # アプリ追加時にここにリンクを追記します
```

---
