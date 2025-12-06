# INTERACTION_FLOW.md

<!--
このファイルでは、ユーザ視点またはシステム視点の「インタラクション状態遷移」を
mermaid の flowchart で記述する (UML 状態マシン図の簡易版)。

目的:
- システムがどのような状態(State)を持ち、どのイベントで状態が変わるかを俯瞰する。
- USE_CASES.md の States 項目から参照される「状態ID」の定義場所とする。

ルール:
- ノードIDは状態IDとして一意に命名する (例: STATE_TITLE, STATE_IN_GAME, STATE_ERROR)。
- 矢印には「イベント名」をできるだけラベルとして明記する。
  - 例: |start_button_clicked|, |login_success|, |timeout|, |sensor_triggered| など。
- ガード条件や細かい例外条件は、必要に応じて [condition] のような簡潔な表記か、
  USE_CASES.md 側の Precondition / ErrorCases に寄せる。
-->

```mermaid
flowchart TD
  %% Goal-based vocabulary learning main flow
  STATE_HOME["STATE_HOME\nHome / Entry"] -->|open_goal_dashboard| STATE_GOAL_DASHBOARD["STATE_GOAL_DASHBOARD\nGoal dashboard (main home)"]

  STATE_GOAL_DASHBOARD -->|start_daily_vocab_session| STATE_DAILY_VOCAB_SESSION["STATE_DAILY_VOCAB_SESSION\nDaily vocabulary session"]
  STATE_DAILY_VOCAB_SESSION -->|finish_daily_vocab_session| STATE_GOAL_DASHBOARD

  STATE_GOAL_DASHBOARD -->|start_reading_check| STATE_READING_CHECK_SESSION["STATE_READING_CHECK_SESSION\nReading comprehension check session"]
  STATE_READING_CHECK_SESSION -->|finish_reading_check| STATE_GOAL_DASHBOARD

  %% Monthly focus review session for mistaken words
  STATE_GOAL_DASHBOARD -->|start_monthly_mistaken_only_review| STATE_REVIEW_MISTAKEN_ONLY_SESSION["STATE_REVIEW_MISTAKEN_ONLY_SESSION\nMonthly mistaken-only review session"]
  STATE_REVIEW_MISTAKEN_ONLY_SESSION -->|finish_monthly_mistaken_only_review| STATE_GOAL_DASHBOARD

  %% Additional menus reachable from goal dashboard
  STATE_GOAL_DASHBOARD -->|open_exam_goal_management| STATE_EXAM_GOAL_MANAGEMENT["STATE_EXAM_GOAL_MANAGEMENT\nExam goal management"]
  STATE_EXAM_GOAL_MANAGEMENT -->|back_to_goal_dashboard| STATE_GOAL_DASHBOARD

  STATE_GOAL_DASHBOARD -->|open_study_history| STATE_STUDY_HISTORY["STATE_STUDY_HISTORY\nStudy history / stats"]
  STATE_STUDY_HISTORY -->|back_to_goal_dashboard| STATE_GOAL_DASHBOARD

  STATE_GOAL_DASHBOARD -->|open_custom_vocab_books| STATE_CUSTOM_VOCAB_BOOKS["STATE_CUSTOM_VOCAB_BOOKS\nCustom word book management"]
  STATE_CUSTOM_VOCAB_BOOKS -->|back_to_goal_dashboard| STATE_GOAL_DASHBOARD

  STATE_GOAL_DASHBOARD -->|open_difficult_word_list| STATE_DIFFICULT_WORD_LIST["STATE_DIFFICULT_WORD_LIST\nDifficult word list"]
  STATE_DIFFICULT_WORD_LIST -->|back_to_goal_dashboard| STATE_GOAL_DASHBOARD

  STATE_GOAL_DASHBOARD -->|open_settings| STATE_SETTINGS["STATE_SETTINGS\nSettings"]
  STATE_SETTINGS -->|back_to_goal_dashboard| STATE_GOAL_DASHBOARD

  STATE_GOAL_DASHBOARD -->|open_help| STATE_HELP["STATE_HELP\nHelp / How to use"]
  STATE_HELP -->|back_to_goal_dashboard| STATE_GOAL_DASHBOARD
``` 