# 仕様 (spec.md)

## やりたいこと

ポケカ AI Battle Challenge に参加し、**サンプルのルールベースエージェントを流用して
valid submission を1回通したうえで**、自己対戦の評価スクリプトで数字ベースに改良を
回し続けられる開発ループを WSL2 上に構築する。

- フェーズA: 最小構成で1回提出を成立させる
- フェーズB: 評価スクリプトを足して、改良 → 計測 → 再提出 のループを回す

## 成果物（提出物）

| ファイル | 内容 |
| --- | --- |
| `main.py` | `agent(obs_dict) -> list[int]` を実装。毎ターン obs を受け取り、選んだ選択肢のインデックス配列を返す |
| `deck.csv` | カードID 60行（1行1枚） |
| `submission.tar.gz` | 上記をトップレベルに含むバンドル。ネスト禁止 |

## エージェント仕様

- `agent(obs_dict)` は `obs_dict["select"]["option"]` の中から選び、`maxCount` 件のインデックスを返す。
- エンジンは**常に合法手のみ**を提示するため、ルール違反の心配は不要。
- 方針は**既存サンプルのルールベース流用**。メガルカリオex / Iono などのサンプルを起点に、
  小さな条件分岐の改良を積み重ねる。

### obs の構造（cabt Engine）
- `logs`: 過去のアクション/イベントのログ
- `current`: 現在の盤面（players, stadium, turn等）。初期デッキ選択フェーズでは None のことがある
- `select`: 選択肢。ここの option のインデックスが action になる
- PlayerState: active / bench(最大5) / hand(自分のみ中身可視) / prize / deckCount / discard /
  状態異常フラグ(poisoned, burned, asleep, paralyzed, confused) 等

## 環境

- WSL2 + Python仮想環境
- `kaggle-environments` と cabt エンジン（cgライブラリ）をKaggleからダウンロードして `engine/` に配置
- Kaggle CLI（`kaggle competitions submit`）または Kaggle UI で提出

## 制約・ルール

- 順位は Elo（勝敗のみ）。スコアの中身は採点されない。各参加者の最上位ボットのみ追跡。
- 1日最大5提出。追跡されるのは最新2提出。
- 新規提出は自分のコピーとの Validation Episode を通過する必要がある。落ちたらログをDLして次に活かす。
- 期間: 2026-08-17まで（最終提出 8/16）。その後2週間対戦を続けて勝者決定。
- 複数アカウント禁止。
- 公式エンジン/カードデータ/サンプルは再配布禁止 → リポジトリにコミットしない。

## ローカル評価ツール（フェーズB）

`tools/selfplay_eval.py`:
- 自己対戦をN回ループ
- 集計: 勝率 / 平均ターン数 / 先攻後攻別勝率 / 勝敗時の平均サイド差 / 引き分け率
- 改良版 vs 旧版の A/Bテストを提出前に実行
- 詳細な評価軸の定義は `docs/eval-axes.md` を参照
