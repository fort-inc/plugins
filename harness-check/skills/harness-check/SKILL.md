---
name: harness-check
description: ハーネスを診断し、Claude Code に読ませている設定と記録の状態に点を付けて、直すとよい所を日本語の診断書 HTML にして開きます。「ハーネスを診て」「診断して」「チェックして」「保全したい」と頼まれた時と、AI の振る舞いを見直したい時に呼びます。
---

# ハーネス診断

ハーネスとは、Claude Code に読ませている設定と記録の一式。この skill は、それを読んで診断書の HTML を作り、ブラウザで開く。判定の基準は同じフォルダの `criteria.md`、診断書の形と文の決まりは同じフォルダの `report.md`。両方を先に読む。

## 守る事

- 相手の環境に何も作らない・変えない・消さない。書き出すのは OS の一時フォルダだけ。Windows は `%TEMP%`、mac と Linux は `$TMPDIR` か `/tmp`。
- 読み取り以外の shell を使わない。使ってよいのは下の6つと、一時フォルダへの書き出し、ブラウザで開く操作だけ。テストやビルドなど相手のコマンドは走らせない。
  - `git --no-optional-locks status --porcelain`
  - `git remote`(名前だけ。URL を出す `-v` は使わない)
  - `git log -5 --format="%h %ad %s" --date=short`
  - `claude --version`
  - 一時フォルダに書いた `harness-check-<年月日-時分>-mask.js`(または `-mask.py`)を `node` か `python3` か `python` で実行する
  - `node -e "console.log(process.env.CLAUDE_CODE_DISABLE_AUTO_MEMORY||'unset')"`
  - `date +%Y%m%d-%H%M`(診断書と控えと script の名前に付ける時刻を取る)
- 探すのは Claude Code の Grep と Glob で行う。shell の `grep`・`find`・`rg` は使わない。
- `claude mcp list` は走らせない。MCP の接続の状態は、このセッションの冒頭に出ている MCP の一覧で見る。表示が無ければ MCP の見る点を「確かめられなかった」にする。
- shell は1命令ずつ打つ。`&&` や `;` で繋がない。1つが止められても他が失われないようにするため。
- Windows でも shell の命令は Bash の道具で打つ。PowerShell の道具は使わない。複数の操作を含む命令が止められやすく、止められた後の打ち直しが代替の実行になってしまうため。
- 命令の中で環境変数を展開しない。一時フォルダのパスは実際の文字列で書く。Windows は `C:/Users/<ユーザー名>/AppData/Local/Temp` と斜線区切りで書く。円記号区切りは Bash の道具で壊れる。mac と Linux は `/tmp`。
- ファイルを書き出すのは Claude Code の Write だけ。shell のリダイレクトやヒアドキュメントで書かない。
- hook や許可の設定で止められたら、別の道具や別の書き方で代替しない。同じ事を別の道具で打ち直さない。書き方を変えて通そうとしない。読み取りの1つが止められた時は、その物を「確かめられなかった」にして診断を続け、止まった内容を「読まなかったもの」に書く。診断書の書き出しが止められた時は、パスの代わりに止まった内容をそのまま伝えて終える。

## 開かないファイル

次は開かない。有る事と置き場だけを見る。`@` の取り込み先、CLAUDE.md から名指しされた文書、hook の実体、記憶のファイルも例外にしない。読む前に必ずこの一覧を当てる。

`.env` と `.env.*`、`.envrc`、`*.pem`、`*.key`、`*.p12`、`*.pfx`、`*.jks`、`*.keystore`、`*.kdbx`、`*.ppk`、`id_rsa*`、`id_ed25519*`、`id_ecdsa*`、`.npmrc`、`.netrc`、`.pypirc`、`.git-credentials`、`*.tfvars`、`credentials*`、`secrets.*`、`.docker/config.json`、`application_default_credentials.json`、名前に service_account か service-account を含む JSON、`.aws/` と `.gcloud/` の下、名前に secret・token・password・passwd・auth・apikey・api_key を含む物。

例外が1つ。settings の hooks に実体として登録されている script は、名前に secret や guard を含んでいても開いてよい。何を止める処理かを読むために要る。

開いてよいファイルでも、引用する行に鍵らしい文字列が含まれる時は、その部分を `***` に替えて引く。鍵らしいとは、`sk-`・`ghp_`・`gho_`・`github_pat_`・`xox[abpr]-`・`AKIA`・`AIza`・`ya29.`・`eyJ`・`glpat-`・`npm_`・`Bearer ` で始まる物。または、空白と `/` と `\` で区切った一片が、丸括弧を含まず 24 文字以上で英字と数字の両方を含む物。

## 設定と MCP の JSON の読み方

`.claude/settings.json`、`.claude/settings.local.json`、`~/.claude/settings.json`、`.mcp.json`、`~/.claude.json` は直接開かない。一時フォルダに下の script を `harness-check-<年月日-時分>-mask.js` の名前で書き、`node <一時フォルダ>/harness-check-<年月日-時分>-mask.js <JSON のパス> [上位キー] [その中のキー]` で読む。時刻は診断書と同じ物にする。前に走らせた時のファイルとぶつからないようにするため。`node` が無ければ同じ場所に `harness-check-<年月日-時分>-mask.py` を書き、`python3` か `python` で同じ引数で読む。どちらも無い時は JSON を読まず、permissions / settings・hooks・MCP の見る点を「確かめられなかった」にする。伏せられない時は読まない。script は見せてよい物だけを通す。出力の `NOT-FOUND` は指したキーが無い事、`READ-FAILED` は読めなかった事を指す。どちらもその JSON を読めなかった物として扱う。

```js
const fs=require('fs'),A=process.argv;
const PASS=new Set(['allow','deny','ask','additionalDirectories','matcher','type','defaultMode','model','name','scope','autoMemoryDirectory','cwd','shell']);
const PRE=/(sk-|ghp_|gho_|github_pat_|xox[abpr]-|AKIA|AIza|ya29\.|eyJ|glpat-|npm_)[^\s"']*/g;
const keep=s=>s.replace(/Bearer\s+\S+/g,'***').replace(PRE,'***').split(/([\s\/\\]+)/).map(w=>[...w].length>=24&&!/[()]/.test(w)&&/[A-Za-z]/.test(w)&&/[0-9]/.test(w)?'***':w).join('');
const fin=v=>Array.isArray(v)?v.map(fin):v&&typeof v==='object'?Object.fromEntries(Object.entries(v).map(([a,b])=>[a,fin(b)])):typeof v==='string'?keep(v):v;
const arg=v=>/^-/.test(v)&&!v.includes('=')?v:v.includes('=')?v.slice(0,v.indexOf('=')+1)+'***':'***';
const bare=w=>w.replace(/^['"]+/,'').replace(/['"]+$/,'');
const pathy=w=>{const b=bare(w);return /^\$\{?CLAUDE_PROJECT_DIR\}?/.test(b)||/^~\//.test(b)||/^\//.test(b)||/^[A-Za-z]:/.test(b)||/^\.\.?\//.test(b)||/\.(py|sh|js|mjs|ps1|cmd|bat|rb)$/.test(b);};
const cmd=s=>{let f=1;return s.split(/(\s+)/).map(t=>/^\s*$/.test(t)?t:f?(f=0,keep(t)):pathy(t)?keep(t):arg(t)).join('');};
const url=s=>{const m=/^([A-Za-z][\w+.-]*:\/\/)([^/?#]*)([^?#]*)/.exec(s);if(!m)return'***';
  const h=m[2].includes('@')?'***@'+m[2].slice(m[2].lastIndexOf('@')+1):m[2];
  return m[1]+h+m[3]+(s.includes('?')?'?***':'')+(s.includes('#')?'#***':'');};
function walk(v,k,sec){
  if(Array.isArray(v))return v.map(x=>sec?walk(x,k,1):(k==='args'&&typeof x==='string')?arg(x):walk(x,k,0));
  if(v&&typeof v==='object')return Object.fromEntries(Object.entries(v).map(([a,b])=>[a,walk(b,a,sec||a==='env'||a==='headers')]));
  if(sec)return'***';
  if(v===null||typeof v==='number'||typeof v==='boolean')return v;
  if(typeof v!=='string')return'***';
  return k==='command'?cmd(v):k==='url'?url(v):PASS.has(k)?keep(v):'***';}
try{let d=JSON.parse(fs.readFileSync(A[2],'utf8'));
  for(const k of A.slice(3)){if(!d||typeof d!=='object'||!(k in d)){console.log('NOT-FOUND');process.exit(2);}d=d[k];}
  console.log(JSON.stringify(fin(walk(d,undefined,0)),null,1));}
catch(e){console.log('READ-FAILED');process.exit(1);}
```

```python
import json,re,sys
PASS={'allow','deny','ask','additionalDirectories','matcher','type','defaultMode','model','name','scope','autoMemoryDirectory','cwd','shell'}
PRE=re.compile(r'(sk-|ghp_|gho_|github_pat_|xox[abpr]-|AKIA|AIza|ya29\.|eyJ|glpat-|npm_)[^\s"\']*')
def keep(s):
    s=PRE.sub('***',re.sub(r'Bearer\s+\S+','***',s))
    return ''.join('***' if len(w)>=24 and not re.search(r'[()]',w) and re.search(r'[A-Za-z]',w) and re.search(r'[0-9]',w) else w for w in re.split(r'([\s/\\]+)',s))
def arg(v):
    if v.startswith('-') and '=' not in v: return v
    return v[:v.index('=')+1]+'***' if '=' in v else '***'
def bare(w): return re.sub(r'[\'"]+$','',re.sub(r'^[\'"]+','',w))
def pathy(w):
    b=bare(w)
    return bool(re.match(r'\$\{?CLAUDE_PROJECT_DIR\}?',b) or re.match(r'~/',b) or b.startswith('/') or re.match(r'[A-Za-z]:',b) or re.match(r'\.\.?/',b) or re.search(r'\.(py|sh|js|mjs|ps1|cmd|bat|rb)$',b))
def cmd(s):
    out=[];first=True
    for t in re.split(r'(\s+)',s):
        if t.strip()=='': out.append(t)
        elif first: out.append(keep(t)); first=False
        elif pathy(t): out.append(keep(t))
        else: out.append(arg(t))
    return ''.join(out)
def url(s):
    m=re.match(r'([A-Za-z][\w+.-]*://)([^/?#]*)([^?#]*)',s)
    if not m: return '***'
    h=m.group(2)
    if '@' in h: h='***@'+h[h.rindex('@')+1:]
    return m.group(1)+h+m.group(3)+('?***' if '?' in s else '')+('#***' if '#' in s else '')
def walk(v,k=None,sec=False):
    if isinstance(v,list):
        o=[]
        for x in v:
            if sec: o.append(walk(x,k,True))
            elif k=='args' and isinstance(x,str): o.append(arg(x))
            else: o.append(walk(x,k,False))
        return o
    if isinstance(v,dict): return {a:walk(b,a,sec or a in ('env','headers')) for a,b in v.items()}
    if sec: return '***'
    if v is None or isinstance(v,(bool,int,float)): return v
    if not isinstance(v,str): return '***'
    if k=='command': return cmd(v)
    if k=='url': return url(v)
    return keep(v) if k in PASS else '***'
def fin(v):
    if isinstance(v,list): return [fin(x) for x in v]
    if isinstance(v,dict): return {a:fin(b) for a,b in v.items()}
    return keep(v) if isinstance(v,str) else v
try:
    d=json.load(open(sys.argv[1],encoding='utf-8'))
    for k in sys.argv[2:]:
        if not isinstance(d,dict) or k not in d: print('NOT-FOUND'); sys.exit(2)
        d=d[k]
    print(json.dumps(fin(walk(d)),ensure_ascii=False,indent=1))
except SystemExit: raise
except Exception:
    print('READ-FAILED'); sys.exit(1)
```

## 1. 集める

user 階層(`~/.claude/`)と起動ディレクトリを分けて集める。診断書でも分けて書く。見る場所の一覧は `criteria.md` にある。

- 全部読む。両階層の CLAUDE.md、`CLAUDE.local.md`、`AGENTS.md`、`.claude/rules/` の中身、CLAUDE.md が `@` で取り込んでいる先、CLAUDE.md から名指しされた文書、毎回読まれる記憶。
- `CLAUDE.md` と `CLAUDE.local.md` は、起動ディレクトリからファイルシステムの根まで各階層を見る。`.claude/rules/`・`.claude/skills/`・`.claude/agents/` は git の根までにする。
- 起動ディレクトリより下の階層は、`CLAUDE.md` と `.claude/` を Glob で数え、その数を「見たもの」に添える。下の `CLAUDE.md` は 10 枚まで読み、二重と矛盾の目だけで見る。11 枚目からと、下の `.claude/skills/`・`.claude/agents/` は数だけを「読まなかったもの」に書く。下の階層の物は起動のたびに読まれる量に入れない。
- 冒頭だけ読む。skill と subagent は frontmatter と行数だけ。例外は2つ。外から持ち込んだ skill と plugin の本文は読む。subagent の本文も読む。
- MCP の定義は3か所。リポジトリ直下の `.mcp.json`。`~/.claude.json` の上位キー `mcpServers`。`~/.claude.json` の `projects` の下の起動ディレクトリのパスの `mcpServers`。後の2つは伏せ読みの引数2と3で取り出す。
- 索引と直近だけ読む。記憶の場所は、伏せ読みした settings に `autoMemoryDirectory` があればそこ。無ければ既定の `~/.claude/projects/<プロジェクト>/memory/`。
- auto memory が止まっているかを見る。settings に `autoMemoryEnabled: false` があるか、`CLAUDE_CODE_DISABLE_AUTO_MEMORY` が `1` なら「今は読まれていない」として扱い、古いフォルダが残っていても読まれる記憶に数えない。
- 起動のたびに読まれる量は、ファイルの行数だけを合計する。MCP は行数に換算せず、接続の数だけを「見たもの」に添える。
- 読まなかった物を控える。診断書に1行で書く。

## 2. 判定する

`criteria.md` の見る点 34 本と加点 8 本を当てる。

- 該当にできるのは、実物を引用できた時だけ。引用とはファイルと行、または実行結果の写し。
- 引用できない物は「確かめられなかった」とし、点を引かない。
- skills / commands、subagents、hooks、MCP は、無ければ「使っていない」にして平均から外す。CLAUDE.md / rules、permissions / settings、memory は必ず見る。
- 点の付け方と、部門ごとの「公式の形」の印は `criteria.md` に従う。
- 数の線は固定する。skill の本文が長い = 500 行超。記録の書き込みが止まっている = 最後の書き込みから 90 日超。最近の履歴 = 直近 30 日。全体の点は小数第一位を四捨五入。

## 3. 返す

`report.md` の雛形を埋めて HTML を1ファイル作り、一時フォルダへ書いて開く。同じ場所に根拠の控えも書く。

- 診断書の名前は `harness-check-<年月日-時分>.html`。開くのは Windows が `start`、mac が `open`、Linux が `xdg-open`。開けない環境ではパスを画面に出す。
- 頭に診断日時、対象パス、`claude --version` の出力を載せる。
- 根拠の控えの名前は `harness-check-<年月日-時分>-basis.md`。推奨ごとに、番号、当てた見る点の番号(`1-3` の形)、引用した実物(パスと行と引用)、重さを並べる。検品用で、顧客には見せない。
- 文は `report.md` の9条に従う。作った後、`report.md` の「使わない語」を、1語ずつ語そのものを Grep で数えて 0 件を確かめる。まとめて正規表現にしない。
- 最後に、診断書と根拠の控えのパスを1行ずつ伝える。点や推奨をチャットへ書き写さない。
