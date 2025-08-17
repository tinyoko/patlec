# 設計書 - Dify音声再生プレイヤーシステム

## 1. システムアーキテクチャ

### 1.1 全体アーキテクチャ
```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   フロントエンド     │◄──►│    バックエンド       │◄──►│   外部API (Dify)  │
│                 │    │                  │    │                 │
│ • HTML5         │    │ • Django 4.x     │    │ • Chat API      │
│ • JavaScript    │    │ • Django REST    │    │ • Knowledge     │
│ • WaveSurfer.js │    │   Framework      │    │   Retrieval     │
│ • CSS3          │    │ • SQLite/        │    │ • Streaming     │
│                 │    │   PostgreSQL     │    │   Response      │
└─────────────────┘    └──────────────────┘    └─────────────────┘
        │                        │
        └────────────────────────┘
              WebSocket/HTTP
```

### 1.2 レイヤーアーキテクチャ
```
┌─────────────────────────────────────────────────────────┐
│                  プレゼンテーション層                        │
│  • HTML/CSS/JavaScript • WaveSurfer.js • AJAX通信      │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│                     API層                               │
│  • Django REST Framework • シリアライザー • 認証        │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│                   ビジネスロジック層                        │
│  • Difyクライアント • タイムスタンプ解析 • 音声処理       │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│                   データアクセス層                          │
│  • Django ORM • モデル • マイグレーション                │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│                   データ層                               │
│  • SQLite (開発) • PostgreSQL (本番) • ファイルストレージ │
└─────────────────────────────────────────────────────────┘
```

## 2. 技術スタック

### 2.1 開発環境
| カテゴリ | 技術・ツール | バージョン | 用途 |
|---------|-------------|-----------|------|
| OS | macOS | 11.0+ | 開発環境 |
| Python | Python | 3.9+ | バックエンド開発 |
| 仮想環境 | venv | 標準ライブラリ | 依存関係分離 |
| バージョン管理 | Git | 2.30+ | ソースコード管理 |
| リポジトリ | GitHub | - | チーム開発・CI/CD |

### 2.2 バックエンド
| カテゴリ | 技術・ライブラリ | バージョン | 用途 |
|---------|-----------------|-----------|------|
| Webフレームワーク | Django | 4.2.7 | メインフレームワーク |
| API | Django REST Framework | 3.14.0 | REST API構築 |
| CORS | django-cors-headers | 4.3.1 | フロントエンド連携 |
| HTTP通信 | requests | 2.31.0 | Dify API連携 |
| 設定管理 | python-decouple | 3.8 | 環境変数管理 |
| 画像処理 | Pillow | 10.0.1 | 画像ファイル処理 |
| 音声解析 | librosa | 0.10.1 | 音声解析・特徴抽出 |
| 音声処理 | pydub | 0.25.1 | 音声ファイル操作 |

### 2.3 フロントエンド
| カテゴリ | 技術・ライブラリ | バージョン | 用途 |
|---------|-----------------|-----------|------|
| マークアップ | HTML5 | - | 基本構造 |
| スタイル | CSS3 | - | デザイン・レスポンシブ |
| スクリプト | JavaScript (ES6+) | - | 動的機能 |
| 音声波形 | WaveSurfer.js | 7.0+ | 音声波形表示・操作 |
| HTTP通信 | Fetch API | - | API通信 |

### 2.4 データベース
| 環境 | データベース | 用途 |
|-----|-------------|------|
| 開発 | SQLite | ローカル開発・テスト |
| 本番 | PostgreSQL | 本番環境・スケーラビリティ |

## 3. 環境設定管理

### 3.1 Python仮想環境設定
```bash
# 仮想環境作成
python -m venv venv

# 仮想環境有効化（macOS）
source venv/bin/activate

# 依存関係インストール
pip install -r requirements.txt

# 仮想環境無効化
deactivate
```

### 3.2 環境変数管理（.env）
```env
# Django設定
DEBUG=True
SECRET_KEY=your-secret-key-here
ALLOWED_HOSTS=localhost,127.0.0.1

# データベース設定
DB_ENGINE=django.db.backends.sqlite3
DB_NAME=db.sqlite3

# Dify API設定
DIFY_API_KEY=your-dify-api-key-here
DIFY_BASE_URL=https://djartipy.com/v1

# メディア設定
MEDIA_ROOT=media/
MEDIA_URL=/media/

# 音声ファイル設定
AUDIO_UPLOAD_MAX_SIZE=104857600  # 100MB
ALLOWED_AUDIO_FORMATS=mp3,wav,mp4,m4a
```

### 3.3 Git管理設定（.gitignore）
```gitignore
# Python仮想環境
venv/
env/
.venv/

# 環境設定ファイル
.env
.env.local
.env.production

# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/

# Django
*.log
local_settings.py
db.sqlite3
db.sqlite3-journal

# メディアファイル
media/
static_collected/

# IDE
.vscode/
.idea/
*.swp
*.swo

# macOS
.DS_Store
.AppleDouble
.LSOverride
```

### 3.4 環境設定テンプレート（.env.example）
```env
# Django設定
DEBUG=True
SECRET_KEY=your-secret-key-here
ALLOWED_HOSTS=localhost,127.0.0.1

# データベース設定
DB_ENGINE=django.db.backends.sqlite3
DB_NAME=db.sqlite3

# Dify API設定（要設定）
DIFY_API_KEY=your-dify-api-key-here
DIFY_BASE_URL=https://djartipy.com/v1

# メディア設定
MEDIA_ROOT=media/
MEDIA_URL=/media/

# 音声ファイル設定
AUDIO_UPLOAD_MAX_SIZE=104857600
ALLOWED_AUDIO_FORMATS=mp3,wav,mp4,m4a
```

## 4. プロジェクト構造

### 4.1 ディレクトリ構造
```
audio_player_project/
├── manage.py
├── requirements.txt
├── .env.example
├── .gitignore
├── README.md
├── audio_player_project/          # メインプロジェクト設定
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── apps/                          # アプリケーション
│   ├── __init__.py
│   ├── audio/                     # 音声管理アプリ
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── serializers.py
│   │   ├── services.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   ├── migrations/
│   │   └── tests/
│   └── chat/                      # チャット管理アプリ
│       ├── __init__.py
│       ├── models.py
│       ├── views.py
│       ├── urls.py
│       ├── serializers.py
│       ├── dify_client.py
│       ├── admin.py
│       ├── apps.py
│       ├── migrations/
│       └── tests/
├── static/                        # 静的ファイル
│   ├── js/
│   │   ├── audio_player.js
│   │   ├── chat_interface.js
│   │   └── main.js
│   ├── css/
│   │   ├── main.css
│   │   ├── player.css
│   │   └── chat.css
│   └── audio/                     # 音声ファイル配置
├── templates/                     # テンプレート
│   ├── base.html
│   ├── player/
│   │   ├── index.html
│   │   └── player.html
│   └── chat/
│       └── interface.html
└── media/                         # アップロードファイル
    └── audio/
```

## 5. データベース設計

### 5.1 ER図
```
┌─────────────────┐    1    ∞ ┌─────────────────┐    ∞    1 ┌─────────────────┐
│   AudioFile     │◄─────────►│   ChatSession   │◄─────────►│   ChatMessage   │
│                 │            │                 │            │                 │
│ • id (PK)       │            │ • id (PK)       │            │ • id (PK)       │
│ • title         │            │ • audio_file_id │            │ • session_id    │
│ • file_path     │            │ • dify_conv_id  │            │ • user_message  │
│ • duration      │            │ • user_id       │            │ • ai_response   │
│ • transcript    │            │ • created_at    │            │ • timestamp_s   │
│ • created_at    │            │                 │            │ • timestamp_e   │
│ • updated_at    │            │                 │            │ • dify_msg_id   │
│                 │            │                 │            │ • created_at    │
└─────────────────┘            └─────────────────┘            └─────────────────┘
```

### 5.2 テーブル定義

#### 5.2.1 AudioFile（音声ファイル）
```python
class AudioFile(models.Model):
    id = models.AutoField(primary_key=True)
    title = models.CharField(max_length=255, verbose_name="タイトル")
    file_path = models.FileField(upload_to='audio/', verbose_name="ファイルパス")
    duration = models.FloatField(verbose_name="再生時間（秒）")
    transcript = models.TextField(blank=True, verbose_name="トランスクリプト")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="作成日時")
    updated_at = models.DateTimeField(auto_now=True, verbose_name="更新日時")
    
    class Meta:
        db_table = 'audio_files'
        verbose_name = '音声ファイル'
        verbose_name_plural = '音声ファイル'
```

#### 5.2.2 ChatSession（チャットセッション）
```python
class ChatSession(models.Model):
    id = models.AutoField(primary_key=True)
    audio_file = models.ForeignKey(AudioFile, on_delete=models.CASCADE, verbose_name="音声ファイル")
    dify_conversation_id = models.CharField(max_length=255, blank=True, verbose_name="Dify会話ID")
    user_identifier = models.CharField(max_length=255, verbose_name="ユーザー識別子")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="作成日時")
    
    class Meta:
        db_table = 'chat_sessions'
        verbose_name = 'チャットセッション'
        verbose_name_plural = 'チャットセッション'
```

#### 5.2.3 ChatMessage（チャットメッセージ）
```python
class ChatMessage(models.Model):
    id = models.AutoField(primary_key=True)
    session = models.ForeignKey(ChatSession, on_delete=models.CASCADE, verbose_name="セッション")
    user_message = models.TextField(verbose_name="ユーザーメッセージ")
    ai_response = models.TextField(verbose_name="AI応答")
    timestamp_start = models.FloatField(null=True, blank=True, verbose_name="開始タイムスタンプ（秒）")
    timestamp_end = models.FloatField(null=True, blank=True, verbose_name="終了タイムスタンプ（秒）")
    dify_message_id = models.CharField(max_length=255, blank=True, verbose_name="DifyメッセージID")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="作成日時")
    
    class Meta:
        db_table = 'chat_messages'
        verbose_name = 'チャットメッセージ'
        verbose_name_plural = 'チャットメッセージ'
```

## 6. API設計

### 6.1 REST API エンドポイント

#### 6.1.1 音声ファイル管理API
```
GET    /api/audio/files/          # 音声ファイル一覧取得
POST   /api/audio/files/          # 音声ファイルアップロード
GET    /api/audio/files/{id}/     # 音声ファイル詳細取得
PUT    /api/audio/files/{id}/     # 音声ファイル更新
DELETE /api/audio/files/{id}/     # 音声ファイル削除
```

#### 6.1.2 チャット管理API
```
POST   /api/chat/sessions/                    # チャットセッション作成
GET    /api/chat/sessions/{id}/               # チャットセッション取得
POST   /api/chat/sessions/{id}/messages/      # メッセージ送信
GET    /api/chat/sessions/{id}/messages/      # メッセージ履歴取得
```

#### 6.1.3 プレイヤー制御API
```
POST   /api/player/seek/          # 指定時刻へのシーク
POST   /api/player/loop/          # ループ再生設定
GET    /api/player/status/        # プレイヤー状態取得
```

### 6.2 Dify API連携設計

#### 6.2.1 DifyClientクラス
```python
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
        
    def extract_timestamp(self, response_text: str) -> Optional[Tuple[float, float]]:
        """[分:秒-分:秒]形式のタイムスタンプ抽出"""
        
    def parse_streaming_response(self, response) -> Dict:
        """SSEストリーミング応答解析"""
```

### 6.3 APIレスポンス形式

#### 6.3.1 音声ファイル応答
```json
{
  "id": 1,
  "title": "特許制度講演_第1回",
  "file_path": "/media/audio/lecture_01.mp3",
  "duration": 3600.5,
  "transcript": "講演のトランスクリプト...",
  "created_at": "2025-01-15T10:00:00Z",
  "updated_at": "2025-01-15T10:00:00Z"
}
```

#### 6.3.2 チャットメッセージ応答
```json
{
  "id": 1,
  "session_id": 1,
  "user_message": "特許権の存続期間は何年ですか？",
  "ai_response": "特許権の存続期間は出願日から20年です。参考箇所: [15:30-16:45]",
  "timestamp_start": 930.0,
  "timestamp_end": 1005.0,
  "dify_message_id": "msg_12345",
  "created_at": "2025-01-15T10:30:00Z"
}
```

## 7. フロントエンド設計

### 7.1 WaveSurfer.js設定
```javascript
const wavesurfer = WaveSurfer.create({
    container: '#waveform',
    waveColor: '#4A90E2',
    progressColor: '#1E3A8A',
    backend: 'WebAudio',
    height: 100,
    pixelRatio: 1,
    normalize: true,
    responsive: true,
    cursorColor: '#FF4444',
    cursorWidth: 2,
    barWidth: 2,
    barRadius: 1
});
```

### 7.2 主要JavaScript機能
```javascript
// 音声プレイヤー制御
class AudioPlayer {
    constructor(containerId) { }
    loadAudio(audioUrl) { }
    play() { }
    pause() { }
    seekTo(seconds) { }
    setLoop(start, end) { }
}

// チャットインターフェース
class ChatInterface {
    constructor(sessionId) { }
    sendMessage(message) { }
    displayMessage(message, isUser) { }
    handleTimestamp(timestamp) { }
}

// タイムスタンプ解析
class TimestampParser {
    static parse(text) { }
    static convertToSeconds(timeString) { }
}
```

## 8. セキュリティ設計

### 8.1 認証・認可
```python
# settings.py
AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
]

# APIキー管理
DIFY_API_KEY = config('DIFY_API_KEY')

# セッション設定
SESSION_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
```

### 8.2 入力検証
```python
# ファイルアップロード検証
ALLOWED_AUDIO_FORMATS = ['mp3', 'wav', 'mp4', 'm4a']
AUDIO_UPLOAD_MAX_SIZE = 100 * 1024 * 1024  # 100MB

# CSRF保護
CSRF_COOKIE_SECURE = True
CSRF_USE_SESSIONS = True
```

### 8.3 CORS設定
```python
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
    "http://127.0.0.1:3000",
]
CORS_ALLOW_CREDENTIALS = True
```

## 9. パフォーマンス設計

### 9.1 音声ファイル最適化
- 音声圧縮: 適切なビットレート設定
- プログレッシブ読み込み: 大容量ファイル対応
- キャッシュ戦略: ブラウザキャッシュ活用

### 9.2 データベース最適化
```python
# インデックス設定
class ChatMessage(models.Model):
    session = models.ForeignKey(ChatSession, on_delete=models.CASCADE, db_index=True)
    created_at = models.DateTimeField(auto_now_add=True, db_index=True)
    
    class Meta:
        indexes = [
            models.Index(fields=['session', 'created_at']),
        ]
```

### 9.3 API最適化
- ページネーション実装
- クエリ最適化（select_related, prefetch_related）
- 非同期処理（Celery導入検討）

## 10. 監視・ログ設計

### 10.1 ログ設定
```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': 'audio_player.log',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file'],
            'level': 'INFO',
            'propagate': True,
        },
        'apps': {
            'handlers': ['file'],
            'level': 'DEBUG',
            'propagate': True,
        },
    },
}
```

### 10.2 監視項目
- API応答時間
- 音声ファイル処理時間
- Dify API連携状況
- エラー率・頻度

## 11. テスト設計

### 11.1 単体テスト
```python
# apps/chat/tests/test_dify_client.py
class DifyClientTestCase(TestCase):
    def test_extract_timestamp(self):
        # タイムスタンプ抽出テスト
        
    def test_send_message(self):
        # メッセージ送信テスト
```

### 11.2 統合テスト
- API連携テスト
- 音声ファイル処理テスト
- フロントエンド・バックエンド連携テスト

## 12. デプロイメント設計

### 12.1 本番環境設定
```python
# settings/production.py
DEBUG = False
ALLOWED_HOSTS = ['your-domain.com']

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': config('DB_NAME'),
        'USER': config('DB_USER'),
        'PASSWORD': config('DB_PASSWORD'),
        'HOST': config('DB_HOST'),
        'PORT': config('DB_PORT'),
    }
}
```

### 12.2 静的ファイル配信
```python
STATIC_ROOT = '/var/www/static/'
MEDIA_ROOT = '/var/www/media/'

# CDN設定
STATICFILES_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
```

この設計書により、効率的なチーム開発と保守性の高いシステム構築が可能になります。