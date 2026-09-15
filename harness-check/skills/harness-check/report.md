# 診断書の雛形

下の HTML をそのまま使い、`{{ }}` の所だけを埋める。1ファイルで完結させる。外から読み込む CSS・JavaScript・Web フォントを足さない。装飾の絵文字を使わない。文字コードは UTF-8。

## 雛形

```html
<!doctype html>
<html lang="ja">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>ハーネス診断書</title>
<style>
:root{--paper:#f2f1ec;--ink:#222831;--ink2:#3b434c;--muted:#6f7781;--faint:#9aa1a8;--line:#e2e0d8;--line2:#eeece5;--ac:#3d5a73;--ac-deep:#2b4055;--ac-soft:#eaeff3;--big:#ac432d;--big-bg:#f7e9e5;--big-ln:#e3c4bb;--mid:#b47c25;--mid-bg:#faf0dd;--mid-ln:#e6d3ac;--sml-bg:#edecea;--sml-ln:#dad8d3;--ok:#2f7a4f}
*{box-sizing:border-box}
html{color-scheme:light}
body{margin:0;background:var(--paper);color:var(--ink);font-family:"Hiragino Kaku Gothic ProN","Yu Gothic Medium","Yu Gothic",Meiryo,system-ui,-apple-system,sans-serif;font-size:14px;line-height:1.75;-webkit-font-smoothing:antialiased}
.sheet{max-width:880px;margin:0 auto;padding:26px 20px 70px}
.spec{border:1px solid var(--line);border-radius:3px;background:#fbfaf7;padding-bottom:22px}
.spec-top{display:flex;justify-content:space-between;align-items:baseline;gap:14px;flex-wrap:wrap;padding:13px 20px;background:var(--ac-deep);color:#fff;border-radius:2px 2px 0 0}
.spec-top b{font-size:13.5px;letter-spacing:.12em}
.spec-top span{font-size:11.5px;color:#b9c8d4;letter-spacing:.06em}
.spec-env{padding:12px 20px;font-size:12px;color:var(--muted);border-bottom:1px dashed var(--line)}
.total{display:flex;align-items:flex-end;gap:20px;padding:22px 20px 18px;flex-wrap:wrap;border-bottom:1px solid var(--line2)}
.total .num{font-size:66px;line-height:.9;font-weight:700;color:var(--ac-deep);font-variant-numeric:tabular-nums;letter-spacing:-.02em}
.total .unit{font-size:17px;color:var(--muted);margin-left:4px}
.tiles{display:grid;grid-template-columns:repeat(auto-fill,minmax(178px,1fr));gap:10px;padding:18px 20px}
.tile{border:1px solid var(--line);background:#fff;border-radius:2px;padding:11px 12px 12px}
.tile .nm{font-size:12px;color:var(--ink2);font-weight:700}
.tile .sc{display:flex;align-items:baseline;gap:3px;margin-top:5px}
.tile .sc b{font-size:26px;line-height:1;color:var(--ac-deep);font-variant-numeric:tabular-nums}
.tile .sc i{font-style:normal;font-size:11px;color:var(--faint)}
.tile .fm{margin-left:6px;font-size:15px;font-weight:700}
.fm.ok{color:var(--ok)}
.fm.ng{color:var(--big)}
.bar{height:5px;background:var(--line2);border-radius:3px;margin-top:8px;overflow:hidden}
.bar i{display:block;height:100%;background:var(--ac);border-radius:3px}
.tile.mid .bar i{background:var(--mid)}
.tile.low .bar i{background:var(--big)}
.tile.off{background:#f4f3f0;border-style:dashed}
.tile.off .nm{color:var(--faint)}
.tile.off .sc b{font-size:13px;line-height:2;color:var(--faint)}
.tile.off .bar i{background:var(--line)}
.tile .one{margin-top:7px;font-size:10.5px;color:var(--muted);line-height:1.4}
.bonus{display:flex;flex-wrap:wrap;gap:8px;align-items:center;padding:4px 20px 18px;border-bottom:1px solid var(--line2)}
.bonus .lb{font-size:11.5px;color:var(--muted);letter-spacing:.08em;margin-right:4px}
.brow{display:flex;align-items:baseline;gap:10px;flex:1 1 100%}
.chip{font-size:12px;border:1px solid var(--ac);border-radius:2px;padding:3px 10px;background:var(--ac-soft);color:var(--ac-deep);font-weight:700;white-space:nowrap}
.chip em{font-style:normal;margin-left:6px}
.brow .why{font-size:12px;color:var(--muted);line-height:1.6}
.close{margin:20px 20px 0;padding:12px 16px;background:var(--ac-soft);border-left:3px solid var(--ac);font-size:13px;color:var(--ac-deep);font-weight:700}
.recs{padding:16px 20px 0}
.recs .lb{font-size:11.5px;color:var(--muted);letter-spacing:.08em}
.rec{border:1px solid var(--line);border-left:3px solid var(--ac);background:#fff;border-radius:2px;padding:12px 14px;margin-top:10px}
.rec.big{border-left-color:var(--big)}
.rec.mid{border-left-color:var(--mid)}
.rec-h{display:flex;gap:9px;align-items:baseline;flex-wrap:wrap;margin-bottom:7px}
.rec-h .st{font-size:13px;font-weight:700;color:var(--ink)}
.rec p{margin:6px 0 0;color:var(--ink2);font-size:12.5px;line-height:1.8}
.w{display:inline-block;min-width:26px;text-align:center;font-size:11.5px;font-weight:700;line-height:1.6;padding:1px 7px;border-radius:2px;border:1px solid}
.w-big{color:var(--big);background:var(--big-bg);border-color:var(--big-ln)}
.w-mid{color:var(--mid);background:var(--mid-bg);border-color:var(--mid-ln)}
.w-sml{color:var(--muted);background:var(--sml-bg);border-color:var(--sml-ln)}
@media (max-width:640px){.sheet{padding:18px 14px 50px}.total .num{font-size:52px}}
</style>
</head>
<body>
<div class="sheet">
<div class="spec">
  <div class="spec-top"><b>ハーネス診断書</b><span>{{診断日時}}　対象: {{対象パス}}　Claude Code {{版}}</span></div>
  <div class="spec-env">ハーネスとは、Claude Code に読ませている設定と記録の一式です。</div>
  <div class="spec-env">{{見たもの}}</div>
  <div class="spec-env">{{読まなかったもの}}</div>
  <div class="total"><div><span class="num">{{全体の点}}</span><span class="unit">点 / 100</span></div></div>
  <div class="tiles">
{{部門タイル}}
  </div>
  <div class="bonus"><span class="lb">加点 +{{加点の合計}}</span>
{{加点}}
  </div>
  <div class="close">直すかどうかは、あなたが決めてください。AI の振る舞いに不満がなければ、このまま使って構いません。</div>
  <div class="recs"><span class="lb">直すなら</span>
{{推奨}}
  </div>
  <div class="close">{{最後の段}}</div>
</div>
</div>
</body>
</html>
```

繰り返す部分の形。`{{部門タイル}}` は7つ並べる。

```html
<div class="tile{{ 空 / mid / low }}">
  <div class="nm">{{部門名}}</div>
  <div class="sc"><b>{{点}}</b><i>/ 20</i><span class="fm {{ok / ng}}">{{○ / ×}}</span></div>
  <div class="bar"><i style="width:{{点を5倍した数}}%"></i></div>
  <div class="one">{{一言}}</div>
</div>
<div class="tile off">
  <div class="nm">{{部門名}}</div>
  <div class="sc"><b>使っていない</b></div>
  <div class="bar"><i style="width:100%"></i></div>
  <div class="one">{{一言}}</div>
</div>
```

```html
<div class="brow"><span class="chip">{{加点の名前}}<em>+2</em></span><span class="why">{{理由の1文}}</span></div>
<div class="rec {{big / mid / sml}}"><div class="rec-h"><span class="st">{{番号}}. {{見出し}}</span><span class="w w-{{big / mid / sml}}">{{大 / 中 / 小}}</span></div><p>{{本文}}</p></div>
```

## 埋め方

| 埋める所 | 何を入れるか |
|---|---|
| `{{診断日時}}` | 診断した日と時刻。`2026-09-15 14:20` の形 |
| `{{対象パス}}` | 起動ディレクトリ。ホームの下なら `~/` から書く |
| `{{版}}` | `claude --version` の出力 |
| `{{見たもの}}` | 見た物を部門の並びで1行。user 階層と起動ディレクトリを分けて書く。使っていない部門はここで「使っていません」と書く。最後に「起動のたびに読まれるのは、〇〇の N 行です」を添える |
| `{{読まなかったもの}}` | 読まなかった物を1行。skill の本文、記憶の古い分、秘匿ファイルなど |
| `{{全体の点}}` | 見た部門の平均を5倍した数。加点で 100 を超えたらそのまま書く |
| `{{部門タイル}}` | 7つ。並びは CLAUDE.md / rules、skills / commands、subagents、hooks、permissions / settings、MCP、memory |
| タイルの色 | 15 点以上は空、8〜14 は `mid`、7 以下は `low`、使っていない部門は `off`。表示だけの区分で、評価名は付けない |
| タイルの印 | 公式の形に合っていれば `ok` の `○`、外れていれば `ng` の `×`。使っていない部門は `<span>` ごと消す |
| `{{一言}}` | 点を引いた理由が分かる1文。満点の部門は何を見たかを書く。使っていない部門は、無くて困らないか・いつ要るかを書く |
| `{{加点の合計}}` | 得た加点の合計。0 なら `.bonus` の行ごと消す |
| `{{加点}}` | 得た物だけ。得ていない加点は出さない |
| `{{推奨}}` | 重い順。`big` が 大、`mid` が 中、`sml` が 小。番号は 1 から連番 |
| `{{本文}}` | 実物の場所を名指しし、見つけた事、AI に何が起きるか、どうするかを続けて書く。大は厚く、中と小は短く |
| `{{最後の段}}` | 「今のハーネスの状態だと、AI はこう振る舞いやすくなります。」で始め、見つかった問題から導いた2〜3行を書き、保全を勧める1文で閉じる |

雛形の構造はそのまま挿入する。`{{部門タイル}}`・`{{加点}}`・`{{推奨}}` へ入れるのは上の形の HTML そのもので、これはエスケープしない。エスケープするのは、その中に入れる末端の文字列だけ。

末端の文字列とは、`{{部門名}}`・`{{一言}}`・`{{加点の名前}}`・`{{理由の1文}}`・`{{見出し}}`・`{{本文}}`・`{{診断日時}}`・`{{対象パス}}`・`{{版}}`・`{{見たもの}}`・`{{読まなかったもの}}`・`{{最後の段}}` に入る文字列。入れる前に `&` を `&amp;`、`<` を `&lt;`、`>` を `&gt;` に、この順で直す。

class の名前・幅の `%`・点の数字には、雛形に並んでいる決まった値と、計算した数字だけを入れる。診断した環境から取った文字列をそこへ入れない。

## 書き方

| # | 決まり |
|---|---|
| 1 | 人が書いた診断として読める文にする。ですます。項目名で文を切らず、同じ型の文を続けない |
| 2 | 冒頭に「ハーネスとは、Claude Code に読ませている設定と記録の一式です」の1文。見た物を部門の並びで1行 |
| 3 | 点は全体と部門だけ。計算式・平均・内訳を書かない |
| 4 | 部門の一言は点を引いた理由が分かる言い方。満点の部門は何を見たかを書く。使っていない部門は「使っていない」と、無くて困らないか・いつ要るかの一言 |
| 5 | 加点は得た物だけ、理由1文付き |
| 6 | 判定の内側の分類の名前、見る点の番号、判定基準の言い回しを出さない。最後の段に「今の状態で起こりやすい事」を、見つかった問題から導いて2〜3行書き、保全を勧める。本人の症状とは書かない。「直すかはあなたが決める」は点の直後に1回だけ |
| 7 | 推奨は重い順。実物の場所を名指しし、見つけた事・AI に何が起きるか・どうするかを書く。書く先・消す先のファイル名まで。大は厚く、中と小は短く |
| 8 | 同じ物は同じ呼び方で通す |
| 9 | 判定基準の抽象語を出さず、その環境の実名に置き換える。下の「使わない語」を出さない。作るたびに Grep で数える |

## 内側の値と、診断書での見せ方

| 内側で持つ物 | 診断書での見せ方 |
|---|---|
| 公式の線 | 部門ごとの「公式の形」の印。○ か ×。外れていれば推奨にも出す |
| 部門の点 | 数字と一言。「10 / 20」と、点を引いた理由が分かる1文 |
| 全体の点 | 数字だけ。「62 点 / 100」。加点で 100 を超えたらそのまま |
| 重さ 大・中・小 | 推奨の印と、推奨の並び順 |
| 判定の内側の分類 | 名前も番号も出さない。推奨の並び順と言い回しに使い、最後の段を導く材料にする |
| 加点 | 得た物だけ、理由1文付き |
| 使っていない部門 | 「使っていない」と、無くて困らないか・いつ要るかの一言 |
| 該当を裏付けた実物 | 推奨の本文に、場所として書く。裏付けの無い物は推奨に出さない |
| 見る点 34 本 | 出さない。推奨は見る点の名前ではなく、その環境で見つけた事として書く |
| 起動のたびに読まれる量 | 「見たもの」の行に、ファイルの行数として添える。MCP は行数に換算せず、接続の数だけを書く |
| 根拠の控え | 見る点の番号と引用は診断書に出さない。同じ一時フォルダの `harness-check-<年月日-時分>-basis.md` に書く。検品用で、顧客には見せない |

## 使わない語

顧客が読む物なので、この skill の中だけで通じる語を出さない。左を使わず右で書く。

| 使わない語 | 代わりに書く語 |
|---|---|
| 「屋根」 | そのフォルダ、その一式 |
| 「入口」 | CLAUDE.md |
| 「面」 | 画面 |
| 「枝」 | ブランチ |
| 「器」 | 置き場、フォルダ |
| 「箱」 | 置き場 |
| 「窓」 | セッション |
| 「関所」 | commit 前の検査 |
| 「検問」 | hook |
| 「正本」 | これが正しいと決めた文書 |
| 「版管理」 | バージョン管理 |
| 「回転」 | 使わない |
| 「締め」 | セッションの終わり |
| 「昇華」 | 使わない |
| 「パトロール」 | 定期の見回り |
| 「走行」 | 実行 |
| 「台帳」 | 一覧 |
| 「隊員」 | 使わない |
| 「所見」 | 見つけた事 |
| 「常時届く」 | 起動のたびに読まれる、毎回読まれる |
| 「手順書」 | skill |
| 「書き口、引き口」 | 記憶を書く場所、記憶を読み返す道筋 |
| 「対象外」 | 使っていない |

欄の名前として使わない語。診断書の中で `痕跡` `影響` `直し方` を項目名にしない。文の中に溶かして書く。

## 作った後に数える

HTML を作ったら、次を確かめてから開く。数えるのは Claude Code の Grep で行う。shell の `grep` は使わない。

1. 上の「使わない語」の左の列を、作った HTML に対して Grep する。0 件であること。
2. `痕跡` `影響` `直し方` を Grep する。項目名として出ていないこと。
3. 見る点の番号(`1-1` のような形)と、内側の分類の番号が出ていないこと。
4. 計算式、平均、`/ 20` 以外の内訳が出ていないこと。
5. 鍵・token・password の値が出ていないこと。伏せ読みの `***` 以外の値が無いこと。
6. `{{` が1つも残っていないこと。
