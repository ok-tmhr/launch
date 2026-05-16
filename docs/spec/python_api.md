# Python API の全体構造（launch-only 版）

## 1. API の全体構造

Launch IR（launch-only）は以下のアクションを持つ：

```sh
LaunchDescription
├── ExecuteProcess
├── DeclareLaunchArgument
├── Group
├── SetEnvironmentVariable
├── IncludeLaunchDescription（後で追加）
└── その他の launch-only アクション（後で追加）
```

## 2. LaunchDescription

```python
class LaunchDescription:
    def __init__(self, actions: list["Action"]):
        self.actions = actions
```

## 3. ExecuteProcess

launch の最も基本的なアクション。

```python
class ExecuteProcess:
    type = "execute_process"

    def __init__(
        self,
        cmd: list[str],
        cwd: str | None = None,
        env: dict[str, str] | None = None,
        shell: bool = False,
        condition: "Condition" | None = None,
    ):
        self.cmd = cmd
        self.cwd = cwd
        self.env = env or {}
        self.shell = shell
        self.condition = condition
```

## 4. DeclareLaunchArgument

```python
class DeclareLaunchArgument:
    type = "declare_launch_argument"

    def __init__(
        self,
        name: str,
        default_value: str | None = None,
        description: str | None = None,
    ):
        self.name = name
        self.default_value = default_value
        self.description = description
```

## 5. Group

```python
class Group:
    type = "group"

    def __init__(
        self,
        actions: list["Action"],
        condition: "Condition" | None = None,
    ):
        self.actions = actions
        self.condition = condition
```

## 6. SetEnvironmentVariable

```python
class SetEnvironmentVariable:
    type = "set_env"

    def __init__(self, name: str, value: str):
        self.name = name
        self.value = value
```

## 7. Condition

```python
class Condition:
    def __init__(self, type: str, args: list):
        self.type = type
        self.args = args
```

## 8. Substitution

```python
class Substitution:
    def __init__(self, type: str, value: str):
        self.type = type
        self.value = value
```

## 9. 使用例（launch-only）

```python
ld = LaunchDescription([
    DeclareLaunchArgument("robot_name", default_value="r2d2"),

    SetEnvironmentVariable("LOG_LEVEL", "info"),

    ExecuteProcess(
        cmd=["/usr/bin/python3", "script.py"],
        cwd="/tmp",
        env={"MODE": "test"},
    ),

    Group(
        actions=[
            ExecuteProcess(cmd=["echo", "inside group"])
        ]
    )
])
```

## 11. ステータス

このドキュメントは **修正版初期ドラフト** であり、
Issue / PR による議論を通じて更新される。
