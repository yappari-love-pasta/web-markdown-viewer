```mermaid
sequenceDiagram
    participant User as ユーザー
    participant Browser as ブラウザ
    participant CAPTCHA as CAPTCHAサーバー（例: Google reCAPTCHA）
    participant Backend as バックエンドサーバー

    User->>Browser: Webページを開く
    Browser->>CAPTCHA: CAPTCHAスクリプトを読み込む
    CAPTCHA-->>Browser: CAPTCHAチャレンジを提供
    User->>Browser: CAPTCHAを解く
    Browser->>CAPTCHA: ユーザーの解答を送信
    CAPTCHA-->>Browser: 検証トークン（captcha_token）を返す
    Browser->>Backend: フォームデータ + captcha_tokenを送信
    Backend->>CAPTCHA: captcha_tokenの検証リクエスト
    CAPTCHA-->>Backend: 検証結果（成功/失敗）を返す
    Backend-->>Browser: フォーム処理結果を返す

