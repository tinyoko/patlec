# タスク分割表 - Dify音声再生プレイヤーシステム

## 1. プロジェクト概要

### 1.1 開発フロー
```
Phase 0: 環境構築 → Phase 1: 基本機能 → Phase 2: Dify連携 → Phase 3: 高度な機能 → Phase 4: テスト・デプロイ
```

### 1.2 作業期間見積もり
- **Phase 0**: 1日
- **Phase 1**: 5-7日
- **Phase 2**: 5-7日
- **Phase 3**: 3-5日
- **Phase 4**: 2-3日
- **合計**: 16-23日

## 2. Phase 0: 環境構築・初期設定

### 2.1 開発環境セットアップ
| タスクID | タスク名 | 内容 | 優先度 | 工数 | 依存関係 |
|---------|---------|------|-------|------|---------|
| ENV-001 | Python仮想環境作成 | venvで仮想環境を作成し有効化 | 高 | 0.5h | - |
| ENV-002 | Gitリポジトリ初期化 | GitHubリポジトリ作成・clone | 高 | 0.5h | - |
| ENV-003 | .gitignore設定 | Python/Django用.gitignore作成 | 高 | 0.5h | ENV-002 |
| ENV-004 | 環境変数テンプレート作成 | .env.example作成 | 高 | 0.5h | ENV-003 |
| ENV-005 | requirements.txt作成 | 依存関係リスト作成 | 高 | 0.5h | ENV-001 |

**チェックリスト**:
- [ ] `python -m venv venv` 実行
- [ ] `source venv/bin/activate` で仮想環境有効化
- [ ] GitHubリポジトリ作成・clone
- [ ] .gitignoreファイル作成（venv/, .env, __pycache__/等）
- [ ] .env.exampleファイル作成
- [ ] requirements.txtファイル作成

### 2.2 Djangoプロジェクト初期化
| タスクID | タスク名 | 内容 | 優先度 | 工数 | 依存関係 |
|---------|---------|------|-------|------|---------|
| ENV-006 | Django関連パッケージインストール | pip install Django DRF等 | 高 | 1h | ENV-005 |
| ENV-007 | Djangoプロジェクト作成 | django-admin startproject実行 | 高 | 0.5h | ENV-006 |
| ENV-008 | Djangoアプリ作成 | audio, chatアプリ作成 | 高 | 0.5h | ENV-007 |
| ENV-009 | settings.py基本設定 | 基本設定・アプリ登録 | 高 | 1h | ENV-008 |
| ENV-010 | 環境変数設定 | .envファイル作成・設定 | 高 | 1h | ENV-004 |

**チェックリスト**:
- [ ] `pip install -r requirements.txt` 実行
- [ ] `django-admin startproject audio_player_project` 実行
- [ ] `python manage.py startapp audio` 実行
- [ ] `python manage.py startapp chat` 実行
- [ ] settings.pyにアプリ追加・基本設定
- [ ] .envファイル作成・APIキー等設定

## 3. Phase 1: 基本機能開発

### 3.1 データベース設計・実装
| タスクID | タスク名 | 内容 | 優先度 | 工数 | 依存関係 |
|---------|---------|------|-------|------|---------|
| DB-001 | AudioFileモデル作成 | 音声ファイル管理モデル | 高 | 2h | ENV-010 |
| DB-002 | ChatSessionモデル作成 | チャットセッション管理モデル | 高 | 1.5h | DB-001 |
| DB-003 | ChatMessageモデル作成 | チャットメッセージ管理モデル | 高 | 2h | DB-002 |
| DB-004 | マイグレーション実行 | データベース初期化 | 高 | 0.5h | DB-003 |
| DB-005 | Django Admin設定 | 管理画面設定 | 中 | 1h | DB-004 |

**実装内容**:
```python
# apps/audio/models.py
class AudioFile(models.Model):
    title = models.CharField(max_length=255)
    file_path = models.FileField(upload_to='audio/')
    duration = models.FloatField()
    transcript = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

# apps/chat/models.py
class ChatSession(models.Model):
    audio_file = models.ForeignKey(AudioFile, on_delete=models.CASCADE)
    dify_conversation_id = models.CharField(max_length=255, blank=True)
    user_identifier = models.CharField(max_length=255)
    created_at = models.DateTimeField(auto_now_add=True)

class ChatMessage(models.Model):
    session = models.ForeignKey(ChatSession, on_delete=models.CASCADE)
    user_message = models.TextField()
    ai_response = models.TextField()
    timestamp_start = models.FloatField(null=True, blank=True)
    timestamp_end = models.FloatField(null=True, blank=True)
    dify_message_id = models.CharField(max_length=255, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
```

### 3.2 REST API実装
| タスクID | タスク名 | 内容 | 優先度 | 工数 | 依存関係 |
|---------|---------|------|-------|------|---------|
| API-001 | DRFシリアライザー作成 | 各モデルのシリアライザー | 高 | 2h | DB-005 |
| API-002 | 音声ファイルAPI実装 | CRUD操作API | 高 | 3h | API-001 |
| API-003 | チャットセッションAPI実装 | セッション管理API | 高 | 2h | API-002 |
| API-004 | チャットメッセージAPI実装 | メッセージ管理API | 高 | 3h | API-003 |
| API-005 | URL設定 | ルーティング設定 | 高 | 1h | API-004 |

**実装内容**:
```python
# apps/audio/serializers.py
class AudioFileSerializer(serializers.ModelSerializer):
    class Meta:
        model = AudioFile
        fields = '__all__'

# apps/audio/views.py
class AudioFileViewSet(viewsets.ModelViewSet):
    queryset = AudioFile.objects.all()
    serializer_class = AudioFileSerializer
```

### 3.3 フロントエンド基本実装
| タスクID | タスク名 | 内容 | 優先度 | 工数 | 依存関係 |
|---------|---------|------|-------|------|---------|
| FE-001 | HTMLテンプレート作成 | 基本レイアウト・ページ構成 | 高 | 2h | API-005 |
| FE-002 | CSS基本スタイル作成 | レスポンシブデザイン | 高 | 3h | FE-001 |
| FE-003 | WaveSurfer.js統合 | 音声波形表示機能 | 高 | 4h | FE-002 |
| FE-004 | 基本JavaScript実装 | API通信・DOM操作 | 高 | 3h | FE-003 |
| FE-005 | ファイルアップロード機能 | 音声ファイルアップロード | 高 | 2h | FE-004 |

**実装内容**:
```javascript
// static/js/audio_player.js
class AudioPlayer {
    constructor(containerId) {
        this.wavesurfer = WaveSurfer.create({
            container: containerId,
            waveColor: '#4A90E2',
            progressColor: '#1E3A8A',
            height: 100,
            responsive: true
        });
    }
    
    loadAudio(audioUrl) {
        this.wavesurfer.load(audioUrl);
    }
}
```

## 4. Phase 2: Dify連携機能開発

### 4.1 Dify APIクライアント実装
| タスクID | タスク名 | 内容 | 優先度 | 工数 | 依存関係 |
|---------|---------|------|-------|------|---------|
| DIFY-001 | DifyClientクラス実装 | 基本API通信クラス | 高 | 3h | API-005 |
| DIFY-002 | ストリーミング応答処理 | SSE応答解析機能 | 高 | 4h | DIFY-001 |
| DIFY-003 | 認証・ヘッダー管理 | APIキー・認証処理 | 高 | 2h | DIFY-002 |
| DIFY-004 | エラーハンドリング | 例外処理・リトライ機能 | 高 | 3h | DIFY-003 |
| DIFY-005 | 会話管理機能 | 会話ID管理・履歴処理 | 中 | 2h | DIFY-004 |

**実装内容**:
```python
# apps/chat/dify_client.py
class DifyClient:
    def __init__(self, api_key: str, base_url: str = "https://djartipy.com/v1"):
        self.api_key = api_key
        self.base_url = base_url
        self.headers = {
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        }
    
    def send_message(self, query: str, user: str, conversation_id: str = "") -> Dict:
        """ストリーミングモードでメッセージ送信"""
        url = f"{self.base_url}/chat-messages"
        data = {
            "inputs": {},
            "query": query,
            "response_mode": "streaming",
            "conversation_id": conversation_id,
            "user": user
        }
        
        response = requests.post(url, headers=self.headers, json=data, stream=True)
        return self.parse_streaming_response(response)
```

### 4.2 タイムスタンプ解析機能
| タスクID | タスク名 | 内容 | 優先度 | 工数 | 依存関係 |
|---------|---------|------|-------|------|---------|
| TS-001 | 正規表現パターン作成 | [分:秒-分:秒]抽出パターン | 高 | 2h | DIFY-005 |
| TS-002 | タイムスタンプ変換機能 | 分:秒→秒変換機能 | 高 | 1.5h | TS-001 |
| TS-003 | 複数タイムスタンプ対応 | 複数区間の処理 | 中 | 2h | TS-002 |
| TS-004 | バリデーション機能 | タイムスタンプ妥当性検証 | 中 | 1.5h | TS-003 |
| TS-005 | 単体テスト作成 | タイムスタンプ解析テスト | 高 | 2h | TS-004 |

**実装内容**:
```python
import re
from typing import Optional, Tuple, List

def extract_timestamp(response_text: str) -> Optional[Tuple[float, float]]:
    """AI応答からタイムスタンプを抽出 [分:秒-分:秒] 形式"""
    pattern = r'\[(\d+):(\d+)-(\d+):(\d+)\]'
    match = re.search(pattern, response_text)
    
    if match:
        start_min, start_sec, end_min, end_sec = map(int, match.groups())
        start_time = start_min * 60 + start_sec
        end_time = end_min * 60 + end_sec
        return (start_time, end_time)
    
    return None
```

### 4.3 チャットインターフェース実装
| タスクID | タスク名 | 内容 | 優先度 | 工数 | 依存関係 |
|---------|---------|------|-------|------|---------|
| CHAT-001 | チャットUI作成 | メッセージ表示・入力UI | 高 | 3h | TS-005 |
| CHAT-002 | リアルタイム通信 | WebSocket/ポーリング実装 | 高 | 4h | CHAT-001 |
| CHAT-003 | メッセージ履歴表示 | 過去メッセージ表示機能 | 中 | 2h | CHAT-002 |
| CHAT-004 | ストリーミング表示 | タイプライター効果 | 中 | 3h | CHAT-003 |
| CHAT-005 | エラー表示処理 | 通信エラー・API制限表示 | 中 | 2h | CHAT-004 |

## 5. Phase 3: 高度な機能開発

### 5.1 自動頭出し・ループ機能
| タスクID | タスク名 | 内容 | 優先度 | 工数 | 依存関係 |
|---------|---------|------|-------|------|---------|
| SEEK-001 | 自動シーク機能 | タイムスタンプ位置への移動 | 高 | 2h | CHAT-005 |
| SEEK-002 | ループ再生機能 | 指定区間の繰り返し再生 | 高 | 3h | SEEK-001 |
| SEEK-003 | ループ制御UI | ループ設定・解除UI | 中 | 2h | SEEK-002 |
| SEEK-004 | 手動区間設定 | ユーザー任意区間設定 | 中 | 3h | SEEK-003 |
| SEEK-005 | ブックマーク機能 | 重要箇所のマーク・管理 | 低 | 2h | SEEK-004 |

### 5.2 ユーザビリティ向上
| タスクID | タスク名 | 内容 | 優先度 | 工数 | 依存関係 |
|---------|---------|------|-------|------|---------|
| UX-001 | キーボードショートカット | スペース再生等のショートカット | 中 | 2h | SEEK-005 |
| UX-002 | 再生速度調整 | 0.5x-2x速度調整機能 | 中 | 1.5h | UX-001 |
| UX-003 | 音量調整 | 音量スライダー・ミュート | 中 | 1h | UX-002 |
| UX-004 | プログレスバー改善 | 時間表示・サムネイル | 低 | 2h | UX-003 |
| UX-005 | ローディング状態表示 | 処理中の視覚的フィードバック | 中 | 1.5h | UX-004 |

### 5.3 データ管理機能
| タスクID | タスク名 | 内容 | 優先度 | 工数 | 依存関係 |
|---------|---------|------|-------|------|---------|
| DATA-001 | 履歴エクスポート | チャット履歴のJSON出力 | 低 | 2h | UX-005 |
| DATA-002 | 学習進捗記録 | 視聴時間・完了率記録 | 低 | 3h | DATA-001 |
| DATA-003 | 検索機能 | 履歴・トランスクリプト検索 | 低 | 3h | DATA-002 |
| DATA-004 | タグ機能 | 音声ファイルのタグ付け | 低 | 2h | DATA-003 |
| DATA-005 | 統計ダッシュボード | 利用統計の可視化 | 低 | 4h | DATA-004 |

## 6. Phase 4: テスト・デプロイ・ドキュメント

### 6.1 テスト実装
| タスクID | タスク名 | 内容 | 優先度 | 工数 | 依存関係 |
|---------|---------|------|-------|------|---------|
| TEST-001 | 単体テスト作成 | モデル・API・関数テスト | 高 | 4h | DATA-005 |
| TEST-002 | 統合テスト作成 | API連携・シナリオテスト | 高 | 3h | TEST-001 |
| TEST-003 | フロントエンドテスト | JavaScript機能テスト | 中 | 3h | TEST-002 |
| TEST-004 | E2Eテスト作成 | ユーザーシナリオテスト | 中 | 4h | TEST-003 |
| TEST-005 | パフォーマンステスト | 負荷・レスポンステスト | 低 | 2h | TEST-004 |

### 6.2 品質向上・リファクタリング
| タスクID | タスク名 | 内容 | 優先度 | 工数 | 依存関係 |
|---------|---------|------|-------|------|---------|
| QA-001 | コードレビュー | 品質チェック・改善 | 高 | 3h | TEST-005 |
| QA-002 | セキュリティ監査 | 脆弱性チェック・修正 | 高 | 2h | QA-001 |
| QA-003 | パフォーマンス最適化 | 速度改善・メモリ最適化 | 中 | 3h | QA-002 |
| QA-004 | エラーハンドリング強化 | 例外処理・ログ改善 | 中 | 2h | QA-003 |
| QA-005 | UIデザイン調整 | 見た目・操作性改善 | 低 | 2h | QA-004 |

### 6.3 ドキュメント・デプロイ
| タスクID | タスク名 | 内容 | 優先度 | 工数 | 依存関係 |
|---------|---------|------|-------|------|---------|
| DOC-001 | README.md作成 | セットアップ・使用方法 | 高 | 2h | QA-005 |
| DOC-002 | API仕様書作成 | OpenAPI形式の仕様書 | 中 | 2h | DOC-001 |
| DOC-003 | ユーザーマニュアル | 操作手順・FAQ | 中 | 3h | DOC-002 |
| DOC-004 | 開発者ガイド | 拡張・カスタマイズ手順 | 低 | 2h | DOC-003 |
| DOC-005 | デプロイ設定 | 本番環境設定・CI/CD | 低 | 3h | DOC-004 |

## 7. 依存関係マップ

### 7.1 クリティカルパス
```
ENV-001 → ENV-006 → ENV-007 → ENV-008 → ENV-009 → ENV-010
    ↓
DB-001 → DB-002 → DB-003 → DB-004 → API-001 → API-002
    ↓
DIFY-001 → DIFY-002 → TS-001 → TS-002 → CHAT-001 → SEEK-001
    ↓
TEST-001 → TEST-002 → QA-001 → DOC-001
```

### 7.2 並行実行可能タスク
- **フロントエンド**: FE-001〜FE-005（API完成後）
- **テスト**: TEST-003〜TEST-005（機能完成後）
- **ドキュメント**: DOC-002〜DOC-004（機能完成後）

## 8. リスク管理

### 8.1 技術リスク
| リスク | 影響度 | 発生確率 | 対策 |
|-------|-------|---------|------|
| Dify API制限・変更 | 高 | 中 | モックAPI準備・代替プラン |
| 大容量音声ファイル処理 | 中 | 高 | ストリーミング・分割処理 |
| ブラウザ互換性問題 | 中 | 低 | クロスブラウザテスト |
| WebAudio API制限 | 低 | 低 | フォールバック実装 |

### 8.2 スケジュールリスク
| リスク | 影響度 | 発生確率 | 対策 |
|-------|-------|---------|------|
| 機能要件の追加・変更 | 高 | 中 | スコープ管理・優先度明確化 |
| 技術習得時間超過 | 中 | 中 | 事前調査・PoC実装 |
| テスト工数不足 | 中 | 高 | 早期テスト開始・自動化 |

## 9. 品質管理

### 9.1 完了条件
各タスクの完了条件：
- [ ] 実装完了
- [ ] 単体テスト通過
- [ ] コードレビュー完了
- [ ] ドキュメント更新

### 9.2 マイルストーン
1. **Phase 0完了**: 開発環境構築完了
2. **Phase 1完了**: 基本機能動作確認
3. **Phase 2完了**: Dify連携・自動頭出し動作確認
4. **Phase 3完了**: 全機能統合テスト完了
5. **プロジェクト完了**: 本番デプロイ・運用開始

## 10. チーム開発ガイドライン

### 10.1 ブランチ戦略
```
main ← develop ← feature/task-id-description
```

### 10.2 コミット規則
```
feat: 新機能追加
fix: バグ修正
docs: ドキュメント更新
refactor: リファクタリング
test: テスト追加
chore: その他の変更
```

### 10.3 プルリクエスト手順
1. feature ブランチで実装
2. テスト実行・通過確認
3. プルリクエスト作成
4. コードレビュー
5. develop ブランチにマージ

この詳細なタスク分割により、効率的で品質の高い開発が可能になります。