# Heroku Deploy Action for YassLab team

YassLab チームのアプリを、Git で Heroku にデプロイするための action です。

[&raquo; 中身を見る（action.yml）](https://github.com/yasslab/heroku_deploy_action/blob/main/action.yml)<br>
[&raquo; 関連リポジトリを見る](https://github.com/yasslab/claude_review_action)

<br>

## なぜ作ったか

それまで使っていた [akhileshns/heroku-deploy](https://github.com/AkhileshNS/heroku-deploy) には、Heroku の API キー（アカウント全体を操作できる）を渡していました。この action は、その action のうち、YassLab のアプリが使っていた機能だけを持ちます。数十行の shell なので、すべて読んで確かめられます。

| 機能 | 振る舞い |
|---|---|
| ログイン | `~/.netrc` に API キーを書く（権限 600） |
| 自動コミット | push の前に作業ツリーに変更があれば、`Commited changes from previous actions` でコミットする（元の action と同じ） |
| デプロイ | 浅い clone なら `--unshallow` してから、`git push https://git.heroku.com/<app>.git <branch>:refs/heads/main`。**強制 push はしない**（後から動いた古い回が、本番を古いコミットに戻さないように） |
| ヘルスチェック | `healthcheck` の URL が 200 でなければ失敗にする。`rollbackonhealthcheckfailed: true` なら `heroku rollback` する（heroku CLI が要る） |
| ログインだけ | `justlogin: true` なら、`~/.netrc` を書くだけでデプロイしない |

元の action の Docker・buildpack・アプリの自動作成・`appdir` は持ちません。

<br>

## 使い方

```yaml
- uses: actions/checkout@v7
- uses: yasslab/heroku_deploy_action@<SHA> # v1.0.0
  with:
    heroku_api_key:  ${{ secrets.HEROKU_API_KEY }}
    heroku_app_name: 'your-app'
    heroku_email:    'you@example.com'
```

本番の API キーを持って本番へ push する action なので、呼ぶ側では **tag ではなく SHA で固定** し、末尾に版をコメントで添えてください。

| 入力 | 必須 | 既定 | 説明 |
|---|---|---|---|
| `heroku_api_key` | ✅ | | Heroku の API キー（secrets から渡す） |
| `heroku_app_name` | ✅ | | デプロイ先のアプリ名 |
| `heroku_email` | ✅ | | Heroku のアカウントのメールアドレス |
| `branch` | | `HEAD` | push する手元のブランチ |
| `healthcheck` | | | デプロイ後に 200 を確かめる URL |
| `delay` | | `0` | ヘルスチェックの前に待つ秒数 |
| `rollbackonhealthcheckfailed` | | `false` | ヘルスチェックに失敗したら直前のリリースに戻す |
| `justlogin` | | `false` | ログインだけしてデプロイしない |

<br>

## Acknowledgments

This action is derived from [akhileshns/heroku-deploy](https://github.com/AkhileshNS/heroku-deploy) by [AkhileshNS](https://github.com/AkhileshNS) (MIT License). The input names and behaviors (the auto-commit and its message, `--unshallow`, the health check with rollback, and `justlogin`) follow the design of the original action as they are; we rewrote only the features YassLab uses as a small shell script. We have relied on your action for years. Thank you very much, AkhileshNS!

## ライセンス

[MIT License](LICENSE)。元の action の著作権表記（Copyright (c) 2020 AkhileshNS）も、LICENSE に残しています。
