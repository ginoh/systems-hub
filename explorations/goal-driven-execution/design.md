# 目標へ収束する実行基盤

更新日: 2026-10-03

## 概要

人は構成・目標・制約を定義し、システムが現状から必要な実行計画を導く。観測・計画・実行を繰り返し、状態や目標の変化に応じて計画を更新する実行基盤を探索する。

[テーマ候補の検討](../../ideas/README.md)から、Generic Control Planeの観測・制御と、Workflow / Execution Platformの実行機構をつなぐ方向を選んだ。仕組みを自分で設計・実装することを重視し、まず小さなPoCでコンセプトを確かめる。

初回のPoCではAPI＋DBの一時環境を対象に、空の状態からの構築、初期化済みDBの再利用、実行途中の目標変更を検証した。実装の土台にはGo、ローカルDocker、CLI＋目標ファイルを採用した。

## 詳細設計の移行先

本体リポジトリは [goal-executor](https://github.com/ginoh/goal-executor)。詳細設計・実装・検証結果は本体側で管理し、このリポジトリにはテーマの概要と参照先を残す。

- [本体README — 実行方法と設計資料への入口](https://github.com/ginoh/goal-executor/blob/main/README.md)
- [設計上の問い](https://github.com/ginoh/goal-executor/blob/main/docs/questions.md)

PRごとの一時環境基盤は本構想の応用先にもなる。システムシミュレータ、構成探索、アーキテクチャレビュー支援などの他候補も引き続き残す。
