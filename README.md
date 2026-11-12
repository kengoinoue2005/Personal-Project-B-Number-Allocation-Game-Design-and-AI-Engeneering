# Number-Allocation-Game-Design-and-AI-Engeneering
A serverless numerical strategy board game (4x4–8x8) in Pure JS. Place tiles to buff adjacent allies via addition and shrink enemies by division. Includes local 2P mode and an adaptive 2/3-step Minimax AI.　
周囲の自陣を加算し敵陣を割り算で削る4×4〜8×8数値戦略ゲーム。Pure JSでブラウザ完結し、盤面に応じ2〜3手先読み＋枝刈り最適化されたAI対戦（先後選択可）と対人戦を搭載。

# 【English Version】

A fully client-side, serverless numerical strategy board game and AI engine built with Pure JavaScript, executing natively within standard web browsers in milliseconds to seconds.

## 1. Game Mechanics & Architecture

* **Core Rules**: Played on 4x4, 6x6, or 8x8 grids, two players alternate placing numbered tiles (1~n^2/2). Placing a tile increments adjacent friendly tiles via addition, while reducing adjacent opponent tiles via exact division (if divisible). Players generally cannot reuse the exact number played by the opponent on the immediately preceding turn (excluding `1` and the final move). The player with the highest total board score wins.
* **Serverless Execution**: Self-contained in a single HTML/JS file with zero backend dependencies, deployable directly via static hosting such as GitHub Pages. Supports both vs AI (with selectable turn order) and vs Human local modes, alongside configurable turn timers (Unlimited, 10s, 30s).
* **Centralized Configuration**: All core parameters—look-ahead depth, beam pruning ratios, card evaluation vectors, and timer options—are consolidated into a single `GAME_SETTINGS` object for rapid balancing and maintainability.

## 2. Architectural Pivot

Initially, a 34-channel Deep Reinforcement Learning (DRL) model was built in PyTorch and trained over 120,000 self-play episodes. However, it failed to seek moves that can be considered as appropriate.

To eliminate black-box failure modes and satisfy strict browser runtime latency constraints, the architecture was refactored into an interpretable framework that explicitly decouples Game-Theoretic Look-Ahead Search from a Card Value Parameter Vector.

## 3. Algorithmic Highlights & Optimizations

### Adaptive Minimax Search & Beam Pruning
To balance computational complexity with strategic depth across varying board dimensions, search parameters scale dynamically:
* **4x4 & 6x6 Grids**: 3-step Minimax look-ahead search with top-15% candidate beam pruning.
* **8x8 Grids**: 2-step Minimax look-ahead search with top-30% candidate beam pruning.
* **Full Spatial Coverage Constraint**: To prevent tactical blind spots caused by aggressive pruning, root-level move filtering guarantees that at least one candidate move is retained for every valid unoccupied cell on the board.

### Domain-Informed Parameter Design & Overfitting Analysis
Card cost vectors were analyzed through self-play simulations in Google Colab using learning rate decay and $\epsilon$-greedy exploration ($s=0.1$). Experiments revealed high sensitivity to initial conditions and demonstrated that pure self-play updates risked overfitting to AI-specific playstyles at the expense of generalization against human opponents. Seeding the evaluation vector with structured domain knowledge derived from game-theoretic analysis yielded an AI capable of consistently defeating experienced human players, including the developer.

### Geometric Tie-Breaking
When multiple candidate moves share identical heuristic evaluations, the engine prioritizes coordinates closest to the geometric center of the grid (minimum squared Euclidean distance), encouraging natural spatial dominance.

# 【日本語版】

外部サーバーに依存せず、Webブラウザのクライアントサイド環境のみで高速動作する対戦型数値戦略ボードゲームおよび思考エンジンです。

## 1. ゲームルールとシステム構成

* **基本ルール**: 4×4、6×6、8×8の盤面上で、手持ちの数字カード（1~n^2/2）を交互に配置します。配置したマスの周囲8マスにある自分の駒は配置数字分だけ加算され、相手の駒は配置数字で割り切れる場合に限りその商へと削減されます。直前に相手が使用した数字は原則使用不可（1および最終手を除く）という制約のもと、最終的な盤面総得点を競います。
* **サーバーレス構成**: 単一HTML/JavaScriptファイルで完結しており、GitHub Pages等でそのまま静的ホスティングが可能です。対戦モードは「vs AI（先手・後手選択可）」と「vs Human（ローカル対戦）」を備え、ターン制限時間（無制限・10秒・30秒）も設定できます。
* **保守性の高い設計**: 盤面サイズごとの先読み手数・枝刈り割合・カード評価パラメータ・制限時間オプションをコード冒頭の `GAME_SETTINGS` オブジェクトに集約し、ゲームバランスを容易に調整できる構造としています。

## 2. 技術的転換（Architectural Pivot）

開発初期には、PyTorchを用いた34チャンネル入力の深層強化学習モデルを構築し、12万回に及ぶ自己対局学習を実施しました。しかし、人間目線で妥当と判断できる手を打つには至らないという課題に直面しました。

ブラックボックスな深層学習モデルにおける局所解の修正難度、およびクライアントサイド実行時の厳格なレスポンス速度制約を考慮し、アーキテクチャを抜本的に再設計しました。深層学習単体への固執を捨て、ゲーム理論に基づく先読み探索アルゴリズムと、手持ち数字の潜在価値を示すパラメータベクトルに明示的に分離する解釈性の高いハイブリッド設計へと転換しました。

## 3. アルゴリズム設計と最適化の工夫

### 盤面規模に応じた適応的ミニマックス探索とビーム枝刈り
探索空間の爆発を防ぎつつ戦略的深度を確保するため、盤面サイズごとに探索深さと枝刈り率を最適化しています。
* **4×4・6×6モード**: 3手先読みミニマックス探索 ＋ 評価値上位 15% のビーム枝刈り
* **8×8モード**: 2手先読みミニマックス探索 ＋ 評価値上位 30% のビーム枝刈り
* **全領域網羅補完ロジック**: 単純な上位抽出による視野狭窄を防ぐため、初手候補においては盤面上のすべての配置可能マスから最低1手以上を候補に残す補完処理を実装しています。

### ドメイン知識に基づくパラメータ設計と過学習の検証
各数字カードの潜在価値の推定にあたり、Google Colab上でAI同士の自己対局シミュレーション（学習率減衰および確率 $s=0.1$ の探索手導入）を実施しました。検証の結果、パラメータ学習は極めて強い初期値依存性を持ち、自己対局のみでの過度な更新は人間プレイヤーに対する汎用性を損なうリスクがあることを特定しました。最終的に、4×4・6×6モードにおいては対局分析から得られたドメイン知識に基づく評価パラメータを採用することで、ルールと定石を熟知した開発者自身をも打ち負かす水準の思考強度を達成しています。

### 幾何学的タイブレーク
評価値が同等の着手候補が複数存在する場合、盤面中心からのユークリッド距離が最小となる座標を優先選択するタイブレークを組み込み、序盤からの自然な盤面制圧を実現しています。
