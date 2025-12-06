# DOMAIN_ER.md

<!--
このファイルでは、ドメインエンティティとその関係を mermaid の erDiagram 形式で記述する。

目的:
- 「このシステムの世界には何が存在し、どう関係しているか」をユーザーと合意する。
- クラス図やテーブル定義など、後続の詳細設計のベースとする。

ルール:
- 永続/非永続に関わらず「ドメインとして意味のあるエンティティ」を列挙する。
- 各エンティティの属性コメントに storage_scope を付与する (任意だが推奨):

  storage_scope 候補:
    - Ephemeral        : 1操作 / 1フレーム内だけで完結
    - Session          : セッション(ログイン中, ゲーム1プレイ中 等)の間維持
    - DeviceLocal      : デバイスローカルに永続化
    - UserPersistent   : ユーザアカウントに紐づき複数デバイス間で共有
    - GlobalPersistent : システム全体で共有される永続データ

- 関係(1:1, 1:N, N:M等)には簡潔な説明ラベルを書くとよい。
-->

```mermaid
erDiagram
  USER ||--o{ EXAM_GOAL : "1 user can have multiple exam goals (1:N)"
  USER {
    string id           PK   "storage_scope: UserPersistent / UUID"
    string displayName       "storage_scope: UserPersistent / optional"
  }

  EXAM_GOAL ||--o{ STUDY_TARGET_AGGREGATE : "Derived daily/weekly/monthly targets (1:N)"
  EXAM_GOAL {
    string id          PK   "storage_scope: UserPersistent / UUID"
    string userId      FK   "storage_scope: UserPersistent"
    string examType         "storage_scope: UserPersistent / e.g. EIKEN"
    string examLevel        "storage_scope: UserPersistent / e.g. Grade1, Grade2"
    date   targetDate       "storage_scope: UserPersistent"
  }

  STUDY_TARGET_AGGREGATE {
    string id            PK   "storage_scope: UserPersistent / UUID"
    string examGoalId    FK   "storage_scope: UserPersistent"
    string timeUnit           "storage_scope: UserPersistent / day, week, month"
    int    targetWords        "storage_scope: UserPersistent / >= 0"
    int    targetMinutes      "storage_scope: UserPersistent / >= 0"
  }

  USER ||--o{ VOCABULARY_BOOK : "User can own multiple custom vocab books (1:N, built-in books have no owner)"
  VOCABULARY_BOOK ||--o{ VOCABULARY_WORD : "A book contains many words (1:N)"
  VOCABULARY_BOOK {
    string id            PK   "storage_scope: GlobalPersistent or UserPersistent / UUID"
    string title              "storage_scope: GlobalPersistent or UserPersistent"
    string examLevel          "storage_scope: GlobalPersistent / linked to exam levels; optional for custom books"
    boolean isBuiltIn         "storage_scope: GlobalPersistent / true for built-in books; false for custom books"
    string ownerUserId   FK   "storage_scope: UserPersistent / nullable / owner for custom books"
  }

  VOCABULARY_WORD ||--o{ WORD_SENSE : "A word can have multiple senses (1:N)"
  VOCABULARY_WORD {
    string id             PK   "storage_scope: GlobalPersistent / UUID"
    string vocabularyBookId FK "storage_scope: GlobalPersistent"
    string lemma               "storage_scope: GlobalPersistent / base form"
    string partOfSpeech        "storage_scope: GlobalPersistent"
    boolean isPolysemous       "storage_scope: GlobalPersistent / true if multiple key senses"
  }

  WORD_SENSE {
    string id             PK   "storage_scope: GlobalPersistent / UUID"
    string vocabularyWordId FK "storage_scope: GlobalPersistent"
    string meaningJa          "storage_scope: GlobalPersistent"
    string exampleSentenceEn  "storage_scope: GlobalPersistent"
    string exampleSentenceJa  "storage_scope: GlobalPersistent"
  }

  USER ||--o{ WORD_SENSE_MASTERY : "User tracks mastery per word sense (1:N)"
  WORD_SENSE ||--o{ WORD_SENSE_MASTERY : "One sense can have many user mastery records (1:N)"
  WORD_SENSE_MASTERY {
    string id             PK   "storage_scope: UserPersistent / UUID"
    string userId         FK   "storage_scope: UserPersistent"
    string wordSenseId    FK   "storage_scope: UserPersistent"
    string masteryStage        "storage_scope: UserPersistent / candidate, shortContextChecked, contextConfirmed"
    datetime lastTestedAt      "storage_scope: UserPersistent / optional"
    datetime nextReviewAt      "storage_scope: UserPersistent / optional / scheduled by forgetting-curve algorithm"
    int      mistakenCount     "storage_scope: UserPersistent / >= 0 / total times answered incorrectly"
    datetime lastMistakenAt    "storage_scope: UserPersistent / optional"
    boolean fromReadingContext "storage_scope: UserPersistent / true if confirmed via reading"
  }

  READING_PASSAGE ||--o{ READING_QUESTION : "One passage can have many questions (1:N)"
  READING_PASSAGE {
    string id             PK   "storage_scope: GlobalPersistent / UUID"
    string examLevel          "storage_scope: GlobalPersistent"
    int    estimatedSeconds   "storage_scope: GlobalPersistent / e.g. around 120 seconds"
    string text               "storage_scope: GlobalPersistent"
  }

  READING_QUESTION {
    string id               PK   "storage_scope: GlobalPersistent / UUID"
    string readingPassageId FK   "storage_scope: GlobalPersistent"
    string questionType         "storage_scope: GlobalPersistent / comprehension, vocabularyInContext etc."
    string prompt               "storage_scope: GlobalPersistent"
  }

  USER ||--o{ READING_SESSION : "User can run many reading sessions (1:N)"
  READING_PASSAGE ||--o{ READING_SESSION : "Each session is for one passage (1:N)"
  READING_SESSION {
    string id               PK   "storage_scope: UserPersistent / UUID"
    string userId           FK   "storage_scope: UserPersistent"
    string readingPassageId FK   "storage_scope: UserPersistent"
    datetime startedAt          "storage_scope: UserPersistent"
    int      allowedSeconds     "storage_scope: UserPersistent / e.g. 120"
    boolean  forceFinished      "storage_scope: UserPersistent / true if finished by countdown"
  }

  USER ||--|| USER_SETTING : "User has at most one settings record (1:1)"
  USER_SETTING {
    string id                          PK   "storage_scope: UserPersistent / UUID"
    string userId                      FK   "storage_scope: UserPersistent"
    string reviewStrictness                 "storage_scope: UserPersistent / strict, standard, relaxed"
    string defaultQuestionDirection         "storage_scope: UserPersistent / EN_TO_JA, JA_TO_EN, MIXED"
    int    mistakenFocusReviewIntervalDays  "storage_scope: UserPersistent / > 0 / min days between mistaken-only review sessions"
    datetime lastMistakenFocusReviewAt      "storage_scope: UserPersistent / optional"
  }