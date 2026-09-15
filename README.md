# fort-plugins

株式会社Fort の Claude Code プラグイン置き場(マーケットプレイス)です。

## 使い方

```
claude plugin marketplace add fort-inc/plugins
claude plugin install <プラグイン名>@fort-plugins
```

セッションの中では `/plugin marketplace add fort-inc/plugins` → `/plugin install` でも同じです。harness-check なら、Claude Code のセッションの中で次の3行を順に打ちます。

```
/plugin marketplace add fort-inc/plugins
/plugin install harness-check@fort-plugins
/reload-plugins
```

## プラグイン一覧

| 名前 | 何をするか | 合言葉 |
|---|---|---|
| review | 成果物や決めごとを出す前に、別の目で見直す | 「見直して」 |
| harness-check | Claude Code に読ませている設定と記録の状態を診断し、直すとよい所を日本語の診断書にする | 「診断して」「ハーネスを診て」 |

harness-check はあなたの環境に何も作らない・変えない・消さないで、読むだけです。診断書は OS の一時フォルダに出ます。

## zip で受け取った場合

GitHub を使わずに zip で受け取った時は、展開すると `harness-check` フォルダが出てきます。これを次の場所に置いてください。

- mac: `~/.claude/skills/harness-check/`
- Windows: `C:\Users\<ユーザー名>\.claude\skills\harness-check\`

置いたら Claude Code を再起動すると、プラグインで入れた時と同じように使えます。

## ライセンス

MIT(`LICENSE` を参照)。
