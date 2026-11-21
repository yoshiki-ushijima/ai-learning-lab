# 利用者（users）

[DB設計の目次](../README.md)、[DB区分一覧](../DB区分一覧.md)。

## 業務上の意味と関係

システム利用者を表す。
初回サンプルでは、社員、上長、管理者のいずれか1ロールのみを持つ。

- `employee_id` は一意
- `login_id` は一意
- `user_role` は定義済みロールのみ許可
- `EMPLOYEE` ロールのユーザーは `group_id`、`break_type_id`、`work_time_type_id` を必須とする

## 列定義

凡例: NULL可が「不可」はNOT NULL、キーが空欄の列は非キー列。列名・型・NULL可・キー・初期値は旧DB定義から維持する。

| 論理名             | 物理名                | 型             | NULL可 | キー・初期値        | 説明                                         |
| ------------------ | --------------------- | -------------- | ------ | ------------------- | -------------------------------------------- |
| ユーザーID         | `user_id`             | `VARCHAR(20)`  | 不可   | PK                  | システム内部の利用者識別子                   |
| 社員ID             | `employee_id`         | `VARCHAR(20)`  | 不可   | UNIQUE              | 業務上の社員識別子                           |
| ログインID         | `login_id`            | `VARCHAR(80)`  | 不可   | UNIQUE              | ログインに使用する識別子                     |
| パスワードハッシュ | `password_hash`       | `VARCHAR(100)` | 不可   |                     | BCrypt等でハッシュ化した値。平文は保持しない |
| 利用者名           | `user_name`           | `VARCHAR(120)` | 不可   |                     | 画面表示、日報登録時のスナップショット元     |
| ロール             | `user_role`           | `VARCHAR(20)`  | 不可   | CHECK               | `EMPLOYEE`、`MANAGER`、`ADMIN`               |
| グループID         | `group_id`            | `VARCHAR(20)`  | 可     |                     | 社員の所属グループID                         |
| グループ名         | `group_name`          | `VARCHAR(120)` | 可     |                     | 所属グループ名のスナップショット             |
| 休憩区分ID         | `break_type_id`       | `VARCHAR(20)`  | 可     |                     | 社員に設定された休憩区分                     |
| 休憩区分名         | `break_type_name`     | `VARCHAR(120)` | 可     |                     | 休憩区分名のスナップショット                 |
| 勤務区分ID         | `work_time_type_id`   | `VARCHAR(20)`  | 可     |                     | 社員に設定された勤務区分                     |
| 勤務区分名         | `work_time_type_name` | `VARCHAR(120)` | 可     |                     | 勤務区分名のスナップショット                 |
| 作成日時           | `created_at`          | `TIMESTAMPTZ`  | 不可   | `CURRENT_TIMESTAMP` | 利用者作成日時                               |
| 更新日時           | `updated_at`          | `TIMESTAMPTZ`  | 不可   | `CURRENT_TIMESTAMP` | 利用者更新日時                               |

主な制約:

- `user_id` を主キーとする。
- `employee_id` と `login_id` は一意とする。
- `user_role` は `EMPLOYEE`、`MANAGER`、`ADMIN` のいずれかとする。
- ロールごとのグループ・休憩区分・勤務区分の必須性はアプリケーションの入力・初期データ処理で検証する。

## 制約・索引

### 主キー・一意制約

| テーブル | 制約                   | 対象列        |
| -------- | ---------------------- | ------------- |
| `users`  | `pk_users`             | `user_id`     |
| `users`  | `uq_users_employee_id` | `employee_id` |
| `users`  | `uq_users_login_id`    | `login_id`    |

### 外部キー

旧DB定義の制約・索引一覧に記載なし。

### CHECK制約

| 制約            | 対象              | 条件                                      |
| --------------- | ----------------- | ----------------------------------------- |
| `ck_users_role` | `users.user_role` | `EMPLOYEE`、`MANAGER`、`ADMIN` のいずれか |

### 明示索引

旧DB定義の制約・索引一覧に記載なし。
