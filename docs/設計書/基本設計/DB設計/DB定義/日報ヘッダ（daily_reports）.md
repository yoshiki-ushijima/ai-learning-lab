# 日報ヘッダ（daily_reports）

[DB設計の目次](../README.md)、[DB区分一覧](../DB区分一覧.md)。

## 業務上の意味と関係

日報ヘッダを表す。
日付、休日区分、実勤務時間、勤務時間内訳、承認状態、承認・差戻しの監査情報を保持する。

- `employee_user_id` と `report_date` の組み合わせは一意
- `employee_user_id` は `EMPLOYEE` ロールのユーザーを参照する
- `group_id` は日報登録者の登録時点の所属グループを保持する
- `approval_status` は定義済み承認状態のみ許可
- `holiday_type` は有効な休日区分マスタを参照する
- `work_minutes` は0以上
- `break_minutes` は0以上
- 勤務時刻がある日報では、`end_minutes` は `start_time_minutes` より後にする
- 勤務時刻がある日報では、休憩区分から算出した `break_minutes` は勤務開始時刻から勤務終了時刻までの時間未満にする
- 勤務時刻がある日報では、`work_time_type_id` を必須とする
- `regular_work_minutes`、`overtime_work_minutes`、`night_work_minutes` は重複しない内訳として算出し、合計が `work_minutes` と一致する必要がある
- 有給休暇の日報では、`start_time_minutes`、`end_minutes`、`break_type_id`、`work_time_type_id`、`break_minutes`、`work_minutes`、`regular_work_minutes`、`overtime_work_minutes`、`night_work_minutes` を保持しない
- 作業なしの休日では、`start_time_minutes`、`end_minutes`、`break_type_id`、`work_time_type_id`、`break_minutes`、`work_minutes`、`regular_work_minutes`、`overtime_work_minutes`、`night_work_minutes` を保持しない

## 列定義

凡例: NULL可が「不可」はNOT NULL、キーが空欄の列は非キー列。列名・型・NULL可・キー・初期値は旧DB定義から維持する。

| 論理名             | 物理名                  | 型              | NULL可 | キー・初期値   | 説明                                       |
| ------------------ | ----------------------- | --------------- | ------ | -------------- | ------------------------------------------ |
| 日報ID             | `report_id`             | `VARCHAR(40)`   | 不可   | PK             | 日報識別子                                 |
| 社員ユーザーID     | `employee_user_id`      | `VARCHAR(20)`   | 不可   | FK             | `users.user_id`。日報の所有者              |
| 社員ID             | `employee_id`           | `VARCHAR(20)`   | 不可   |                | 登録時点の社員IDスナップショット           |
| 社員名             | `employee_name`         | `VARCHAR(120)`  | 不可   |                | 登録時点の社員名スナップショット           |
| グループID         | `group_id`              | `VARCHAR(20)`   | 不可   |                | 登録時点の所属グループID                   |
| グループ名         | `group_name`            | `VARCHAR(120)`  | 不可   |                | 登録時点の所属グループ名                   |
| 日報日             | `report_date`           | `DATE`          | 不可   | UNIQUEの構成列 | 社員ごとの日報対象日                       |
| 休日区分           | `holiday_type`          | `VARCHAR(20)`   | 不可   | FK             | `holiday_types.holiday_type`               |
| 休憩区分ID         | `break_type_id`         | `VARCHAR(20)`   | 可     | FK             | 登録時に使用した休憩区分                   |
| 休憩区分名         | `break_type_name`       | `VARCHAR(120)`  | 可     |                | 登録時点の休憩区分名                       |
| 勤務区分ID         | `work_time_type_id`     | `VARCHAR(20)`   | 可     | FK             | 登録時に使用した勤務区分                   |
| 勤務区分名         | `work_time_type_name`   | `VARCHAR(120)`  | 可     |                | 登録時点の勤務区分名                       |
| 勤務開始時刻（分） | `start_time_minutes`    | `INTEGER`       | 可     |                | 0:00からの経過分                           |
| 勤務終了時刻（分） | `end_time_minutes`      | `INTEGER`       | 可     |                | 0:00からの経過分                           |
| 休憩時間分         | `break_minutes`         | `INTEGER`       | 可     |                | 休憩区分から算出した休憩時間               |
| 実勤務時間分       | `work_minutes`          | `INTEGER`       | 可     |                | 勤務時間から休憩時間を差し引いた時間       |
| 通常勤務時間分     | `regular_work_minutes`  | `INTEGER`       | 可     |                | 通常勤務帯に該当する時間                   |
| 残業時間分         | `overtime_work_minutes` | `INTEGER`       | 可     |                | 通常勤務帯外の時間                         |
| 深夜勤務時間分     | `night_work_minutes`    | `INTEGER`       | 可     |                | 深夜勤務帯に該当する時間                   |
| 備考               | `remarks`               | `VARCHAR(1000)` | 可     |                | 日報の備考                                 |
| 承認状態           | `approval_status`       | `VARCHAR(20)`   | 不可   | CHECK          | `DRAFT`、`PENDING`、`REJECTED`、`APPROVED` |
| 提出日時           | `submitted_at`          | `TIMESTAMPTZ`   | 可     |                | 既存データの提出日時（参照のみ）           |
| 承認者ユーザーID   | `approver_user_id`      | `VARCHAR(20)`   | 可     |                | 承認者のユーザーID                         |
| 承認者名           | `approver_name`         | `VARCHAR(120)`  | 可     |                | 既存データの承認者名（参照のみ）           |
| 承認日時           | `approved_at`           | `TIMESTAMPTZ`   | 可     |                | 承認日時                                   |
| 差戻し者ユーザーID | `rejector_user_id`      | `VARCHAR(20)`   | 可     |                | 差戻し者のユーザーID                       |
| 差戻し者名         | `rejector_name`         | `VARCHAR(120)`  | 可     |                | 既存データの差戻し者名（参照のみ）         |
| 差戻し日時         | `rejected_at`           | `TIMESTAMPTZ`   | 可     |                | 差戻し日時                                 |
| 差戻しコメント     | `reject_comment`        | `VARCHAR(1000)` | 可     |                | 最新の差戻しコメント                       |

主な制約:

- `report_id` を主キーとする。
- `employee_user_id` と `report_date` の組み合わせを一意とする。
- `employee_user_id` は `users.user_id`、`holiday_type` は `holiday_types.holiday_type`、`break_type_id` は `break_types.break_type_id`、`work_time_type_id` は `work_time_types.work_time_type_id` を参照する。
- `approval_status` は `DRAFT`、`PENDING`、`REJECTED`、`APPROVED` のいずれかとする。
- `start_time_minutes` または `end_time_minutes` がNULLでない場合、`end_time_minutes` は `start_time_minutes` より後でなければならない。
- 休日区分に応じた各項目の必須・禁止、作業明細合計との一致、ロールと社員種別の整合性はアプリケーションで検証する。
- `approver_user_id` と `rejector_user_id` は現行DDLでは外部キーを設定していない。参照権限とロールの検証はアプリケーションで行う。

## 制約・索引

### 主キー・一意制約

| テーブル        | 制約                             | 対象列                            |
| --------------- | -------------------------------- | --------------------------------- |
| `daily_reports` | `pk_daily_reports`               | `report_id`                       |
| `daily_reports` | `uq_daily_reports_employee_date` | `employee_user_id`, `report_date` |

### 外部キー

| 制約                              | 子テーブル・列                    | 親テーブル・列                      |
| --------------------------------- | --------------------------------- | ----------------------------------- |
| `fk_daily_reports_employee`       | `daily_reports.employee_user_id`  | `users.user_id`                     |
| `fk_daily_reports_holiday_type`   | `daily_reports.holiday_type`      | `holiday_types.holiday_type`        |
| `fk_daily_reports_break_type`     | `daily_reports.break_type_id`     | `break_types.break_type_id`         |
| `fk_daily_reports_work_time_type` | `daily_reports.work_time_type_id` | `work_time_types.work_time_type_id` |

### CHECK制約

| 制約                          | 対象                                                   | 条件                                                  |
| ----------------------------- | ------------------------------------------------------ | ----------------------------------------------------- |
| `ck_daily_reports_status`     | `daily_reports.approval_status`                        | `DRAFT`、`PENDING`、`REJECTED`、`APPROVED` のいずれか |
| `ck_daily_reports_time_order` | `daily_reports.start_time_minutes`、`end_time_minutes` | いずれかがNULL、または終了分が開始分より大きい        |

### 明示索引

| 索引                        | テーブル        | 列                                                                               | 用途                       |
| --------------------------- | --------------- | -------------------------------------------------------------------------------- | -------------------------- |
| `ix_daily_reports_calendar` | `daily_reports` | `report_date`, `employee_user_id`, `group_id`, `approval_status`, `holiday_type` | カレンダー・一覧・状態検索 |

## 既存状態・監査情報の互換性

| 操作     | 保存前               | 保存後   | 更新範囲                                           |
| -------- | -------------------- | -------- | -------------------------------------------------- |
| 日報登録 | なし                 | `DRAFT`  | 日報入力項目、計算値、作業明細。既存監査項目はNULL |
| 日報編集 | `DRAFT` / `REJECTED` | 同じ状態 | 日報入力項目、計算値、作業明細。既存監査情報は維持 |

4種類の状態コードは検索・表示・編集可否判定に使用する。提出日時、承認者・日時、差戻し者・日時・コメントは既存データを参照するために保持する。本設計にこれらを生成・更新する業務操作は含めない。

この節は保持しているDB項目・旧DB資料の保存規則の記録であり、現行の画面・APIへ状態による制御を追加する指示ではない。
