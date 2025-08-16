```mermaid
sequenceDiagram
autonumber
title DNSキャッシュポイズニングのシーケンス（概略）

participant User as ユーザー(クライアント)
participant Rec as リカーシブDNS(キャッシュ)
participant Auth as 正規権威DNS
participant Att as 攻撃者
participant Fake as 偽権威DNS(攻撃者が偽装)

note over User,Rec: ① 被害者の利用開始（例: example.com の A レコードを引きたい）
User->>Rec: example.com の A レコードを問い合わせ
Rec-->>User: キャッシュ未ヒット（内部処理）

note over Rec,Auth: ② 再帰問い合わせのトリガー（未キャッシュ）
Rec->>Auth: example.com の問い合わせ（TXID & 送信ポートを含む）

note over Att,Rec: ③ 攻撃者がレースを開始（大量の偽応答をばら撒く）
Att-)+Rec: 偽の権威応答を多数送信<br/>（TXID/ポートを総当たり、<br/>example.com の権威NSやAを偽装）
Att--)Rec: （正規応答到着より先に届けば勝ち）

par 正規応答
    Auth-->>Rec: 正規の権威応答（本物のNS/A）
and 偽応答の雨
    Att-->>Rec: 偽の権威応答（TXID一致を狙う）
end

alt 偽応答が先に一致
    note over Rec: ④ キャッシュ汚染発生<br/>・NSを攻撃者のドメインに差し替え<br/>・Aレコードを悪性IPに設定<br/>・TTLも悪性にキャッシュ
    Rec-->>User: 「example.com = 悪性IP」を応答（キャッシュ済み）
else 正規応答が先に到着
    note over Rec: 失敗（今回の試行は不発）<br/>→ 攻撃者はサブドメインを変えて再試行（例: a1.example.com など）
    Rec-->>User: 正規の応答
end


note over User,Rec: ⑤ 後続利用者が同名/同ゾーンを引く
User->>Rec: example.com（または *.example.com）を問い合わせ
Rec-->>User: キャッシュ済みの偽A/偽NSを返す

note over User,Fake: ⑥ 被害成立（フィッシング/マルウェア配布等）
User->>Fake: 悪性IPへ接続（HTTPSでも偽証明書/不一致で警告の恐れ）
Fake-->>User: 偽サイト/悪性コンテンツ提供

%% --- 補足: 代表的な対策（参考メモ） ---
note over Rec,Auth: 対策メモ：<br/>・ソースポートランダム化<br/>・TXIDの十分な乱数性<br/>・0x20 大文字小文字ミキシング<br/>・DNSSEC 検証で権威応答の完全性を保証<br/>・短TTL/信頼できる上流・レート制限
