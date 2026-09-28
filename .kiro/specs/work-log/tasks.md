# Implementation Plan: work-log

## Overview

HTML/CSS/JavaScript (ES6+) のみで構成するシングルページ作業記録アプリを実装します。
`index.html` 内の `<script>` タグに Storage・Validator・LogForm・LogList・App の5モジュールを順に実装し、最後に依存関係を接続してアプリを完成させます。
テストは Vitest + fast-check を使用し、`tests/` ディレクトリ配下に配置します。

## Tasks

- [ ] 1. プロジェクト構成とテスト環境のセットアップ
  - `work-log/` ディレクトリを作成し、`index.html`（スケルトン）と `tests/` ディレクトリを用意する
  - `package.json` を作成し、Vitest と fast-check を devDependencies に追加する
  - `vitest.config.js`（または package.json の `test` スクリプト）を設定し `vitest --run` で全テストが実行できる状態にする
  - _Requirements: 1.1, 2.1, 3.1_

- [ ] 2. Storage モジュールの実装とテスト
  - [ ] 2.1 Storage モジュールを実装する
    - `index.html` の `<script>` 内に `Storage` オブジェクトを定義する
    - `load()`: localStorage から JSON をデシリアライズして返す。例外時は空配列を返しコンソールに `[work-log] Failed to parse storage data:` + エラーを出力する
    - `save(logs)`: `JSON.stringify` で保存し、`QuotaExceededError` 等の例外を catch して `{ success: false, error }` を返す。成功時は `{ success: true }` を返す
    - `STORAGE_KEY = 'work-log-entries'` を定数として定義する
    - _Requirements: 4.1, 4.2, 4.3, 4.5, 4.6_

  - [ ]* 2.2 Property 1: Storage ラウンドトリップのプロパティテストを書く
    - `tests/storage.test.js` に fast-check を使用して記述する
    - **Property 1: Storage ラウンドトリップ**
    - **Validates: Requirements 4.4, 4.1, 4.2, 4.3**
    - `fc.array(fc.record({ id: fc.string(), description: fc.string({ minLength: 1, maxLength: 200 }), duration: fc.integer({ min: 1, max: 480 }), date: fc.string(), createdAt: fc.integer({ min: 0 }) }))` でランダムなリストを生成し、save → load のラウンドトリップで完全一致を検証する
    - `// Feature: work-log, Property 1: Storage ラウンドトリップ` タグを付与する

  - [ ]* 2.3 Property 2: 不正JSON耐性のプロパティテストを書く
    - `tests/storage.test.js` に追記する
    - **Property 2: 不正JSON耐性**
    - **Validates: Requirements 4.5**
    - `fc.string()` で生成した任意文字列を localStorage に直接セットした上で `Storage.load()` が例外をスローせず空配列を返すことを検証する
    - `// Feature: work-log, Property 2: 不正JSON耐性` タグを付与する

  - [ ]* 2.4 Storage のユニットテストを書く
    - `tests/storage.test.js` に追記する
    - QuotaExceededError 発生時に `{ success: false }` を返し既存データが変更されないことを確認する (要件4.6)

- [ ] 3. Validator モジュールの実装とテスト
  - [ ] 3.1 Validator モジュールを実装する
    - `index.html` の `<script>` 内に `Validator` オブジェクトを定義する（Storage の後に記述）
    - `validateDescription(value)`: 空 → `DESCRIPTION_EMPTY`、201文字以上 → `DESCRIPTION_TOO_LONG`、それ以外 → `{ valid: true }`
    - `validateDuration(value)`: 1〜480の整数でない場合 → `DURATION_INVALID`
    - `validateDate(value)`: 空 → `DATE_EMPTY`、フォーマット不正または実在しない日付 → `DATE_INVALID`（正規表現 + `new Date` による実在確認）
    - `validateAll(fields)`: 3フィールドを一括検証し `{ errors: {} }` を返す
    - `ERROR_MESSAGES` 定数オブジェクトをモジュール外に定義する
    - _Requirements: 1.5, 1.6, 1.7, 1.8, 1.9_

  - [ ]* 3.2 Property 3: 作業内容長バリデーションのプロパティテストを書く
    - `tests/validator.test.js` に fast-check を使用して記述する
    - **Property 3: バリデーション — 作業内容長**
    - **Validates: Requirements 1.5, 1.6**
    - 長さ1〜200の文字列 → `valid: true`、長さ0または201以上 → `valid: false` を検証する
    - `// Feature: work-log, Property 3: バリデーション — 作業内容長` タグを付与する

  - [ ]* 3.3 Property 4: 作業時間範囲バリデーションのプロパティテストを書く
    - `tests/validator.test.js` に追記する
    - **Property 4: バリデーション — 作業時間範囲**
    - **Validates: Requirements 1.7**
    - `fc.integer({ min: 1, max: 480 })` → `valid: true`、範囲外の整数または非整数 → `valid: false` を検証する
    - `// Feature: work-log, Property 4: バリデーション — 作業時間範囲` タグを付与する

  - [ ]* 3.4 Property 5: 日付バリデーションのプロパティテストを書く
    - `tests/validator.test.js` に追記する
    - **Property 5: バリデーション — 日付形式と実在性**
    - **Validates: Requirements 1.8, 1.9**
    - 有効なYYYY-MM-DD日付文字列 → `valid: true`、フォーマット不正・実在しない日付 → `valid: false` を検証する
    - `// Feature: work-log, Property 5: バリデーション — 日付形式と実在性` タグを付与する

- [ ] 4. チェックポイント — Storage・Validator のテストをすべて通過させる
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 5. LogForm コンポーネントの実装とテスト
  - [ ] 5.1 index.html にフォームのHTML構造を実装する
    - `<form id="log-form">` の中に作業内容・作業時間・日付の3つの `<input>` と対応する `<label>` を配置する
    - 各 `<label>` の `for` 属性と `<input>` の `id` 属性を一致させる（例: `for="description"` / `id="description"`）
    - 各フィールドの直下にエラー表示用の `<p class="error-message" role="alert">` を配置し、初期状態は空にする
    - フォーム上部にStorage全体エラー用の `<p id="form-save-error" role="alert">` を配置する
    - 送信ボタンを配置する
    - _Requirements: 1.1, 5.2_

  - [ ] 5.2 LogForm モジュールを実装する
    - `init(onSubmit)`: `#log-form` の `submit` イベントにハンドラを登録する
    - ハンドラ内で `Validator.validateAll()` を呼び出し、エラーがあればフィールド直下のエラー要素にメッセージをセットして `return`
    - バリデーション通過後は `onSubmit({ description, duration, date })` を呼び出す
    - `reset()`: 全フィールドの値をクリアし、全エラーメッセージ要素を空にする
    - `showSaveError(message)`: `#form-save-error` にメッセージをセットする
    - _Requirements: 1.2, 1.4, 1.5, 1.6, 1.7, 1.8, 1.9, 1.10_

  - [ ]* 5.3 LogForm のユニットテストを書く
    - `tests/logform.test.js` に JSDOM 環境（Vitest の `environment: 'jsdom'`）を使用して記述する
    - フォームに3つのフィールドと対応するlabelが存在することを確認する (要件1.1, 5.2)
    - 送信後にフォームがリセットされることを確認する (要件1.4)
    - `showSaveError()` 呼び出し後にエラーが表示され、`reset()` 後にクリアされることを確認する (要件1.10)

- [ ] 6. LogList コンポーネントの実装とテスト
  - [ ] 6.1 index.html にリスト表示のHTML構造を実装する
    - `<section id="log-list">` を配置し、内部に `<ul id="log-entries">` と空メッセージ用 `<p id="empty-message">` を配置する
    - Storage 読み込み失敗時用のエラー表示領域 `<p id="list-load-error" role="alert">` を配置する
    - _Requirements: 2.1, 2.3, 2.6_

  - [ ] 6.2 LogList モジュールを実装する
    - `render(logs, onDelete)`: `#log-entries` を毎回再描画する
    - `logs` が空配列のとき `#empty-message` を表示し `<ul>` を空にする（要件2.3, 3.5）
    - `logs` が1件以上のとき `createdAt` の降順でソートし、各エントリを `<li>` として描画する（要件2.4）
    - 各 `<li>` には description・duration（分単位の整数）・date（YYYY-MM-DD形式）と削除ボタン（`data-id` 属性に id をセット）を含める（要件2.2, 3.1）
    - 削除ボタンの `click` イベントで `onDelete(id)` を呼び出す
    - `showLoadError(message)`: `#list-load-error` にメッセージを表示し `<ul>` と `#empty-message` を非表示にする（要件2.6）
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 3.1, 3.5_

  - [ ]* 6.3 Property 6: WorkLog 表示完全性のプロパティテストを書く
    - `tests/loglist.test.js` に fast-check + JSDOM を使用して記述する
    - **Property 6: WorkLog 表示完全性**
    - **Validates: Requirements 2.1, 2.2**
    - `fc.record(workLogArbitrary)` で生成した任意の WorkLog について、`render()` 後のDOMに description・duration・date がすべて含まれることを検証する
    - `// Feature: work-log, Property 6: WorkLog 表示完全性` タグを付与する

  - [ ]* 6.4 Property 7: ソート順のプロパティテストを書く
    - `tests/loglist.test.js` に追記する
    - **Property 7: ソート順の保証**
    - **Validates: Requirements 2.4**
    - `fc.array(fc.record(workLogArbitrary), { minLength: 2 })` で生成したリストについて、`render()` 後のDOM行順が `createdAt` 降順になっていることを検証する
    - `// Feature: work-log, Property 7: ソート順の保証` タグを付与する

  - [ ]* 6.5 LogList のユニットテストを書く
    - `tests/loglist.test.js` に追記する
    - 空リスト時に「作業記録はまだありません」が表示される (要件2.3)
    - 各エントリに削除ボタンが1つ表示される (要件3.1)

- [ ] 7. チェックポイント — LogForm・LogList のテストをすべて通過させる
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 8. App コントローラーの実装とテスト
  - [ ] 8.1 App コントローラーモジュールを実装する
    - `index.html` の `<script>` 内に `App` オブジェクトを定義する（最後のモジュール）
    - `init()`: `Storage.load()` を呼び出し、失敗なら `LogList.showLoadError(ERROR_MESSAGES.LOAD_FAILED)` を表示してリターン。成功なら `LogList.render(logs, App.deleteLog)` で初期描画し、`LogForm.init(App.addLog)` でフォームを初期化する（要件2.5, 2.6）
    - `addLog(input)`: 新しい WorkLog オブジェクトを生成（id は `Date.now().toString(36) + Math.random().toString(36).slice(2)`、createdAt は `Date.now()`）し `Storage.save()` を呼び出す。失敗なら `LogForm.showSaveError()` を呼び出す。成功なら `LogForm.reset()` し `LogList.render()` で再描画する（要件1.2, 1.3, 1.4, 1.10）
    - `deleteLog(id)`: 現在のリストから対象を除外し `Storage.save()` を呼び出す。失敗なら削除失敗エラーを表示し元リストで再描画する。成功なら `LogList.render()` で再描画する（要件3.2, 3.3, 3.4）
    - `DOMContentLoaded` イベントで `App.init()` を呼び出す（要件2.5）
    - _Requirements: 1.2, 1.3, 1.4, 1.10, 2.5, 2.6, 3.2, 3.3, 3.4_

  - [ ]* 8.2 Property 8: 削除後の不在のプロパティテストを書く
    - `tests/app.test.js` に fast-check + JSDOM を使用して記述する
    - **Property 8: 削除後の不在**
    - **Validates: Requirements 3.2**
    - `fc.array(fc.record(workLogArbitrary), { minLength: 1 })` と `fc.nat()` (index) で対象を選択し、`App.deleteLog()` 後に `Storage.load()` のリストに該当 id が含まれないことを検証する
    - `// Feature: work-log, Property 8: 削除後の不在` タグを付与する

  - [ ]* 8.3 App のユニットテストを書く
    - `tests/app.test.js` に追記する
    - 起動時に Storage から WorkLog が読み込まれて LogList が初期化される (要件2.5)
    - Storage 読み込み失敗時にエラーが表示される (要件2.6)
    - 削除成功後に LogList が更新される (要件3.3)
    - 削除失敗時にエラーが表示され元データが保持される (要件3.4)

- [ ] 9. CSS スタイルとアクセシビリティ対応の実装
  - [ ] 9.1 レスポンシブレイアウトを実装する
    - `<head>` に `<meta name="viewport" content="width=device-width, initial-scale=1">` を追加する
    - CSS でビューポート幅375px以上で水平スクロールが発生しないレイアウトを実装する（`max-width`・`box-sizing: border-box` を活用）
    - _Requirements: 5.1_

  - [ ] 9.2 フォーカスインジケーターを実装する
    - フォーカスを受けた要素に `outline` スタイルを CSS で明示的に設定する（ブラウザデフォルトの `outline: none` を上書きしない）
    - `<input>`・`<button>` 要素が Tab キーでフォーカス移動可能であることを確認する（`tabindex` はデフォルトのままで対応）
    - _Requirements: 5.3, 5.4_

- [ ] 10. 最終チェックポイント — すべてのテストを通過させてアプリを完成させる
  - Ensure all tests pass, ask the user if questions arise.
  - `index.html` をブラウザで直接開いて基本動作（追加・一覧表示・削除・リロード後の永続化）を確認する

## Notes

- タスクに `*` が付いたサブタスクはオプションです。MVPを優先する場合はスキップできます
- 各タスクは前のタスクの成果物を前提としています。順番通りに実装してください
- テストは `npx vitest --run` で実行します（ウォッチモードは使わない）
- `index.html` の `<script>` 内モジュール順: `ERROR_MESSAGES` → `Storage` → `Validator` → `LogForm` → `LogList` → `App`
- JSDOM 環境が必要なテストファイルには `// @vitest-environment jsdom` をファイル先頭に記載してください
- プロパティテストは各最低100回のイテレーションで実行します（fast-check のデフォルト）

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1"] },
    { "id": 1, "tasks": ["2.1", "3.1"] },
    { "id": 2, "tasks": ["2.2", "2.3", "2.4", "3.2", "3.3", "3.4"] },
    { "id": 3, "tasks": ["5.1", "6.1"] },
    { "id": 4, "tasks": ["5.2", "6.2"] },
    { "id": 5, "tasks": ["5.3", "6.3", "6.4", "6.5"] },
    { "id": 6, "tasks": ["8.1", "9.1", "9.2"] },
    { "id": 7, "tasks": ["8.2", "8.3"] }
  ]
}
```
