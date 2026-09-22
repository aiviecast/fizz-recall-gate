# fizz-recall-gate

Fizz の **recall ゲート**。recall の生結果を「brain が実際に使う最小限」に絞り、
デッドラインに間に合わなかった時の「記憶なし」フォールバックを供給する。

recall は hot path で 30ms デッドライン付き。L1/L2/session の参照が間に合えば
生 `RecallResult` が組み上がるが、間に合わなければ記憶なしで進める必要がある。
この部品は 2 つの役割を持つ:

1. **gate(r, cap)** — 生結果の `recent` を直近 `cap` 件に絞る。会話は前の話題を
   引きずらない方針なので、recall 全量はプロンプトに流さず summary + 直近わずか
   だけを通す。
2. **empty_*** — 記憶なしの well-formed な `RecallResult` を作る。デッドライン
   超過・新規ユーザ時に downstream が常に整った形を受け取れる。

時間計測(実際の 30ms 判定)は呼び出し側(orchestrator)の責務。この部品は純粋。

## I/O

```jsonc
{"op":"gate","result":{...RecallResult...},"cap":4}
{"op":"empty","user":{...MemoryUser...}}
{"op":"empty_new","user_id":7,"name":"ねこ","ts":100}
```

いずれも `{...RecallResult...}` を 1 行で返す。`cap` 省略時は既定 `RECENT_CAP=4`。

## 開発

```sh
almide check src/main.almd
almide test
almide build src/main.almd -o build/fizz-recall-gate
```

ツールチェーン: [almide](https://github.com/almide/almide) v0.26.6+。

## 契約

[fizz-protocol](https://github.com/aiviecast/fizz-protocol) の `memory` モジュール
(`RecallResult` / `MemoryUser` / `MemoryEvent` とその Codec)に依存。
