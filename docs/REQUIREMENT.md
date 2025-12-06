# Project Requirements

<!--
このファイルはプロジェクトの要件定義のメインエントリです。
Goal / Domain / Interaction / Architecture の4本柱をまとめて記述します。

他の設計ファイルとの関係:
- ドメイン構造: DOMAIN_ER.md
- インタラクション状態遷移: INTERACTION_FLOW.md
- コンポーネント構成: ARCHITECTURE_DIAGRAM.md
- ユースケース詳細: USE_CASES.md

運用ルール:
- 「TBD」や空欄のままにせず、すべての項目に具体的な内容を埋めてから次フェーズに進むこと。
- 説明文は必要に応じて削除してよいが、見出し・項目名は原則残すこと。
-->

---

## 1. Overview (Goal / Scope)

### 1.1 Product Summary

<!-- プロジェクトの概要を短く定義する。 -->

- Goal:  
  - Help exam-focused English learners eliminate unknown words for their target exam by planning and executing daily vocabulary study and reading checks based on built-in vocabulary books.  
- Target Users:  
  - Learners from high school students to advanced candidates (for example Eiken Grade 1) who study for English exams and want to use short spare time on mobile devices.  
- One-line Description:  
  - Goal-based English vocabulary book application that turns exam goals into daily / weekly / monthly word-count and time targets and visualizes cleared versus remaining unknown words.  

### 1.2 Scope

<!--
スコープ内とスコープ外を明確に切り分ける。
「やらないこと」を書くことでスコープの暴走を防ぐ。
-->

- In Scope:  
  - Provide built-in vocabulary books organized by exam level or school grade.  
  - Allow the learner to set a final exam goal (exam type, level, target date) and derive daily / weekly / monthly study targets for both time and number of words.  
  - Provide daily vocabulary study sessions using built-in vocabulary books and optional custom words.  
  - Support question directions of English-to-Japanese and Japanese-to-English (chosen by the learner per session or configuration).  
  - Track known / unknown status per word and study time, and visualize numbers and ratios of cleared versus remaining unknown words.  
  - Provide optional reading comprehension checks with longer texts and feed difficult words / sentences back into vocabulary review.  
- Out of Scope:  
  - Detailed test-preparation features beyond vocabulary and reading (for example full mock exams, listening, speaking, writing practice).  
  - Complex calendar integration with external services (for example sync with external calendar providers).  
  - Spelling test style questions that require full English word input.  

### 1.3 Success Criteria & Constraints

<!--
成功判定の基準と、守るべき制約条件を記述する。
例: 指標(DAU, 継続率, 応答時間)や締切、対応プラットフォームなど。
-->

- Success Criteria:  
  - The learner can always view the final exam goal and derived daily / weekly / monthly study targets on a main screen without navigating deeply.  
  - The learner can check numbers and ratios of cleared versus remaining unknown words for each built-in vocabulary book linked to an exam level.  
  - The learner can optionally run reading comprehension checks and see whether studied words are understood in context.  
  - Quantitative KPIs (for example retention rate or average unknown-word reduction) will be defined after collecting prototype usage data; this is intentionally deferred and will be documented later.  
- Constraints:  
  - The application must be comfortable to use in short spare time on mobile devices (for example simple operations and short interaction cycles).  
  - The primary content focus is English vocabulary and reading; other skills such as listening or speaking are not covered in the initial scope.  
  - Target exam types include at least Eiken levels and high school level vocabulary; support for other exam brands will be considered later.  

---

## 2. Domain Model (Domain)

### 2.1 Domain Entities

<!--
ドメインエンティティとその関係は DOMAIN_ER.md に mermaid の erDiagram 形式で記述する。

DOMAIN_ER.md では、各エンティティに storage_scope をコメントで付与する運用を想定する:
- Ephemeral          : 1操作 / 1フレーム内だけで完結する一時データ
- Session            : セッション(ログイン中, ゲーム1プレイ中 等)の間維持されるデータ
- DeviceLocal        : デバイス上に永続化され、主にそのデバイスだけで利用されるデータ
- UserPersistent     : ユーザアカウントに紐づき、複数デバイス間で共有されるデータ
- GlobalPersistent   : システム全体で共有される永続データ
-->

- Domain ER Diagram: see `DOMAIN_ER.md`  

### 2.2 Domain Notes (Optional)

<!--
ER 図だけでは表現しづらいルールや補足がある場合に記述する。
何もなければ空でもよい。
-->

- Notes:  
  - Each vocabulary word can have one or more senses; polysemous words (for example, words whose meaning changes significantly by context) must be treated per sense when judging understanding.  
  - The application manages multi-stage mastery states for each target word (or word sense), such as: candidate (seen only in card study), short-context-checked (passed short sentence tests), and context-confirmed (passed reading comprehension or equivalent high-context tests).  
  - A word is counted as "cleared" in progress metrics only when all required senses reach the context-confirmed state; simpler words (for example, many nouns with a single main sense) may reach this state via short-context tests, while polysemous words require reading comprehension checks.  
  - The forgetting-curve-based review scheduler uses a strict default profile (shorter review intervals and fewer allowed lapses), but learners can adjust the strictness level (for example, strict / standard / relaxed) via application settings; the scheduling logic must remain stable and consistent across profiles.  
  - Daily vocabulary sessions and mistaken-only review sessions use sessionId values that are ephemeral identifiers for the interaction; no dedicated persistent session entities are used, and durable data is stored in word sense mastery records and user settings (for example, last executed mistaken-only review timestamp).  

---

## 3. Interaction / UI / Operations

### 3.1 Interaction State Flow

<!--
システム全体のインタラクション状態遷移は INTERACTION_FLOW.md に mermaid flowchart で記述する。

ルール:
- ノードIDは状態(State)IDとして扱う（例: STATE_TITLE, STATE_IN_GAME 等）。
- 矢印には可能な限り「イベント名」をラベルとして付ける。
  例: |start_button_clicked|, |timer_expired|, |message_received| など。
-->

- Interaction State Diagram: see `INTERACTION_FLOW.md`  

### 3.2 Public Operations / APIs

<!--
クライアント(人間・他システム・外部アプリ等)から見える操作/インターフェースを列挙する。
Web の場合は HTTP パス、CLI の場合はコマンド、ゲーム/組み込みの場合は公開APIやメッセージ名など。

列挙した OP_ID は USE_CASES.md の Operations から参照される前提。
-->

| OP_ID                              | Interface / Path / Command                                    | Summary                                                                                   |
|------------------------------------|---------------------------------------------------------------|-------------------------------------------------------------------------------------------|
| OP_VIEW_GOAL_DASHBOARD             | GET /app/goal-dashboard                                      | Show final exam goal, derived targets, and progress (cleared / remaining unknown words).  |
| OP_UPDATE_STUDY_GOALS              | POST /app/study-goals                                        | Define or update exam goal and derived daily / weekly / monthly targets.                  |
| OP_START_DAILY_VOCAB_SESSION       | POST /app/daily-vocab-sessions                               | Start or resume a daily vocabulary study session from built-in vocabulary books.          |
| OP_ANSWER_VOCAB_CARD               | POST /app/daily-vocab-sessions/{sessionId}/answers           | Submit an answer for a vocabulary card (known / unknown, EN->JA or JA->EN).              |
| OP_START_READING_CHECK_SESSION     | POST /app/reading-check-sessions                             | Start a reading comprehension check session using a longer text.                          |
| OP_ANSWER_READING_CHECK            | POST /app/reading-check-sessions/{sessionId}/answers         | Submit reading comprehension answers and record difficult words / sentences for review.   |
| OP_START_MISTAKEN_ONLY_REVIEW_SESSION | POST /app/mistaken-only-review-sessions                    | Start a monthly focus review session for mistaken or difficult words only.                |
| OP_ANSWER_MISTAKEN_REVIEW_CARD     | POST /app/mistaken-only-review-sessions/{sessionId}/answers  | Submit an answer for a card in the mistaken-only review session.                          |
| OP_MANAGE_EXAM_GOALS               | GET/POST /app/exam-goals                                     | View and manage exam goals (create, update, archive).                                     |
| OP_VIEW_STUDY_HISTORY              | GET /app/study-history                                       | View study history and statistics over days, weeks, and months.                           |
| OP_MANAGE_CUSTOM_VOCAB_BOOKS       | GET/POST /app/custom-vocab-books                             | View and manage custom vocabulary books created by the learner.                           |
| OP_VIEW_DIFFICULT_WORD_LIST        | GET /app/difficult-words                                     | View list of difficult or repeatedly mistaken words and their mastery states.             |
| OP_UPDATE_APP_SETTINGS             | GET/POST /app/settings                                       | View and update application-level settings (for example notification, defaults).          |
| OP_VIEW_HELP                       | GET /app/help                                                | View help and usage guide for the application.                                            |

### 3.3 Use Cases

<!--
ユースケースの詳細仕様は USE_CASES.md に記述する。

ここでは「ユースケースの一覧」や「重要なユースケースIDだけ」を簡潔に列挙しておくとよい。
-->

- Key Use Case IDs:  
  - UC_PLAN_AND_STUDY_VOCAB_BY_EXAM_GOAL  
  - UC_MONTHLY_FOCUS_REVIEW_MISTAKEN_WORDS  
  - UC_MANAGE_EXAM_GOALS  
  - UC_MANAGE_CUSTOM_VOCAB_BOOKS  
  - UC_VIEW_STUDY_HISTORY  
  - UC_CONFIGURE_APP_SETTINGS  

---

## 4. Architecture

### 4.1 Components & Data Flow

<!--
システムを構成する主要なコンポーネント(箱)とデータフローは ARCHITECTURE_DIAGRAM.md に mermaid flowchart で記述する。

ルール:
- ノードはコンポーネント(例: Frontend, API Server, GameServer, DeviceController 等)。
- 外部サービス・外部デバイスはノード名に「(外部)」を付ける。
- 矢印には可能な限り「代表的な操作/プロトコル名」をラベルとして付ける。
  例: |HTTP /api/login|, |gRPC Match|, |I2C Read| など。
-->

- Architecture Diagram: see `ARCHITECTURE_DIAGRAM.md`  

### 4.2 Storage Scope Policy

<!--
DOMAIN_ER.md で付与した storage_scope を、具体的なストレージ/媒体にマッピングする方針を記述する。
例: UserPersistent -> クラウドDB, Session -> サーバメモリ+キャッシュ 等。
-->

- Ephemeral:  
  - Stored only in application process memory on client or server during a single interaction (for example current card being shown, countdown timers) and not persisted after the operation ends.  
- Session:  
  - Stored in short-lived session mechanisms such as authentication tokens or server-side session stores, mainly for keeping the learner logged in and tracking current session identifiers.  
- DeviceLocal:  
  - Stored on the learner's device (for example local storage or on-device database) for offline-friendly features such as cached vocabulary content or last viewed progress; synchronized with UserPersistent data when online.  
- UserPersistent:  
  - Stored in a cloud database per user account and shared across devices, including exam goals, study targets, word sense mastery records, reading sessions, custom vocabulary books, and user settings (for example review strictness and default question direction).  
- GlobalPersistent:  
  - Stored in shared system-wide databases, including built-in vocabulary books, word senses, and reading passages / questions that are common to all learners.  

### 4.3 Non-Functional Requirements (Architecture-related)

<!--
アーキテクチャ設計に強く影響する非機能要件のみを記述する。
詳細なSLAや細かい数値要件がある場合は別ドキュメントに分けてもよい。
-->

- Performance / Throughput:  
  - Typical API responses for study and progress views should complete within a few hundred milliseconds under normal load, and the system should comfortably support concurrent usage by typical class sizes and self-learners without noticeable delay.  
- Security / Authentication / Authorization:  
  - All external communication must use HTTPS, user accounts must be authenticated (for example via email/password or external identity provider), and per-user data such as mastery records and goals must not be accessible to other users. No hardcoded secrets or API keys may be stored in the codebase.  
- Availability / Reliability / Backup:  
  - The service should be reasonably available for daily study (for example target monthly uptime around 99%); user-persistent data such as mastery records and goals must be regularly backed up so that individual failures do not cause irreversible loss of long-term learning history.  
- Observability (Logging / Metrics / Tracing / Alerting):  
  - The system should log key operations (for example session creation, answer submissions, scheduler decisions) and export metrics to monitor usage, error rates, and scheduling anomalies; minimal tracing or structured logging should allow analysis of learning flows without exposing personal content.  
- Other NFRs:  
  - The learning algorithm must support configurable review strictness while keeping a safe default: strict forgetting-curve scheduling aimed at exam preparation, tunable per learner from application settings.  