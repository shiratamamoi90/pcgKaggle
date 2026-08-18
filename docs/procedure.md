# 手順 (procedure.md)

## フェーズA：valid submission を1回通す

1. **参加登録**
   - Kaggleでコンペに参加登録し、Rulesに同意。
   - Data Page からエンジン・サンプル・カードデータをダウンロード。
2. **環境構築（WSL2）**
   - Python仮想環境を作成。
   - `kaggle-environments` をインストール。cgライブラリ/エンジンを `engine/` に配置。
3. **サンプル理解**
   - 「[Beginner Guide] From Deck List to First Valid Sub」を Copy & Edit して中身を読む。
   - サンプルのルールベース `main.py` と `deck.csv`（例：メガルカリオex）を手元に用意。
4. **ローカル対戦の確認**
   ```python
   from kaggle_environments import make
   from agent import agent  # main.py 内の agent をimport

   with open("deck.csv") as f:
       deck = [int(line) for line in f.readlines() if line.strip()]

   env = make("cabt", configuration={"decks": [deck, deck]})
   env.run([agent, agent])

   with open("result.html", "w") as f:
       f.write(env.render(mode="html"))
   ```
   - `result.html` を開いて対戦が成立しているか観戦。
5. **バンドル作成**
   ```bash
   tar -czvf submission.tar.gz *
   tar -tzf submission.tar.gz   # main.py / deck.csv がトップレベルにあるか確認（ネスト禁止）
   ```
6. **提出**
   - My Submissions からアップロード、または
     `kaggle competitions submit pokemon-tcg-ai-battle -f submission.tar.gz -m "first valid sub"`
   - Validation Episode を通過するか確認。落ちたらログをDLして原因を特定。

## フェーズB：改良ループ

7. **評価スクリプト作成** (`tools/selfplay_eval.py`)
   - 自己対戦をN回（まず数百回）ループ。
   - 集計: 勝率 / 平均ターン数 / 先後別勝率 / 勝敗時の平均サイド差 / 引き分け率。
8. **1点だけ改良**
   - `agent` のロジックを小さく1箇所だけ変更（例：特定状況での退却判断）。
9. **A/Bテスト**
   - 改良版 vs 旧版を `selfplay_eval.py` で対戦させ、数字が改善したか確認。
   - 数百〜1000試合回さないとノイズに埋もれる点に注意。
10. **提出**
    - 数字が改善したものだけ提出。ラダーのEloの動きを観察。
11. **繰り返し**
    - 8〜10を反復。慣れたら相性マトリクスや盤面評価関数のスコア検証へ。

## Claude Code 運用メモ

- 変更 → A/Bテスト → 提出提案、の順を崩さない。
- 提出前に必ず `tar -tzf` でネスト確認。
- 「強くなった」は数字の裏付けがある場合のみ。
