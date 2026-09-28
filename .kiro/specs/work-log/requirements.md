# Requirements Document

## Introduction

本ドキュメントは、シンプルな作業記録Webアプリ「work-log」の要件を定義します。
ユーザーは日々の作業内容・作業時間・日付を登録し、過去の記録を一覧で確認・削除できます。
アプリはHTML、CSS、JavaScriptのみで構築し（フレームワーク不使用）、データはブラウザのlocalStorageに永続化します。

## Glossary

- **App**: 作業記録Webアプリケーション全体
- **WorkLog**: 作業内容・作業時間・日付の3要素からなる1件の作業記録
- **LogForm**: 作業記録を入力・送信するフォームUI
- **LogList**: 登録済みの作業記録を一覧表示するUIコンポーネント
- **Storage**: ブラウザのlocalStorageを操作するデータ永続化レイヤー
- **作業内容**: 作業の説明テキスト（1文字以上、200文字以内）
- **作業時間**: 作業にかかった時間（分単位、1以上480以下の整数）
- **日付**: 作業を行った日付（YYYY-MM-DD形式）

---

## Requirements

### 要件1: 作業記録の登録

**ユーザーストーリー:** 開発者として、作業内容・作業時間・日付を入力して作業記録を登録したい。そうすることで、日々の作業をアプリ上で管理できる。

#### 受け入れ基準

1. THE LogForm SHALL 作業内容・作業時間・日付の3つの入力フィールドを表示する。
2. WHEN ユーザーが作業内容・作業時間・日付をすべて入力して送信操作を行ったとき、THE App SHALL 新しい WorkLog を作成して Storage に保存する。
3. WHEN Storage への保存が成功したとき、THE App SHALL LogList を最新の状態に更新する。
4. WHEN 送信が完了したとき、THE LogForm SHALL すべての入力フィールドを空の状態にリセットする。
5. IF 作業内容が空のとき、THEN THE LogForm SHALL「作業内容を入力してください」というエラーメッセージを表示し、送信を中止する。
6. IF 作業内容が200文字を超えるとき、THEN THE LogForm SHALL「作業内容は200文字以内で入力してください」というエラーメッセージを表示し、送信を中止する。
7. IF 作業時間が1未満または480を超える整数でないとき、THEN THE LogForm SHALL「作業時間は1〜480分の整数で入力してください」というエラーメッセージを表示し、送信を中止する。
8. IF 日付が空のとき、THEN THE LogForm SHALL「日付を入力してください」というエラーメッセージを表示し、送信を中止する。
9. IF 日付がYYYY-MM-DD形式でないか実在しない日付のとき、THEN THE LogForm SHALL「有効な日付を入力してください」というエラーメッセージを表示し、送信を中止する。
10. IF Storage への保存が失敗したとき、THEN THE App SHALL「保存に失敗しました。もう一度お試しください」というエラーメッセージを表示し、入力内容を保持する。

---

### 要件2: 作業記録の一覧表示

**ユーザーストーリー:** 開発者として、登録した作業記録を一覧で確認したい。そうすることで、過去の作業履歴を把握できる。

#### 受け入れ基準

1. WHEN App が Storage からの WorkLog 読み込みを完了したとき、THE LogList SHALL Storage に保存されているすべての WorkLog を表示する。
2. WHILE WorkLog が1件以上 Storage に存在するとき、THE LogList SHALL 各 WorkLog の作業内容・作業時間（分単位の整数、例: 90）・日付（YYYY-MM-DD形式）を1行にまとめて表示する。
3. WHILE Storage に WorkLog が0件のとき、THE LogList SHALL「作業記録はまだありません」というメッセージを表示する。
4. THE LogList SHALL WorkLog を登録日時の降順（新しい順）で表示する。
5. WHEN App が起動したとき、THE App SHALL Storage から WorkLog を読み込み、LogList を初期化する。
6. IF App の起動時に Storage からの WorkLog 読み込みが失敗したとき、THEN THE App SHALL Storage が利用できないことを示すエラーメッセージを LogList の領域に表示し、WorkLog の一覧表示を行わない。

---

### 要件3: 作業記録の削除

**ユーザーストーリー:** 開発者として、不要な作業記録を削除したい。そうすることで、一覧を整理できる。

#### 受け入れ基準

1. THE LogList SHALL 各 WorkLog エントリーに対して削除ボタンを1つ表示する。
2. WHEN ユーザーが削除ボタンをクリックしたとき、THE App SHALL 対象の WorkLog を Storage から削除する。
3. WHEN Storage からの削除が成功したとき、THE App SHALL LogList を最新の状態に更新する。
4. IF Storage からの削除が失敗したとき、THEN THE App SHALL 削除失敗を示すエラーメッセージを表示し、対象の WorkLog を削除前の状態のまま保持する。
5. IF Storage 内の WorkLog が0件のとき、THEN THE LogList SHALL「作業記録はまだありません」というメッセージを表示する。

---

### 要件4: データの永続化

**ユーザーストーリー:** 開発者として、ページをリロードしても作業記録が消えないようにしたい。そうすることで、ブラウザを閉じても記録を維持できる。

#### 受け入れ基準

1. WHEN WorkLog が作成されたとき、THE Storage SHALL WorkLog をブラウザのlocalStorageにJSON形式でシリアライズして保存する。
2. WHEN WorkLog が削除されたとき、THE Storage SHALL 更新後のWorkLogリストをブラウザのlocalStorageにJSON形式でシリアライズして保存する。
3. WHEN App が起動したとき、THE Storage SHALL localStorageからWorkLogリストをデシリアライズして読み込み、UIに反映する。
4. THE Storage SHALL 任意の有効なWorkLogリストについて、シリアライズしてからデシリアライズした結果が元のリストと同一件数・同一フィールド値で等価であることを保証する（ラウンドトリップ特性）。
5. IF localStorageのデータが破損または不正なJSON形式のとき、THEN THE Storage SHALL 空のWorkLogリストを返し、エラーの内容を示すメッセージをコンソールに出力する。
6. IF localStorageへの書き込みが容量超過などで失敗したとき、THEN THE Storage SHALL 呼び出し元に失敗を通知し、既存データを変更しない。

---

### 要件5: UIのアクセシビリティとレスポンシブ対応

**ユーザーストーリー:** 開発者として、様々なデバイスで快適にアプリを使いたい。そうすることで、PCでもスマートフォンでも作業記録を管理できる。

#### 受け入れ基準

1. THE App SHALL ビューポート幅375px以上のPCブラウザおよびスマートフォンブラウザで水平スクロールなしに正常に表示・操作できるレスポンシブレイアウトを提供する。
2. THE LogForm SHALL 各入力フィールドに対応するlabelを持ち、`for`属性と`id`属性を一致させてスクリーンリーダーが読み取れる形式で関連付ける。
3. WHEN ユーザーがキーボードのみで操作するとき、THE App SHALL Tabキーによるフォーカス移動とEnterキーによる実行で、すべての操作（入力・送信・削除）をキーボードで完結できるフォーカス順序を維持する。
4. THE App SHALL フォーカスを受けた要素に視覚的なフォーカスインジケーター（アウトラインなど）を表示する。
