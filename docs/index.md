# 「PostgreSQL」

The PostgreSQL driver for Suru, used with the database layer 「蔵」. It calls libpq through external functions.
Add it with `kumu 追加 github.com/suru-lang/postgresql` and write `「PostgreSQL」 を 読み込んで 使う` in your file.

`先 を PostgreSQL駆動 で DB開封する` connects. 先 is a libpq connection string (「host=localhost dbname=店 user=花子」 or 「postgresql://花子@localhost/店」). With 「」, the environment variables PGHOST, PGUSER, PGPASSWORD and so on are used.
Then use the functions of 「蔵」 (`DB実行する`, `DB差込問合する`, `DB行問合する`, `DB取引する` and so on).

- 「?」 is rewritten to 「$1」, 「$2」 and so on (not inside strings or quoted names).
- `DB実行する` without values can run several statements separated by 「;」.
- `DB最終番号取得する` is SELECT lastval() (the sequence value last advanced on the same connection).
- After connecting, client_min_messages is set to WARNING, so NOTICE is not written to standard error.
- Values read: integers, decimals and numeric are `DB数`, boolean is `DB真偽`, bytea is `DBバイト列`, and the rest are `DB文字列`.

A failure is a runtime error such as 「PostgreSQL で『SELEC 1』を実行するのに失敗しました(ERROR:  syntax error at or near "SELEC" …)」, which `試行する` can catch.

```{toctree}
:maxdepth: 2
:caption: API
:glob:

api/*
```
