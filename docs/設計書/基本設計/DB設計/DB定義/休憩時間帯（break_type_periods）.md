# 休憩時間帯（break_type_periods）

[DB設計の目次](../README.md)、[DB区分一覧](../DB区分一覧.md)。

## 業務上の意味と関係

休憩区分に紐づく休憩時間帯を表す。
1つの休憩区分に複数の休憩時間帯を持てる。

- `end_minutes` は `start_minutes` より後にする
- 同一休憩区分内の休憩時間帯は、初期データ作成時に重複しないようにする

## 列定義

凡例: NULL可が「不可」はNOT NULL、キーが空欄の列は非キー列。休憩区分IDと表示順は複合主キーの構成列。

| 論理名     | 物理名          | 型            | NULL可 | キー・初期値 | 説明                        |
| ---------- | --------------- | ------------- | ------ | ------------ | --------------------------- |
| 休憩区分ID | `break_type_id` | `VARCHAR(20)` | 不可   | 複合PK、FK   | `break_types.break_type_id` |
| 開始分     | `start_minutes` | `INTEGER`     | 不可   |              | 0:00からの経過分            |
| 終了分     | `end_minutes`   | `INTEGER`     | 不可   |              | 0:00からの経過分            |
| 表示順     | `display_order` | `INTEGER`     | 不可   | 複合PK       | 同一休憩区分内の順序        |

主な制約・識別単位:

- `break_type_id` は `break_types.break_type_id` を参照する。
- 独立した時間帯IDは設けず、複合主キーを使用する。
- `break_type_id` と `display_order` の組み合わせを複合主キーとする。
- 同じ休憩区分内の表示順の重複は複合主キーで禁止する。
- `end_minutes` は `start_minutes` より後でなければならない。DBのCHECK制約で検証する。

## 制約・索引

### 主キー・一意制約

主キーは休憩区分IDと表示順の複合キーとする。

| 制約                    | 対象列                           |
| ----------------------- | -------------------------------- |
| `pk_break_type_periods` | `break_type_id`, `display_order` |

### 外部キー

| 制約                         | 子テーブル・列                     | 親テーブル・列              |
| ---------------------------- | ---------------------------------- | --------------------------- |
| `fk_break_type_periods_type` | `break_type_periods.break_type_id` | `break_types.break_type_id` |

### CHECK制約

| 対象列                         | 条件                          |
| ------------------------------ | ----------------------------- |
| `start_minutes`                | 0以上1439以下                 |
| `end_minutes`                  | 1以上1440以下                 |
| `start_minutes`, `end_minutes` | `end_minutes > start_minutes` |

### 明示索引

旧DB定義の制約・索引一覧に記載なし。
