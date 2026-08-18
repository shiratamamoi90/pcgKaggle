# ポケカ AI Battle Challenge — ドキュメントと champion

Kaggle [Pokémon TCG AI Battle Challenge](https://www.kaggle.com/competitions/pokemon-tcg-ai-battle)
（Simulation カテゴリ, 2026-06-20〜08-16）の記録と、最終的に champion となったエージェント。

**最終結果: 846.1 / 673位 6,892チーム中（上位 9.8%）、提出総数 50。**
※締切後もラダーは対戦を続けているため順位は暫定（08-17 のスナップショット）。

このリポジトリの価値は「強いエージェント」ではなく、
**何が効かないかを対照群を置いて50提出ぶん測り切った記録**の側にある。

## 中身

| パス | 内容 |
|---|---|
| `docs/FINDINGS.md` | **何を測り何が分かったか。** 主張はすべて数字と紐づけ済み |
| `docs/CHRONICLE.md` | **全48施策の詳細記録。** 1件ごとに 目標 / 実施 / 成果または悪化 / 次の手順 |
| `docs/LINEAGE.md` | 技術の正典。全世代の中身・復元手順・**死んだ軸の台帳**（③k から読む） |
| `docs/README.md` | 現在地・最終結果・Strategy カテゴリの仕様 |
| `docs/WRITEUP-GUIDE.md` | Strategy カテゴリ（技術レポート）向けの整理 |
| `docs/spec.md` `procedure.md` `eval-axes.md` | Simulation 期の設計意図（当時の記載を保持） |
| `CLAUDE.md` | 運用ルール（測定のルール・提出の制約）。2部構成で旧記載を全文保持 |
| `champion/` | 実績 champion = 6日版 Majkel-BC（`submission_majkel_bc_6d.tar.gz` の中身） |

## champion/

| ファイル | 備考 |
|---|---|
| `main.py` | 推論本体。探索予算は `NN_SEARCH=1`（貪欲）が正しい設定 |
| `model.pth` | md5 `ef223b09fc087a8fa421e646c5517fa1` |
| `deck.csv` | 全提出 tar の実デッキ（Alakazam ex + Dudunsparce） |

提出 tar は上記3ファイルに公式エンジン `cg/` を同梱した非ネスト構成だが、
**`cg/` は再配布禁止のためこのリポジトリには含めない。** 実行するには Kaggle から各自取得して配置する。

## 含まれないもの

- `cg/`（公式エンジン）, `opponents/`, `external/`（第三者の公開エージェント・カーネル） — **再配布禁止**
- `submission_*.tar.gz` — `cg/` を同梱しているため同上
- `episodes/`（実ラダーリプレイ 145GB）, `venv/`, `models_*/`, `logs/`, `scratchpad/` — ローカル保持

## 測定に関する最重要の注意

過去の数値を引用する前に、**それがどの測定器で取られたか**を確認すること。
`opponents/grimmsnarl` 由来の数値は無効（07-06製の古い方策で、実ラダーと **42.4pp** ずれていた）。
有効なのは `opponents/metapool`（実ラダーとの差 0.7pp）。詳細は `docs/FINDINGS.md` §4。
