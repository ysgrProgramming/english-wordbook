# USE_CASES.md

<!--
このファイルでは、ユースケース仕様をテキストで記述する。
UMLの Fully Dressed Use Case を簡略化し、Domain / State / Operations と
トレースしやすい構造にしている。

関係:
- States: INTERACTION_FLOW.md の状態IDを参照する。
- Operations: REQUIREMENT.md の Public Operations / APIs の OP_ID を参照する。
- Domain: DOMAIN_ER.md のエンティティ・ルールと整合している必要がある。

ルール:
- 各ユースケースは UC-ID 単位で 1 ブロックとして記述する。
- Precondition / Postcondition はできる限り Domain の用語で書く。
- States / Operations は配列のように [] 内にIDを列挙する。
- ErrorCases は代表的なものだけでよいが、「どう扱うか」も簡潔に書くこと。
-->

## Use Cases

### UC-1

- UC_ID: UC_PLAN_AND_STUDY_VOCAB_BY_EXAM_GOAL  
- Title: Plan and study vocabulary toward exam goal  
- Actor: ExamLearner  

- Goal:  
  - The learner plans vocabulary study toward a specific exam goal and target date, studies daily using built-in vocabulary books and optional custom words, and can continuously reduce unknown words while monitoring progress against daily, weekly, and monthly targets.  
  - For polysemous words whose meanings change by context, understanding is finally confirmed through reading comprehension sessions; for simpler words (for example, many nouns with a single main sense), short-context tests (sentence-level meaning checks or cloze-style questions) can be used to confirm understanding.  

- Precondition:  
  - The learner can access the vocabulary book application.  
  - At least one built-in vocabulary book is available for the learner's exam level (for example, Eiken level or high school grade level).  

- Postcondition:  
  - The learner has updated study records for the day (for example, candidate / short-context-checked / context-confirmed mastery stages for words or senses, and study time).  
  - The application can show the learner's progress toward the final exam goal, including numbers and ratios of "cleared" words and remaining unknown words, where "cleared" is counted only for senses that reached context-confirmed state via appropriate tests (reading comprehension for polysemous words, or short-context tests for simple words).  
  - If the learner chose to run a reading comprehension check, the result is reflected and difficult sentences or words are scheduled for re-study; reading sessions provide explanations and full-text review after a timed reading phase.  

- States: [ STATE_HOME -> STATE_GOAL_DASHBOARD -> STATE_DAILY_VOCAB_SESSION -> STATE_GOAL_DASHBOARD, STATE_GOAL_DASHBOARD -> STATE_READING_CHECK_SESSION -> STATE_GOAL_DASHBOARD ]  
  <!-- INTERACTION_FLOW.md の状態ID列。例: [STATE_HOME -> STATE_PROFILE_EDIT -> STATE_PROFILE_VIEW] -->

- Operations: [ OP_VIEW_GOAL_DASHBOARD, OP_UPDATE_STUDY_GOALS, OP_START_DAILY_VOCAB_SESSION, OP_ANSWER_VOCAB_CARD, OP_START_READING_CHECK_SESSION, OP_ANSWER_READING_CHECK ]  
  <!-- REQUIREMENT.md の OP_ID や主要な操作名を列挙する。 -->

- ErrorCases:
  - No appropriate built-in vocabulary book exists for the selected exam level.  
  - Study goal definitions (for example, daily word count) are not consistent with the final exam date (for example, the required number of words per day is unrealistically high).  
  - Network or storage error prevents saving study records or loading reading comprehension content.  

---

### UC-2

- UC_ID: UC_MONTHLY_FOCUS_REVIEW_MISTAKEN_WORDS  
- Title: Run monthly focus review session for mistaken words  
- Actor: ExamLearner  

- Goal:  
  - The learner occasionally runs a special review-only session that focuses on mistaken or difficult words, without mixing new words, while keeping normal daily sessions as a mix of new and due-for-review words scheduled by the forgetting-curve-based algorithm.  

- Precondition:  
  - The learner can access the goal dashboard.  
  - There are enough mistaken or difficult words accumulated for meaningful review.  
  - A monthly focus review session has not been executed within the last configured interval (for example, about one month) to avoid overusing this mode and blocking progress on new words.  

- Postcondition:  
  - Mistaken or difficult words targeted in the session have updated mastery records (for example, updated mastery stage, lastTestedAt, and nextReviewAt).  
  - The system records the execution time of the focus review session and enforces the interval until the next allowed focus review.  

- States: [ STATE_GOAL_DASHBOARD -> STATE_REVIEW_MISTAKEN_ONLY_SESSION -> STATE_GOAL_DASHBOARD ]  

- Operations: [ OP_START_MISTAKEN_ONLY_REVIEW_SESSION, OP_ANSWER_MISTAKEN_REVIEW_CARD ]  

- ErrorCases:
  - No sufficient mistaken or difficult words exist to start a meaningful focus review session.  
  - The learner tries to start the focus review mode again before the allowed interval has passed.  
  - Network or storage error prevents loading or saving review results.  

<!--
UC-3 以降も同じテンプレートをコピーして必要なだけ追加する。
-->


