# 上長グループ権限（manager_group_permissions）

[DB設計の目次](../README.md)、[DB区分一覧](../DB区分一覧.md)。

## 業務上の意味と関係

上長が閲覧できるグループを表す。
上長の日報参照の権限制御に使用する。

- `manager_user_id` は `MANAGER` ロールのユーザーを参照する
- `manager_user_id` と `group_id` の組み合わせは一意
- 主キーは `manager_user_id + group_id` の複合キー
- 上長は、閲覧対象グループに所属する社員の日報のみ参照できる
- 同一グループを複数の上長に紐づけられるため、同一社員の日報を複数の上長が参照できる

## 列定義

凡例: NULL可が「不可」はNOT NULL、キーが空欄の列は非キー列。列名・型・NULL可・キー・初期値は旧DB定義から維持する。

| 論理名             | 物理名            | 型            | NULL可 | キー・初期値 | 説明                                |
| ------------------ | ----------------- | ------------- | ------ | ------------ | ----------------------------------- |
| 上長ユーザーID     | `manager_user_id` | `VARCHAR(20)` | 不可   | 複合PK、FK   | `users.user_id`。日報を閲覧する上長 |
| 閲覧対象グループID | `group_id`        | `VARCHAR(20)` | 不可   | 複合PK       | 上長が閲覧できるグループID          |

主な制約:

- `manager_user_id` と `group_id` の組み合わせを主キーとする。
- `manager_user_id` は `users.user_id` を参照する。
- `manager_user_id` が `MANAGER` ロールであること、および `group_id` が日報の所属グループと一致することはアプリケーションで検証する。

## 制約・索引

### 主キー・一意制約

| テーブル                    | 制約                           | 対象列                        |
| --------------------------- | ------------------------------ | ----------------------------- |
| `manager_group_permissions` | `pk_manager_group_permissions` | `manager_user_id`, `group_id` |

### 外部キー

| 制約                                | 子テーブル・列                              | 親テーブル・列  |
| ----------------------------------- | ------------------------------------------- | --------------- |
| `fk_manager_group_permissions_user` | `manager_group_permissions.manager_user_id` | `users.user_id` |

### CHECK制約

旧DB定義の制約・索引一覧に記載なし。

### 明示索引

旧DB定義の制約・索引一覧に記載なし。
