# Dify音声再生プレイヤーシステム開発指示書

## プロジェクト概要

講演音声ファイルのトランスクリプトを活用したDifyチャットフローと連携する音声再生プレイヤーシステムを構築します。DjangoをバックエンドとしてWaveSurfer.jsによる音声波形表示機能を持つプレイヤーを作成し、DifyチャットフローのAPI機能により、チャット応答とともに時間範囲での自動頭出し・ループ再生機能を提供します。

## システム要件

### 技術スタック
- **バックエンド**: Django 4.x + Django REST Framework
- **フロントエンド**: HTML5 + JavaScript + WaveSurfer.js
- **データベース**: SQLite（開発用）/ PostgreSQL（本番用）
- **外部API**: Dify Chat API

### 主要機能
1. 音声ファイル管理・再生
2. WaveSurfer.jsによる波形表示
3. Difyチャットフローとの連携
4. タイムスタンプ解析による自動頭出し
5. 指定区間のループ再生

## DifyチャットフローAPI仕様

### ベースURL
```
https://djartipy.com/v1
```

### 認証
```
Authorization: Bearer {API_KEY}
```

### 主要エンドポイント
- `POST /chat-messages`: チャットメッセージ送信
- `GET /messages`: 会話履歴取得
- `GET /conversations`: 会話一覧取得

### 応答形式
チャット応答には参考箇所として `[分:秒-分:秒]` 形式のタイムスタンプが含まれます。

## 開発要件

### 1. Djangoプロジェクト構造

```
audio_player_project/
├── manage.py
├── requirements.txt
├── audio_player_project/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── apps/
│   ├── audio/
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── serializers.py
│   │   └── services.py
│   ├── chat/
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── serializers.py
│   │   └── dify_client.py
├── static/
│   ├── js/
│   ├── css/
│   └── audio/
└── templates/
    └── player/
```

### 2. モデル設計

#### AudioFile モデル
```python
class AudioFile(models.Model):
    title = models.CharField(max_length=255)
    file_path = models.FileField(upload_to='audio/')
    duration = models.FloatField()  # 秒
    transcript = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

#### ChatSession モデル
```python
class ChatSession(models.Model):
    audio_file = models.ForeignKey(AudioFile, on_delete=models.CASCADE)
    dify_conversation_id = models.CharField(max_length=255, blank=True)
    user_identifier = models.CharField(max_length=255)
    created_at = models.DateTimeField(auto_now_add=True)
```

#### ChatMessage モデル
```python
class ChatMessage(models.Model):
    session = models.ForeignKey(ChatSession, on_delete=models.CASCADE)
    user_message = models.TextField()
    ai_response = models.TextField()
    timestamp_start = models.FloatField(null=True, blank=True)  # 秒
    timestamp_end = models.FloatField(null=True, blank=True)    # 秒
    dify_message_id = models.CharField(max_length=255, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
```

### 3. DifyAPI連携クライアント

#### dify_client.py
```python
import requests
import re
from typing import Dict, Optional, Tuple

class DifyClient:
    def __init__(self, api_key: str, base_url: str = "https://djartipy.com/v1"):
        self.api_key = api_key
        self.base_url = base_url
        self.headers = {
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        }
    
    def send_message(self, query: str, user: str, conversation_id: str = "") -> Dict:
        """Difyにメッセージを送信"""
        # 実装要件：ストリーミングモードでメッセージ送信
        
    def extract_timestamp(self, response_text: str) -> Optional[Tuple[float, float]]:
        """応答からタイムスタンプを抽出 [分:秒-分:秒] 形式"""
        # 実装要件：正規表現でタイムスタンプを抽出し、秒に変換
        
    def parse_streaming_response(self, response) -> Dict:
        """ストリーミング応答を解析"""
        # 実装要件：SSEストリーミング応答を解析
```

### 4. API エンドポイント

#### AudioFile API
- `GET /api/audio/files/` - 音声ファイル一覧
- `POST /api/audio/files/` - 音声ファイルアップロード
- `GET /api/audio/files/{id}/` - 音声ファイル詳細

#### Chat API
- `POST /api/chat/sessions/` - チャットセッション作成
- `POST /api/chat/sessions/{id}/messages/` - メッセージ送信
- `GET /api/chat/sessions/{id}/messages/` - メッセージ履歴

#### Player API
- `POST /api/player/seek/` - 指定時刻への移動
- `POST /api/player/loop/` - ループ再生設定

### 5. フロントエンド要件

#### WaveSurfer.js設定
```javascript
const wavesurfer = WaveSurfer.create({
    container: '#waveform',
    waveColor: '#4A90E2',
    progressColor: '#1E3A8A',
    backend: 'WebAudio',
    height: 100,
    pixelRatio: 1,
    normalize: true,
    responsive: true
});
```

#### 主要機能実装
1. **音声波形表示**: WaveSurfer.jsで音声ファイルの波形を表示
2. **再生コントロール**: 再生/停止/シークバー
3. **チャットインターフェース**: Difyとのチャット機能
4. **タイムスタンプ連動**: チャット応答から抽出したタイムスタンプで自動頭出し
5. **ループ再生**: 指定区間の繰り返し再生
6. **リアルタイム更新**: WebSocketまたはポーリングでチャット状態更新

### 6. 環境設定

#### requirements.txt
```
Django==4.2.7
djangorestframework==3.14.0
django-cors-headers==4.3.1
requests==2.31.0
python-decouple==3.8
Pillow==10.0.1
librosa==0.10.1  # 音声解析用
pydub==0.25.1    # 音声処理用
```

#### settings.py 設定
```python
# Dify API設定
DIFY_API_KEY = config('DIFY_API_KEY')
DIFY_BASE_URL = config('DIFY_BASE_URL', default='https://djartipy.com/v1')

# CORS設定
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
    "http://127.0.0.1:3000",
]

# メディアファイル設定
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# 音声ファイル設定
AUDIO_UPLOAD_MAX_SIZE = 100 * 1024 * 1024  # 100MB
ALLOWED_AUDIO_FORMATS = ['mp3', 'wav', 'mp4', 'm4a']
```

### 7. 実装の優先順位

#### Phase 1: 基本機能
1. Djangoプロジェクト初期化
2. モデル作成・マイグレーション
3. 音声ファイルアップロード機能
4. WaveSurfer.js基本再生機能

#### Phase 2: Dify連携
1. DifyAPIクライアント実装
2. チャット機能実装
3. タイムスタンプ抽出機能
4. 自動頭出し機能

#### Phase 3: 高度な機能
1. ループ再生機能
2. チャット履歴管理
3. UI/UX改善
4. エラーハンドリング強化

### 8. テスト要件

#### 単体テスト
- DifyAPIクライアントのテスト
- タイムスタンプ抽出のテスト
- 音声ファイル処理のテスト

#### 統合テスト
- Dify API連携のエンドツーエンドテスト
- フロントエンド・バックエンド連携テスト

### 9. セキュリティ考慮事項

1. **API キー管理**: 環境変数での安全な管理
2. **ファイルアップロード**: ファイル形式・サイズ制限
3. **CSRF保護**: Django標準のCSRF保護
4. **入力検証**: ユーザー入力の適切な検証

### 10. パフォーマンス最適化

1. **音声ファイル圧縮**: 適切な音質での圧縮
2. **キャッシュ戦略**: 頻繁にアクセスされるデータのキャッシュ
3. **非同期処理**: 重い処理の非同期化
4. **CDN活用**: 静的ファイルの配信最適化

## 開発開始手順

1. **プロジェクト初期化**
   ```bash
   django-admin startproject audio_player_project
   cd audio_player_project
   python manage.py startapp audio
   python manage.py startapp chat
   ```

2. **依存関係インストール**
   ```bash
   pip install -r requirements.txt
   ```

3. **データベース設定**
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

4. **開発サーバー起動**
   ```bash
   python manage.py runserver
   ```

## 成果物

- 完全に動作するDjangoアプリケーション
- WaveSurfer.jsによる音声プレイヤー
- DifyチャットフローAPI連携機能
- タイムスタンプ解析・自動頭出し機能
- ループ再生機能
- 包括的なドキュメント

このシステムにより、講演音声の内容に基づいた対話的な学習体験を提供し、ユーザーは関連する音声箇所を即座に確認できるようになります。