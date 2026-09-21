# テーマ候補と検討状況

更新日: 2026-09-21

候補の概要と比較の経緯を残す。以下の機能・設計は探索案であり、実装要件や既存製品に対する新規性を確認したものではない。

## 当初の六つの候補

| 候補 | コンセプト・設計上の関心 | 現在の検討状況 |
| --- | --- | --- |
| Small Cloud / Container Platform | イメージ・台数・資源量などの宣言からコンテナを配置・維持する。Kubernetesの再実装自体を目的にせず、小〜中規模の環境に必要な機能を考える | 概要まで検討。関心はあるが主題には選んでいない |
| Generic Control Plane / Reconciliation Platform | desired stateとactual stateを比較し、コンテナ以外も制御する。resource / controller model、依存関係、失敗状態、結果整合性、provider abstractionを考える | 関心が強い。目標へ収束する実行基盤と関連付けて検討 |
| Workflow / Execution Platform | 独自の実行モデルを考える。durable execution、retry、timeout、cancellation、並行実行、永続化、scheduler、人の介入などを扱う | 関心が強い。Control Planeとの接続を検討 |
| Deployment System | 実行先は既存基盤を使い、稼働中システムを安全に更新することへ集中する | Argo CDやArgo Rolloutsなどとの重なりを意識し、深掘りは保留。不採用ではない |
| Distributed Storage / Object Store | 小さなオブジェクトストレージを育てる。データ配置、複製、障害復旧を自分で設計する | 概要まで検討。複数プロセスへの保存・障害後の読み出し・複製修復が試作例 |
| Software System Simulator | 構成を実行可能なモデルとして扱い、負荷・障害・制御方式を変えて挙動を比較する | 用途、限界、構成探索、学習への利用まで検討。候補として残す |

### 各案で探索できる範囲

- コンテナ基盤: placement、health check、restart、rolling deployment、service discovery、config / secret、logging、metrics、autoscaling、node failure recovery。
- Control Plane: container、VM、database、repository、DNS、cloud resource、local processなどを対象にできるか。
- 実行基盤: worker discovery、dependency graph、replay、idempotency、distributed locking。用途はCI/CDに限らず、deployment、automation、data processing、operational workflow、AI / agent executionも考えられる。
- デプロイ: rolling update、canary、blue/green、rollback、health evaluation、deployment lock、progressive delivery、environment promotion、audit log。
- ストレージ: replication、sharding、consistent hashing、metadata、checksum、rebalancing、garbage collection、versioning、erasure coding。
- シミュレータ: latency、throughput、concurrency、queue size、instance count、failure probabilityをモデル化し、retry storm、thundering herd、autoscaling、circuit breaker、cache、sharding、node failureなどを観察する。

これらをすべて実装する想定ではない。興味を持つ問いを選ぶための材料とする。

## 議論から具体化した候補

### 目標へ収束する実行基盤 — 現在の中心

人が構成・目標・制約を指定し、観測状態から必要な実行計画を生成・更新する。ユーザーがパイプラインを事前に書かなくてもよい形を考える。

Generic Control PlaneとWorkflow / Execution Platformを組み合わせる具体的な方向性であり、完全に独立した三つの製品を作る想定ではない。

小さく観測・計画・実行を一周させ、その部分を個別に検証・発展させる方針に合意した。最初の題材はAPI＋DBの一時環境が案として挙がっている。

- [構想とPoC](../explorations/goal-driven-execution/design.md)
- [設計上の問い](../explorations/goal-driven-execution/questions.md)

### PRごとの一時環境基盤

PR作成時にAPI・DBなどを用意し、更新に追従し、PR終了時に片付ける。開発ツールやエディタよりも、開発中のシステムを動かす環境が対象。

独立した候補として残すとともに、目標へ収束する実行基盤の応用先にもなる。差を考える軸は、構築の手軽さ、PRと手動操作の共通化、データの保持・初期化・複製など。Composeの再利用も案の一つであり、採用済みではない。

既存の選択肢として議論中に確認した公式資料:

- [Argo CD Pull Request Generator](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Pull-Request/)
- [GitLab Review Apps](https://docs.gitlab.com/ci/review_apps/)
- [Okteto Preview Environments](https://www.okteto.com/docs/previews)
- [Render Preview Environments](https://render.com/docs/preview-environments)

用途自体に新規性があるとは扱わない。既存と重なってもシステム構築の知識になる一方、自分なりの機能・着眼点・簡潔さを持ちたい。

### CI/CDの実行モデルに関する別案

完成したCI/CD製品より、アイデアのPoCとして検討した。

| 案 | コアコンセプト |
| --- | --- |
| 分岐できる実行 | 保存した入力や実行状態から、条件を変えた別の試行を作る。何を再現できるかが設計課題 |
| 環境を含む実行セッション | 処理・環境・人の操作を一つのセッションで管理する。失敗時に環境を保持し、調査・再実行へつなぐ |
| 根拠を積み上げる検証 | 成果物と検証結果の有効範囲を管理し、変更によって無効になった根拠を取り直す |
| 制約から処理を合成する実行 | 操作の入出力・条件から計画を作る。目標へ収束する実行基盤につながる |

CUEへの関心もあるが、言語や実装方式は未決定。これらを現在のPoCにすべて含めるわけではない。

### シミュレーションを使った構成探索

要求と制約から候補を生成し、仮想実験で評価して設計案を絞り込む。台数などのパラメータ調整、既知の構成の選択、部品からの構造生成という発展が考えられる。

性能だけでなく機能上の意味を維持する制約が必要。例えば同期処理を非同期化すれば、「応答時には処理が完了している」という要求を変えてしまう可能性がある。

シミュレーションで分かるのはモデルの仮定のもとでの成立条件・弱点・トレードオフであり、実システムの性能を無条件に保証するものではない。実コンテナ等を動かすアーキテクチャベンチマークも別方向として挙がったが、仮想モデルの案も関心がある。

参考として議論中に確認した資料: [Simulation optimization: A review of algorithms and applications](https://arxiv.org/abs/1706.08591)

### アーキテクチャレビュー支援

設計判断を、根拠と検証可能な問いに分解する。要求・制約・構成・採用理由から、前提、トレードオフ、代替案、未確認事項と検証方法を整理する。

設計判断を補強し、体系的な学習に役立てたいという動機がある。シミュレータはレビュー全体ではなく、性能や障害などの個別の問いを検証する手段として組み込める。

AIによる質問・代替案の提案も考えられるが、事実・推測・検証結果を区別して管理する。作る楽しさの重心は、実行制御より設計知識の表現と対話・情報管理へ移る。候補として残す。

## 選定の前提

既存製品との差別化だけを目的にせず、作る面白さ、得たい知識、短期間で見える成果を比較する。コンテナ技術は関心のある領域だが、将来の仕事への準備に限定しない。

今後KaaS（Kubernetesクラスタの払い出し）に関わる可能性も議論した。アプリの実行管理とは層が異なり、個人開発で先に取り組むかは未決定。この可能性を理由に候補を自動的に採用・除外しない。
