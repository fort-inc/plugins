---
name: harness-check
description: ハーネス診断書を作ります。Claude Code に読ませている設定と記録の一式を読んで点を付け、直すとよい所を日本語の診断書 HTML にして開きます。「ハーネス診断」「ハーネス保全」と頼まれた時に呼びます。Claude Code 本体の不調を調べる /doctor とは別物で、CLAUDE.md・rules・skill・hook・記憶の中身を読みます。
---

# ハーネス診断

ハーネスとは、Claude Code に読ませている設定と記録の一式。この skill は、それを読んで診断書の HTML を作り、ブラウザで開く。判定の基準は同じフォルダの `criteria.md`、診断書の形と文の決まりは同じフォルダの `report.md`。両方を先に読む。

## 守る事

- 相手の環境に何も作らない・変えない・消さない。書き出すのは OS の一時フォルダだけ。場所は下の `node` の命令で取る。node が無ければ、Windows は `C:/Users/<ユーザー名>/AppData/Local/Temp`、mac と Linux は `/tmp`。
- 読み取り以外の shell を使わない。使ってよいのは下の8つと、一時フォルダへの書き出し、ブラウザで開く操作、Glob と Grep が使えない時の読み取り専用の代替だけ。テストやビルドなど相手のコマンドは走らせない。
  - `git --no-optional-locks status --porcelain --branch`
  - `git remote`。名前だけを見る。URL を出す `-v` は使わない
  - `git log -5 --format="%h %ad %s" --date=short`。末尾に `-- <パス>` を付けて、特定のファイルの直近 5 件を見てよい
  - `claude --version`
  - `node -e "console.log(process.env.CLAUDE_CODE_DISABLE_AUTO_MEMORY||'unset')"`
  - `date +%Y%m%d-%H%M`。診断書と控えの名前に付ける時刻を取る。PowerShell で打つ時は `Get-Date -Format yyyyMMdd-HHmm`。
  - `node -e "console.log(require('os').tmpdir())"`。一時フォルダの場所を取る
  - `ls -la <パス>`。ファイルの大きさと最終更新を見る時だけ。複数のパスを並べてよい。PowerShell で打つ時は `Get-Item <パス>`
- 探すのは Claude Code の Grep と Glob で行う。shell の `grep`・`find`・`rg` は使わない。Glob と Grep が使えない時だけ、読み取り専用の shell 命令で代替してよい。繋ぎと書き込みはしない。
- `claude mcp list` は走らせない。MCP の接続の状態は、このセッションの冒頭に出ている MCP の一覧で見る。表示が無ければ、一覧で見るセルを「確かめられず」にする。
- shell は1命令ずつ打つ。`&&` や `;` で繋がない。1つが止められても他が失われないようにするため。読み取り専用の命令は、繋がずに1命令へ複数のパスを渡してよい。
- Windows でも shell の命令は Bash の道具で打つ。PowerShell の道具は使わない。複数の操作を含む命令が止められやすく、止められた後の打ち直しが代替の実行になってしまうため。例外は 2 つ。`claude --version` は Windows では PowerShell の道具で打つ。Git Bash では PATH に無い事がある。Git for Windows が無く Bash の道具が無い時は、同じ読み取り専用の命令を PowerShell の道具で打ってよい。
- 命令の中で環境変数を展開しない。一時フォルダのパスは、取った実際の文字列を斜線区切りで書く。円記号区切りは Bash の道具で壊れる。
- ファイルを書き出すのは Claude Code の Write だけ。shell のリダイレクトやヒアドキュメントで書かない。
- hook や許可の設定で止められたら、別の道具や別の書き方で代替しない。同じ事を別の道具で打ち直さない。書き方を変えて通そうとしない。読み取りの1つが止められた時は、その物を「確かめられず」にして診断を続け、止まった内容を「見たもの」の行に読まなかった物として書く。診断書の書き出しが止められた時は、パスの代わりに止まった内容をそのまま伝えて終える。

## 開かないファイル

次は開かない。有る事と置き場だけを見る。`@` の取り込み先、CLAUDE.md から名指しされた文書、hook の実体、記憶のファイルも例外にしない。読む前に必ずこの一覧を当てる。

`.env` と `.env.*`、`.envrc`、`*.pem`、`*.key`、`*.p12`、`*.pfx`、`*.jks`、`*.keystore`、`*.kdbx`、`*.ppk`、`id_rsa*`、`id_ed25519*`、`id_ecdsa*`、`.npmrc`、`.netrc`、`.pypirc`、`.git-credentials`、`*.tfvars`、`credentials*`、`secrets.*`、`.docker/config.json`、`application_default_credentials.json`、名前に service_account か service-account を含む JSON、`.aws/` と `.gcloud/` の下、名前に secret・token・password・passwd・auth・apikey・api_key を含む物。

例外が1つ。settings の hooks に実体として登録されている script は、名前に secret や guard を含んでいても開いてよい。何を止める処理かを読むために要る。

## 設定と接続定義の JSON の読み方

`.claude/settings.json`、`.claude/settings.local.json`、`~/.claude/settings.json`、`.mcp.json`、`~/.claude.json` も、他のファイルと同じく Read で開く。Read が止められた時は代替せず、そのセルを「確かめられず」にする。

## 1. 集める

user 階層と起動ディレクトリを分けて集める。user 階層とは `~/.claude/` を指す。診断書でも分けて書く。見る場所の一覧は `criteria.md` にある。

- 全部読む。両階層の CLAUDE.md、`CLAUDE.local.md`、`AGENTS.md`、`.claude/rules/` の中身、CLAUDE.md が `@` で取り込んでいる先、CLAUDE.md から名指しされた文書、毎回読まれる記憶。
- `CLAUDE.md`、`CLAUDE.local.md`、`.claude/CLAUDE.md` は、起動ディレクトリからファイルシステムの根まで、各階層のフォルダ直下に有るかを Read で直接開いて確かめる。無ければ「存在しない」が即座に返る。Glob は使わない。Windows のドライブ直下とユーザーフォルダ直下も同じ。`.claude/rules/`・`.claude/skills/`・`.claude/agents/` は git の根までにする。
- `.claude/skills/` 直下に直に置かれたファイルは、両階層とも `.claude/skills/*.md` の Glob で数え、そのパスと冒頭 3 行を読む。根拠 2-1 と 1-9 の判定に使う。
- 起動ディレクトリより下の階層は、`CLAUDE.md` と `.claude/` を Glob で数え、その数を「見たもの」に添える。下の `CLAUDE.md` は浅い順に 10 枚まで読み、二重と矛盾の目だけで見る。11 枚目からと、下の `.claude/skills/`・`.claude/agents/` は数だけを「見たもの」の行に書く。下の階層の物は起動のたびに読まれる量に入れない。
- 冒頭だけ読む。skill と subagent は frontmatter と行数だけ。例外は3つ。外から持ち込んだ skill と plugin の本文は読む。subagent の本文も読む。外から持ち込んだ skill とは、plugin として入った物か、導入の跡が記録にある物を指す。入っている plugin は `~/.claude/plugins/installed_plugins.json` を Read で開いて見る。`installPath` が実体の場所で、そこに無い plugin は無いとする。description が重なる skill の対と、常時読まれる決まりと同じ事柄を扱う skill は、本文も読んで見比べる。
- hook の実体と plugin の本文は、Read の範囲指定で 200 行ずつ読む。止める処理と道具の範囲を読み取れたらそこで止める。
- MCP の定義は4か所。リポジトリ直下の `.mcp.json`。`~/.claude.json` の上位キー `mcpServers`。`~/.claude.json` の `projects` の下の起動ディレクトリのパスの `mcpServers`。入っている plugin のフォルダ直下の `.mcp.json`。2 つ目と 3 つ目は `~/.claude.json` を Read で開いて取り出す。
- 索引と直近だけ読む。記憶の場所は、CLAUDE.md から指された先。索引は必ず読む。索引以外の記憶のファイルは、最終更新の新しい順に 5 件まで読む。auto memory が動いている時の置き場は、settings に `autoMemoryDirectory` があればそこ。無ければ既定の `~/.claude/projects/<プロジェクト>/memory/`。
- auto memory が止まっているかを見る。settings に `autoMemoryEnabled: false` があるか、`CLAUDE_CODE_DISABLE_AUTO_MEMORY` が `1` なら「今は読まれていない」として扱い、古いフォルダが残っていても読まれる記憶に数えない。
- 起動のたびに読まれる量は、ファイルの行数だけを合計する。MCP は行数に換算せず、接続の数だけを「見たもの」に添える。
- 読まなかった物を控え、診断書の「見たもの」の行に1文で書く。

## 2. 判定する

`criteria.md` の 46 セルを当て、採点が終わってから加点を当てる。

- セルの判定は クリア / 未クリア / 確かめられず。未クリアにできるのは実物を引用できた時だけ。見る場所が読めなければ確かめられず。
- 同じ事実が複数のセルに当たれば両方に当てる。同じセルに事実が幾つあっても未クリアは1回。
- 跡を探す場所は 3 つ。CLAUDE.md から指された記録の直近 5 件、`git log` の直近 5 件、hook や skill の本文が名指しする記録先。そこに無ければ跡は無いとする。
- skills / commands、subagents、hooks、MCP は、見る場所にハーネスのつもりで置かれた物が 1 つも無ければ「使っていない」にして分母から外す。置き場や形が公式と違う物でも、有れば使っている。CLAUDE.md / rules、permissions / settings、memory、全体は必ず見る。
- 部門のセルの過半を確かめられなければ、部門ごと「確かめられなかった」にして分母から外す。
- 見た部門が 1 つも無ければ、全体の点は出さず「確かめられなかった」と書く。
- 点の付け方は `criteria.md` に従う。
- 加点は、集めた物のうち 46 セルのどの判定にも使わなかった仕掛けを並べ、1 つずつ `criteria.md` の加点の分類に当てる。目的が名前か説明か記録から分かり、直近 30 日に回った跡を引用できる物だけ数える。分類ごとに 1 つでもあれば 1 点。加点は回っている分類の数で、上限 6。
- 数の線は固定する。skill の本文が長い = 500 行超。記録の書き込みが止まっている = 最後の書き込みから 90 日超。最近の履歴 = 直近 30 日。記憶の索引の上限 = 200 行か 25KB。全体の点は小数第一位を四捨五入。
- ファイルの大きさは `ls -la` で取る。最終更新は、git に入っているファイルは `git log -5 -- <パス>` の先頭の日付、入っていないファイルは `ls -la` の日時。中身に書かれた日付は使わない。25KB、90 日、30 日の線はこれで見る。

## 3. 返す

`report.md` の雛形を埋めて HTML を1ファイル作り、一時フォルダへ書いて開く。同じ場所に根拠の控えも書く。

- 診断書の名前は `harness-check-<年月日-時分>.html`。開くのは Windows が `start`、mac が `open`、Linux が `xdg-open`。開けない環境ではパスを画面に出す。
- 頭に診断日時、対象パス、`claude --version` の出力を載せる。
- 根拠の控えの名前は `harness-check-<年月日-時分>-basis.md`。部門ごとに、満点、セルごとの判定、未クリアのセルに当てた根拠の番号と重さと引用、部門の点を小数のまま並べる。続けて、部門ごとの ○ の数・大の数・中の数・全項目の数と、加点の分類ごとに候補にした仕掛けの一覧・条件の判定・引いた跡と、全体の点の計算を書く。検品用で、顧客には見せない。
- 画面に返す文は全部日本語にする。
- 診断書を作った後の検品の Grep に look-around を使わない。
- 文は `report.md` の9条に従う。
- 最後に、診断書と根拠の控えのパスを1行ずつ伝える。点や推奨をチャットへ書き写さない。
