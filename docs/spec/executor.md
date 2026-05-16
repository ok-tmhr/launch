# Executor Specification (Initial Draft)

## 概要

本ドキュメントは、Launch IR（launch-only）を実行するための
**最小 Executor** の設計を定義する。

Executor は IR を逐次評価し、必要に応じて外部プロセスの起動・環境変数の設定・引数の解決を行う。
本仕様は最小構成であり、将来的な拡張（IncludeLaunchDescription、Timer、イベント処理など）を前提にしている。

## 1. 設計方針

### 1.1 目的

Executor の目的は以下の通り：

- Launch IR を逐次的に評価し、定義されたアクションを実行する
- Condition / Substitution を適切なタイミングで評価する
- 外部プロセスの起動を安全に扱う
- 最小限の状態管理で動作する

### 1.2 原則

- **IR-first**：Executor は IR を直接評価する
- **Sequential**：最小構成では逐次実行のみを扱う
- **Stateless-first**：必要最小限の状態のみ保持する
- **Minimal**：複雑なイベント処理や並列実行は扱わない

## 2. Executor の責務

Executor が行うこと：

- LaunchDescription の actions を順に評価する
- DeclareLaunchArgument の登録
- SetEnvironmentVariable の適用
- Condition の評価
- Substitution の評価
- ExecuteProcess の実行（同期）

Executor が行わないこと：

- 並列実行
- イベント駆動処理
- Launch ファイルのネスト読み込み
- プロセス監視・再起動
- ログの高度な管理

## 3. 実行モデル

Executor は以下の順序で IR を評価する：

1. LaunchDescription の actions を先頭から順に処理する
2. 各アクションの condition を評価する
3. condition が false の場合はスキップ
4. アクション固有の処理を実行する

Group は以下のように扱う：

- Group 自体の condition を評価する
- true の場合、内部の actions を順に評価する

## 4. DeclareLaunchArgument の評価

### 4.1 概要

DeclareLaunchArgument は Executor 内部の「引数テーブル」に登録される。

### 4.2 処理内容

- name をキーとして登録
- default_value があれば初期値として設定
- description は Executor では使用しない（保持のみ）

## 5. SetEnvironmentVariable の評価

### 5.1 概要

環境変数を設定する。

### 5.2 処理内容

- Executor の「環境変数テーブル」に name/value を登録
- ExecuteProcess 実行時にプロセス環境へ反映する

## 6. Condition の評価

### 6.1 概要

Condition はアクション実行前に評価される。

### 6.2 評価ルール

- `and` / `or` / `not` は再帰的に評価
- `equals` は文字列比較
- `exists` は引数テーブルにキーが存在するかを判定

### 6.3 Substitution との関係

Condition の args 内に Substitution が含まれる場合、
Substitution を先に評価してから Condition を評価する。

## 7. Substitution の評価

### 7.1 概要

Substitution は実行時に動的に値を生成する。

### 7.2 種類

- `env`：環境変数テーブルから取得
- `eval`：簡易式の評価（最小構成では文字列置換レベル）
- `text`：そのまま返す

## 8. ExecuteProcess の評価

### 8.1 概要

外部プロセスを起動する。

### 8.2 処理内容

- cmd の Substitution を評価
- cwd があれば設定
- env は Executor の環境変数テーブルをマージ
- shell が true の場合はシェル経由で実行
- プロセスを同期的に実行し、終了コードを取得する

### 8.3 終了コードの扱い

最小構成では：

- 終了コードは Executor 内で保持するのみ
- エラー時の再試行や停止処理は行わない

## 9. 疑似コード（簡易）

```python
def run(ld: LaunchDescription):
    args = {}
    env = {}

    for action in ld.actions:
        eval_action(action, args, env)


def eval_action(action, args, env):
    if not eval_condition(action.condition, args, env):
        return

    if action.type == "declare_launch_argument":
        args[action.name] = action.default_value

    elif action.type == "set_env":
        env[action.name] = action.value

    elif action.type == "group":
        for a in action.actions:
            eval_action(a, args, env)

    elif action.type == "execute_process":
        cmd = eval_substitutions(action.cmd, args, env)
        run_process(cmd, cwd=action.cwd, env=env, shell=action.shell)
```

## 10. ステータス

このドキュメントは **初期ドラフト** であり、
Issue / PR による議論を通じて更新される。
