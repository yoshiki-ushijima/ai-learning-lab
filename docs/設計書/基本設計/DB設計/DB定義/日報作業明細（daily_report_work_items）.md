# 日報作業明細（daily_report_work_items）

[DB設計の目次](../README.md)、[DB区分一覧](../DB区分一覧.md)。

## 業務上の意味と関係

日報の案件別・作業分類別の作業明細を表す。
1日の日報に複数件登録できる。

- `work_minutes` は1以上
- 同一日報内の作業明細は `display_order` の昇順で表示する
- 作業明細の合計時間は、実勤務時間がある日報の `work_minutes` と一致する必要がある
- 有給休暇の日報には作業明細を登録しない
- 作業なしの休日には作業明細を登録しない

## 列定義

凡例: NULL可が「不可」はNOT NULL、キーが空欄の列は非キー列。列名・型・NULL可・キー・初期値は旧DB定義から維持する。

| 論理名     | 物理名             | 型            | NULL可 | キー・初期値 | 説明                               |
| ---------- | ------------------ | ------------- | ------ | ------------ | ---------------------------------- |
| 作業明細ID | `work_item_id`     | `VARCHAR(40)` | 不可   | PK           | 作業明細識別子                     |
| 日報ID     | `report_id`        | `VARCHAR(40)` | 不可   | FK           | `daily_reports.report_id`          |
| 案件ID     | `project_id`       | `VARCHAR(20)` | 不可   | FK           | `projects.project_id`              |
| 作業分類ID | `work_category_id` | `VARCHAR(20)` | 不可   | FK           | `work_categories.work_category_id` |
| 作業時間分 | `work_minutes`     | `INTEGER`     | 不可   | CHECK        | 1分以上の作業時間                  |
| 表示順     | `display_order`    | `INTEGER`     | 不可   |              | 同一日報内の表示順                 |

主な制約:

- `work_item_id` を主キーとする。
- `report_id` は `daily_reports.report_id`、`project_id` は `projects.project_id`、`work_category_id` は `work_categories.work_category_id` を参照する。
- `work_minutes` は1以上とする。
- 同一日報内の作業明細合計が日報ヘッダの `work_minutes` と一致することは、アプリケーションで検証する。

## 制約・索引

### 主キー・一意制約

| テーブル                  | 制約                         | 対象列         |
| ------------------------- | ---------------------------- | -------------- |
| `daily_report_work_items` | `pk_daily_report_work_items` | `work_item_id` |

### 外部キー

| 制約                             | 子テーブル・列                             | 親テーブル・列                     |
| -------------------------------- | ------------------------------------------ | ---------------------------------- |
| `fk_daily_report_items_report`   | `daily_report_work_items.report_id`        | `daily_reports.report_id`          |
| `fk_daily_report_items_project`  | `daily_report_work_items.project_id`       | `projects.project_id`              |
| `fk_daily_report_items_category` | `daily_report_work_items.work_category_id` | `work_categories.work_category_id` |

### CHECK制約

| 制約                            | 対象                                   | 条件  |
| ------------------------------- | -------------------------------------- | ----- |
| `ck_daily_report_items_minutes` | `daily_report_work_items.work_minutes` | 1以上 |

### 明示索引

| 索引                           | テーブル                  | 列          | 用途                   |
| ------------------------------ | ------------------------- | ----------- | ---------------------- |
| `ix_daily_report_items_report` | `daily_report_work_items` | `report_id` | 日報単位の作業明細取得 |
