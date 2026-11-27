[English](README.md) | 日本語

# PostgreSQL(する のデータベースの層「蔵」の駆動)

日本語のプログラミング言語「[する](https://github.com/suru-lang/suru)」から、データベースの層 [「蔵」](https://github.com/suru-lang/db) を通して PostgreSQL を使う駆動です。libpq を外部の関数として呼びます。

## 入れ方

[kumu](https://github.com/suru-lang/suru/blob/main/README_jp.md#プロジェクトを組むkumu) のプロジェクトで:

```sh
kumu 追加 github.com/suru-lang/postgresql
```

「蔵」(suru-lang/db)も依存として一緒に入ります。システムに libpq が要ります(組.suru の `外部依存`。無ければ kumu が入れ方を出して止まります)。

する 本体は 0.9.1 以上が要ります(ライブラリが C の関数を「以下は外部の入出力動作です … 以上です」で書くため)。

- macOS: `brew install libpq` して `export PKG_CONFIG_PATH="$(brew --prefix libpq)/lib/pkgconfig"`
- Debian・Ubuntu: `sudo apt install libpq-dev`
- Fedora: `sudo dnf install libpq-devel`

## 使い方

```
「蔵」 を 読み込んで 使う
「PostgreSQL」 を 読み込んで 使う
「標準」 を 読み込んで 使う

DB行 を 品表示文にする は 文字列
行 を 品表示文にする とは （行 から 「名前」 を DB列取得して DB文字列取出する） + 「 は 」 + （行 から 「値段」 を DB列取得して DB値文字列にする） + 「 円です」

主処理する とは 入出力手順
  db とは 「」 を PostgreSQL駆動 で DB開封した 結果
  db に 「DROP TABLE IF EXISTS 品; CREATE TABLE 品 (番号 BIGSERIAL PRIMARY KEY, 名前 TEXT NOT NULL, 値段 DOUBLE PRECISION)」 を DB実行する
  db で （db に 「INSERT INTO 品 (名前, 値段) VALUES ('りんご', 120), ('みかん', 80.5), ('ぶどう', 450)」 を DB実行する） を DB取引する
  行群 とは db に 「SELECT 名前, 値段 FROM 品 WHERE 値段 < ? ORDER BY 値段」 を （（200 の DB数） を 空 に 挿入する） で DB行問合した 結果
  （行群 の それぞれ を 品表示文にして 改行 で 結合する） を 表示する
  db に 「DROP TABLE 品」 を DB実行する
  db を DB閉鎖する
  「以上です」 を 表示する

主処理する
```

関数はどれも「蔵」のものです(「蔵」の README を見てください)。`PostgreSQL駆動` を `SQLite駆動` に替えれば、同じプログラムが SQLite で動きます。例は [例/店.suru](例/店.suru) です。

## 決めたこと

1. **つなぐ。** `先 を PostgreSQL駆動 で DB開封する` でつなぎます。`先` は libpq の接続文字列です(`「host=localhost dbname=店 user=花子」` や `「postgresql://花子@localhost/店」`)。`「」` なら、libpq が環境変数 `PGHOST`・`PGUSER`・`PGPASSWORD`・`PGDATABASE` などを使います。つないだら `client_min_messages` を WARNING にするので、NOTICE は標準エラーに出ません。
2. **プレースホルダ。** 文は「蔵」のどこでも同じく「?」で書きます。駆動が `$1, $2, …` に書き換え(文字列と囲んだ名前の中はそのまま)、値は `PQexecParams` で別に渡すので、値の中の引用符が SQL を壊すことはありません。バイト列はバイナリで送ります。
3. **何文でも。** 値のない `DB実行する` は `PQexec` を使うので、「;」で区切った何文でも一度に実行できます。
4. **読んだ値。** 整数・小数・numeric の列は `DB数`、boolean は `DB真偽`、bytea は `DBバイト列`、NULL は `DB無`、ほかは `DB文字列`(日付などは PostgreSQL が文字で書いたもの)です。
5. **最後の番号。** `DB最終番号取得する` は `SELECT lastval()` で、同じ接続で最後に連番(`BIGSERIAL` の列など)から取った値です。`INSERT … RETURNING 番号` を `DB問合する` で使うこともできます。
6. **失敗は実行時エラー。** 「PostgreSQL で『SELEC 1』を実行するのに失敗しました(ERROR:  syntax error at or near "SELEC" …)」のように PostgreSQL の理由が付き、`試行する` で受け止められます。失敗しても結果は解放します。

## API ドキュメント

API ドキュメントは `PostgreSQL.suru` の注釈から作ります。ファイルの頭の注釈と、公開の定義のすぐ上の注釈で、英語で書いています。公開先は https://suru-lang-postgresql.readthedocs.io/en/latest/ です。

```sh
kumu 文書                                                            # docs/api/PostgreSQL.md を書く
pip install -r docs/requirements.txt && sphinx-build docs docs/_build   # docs/_build に HTML
```

`docs/` と `.readthedocs.yaml` は `kumu 文書 --雛形`(する 0.11.0 以上)で書きました。Read the Docs はビルドのたびに kumu 0.11.0 を取ってきて `kumu 文書 --出力 docs/api` を動かしてから Sphinx を動かすので、作ったページはコミットしません。エディタのホバーにも同じ説明が出ます。

## 試験

試験は本物の PostgreSQL のサーバーを使います。つなぐ先は環境変数(`PGHOST`・`PGUSER`・`PGPASSWORD`・`PGDATABASE`)で決めます。`人_…` という名前の表を作って消します。

```sh
PGHOST=localhost PGUSER=postgres PGPASSWORD=… kumu 試験
kumu 点検
```

## 今後の候補

- 日付・時刻・JSON の列を、文字列でなく する の値で読む
- LISTEN / NOTIFY
- たくさんの行を速く入れる COPY

## ライセンス

[MIT](LICENSE)
