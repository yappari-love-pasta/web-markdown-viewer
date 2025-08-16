```mermaid
sequenceDiagram
    participant CloudWatch
    participant AutoScalingGroup
    participant EC2
    participant ELB

    Note over CloudWatch: スケーリングポリシーに従い<br>CPU使用率などを監視

    CloudWatch->>AutoScalingGroup: アラーム発火（例: CPU使用率80%以上）
    AutoScalingGroup->>AutoScalingGroup: スケールアウト判定
    AutoScalingGroup->>EC2: 新しいEC2インスタンスを起動
    EC2->>ELB: ヘルスチェックに登録
    ELB->>EC2: ヘルスチェック通過
    ELB->>AutoScalingGroup: インスタンス登録完了通知
    Note over AutoScalingGroup,ELB: トラフィックが新規インスタンスに分散される
