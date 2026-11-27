English | [日本語](README_jp.md)

# PostgreSQL (a driver for Suru's database layer 蔵)

A driver that lets [Suru (「する」)](https://github.com/suru-lang/suru), a programming language in Japanese, use PostgreSQL through its database layer [「蔵」](https://github.com/suru-lang/db). It calls libpq as foreign functions.

## Installing

In a [kumu](https://github.com/suru-lang/suru#building-projects-kumu) project:

```sh
kumu 追加 github.com/suru-lang/postgresql
```

「蔵」 (suru-lang/db) comes along as a dependency. libpq is required on the system (the `外部依存` in 組.suru; if it is missing, kumu shows how to install it and stops).

Suru 0.9.1 or later is required (the library writes its C functions with 「以下は外部の入出力動作です … 以上です」).

- macOS: `brew install libpq`, then `export PKG_CONFIG_PATH="$(brew --prefix libpq)/lib/pkgconfig"`
- Debian, Ubuntu: `sudo apt install libpq-dev`
- Fedora: `sudo dnf install libpq-devel`

## Usage

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

All the functions are those of 「蔵」 (see its README). Replacing `PostgreSQL駆動` with `SQLite駆動` runs the same program on SQLite. An example is [例/店.suru](例/店.suru).

## Decisions

1. **Connecting.** `先 を PostgreSQL駆動 で DB開封する` connects. `先` is a libpq connection string (`「host=localhost dbname=店 user=花子」` or `「postgresql://花子@localhost/店」`). With `「」`, libpq uses the environment variables `PGHOST`, `PGUSER`, `PGPASSWORD`, `PGDATABASE` and so on. After connecting it sets `client_min_messages` to WARNING, so NOTICE messages are not written to standard error.
2. **Placeholders.** Statements are written with 「?」 as in all of 「蔵」. The driver changes them to `$1, $2, …` (leaving strings and quoted names alone) and passes the values separately with `PQexecParams`, so quotes in values can't break the SQL. Byte strings are sent as binary.
3. **Several statements.** `DB実行する` without values uses `PQexec`, so several statements separated by 「;」 can be run at once.
4. **Values read back.** Integer, decimal and numeric columns become `DB数`, boolean `DB真偽`, bytea `DBバイト列`, NULL `DB無`, and everything else `DB文字列` (as PostgreSQL writes it as text, e.g. dates).
5. **The last id.** `DB最終番号取得する` is `SELECT lastval()`: the value most recently taken from a sequence (such as a `BIGSERIAL` column) on the same connection. `INSERT … RETURNING 番号` with `DB問合する` also works.
6. **Failures are runtime errors.** They carry PostgreSQL's message, like 「PostgreSQL で『SELEC 1』を実行するのに失敗しました(ERROR:  syntax error at or near "SELEC" …)」 ("failed to run 『SELEC 1』 in PostgreSQL (…)"), and are catchable with `試行する`. Results are freed even on failure.

## API documentation

The API documentation is made from the comments in `PostgreSQL.suru`: the head comment of the file and the comment right above each public definition, written in English. It is published at https://suru-lang-postgresql.readthedocs.io/en/latest/.

```sh
kumu 文書                                                            # writes docs/api/PostgreSQL.md
pip install -r docs/requirements.txt && sphinx-build docs docs/_build   # HTML in docs/_build
```

`docs/` and `.readthedocs.yaml` were written by `kumu 文書 --雛形` (Suru 0.11.0 or later). On every build, Read the Docs downloads kumu 0.11.0, runs `kumu 文書 --出力 docs/api` and then Sphinx, so the generated pages are not committed. The editor hover shows the same descriptions.

## Tests

The tests use a real PostgreSQL server, chosen with the environment variables (`PGHOST`, `PGUSER`, `PGPASSWORD`, `PGDATABASE`). They create and drop tables named `人_…`.

```sh
PGHOST=localhost PGUSER=postgres PGPASSWORD=… kumu 試験
kumu 点検
```

## Future work

- Read date, time and JSON columns into Suru values instead of strings
- LISTEN / NOTIFY
- COPY for loading many rows quickly

## License

[MIT](LICENSE)
