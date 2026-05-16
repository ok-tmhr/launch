# Launch IR Specification (Initial Draft)

## 概要

本ドキュメントは、Launch システムをゼロベースで再設計するための
**Launch IR（Intermediate Representation / 中間表現）** の初期仕様を定義する。

Launch IR は、YAML / XML / Python API などのフロントエンドがコンパイルする
**抽象構文木（AST）** であり、Executor が解釈する唯一のデータ構造となる。

本仕様は最小構成であり、将来的な拡張（イベント、タイマー、条件分岐など）を前提にしている。

## 1. 設計方針

### 1.1 目的

Launch IR の目的は以下の通り：

- Launch 構成を **純粋データ構造** として表現する
- フロントエンド（YAML / XML / Python）を統一的に扱う
- Executor が解釈しやすい AST を提供する
- スキーマファースト設計に適した構造を持つ
- 拡張可能で後方互換性を維持しやすい

### 1.2 原則

- **Immutable**：IR は不変データとして扱う
- **Declarative**：ロジックは含めず、構成のみを表現する
- **Schema-friendly**：JSON Schema / XML Schema に容易に変換可能
- **Frontend-agnostic**：どの記述形式でも同じ IR に変換される
- **Minimal-first**：最小構成から始め、必要に応じて拡張する

## 2. IR の全体構造

Launch IR は以下のトップレベル構造を持つ：

```sh
LaunchDescription
└── actions: Action[]
```

Action は以下のユニオン型で構成される：

```sh
Action = ExecuteProcess | DeclareLaunchArgument | Group | SetEnvironmentVariable
```

（IncludeLaunchDescription / Timer などは将来追加）

## 3. LaunchDescription

### 3.1 概要

LaunchDescription は Launch ファイル全体を表すトップレベルノード。

### 3.2 構造

```yaml
LaunchDescription:
  actions: Action[]
```

- `actions`
  - 実行されるアクションのリスト
  - 順序は保持される

## 4. ExecuteProcess

### 4.1 概要

ExecuteProcess は外部プロセスを起動するための最小単位。

### 4.2 構造

```yaml
ExecuteProcess:
  type: execute_process
  cmd: string[]
  cwd: string | null
  env: object | null
  shell: boolean
  condition: Condition | null
```

### 4.3 説明

- `cmd`
  - 実行するコマンドと引数
- `cwd`
  - 実行ディレクトリ
- `env`
  - 環境変数のマップ
- `shell`
  - シェル経由で実行するか
- `condition`
  - 条件付き実行

## 5. DeclareLaunchArgument

### 5.1 概要

Launch ファイル内で使用する引数を宣言する。

### 5.2 構造

```yaml
DeclareLaunchArgument:
  type: declare_launch_argument
  name: string
  default_value: string | null
  description: string | null
```

## 6. Group

### 6.1 概要

Group は複数のアクションをまとめ、条件を適用する。

### 6.2 構造

```yaml
Group:
  type: group
  actions: Action[]
  condition: Condition | null
```

### 6.3 説明

- Group 内の actions は順序通りに評価される
- 条件が false の場合、内部の actions はすべて無視される

## 7. SetEnvironmentVariable

### 7.1 概要

環境変数を設定する。

### 7.2 構造

```yaml
SetEnvironmentVariable:
  type: set_env
  name: string
  value: string
```

## 8. Condition

### 8.1 概要

アクションの実行可否を制御するブール式。

### 8.2 構造（最小構成）

```yaml
Condition:
  type: and | or | not | equals | exists
  args: Condition[] | string[]
```

## 9. Substitution

### 9.1 概要

実行時に評価される動的値。

### 9.2 構造（最小構成）

```yaml
Substitution:
  type: env | eval | text
  value: string
```

## 10. 将来拡張（予定）

以下は将来追加される可能性が高い IR ノード：

- IncludeLaunchDescription
- Timer
- EventHandler
- OnExit / OnStart
- AsyncGroup
- ParallelGroup

これらは別ドキュメントで定義する。

## 11. 付録：最小 IR の例

```yaml
LaunchDescription:
  actions:
    - type: declare_launch_argument
      name: mode
      default_value: debug

    - type: set_env
      name: LOG_LEVEL
      value: info

    - type: execute_process
      cmd: ["echo", "hello"]
      shell: false

    - type: group
      actions:
        - type: execute_process
          cmd: ["echo", "inside group"]
      condition:
        type: equals
        args: ["$(mode)", "debug"]
```

## 12. ステータス

このドキュメントは **初期ドラフト** であり、
Issue / PR による議論を通じて更新される。
