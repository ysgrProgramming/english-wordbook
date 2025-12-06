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
  - The system records the execution time of the focus review session and enforces the interval until the next allowed focus review (for example, by updating user settings such as lastMistakenFocusReviewAt).  

- States: [ STATE_GOAL_DASHBOARD -> STATE_REVIEW_MISTAKEN_ONLY_SESSION -> STATE_GOAL_DASHBOARD ]  

- Operations: [ OP_START_MISTAKEN_ONLY_REVIEW_SESSION, OP_ANSWER_MISTAKEN_REVIEW_CARD ]  

- ErrorCases:
  - No sufficient mistaken or difficult words exist to start a meaningful focus review session.  
  - The learner tries to start the focus review mode again before the allowed interval has passed.  
  - Network or storage error prevents loading or saving review results.  

---

### UC-3

- UC_ID: UC_MANAGE_EXAM_GOALS  
- Title: Manage exam goals for vocabulary study  
- Actor: ExamLearner  

- Goal:  
  - The learner creates, updates, or archives exam goals (exam type, level, target date) so that daily / weekly / monthly study targets can be derived and kept up to date.  

- Precondition:  
  - The learner can access the exam goal management screen from the goal dashboard.  

- Postcondition:  
  - Exam goal records are created or updated in a consistent state, and obsolete goals can be archived instead of deleted to preserve history.  
  - The goal dashboard reflects the currently active exam goal and its derived study targets.  

- States: [ STATE_GOAL_DASHBOARD -> STATE_EXAM_GOAL_MANAGEMENT -> STATE_GOAL_DASHBOARD ]  

- Operations: [ OP_MANAGE_EXAM_GOALS, OP_VIEW_GOAL_DASHBOARD ]  

- ErrorCases:
  - The learner sets an exam date in the past or an invalid combination of exam type and level.  
  - Network or storage error prevents saving changes to exam goals.  

---

### UC-4

- UC_ID: UC_MANAGE_CUSTOM_VOCAB_BOOKS  
- Title: Manage custom vocabulary books and words  
- Actor: ExamLearner  

- Goal:  
  - The learner creates and maintains custom vocabulary books (for example from past papers or reading) and optionally links them to exam goals or study sessions.  

- Precondition:  
  - The learner can access the custom vocabulary book management screen from the goal dashboard.  

- Postcondition:  
  - Custom vocabulary books and their words are created, updated, or deleted as requested, and are available as sources for future study sessions.  

- States: [ STATE_GOAL_DASHBOARD -> STATE_CUSTOM_VOCAB_BOOKS -> STATE_GOAL_DASHBOARD ]  

- Operations: [ OP_MANAGE_CUSTOM_VOCAB_BOOKS ]  

- ErrorCases:
  - The learner tries to delete a custom book that is still referenced by active settings or reports.  
  - Network or storage error prevents saving changes to custom vocabulary books or words.  

---

### UC-5

- UC_ID: UC_VIEW_STUDY_HISTORY  
- Title: View study history and statistics  
- Actor: ExamLearner  

- Goal:  
  - The learner reviews study history and key statistics (for example study time, cleared words, reading sessions) over days, weeks, and months to understand long-term progress.  

- Precondition:  
  - The learner can access the study history / stats screen from the goal dashboard.  

- Postcondition:  
  - The learner can see aggregated metrics and trends, filtered by time range and (optionally) exam goal or vocabulary book.  

- States: [ STATE_GOAL_DASHBOARD -> STATE_STUDY_HISTORY -> STATE_GOAL_DASHBOARD ]  

- Operations: [ OP_VIEW_STUDY_HISTORY ]  

- ErrorCases:
  - No study data exists yet for the selected range (the system should handle this gracefully).  
  - Network or storage error prevents loading study history.  

---

### UC-6

- UC_ID: UC_CONFIGURE_APP_SETTINGS  
- Title: Configure application and review settings  
- Actor: ExamLearner  

- Goal:  
  - The learner configures application-level settings such as review strictness, default question direction, and monthly mistaken-only review policy so that the app behavior matches their learning style.  

- Precondition:  
  - The learner can access the settings screen from the goal dashboard.  

- Postcondition:  
  - User settings are updated and will be used by the review scheduler and UI (for example default direction EN->JA or JA->EN, strictness profile, minimum interval for mistaken-only review).  

- States: [ STATE_GOAL_DASHBOARD -> STATE_SETTINGS -> STATE_GOAL_DASHBOARD ]  

- Operations: [ OP_UPDATE_APP_SETTINGS ]  

- ErrorCases:
  - The learner enters invalid setting values (for example negative interval days); the system should validate and reject such inputs with clear messages.  
  - Network or storage error prevents saving updated settings.  
