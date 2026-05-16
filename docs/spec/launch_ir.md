# Launch IR Specification (Initial Draft)

## 概要

本ドキュメントは、ROS 2 Launch システムをゼロベースで再設計するための
**Launch IR（Intermediate Representation / 中間表現）** の初期仕様を定義する。

Launch IR は、YAML / XML / Python API などのフロントエンドがコンパイルする
**抽象構文木（AST）** であり、Executor が解釈する唯一のデータ構造となる。

本仕様は最小構成であり、将来的な拡張（イベント、タイマー、ライフサイクル等）を前提にしている。

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
Action = Node | Group
```

（Timer / IncludeLaunch / LifecycleNode などは将来追加）

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
  - Node / Group の順序は保持される

## 4. Node

### 4.1 概要

Node は ROS ノード（または一般プロセス）を起動するための最小単位。

### 4.2 構造

```yaml
Node:
  type: node
  package: string
  executable: string
  name: string | null
  namespace: string | null
  parameters: Parameter[] | null
  remappings: Remap[] | null
  env: EnvVar[] | null
  condition: Condition | null
```

### 4.3 説明

- `package` / `executable`
  - 起動対象のプロセス
- `name` / `namespace`
  - ROS 名の設定
- `parameters`
  - 型付きパラメータ
- `remappings`
  - ROS 名のリマッピング
- `env`
  - 環境変数
- `condition`
  - 条件付き実行

## 5. Group

### 5.1 概要

Group はスコープ（namespace / parameters / env / condition）を提供する。

### 5.2 構造

```yaml
Group:
  type: group
  namespace: string | null
  parameters: Parameter[] | null
  env: EnvVar[] | null
  condition: Condition | null
  actions: Action[]
```

### 5.3 説明

- Group 内の Node / Group に対してスコープが適用される
- 条件が false の場合、内部の actions はすべて無視される

## 6. Parameter

### 6.1 概要

型付きパラメータを表す。

### 6.2 構造

```yaml
Parameter:
  name: string
  value: string | number | boolean | list | dict | Substitution
```

## 7. Remap

### 7.1 概要

ROS 名のリマッピング。

### 7.2 構造

```yaml
Remap:
  from: string
  to: string
```

## 8. EnvVar

### 8.1 概要

環境変数の設定。

### 8.2 構造

```yaml
EnvVar:
  name: string
  value: string | Substitution
```

## 9. Condition

### 9.1 概要

アクションの実行可否を制御するブール式。

### 9.2 構造（最小構成）

```yaml
Condition:
  type: and | or | not | equals | exists
  args: Condition[] | string[]
```

## 10. Substitution

### 10.1 概要

実行時に評価される動的値。

### 10.2 構造（最小構成）

```yaml
Substitution:
  type: find_package | env | eval | text
  value: string
```

## 11. 将来拡張（予定）

以下は将来追加される可能性が高い IR ノード：

- Timer
- IncludeLaunch
- LifecycleNode
- EventHandler
- OnExit / OnStart
- AsyncGroup
- ParallelGroup

これらは別ドキュメントで定義する。

## 12. 付録：最小 IR の例

```yaml
LaunchDescription:
  actions:
    - type: group
      namespace: robot1
      actions:
        - type: node
          package: demo_nodes_cpp
          executable: talker
          name: talker
          parameters:
            - name: rate
              value: 10
```

## 13. ステータス

このドキュメントは **初期ドラフト** であり、
Issue / PR による議論を通じて更新される。

必要であれば、次に **`docs/spec/python_api.md` の初期ドラフト** も作成できます。
