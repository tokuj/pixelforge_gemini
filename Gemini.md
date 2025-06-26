# PixelForge - ピクセルアート変換Webアプリケーション開発指示書

## プロジェクト概要
写真をアップロードすると、ピクセル・ドット調の画像に変換するWebアプリケーション「PixelForge」を開発してください。処理速度とユーザー体験を重視し、基本的な画像処理はフロントエンドで完結させ、共有機能のみバックエンドを使用します。

## アプリケーション情報
- **名称**: PixelForge
- **サブタイトル**: Transform your photos into pixel art masterpieces
- **プロジェクトID**: coproto-gcp
- **リージョン**: asia-northeast1

## 技術スタック
- **フロントエンド**: Next.js 14+ (App Router)
- **UIライブラリ**: Material-UI (MUI) v5
- **画像処理**: Canvas API（フロントエンド完結）
- **バックエンド**: Python + Google Cloud Functions（共有機能のみ）
- **ホスティング**: Google Cloud Run（フロントエンド）
- **ストレージ**: Google Cloud Storage（共有画像の一時保存）
- **データベース**: Firestore（共有URL管理）

## アーキテクチャ判断

### フロントエンド処理で完結させる機能
- **画像のピクセル化処理**: Canvas APIで十分高速
- **プリセット適用**: 事前定義のパラメータ適用
- **エフェクト処理**: リアルタイムで処理可能
- **画像ダウンロード**: Blob URLで実装

### バックエンドが必要な機能
- **共有機能**: 画像の一時保存とURL生成
- **アクセス統計**: 共有された画像の閲覧数カウント

## ブランディング実装

### ロゴ・アイコン配置
```
pixelforge/                        # リポジトリルート
├── images/                        # ブランドアセット
│   ├── logo.svg                  # 横長ロゴ（200x60）
│   ├── icon.svg                  # 正方形アイコン（256x256）
│   └── favicon.svg               # ファビコン（32x32）
├── app/                          # Next.js App Router
├── components/                   # Reactコンポーネント
└── public/                       # 静的ファイル
```

### ロゴの実装
```typescript
// components/common/Logo.tsx
import Image from 'next/image';
import logoSvg from '@/images/logo.svg';
import iconSvg from '@/images/icon.svg';

export const Logo: React.FC<{ variant?: 'full' | 'icon' }> = ({ variant = 'full' }) => {
  if (variant === 'icon') {
    return <Image src={iconSvg} alt="PixelForge" width={40} height={40} />;
  }
  return <Image src={logoSvg} alt="PixelForge" width={160} height={48} priority />;
};
```

### メタデータ設定
```typescript
// app/layout.tsx
import faviconSvg from '@/images/favicon.svg';
import iconSvg from '@/images/icon.svg';

export const metadata: Metadata = {
  title: 'PixelForge - Transform your photos into pixel art',
  description: 'Convert your images into stunning pixel art with customizable settings, presets, and effects.',
  icons: {
    icon: faviconSvg.src,
    apple: iconSvg.src,
  },
  openGraph: {
    title: 'PixelForge',
    description: 'Transform your photos into pixel art masterpieces',
    url: 'https://pixelforge-xxxxx-an.a.run.app',
    siteName: 'PixelForge',
    images: [
      {
        url: '/og-image.png',
        width: 1200,
        height: 630,
      },
    ],
    locale: 'en_US',
    type: 'website',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'PixelForge',
    description: 'Transform your photos into pixel art masterpieces',
    images: ['/og-image.png'],
  },
};
```

### カラーパレット定義
```typescript
// styles/theme.ts
import { createTheme } from '@mui/material/styles';

export const pixelForgeColors = {
  primary: '#45B7D1',      // 水色
  secondary: '#FF6B6B',    // 赤
  accent1: '#4ECDC4',      // 青緑
  accent2: '#96CEB4',      // 緑
  accent3: '#DDA0DD',      // 紫
  background: '#f8f9fa',
  surface: '#ffffff',
  text: '#333333',
};

export const pixelForgeTheme = createTheme({
  palette: {
    primary: {
      main: pixelForgeColors.primary,
      light: pixelForgeColors.accent1,
      dark: '#35a7c1',
    },
    secondary: {
      main: pixelForgeColors.secondary,
      light: pixelForgeColors.accent3,
      dark: '#ee5555',
    },
    success: {
      main: pixelForgeColors.accent2,
    },
    background: {
      default: pixelForgeColors.background,
      paper: pixelForgeColors.surface,
    },
    text: {
      primary: pixelForgeColors.text,
    },
  },
  typography: {
    fontFamily: '"Inter", "Roboto", "Helvetica", "Arial", sans-serif',
    h1: {
      fontWeight: 700,
      letterSpacing: '-0.02em',
    },
  },
  shape: {
    borderRadius: 8,
  },
  components: {
    MuiButton: {
      styleOverrides: {
        root: {
          textTransform: 'none',
          fontWeight: 600,
        },
      },
    },
    MuiAppBar: {
      styleOverrides: {
        root: {
          boxShadow: '0 2px 4px rgba(0,0,0,0.1)',
        },
      },
    },
  },
});
```

## 機能要件

### 1. 画像アップロード機能
- **入力方法**
  - ドラッグ&ドロップ
  - ファイル選択ボタン
  - クリップボードから貼り付け（Ctrl+V）
- **対応形式**: JPEG, PNG, GIF, WebP
- **最大ファイルサイズ**: 10MB
- **推奨画像サイズ**: 最大2048x2048px（それ以上は自動リサイズ）

### 2. ピクセル化処理
- **ピクセルサイズ**: 2px〜64px（スライダーで調整）
- **処理アルゴリズム**:
  ```
  1. 指定ピクセルサイズごとにグリッド分割
  2. 各グリッド内の平均色を計算
  3. グリッド全体をその平均色で塗りつぶし
  ```
- **リアルタイムプレビュー**: 250ms debounceで更新

### 3. プリセット機能
- **ゲーム機プリセット**:
  - ファミコン風（256×224px、52色パレット）
  - ゲームボーイ風（160×144px、4階調グリーン）
  - スーパーファミコン風（256×224px、32,768色から256色）
- **アートスタイルプリセット**:
  - 8ビットアート（ピクセルサイズ8px、256色）
  - 16ビットアート（ピクセルサイズ4px、65,536色）
  - モダンピクセルアート（ピクセルサイズ2px、フルカラー）
- **カスタムプリセット**: ユーザーが設定を保存（LocalStorage）

### 4. カラーパレット機能
- **色数制限**: 2, 4, 8, 16, 32, 64, 128, 256色
- **パレット選択**:
  - 自動生成（k-means法で代表色抽出）
  - プリセットパレット（レトロゲーム機パレット）
  - カスタムパレット（ユーザー定義）
- **ディザリング**: Floyd-Steinbergアルゴリズム（オプション）

### 5. エフェクト機能
- **カラーエフェクト**:
  - モノクロ
  - セピア
  - ネガティブ
  - カスタムカラーフィルター
- **後処理エフェクト**:
  - グリッドライン表示
  - スキャンライン効果
  - CRTモニター風湾曲

### 6. 共有機能
- **共有URL生成**: 
  - 短縮URL形式: `https://pixelforge-xxxxx-an.a.run.app/p/[8文字のID]`
  - 有効期限: 7日間
- **共有オプション**:
  - 処理済み画像のみ
  - 処理パラメータ付き（他のユーザーが同じ設定で編集可能）
- **SNS共有**: Twitter、Facebook、LINE対応
- **OGP対応**: 共有時のプレビュー画像自動生成

### 7. ダウンロード機能
- **形式選択**: PNG, JPEG, WebP
- **サイズオプション**:
  - オリジナルサイズ
  - 2倍、4倍、8倍拡大（ニアレストネイバー法）
- **メタデータ保持**: 元画像のEXIF情報を保持するオプション

## プロジェクト構造

```
pixelforge/
├── images/                         # ブランドアセット
│   ├── logo.svg                   # 横長ロゴ
│   ├── icon.svg                   # アプリアイコン
│   └── favicon.svg                # ファビコン
├── app/
│   ├── page.tsx                   # メインページ
│   ├── layout.tsx                 # レイアウト定義
│   ├── globals.css                # グローバルスタイル
│   ├── p/[id]/page.tsx           # 共有ページ
│   └── api/
│       ├── share/
│       │   └── route.ts          # 共有API
│       └── shared/[id]/
│           └── route.ts          # 共有画像取得API
├── components/
│   ├── ImageUploader/
│   │   ├── index.tsx             # メインコンポーネント
│   │   ├── DropZone.tsx          # ドロップゾーン
│   │   └── ClipboardHandler.tsx  # クリップボード処理
│   ├── PixelConverter/
│   │   ├── index.tsx             # メインコンポーネント
│   │   ├── Canvas.tsx            # Canvas描画
│   │   └── Controls.tsx          # 設定コントロール
│   ├── PresetSelector/
│   │   ├── index.tsx             # プリセット選択UI
│   │   ├── PresetCard.tsx        # プリセットカード
│   │   └── CustomPresetDialog.tsx # カスタムプリセット作成
│   ├── ColorPalette/
│   │   ├── index.tsx             # パレット管理
│   │   ├── PaletteEditor.tsx     # パレット編集
│   │   └── ColorPicker.tsx       # 色選択
│   ├── ShareDialog/
│   │   ├── index.tsx             # 共有ダイアログ
│   │   └── SocialButtons.tsx     # SNSシェアボタン
│   ├── layout/
│   │   ├── Header.tsx            # ヘッダー
│   │   └── Footer.tsx            # フッター
│   └── common/
│       ├── Logo.tsx              # ロゴコンポーネント
│       ├── LoadingOverlay.tsx    # ローディング表示
│       └── ErrorBoundary.tsx     # エラーハンドリング
├── lib/
│   ├── pixelate/
│   │   ├── core.ts              # ピクセル化コアロジック
│   │   ├── dithering.ts         # ディザリング処理
│   │   ├── colorQuantization.ts # 色数削減アルゴリズム
│   │   └── effects.ts           # エフェクト処理
│   ├── presets/
│   │   ├── index.ts             # プリセット定義
│   │   └── gameConsoles.ts      # ゲーム機プリセット
│   ├── share/
│   │   └── urlGenerator.ts      # URL生成ロジック
│   ├── config.ts                # アプリ設定
│   └── utils/
│       ├── imageUtils.ts        # 画像処理ユーティリティ
│       └── colorUtils.ts        # 色計算ユーティリティ
├── hooks/
│   ├── useImageProcessing.ts    # 画像処理フック
│   ├── usePresets.ts           # プリセット管理フック
│   └── useShare.ts             # 共有機能フック
├── types/
│   ├── image.ts                # 画像関連の型定義
│   └── preset.ts               # プリセットの型定義
├── styles/
│   └── theme.ts                # MUIテーマ設定
├── public/
│   ├── manifest.json           # PWAマニフェスト
│   └── og-image.png           # OGP画像
├── functions/                   # Google Cloud Functions
│   ├── requirements.txt
│   ├── main.py                 # エントリーポイント
│   ├── share_handler.py        # 共有処理
│   └── storage_manager.py      # GCS操作
├── Dockerfile                   # Cloud Run用
├── next.config.js              # Next.js設定
├── package.json
├── tsconfig.json
├── .env.production             # 本番環境変数
├── cloudbuild.yaml             # Cloud Run デプロイ設定
└── cloudbuild-functions.yaml   # Cloud Functions デプロイ設定
```

## 実装詳細

### フロントエンド実装

#### 1. 画像処理パイプライン
```typescript
// lib/pixelate/core.ts
export class PixelArtProcessor {
  private canvas: HTMLCanvasElement;
  private ctx: CanvasRenderingContext2D;
  private worker?: Worker;

  async process(image: ImageData, options: ProcessOptions): Promise<ImageData> {
    // 大きな画像の場合はWeb Workerを使用
    if (image.width * image.height > 1_000_000) {
      return this.processInWorker(image, options);
    }
    
    return this.processInMainThread(image, options);
  }

  private processInMainThread(image: ImageData, options: ProcessOptions): ImageData {
    const { pixelSize, colorPalette, useDithering } = options;
    
    // ピクセル化処理
    const pixelated = this.pixelate(image, pixelSize);
    
    // カラーパレット適用
    if (colorPalette) {
      return this.applyColorPalette(pixelated, colorPalette, useDithering);
    }
    
    return pixelated;
  }
}
```

#### 2. プリセットシステム
```typescript
// lib/presets/index.ts
export interface Preset {
  id: string;
  name: string;
  icon: string;
  category: 'console' | 'art' | 'custom';
  settings: {
    pixelSize: number;
    colorPalette?: Color[];
    colorCount?: number;
    effects?: Effect[];
    outputSize?: { width: number; height: number };
    dithering?: boolean;
  };
}

// LocalStorageでカスタムプリセットを管理
export class PresetManager {
  private readonly STORAGE_KEY = 'pixelforge_custom_presets';
  
  saveCustomPreset(preset: Preset): void {
    const presets = this.loadCustomPresets();
    presets.push(preset);
    localStorage.setItem(this.STORAGE_KEY, JSON.stringify(presets));
  }
  
  loadCustomPresets(): Preset[] {
    const stored = localStorage.getItem(this.STORAGE_KEY);
    return stored ? JSON.parse(stored) : [];
  }
  
  deleteCustomPreset(id: string): void {
    const presets = this.loadCustomPresets();
    const filtered = presets.filter(p => p.id !== id);
    localStorage.setItem(this.STORAGE_KEY, JSON.stringify(filtered));
  }
}
```

#### 3. 共有機能実装
```typescript
// hooks/useShare.ts
export function useShare() {
  const [isSharing, setIsSharing] = useState(false);
  
  const shareImage = async (
    imageBlob: Blob,
    settings: ProcessSettings
  ): Promise<ShareResult> => {
    setIsSharing(true);
    
    try {
      const formData = new FormData();
      formData.append('image', imageBlob);
      formData.append('settings', JSON.stringify(settings));
      
      const response = await fetch('/api/share', {
        method: 'POST',
        body: formData,
      });
      
      if (!response.ok) {
        throw new Error('Share failed');
      }
      
      const result = await response.json();
      return {
        shareId: result.shareId,
        shareUrl: result.shareUrl,
        expiresAt: result.expiresAt,
      };
    } finally {
      setIsSharing(false);
    }
  };
  
  return { shareImage, isSharing };
}
```

### バックエンド実装（Google Cloud Functions）

#### 1. 共有エンドポイント
```python
# functions/main.py
import functions_framework
from share_handler import create_share
from storage_manager import StorageManager
import json

@functions_framework.http
def pixelforge_share(request):
    """画像共有エンドポイント"""
    # CORS対応
    if request.method == 'OPTIONS':
        headers = {
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Methods': 'POST',
            'Access-Control-Allow-Headers': 'Content-Type',
        }
        return ('', 204, headers)
    
    if request.method != 'POST':
        return {'error': 'Method not allowed'}, 405
    
    # 画像とメタデータを取得
    image_file = request.files.get('image')
    settings = request.form.get('settings')
    
    if not image_file:
        return {'error': 'No image provided'}, 400
    
    # GCSに保存
    storage = StorageManager()
    image_url = storage.upload_image(image_file)
    
    # Firestoreに共有情報を保存
    share_id = create_share(image_url, json.loads(settings))
    
    return {
        'shareId': share_id,
        'shareUrl': f'https://pixelforge-xxxxx-an.a.run.app/p/{share_id}',
        'expiresAt': '7 days from now'
    }
```

#### 2. ストレージ管理
```python
# functions/storage_manager.py
from google.cloud import storage
import uuid
from datetime import datetime, timedelta

class StorageManager:
    def __init__(self):
        self.client = storage.Client()
        self.bucket = self.client.bucket('coproto-gcp-pixelforge-shares')
    
    def upload_image(self, image_file, expiry_days=7):
        # ユニークなファイル名を生成
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        blob_name = f"{timestamp}_{uuid.uuid4().hex[:8]}.png"
        blob = self.bucket.blob(blob_name)
        
        # メタデータを設定
        blob.metadata = {
            'uploaded_at': datetime.now().isoformat(),
            'expires_at': (datetime.now() + timedelta(days=expiry_days)).isoformat()
        }
        
        # アップロード
        blob.upload_from_file(
            image_file,
            content_type='image/png'
        )
        
        # 公開URLを返す
        return blob.public_url
```

## パフォーマンス最適化

### 1. 画像処理の最適化
- **チャンク処理**: 大きな画像を分割して処理
- **Web Worker活用**: メインスレッドのブロックを防止
- **キャッシング**: 処理結果をメモリキャッシュ
- **遅延処理**: スライダー操作時はdebounce/throttle

### 2. メモリ管理
```typescript
// 使用後のリソースを解放
function cleanup() {
  canvas.width = 0;
  canvas.height = 0;
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  URL.revokeObjectURL(blobUrl);
}
```

### 3. 共有機能の最適化
- **画像圧縮**: 共有前に画像を最適化
- **CDN配信**: Cloud CDNでグローバル配信
- **キャッシュ戦略**: 7日間のブラウザキャッシュ

## セキュリティ考慮事項

1. **アップロードの検証**
   - ファイルタイプの厳密なチェック
   - ファイルサイズ制限の実装
   - 悪意のあるコンテンツのスキャン

2. **共有機能のセキュリティ**
   - レート制限（1分間に5回まで）
   - 一時URLの使用期限設定
   - XSS対策

## デプロイ設定

### Google Cloud Run設定（フロントエンド）
```dockerfile
# Dockerfile
FROM node:20-alpine AS base

# 依存関係のインストール
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# ビルド
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build

# 実行
FROM base AS runner
WORKDIR /app
ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs
EXPOSE 8080
ENV PORT 8080

CMD ["node", "server.js"]
```

```yaml
# cloudbuild.yaml (フロントエンド用)
steps:
  # Dockerイメージをビルド
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/coproto-gcp/pixelforge', '.']
  
  # イメージをContainer Registryにプッシュ
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/coproto-gcp/pixelforge']
  
  # Cloud Runにデプロイ
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'run'
      - 'deploy'
      - 'pixelforge'
      - '--image=gcr.io/coproto-gcp/pixelforge'
      - '--region=asia-northeast1'
      - '--platform=managed'
      - '--allow-unauthenticated'
      - '--memory=1Gi'
      - '--cpu=1'
      - '--max-instances=10'
      - '--min-instances=0'
      - '--set-env-vars=PROJECT_ID=coproto-gcp,NEXT_PUBLIC_API_URL=https://asia-northeast1-coproto-gcp.cloudfunctions.net'

images:
  - 'gcr.io/coproto-gcp/pixelforge'
```

```json
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone',
  images: {
    domains: ['storage.googleapis.com'],
  },
  env: {
    PROJECT_ID: 'coproto-gcp',
    GCS_BUCKET: 'coproto-gcp-pixelforge-shares',
  },
}

module.exports = nextConfig
```

### Google Cloud Functions設定（バックエンド）
```yaml
# cloudbuild-functions.yaml
steps:
  # Cloud Functionsをデプロイ
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'functions'
      - 'deploy'
      - 'pixelforge-share'
      - '--entry-point=pixelforge_share'
      - '--runtime=python311'
      - '--trigger-http'
      - '--allow-unauthenticated'
      - '--region=asia-northeast1'
      - '--memory=512MB'
      - '--timeout=60s'
      - '--max-instances=100'
      - '--set-env-vars=PROJECT_ID=coproto-gcp,BUCKET_NAME=coproto-gcp-pixelforge-shares,SHARE_EXPIRY_DAYS=7'
      - '--source=./functions'
```

### デプロイコマンド
```bash
# アプリケーション名を環境変数に設定
export APP_NAME=pixelforge

# フロントエンド（Cloud Run）のデプロイ
gcloud builds submit --config cloudbuild.yaml --project coproto-gcp

# バックエンド（Cloud Functions）のデプロイ
gcloud builds submit --config cloudbuild-functions.yaml --project coproto-gcp

# 必要なGCPサービスの有効化
gcloud services enable run.googleapis.com cloudfunctions.googleapis.com \
  cloudbuild.googleapis.com storage.googleapis.com firestore.googleapis.com \
  --project coproto-gcp

# GCSバケットの作成（PixelForge専用）
gsutil mb -p coproto-gcp -c standard -l asia-northeast1 \
  gs://coproto-gcp-pixelforge-shares/

# バケットのライフサイクル設定（7日後に自動削除）
cat > lifecycle.json << EOF
{
  "lifecycle": {
    "rule": [
      {
        "action": {"type": "Delete"},
        "condition": {"age": 7}
      }
    ]
  }
}
EOF
gsutil lifecycle set lifecycle.json gs://coproto-gcp-pixelforge-shares/
```

## 環境変数設定

### フロントエンド環境変数
```env
# .env.production
PROJECT_ID=coproto-gcp
NEXT_PUBLIC_API_URL=https://asia-northeast1-coproto-gcp.cloudfunctions.net
NEXT_PUBLIC_SHARE_BASE_URL=https://pixelforge-xxxxx-an.a.run.app
NEXT_PUBLIC_GCS_BUCKET=coproto-gcp-pixelforge-shares
```

### API エンドポイント設定
```typescript
// lib/config.ts
export const config = {
  projectId: 'coproto-gcp',
  api: {
    shareEndpoint: `${process.env.NEXT_PUBLIC_API_URL}/pixelforge-share`,
    getSharedEndpoint: (id: string) => 
      `${process.env.NEXT_PUBLIC_API_URL}/pixelforge-get?id=${id}`,
  },
  storage: {
    bucketName: 'coproto-gcp-pixelforge-shares',
    publicUrl: (filename: string) => 
      `https://storage.googleapis.com/coproto-gcp-pixelforge-shares/${filename}`,
  },
  share: {
    baseUrl: process.env.NEXT_PUBLIC_SHARE_BASE_URL || 'http://localhost:3000',
  },
};
```

## IAM設定

```bash
# Cloud RunからCloud Functions/GCSへのアクセス権限設定
gcloud projects add-iam-policy-binding coproto-gcp \
  --member="serviceAccount:pixelforge@coproto-gcp.iam.gserviceaccount.com" \
  --role="roles/cloudfunctions.invoker"

gcloud projects add-iam-policy-binding coproto-gcp \
  --member="serviceAccount:pixelforge@coproto-gcp.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

# Cloud FunctionsからGCS/Firestoreへのアクセス権限設定
gcloud projects add-iam-policy-binding coproto-gcp \
  --member="serviceAccount:coproto-gcp@appspot.gserviceaccount.com" \
  --role="roles/storage.objectAdmin"

gcloud projects add-iam-policy-binding coproto-gcp \
  --member="serviceAccount:coproto-gcp@appspot.gserviceaccount.com" \
  --role="roles/datastore.user"
```

## CI/CD設定

### GitHub Actions設定
```yaml
# .github/workflows/deploy.yml
name: Deploy to Google Cloud

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  PROJECT_ID: coproto-gcp
  REGION: asia-northeast1

jobs:
  deploy-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - id: 'auth'
        uses: 'google-github-actions/auth@v1'
        with:
          credentials_json: '${{ secrets.GCP_SA_KEY }}'
      
      - name: 'Set up Cloud SDK'
        uses: 'google-github-actions/setup-gcloud@v1'
      
      - name: 'Configure Docker'
        run: gcloud auth configure-docker
      
      - name: 'Build and Deploy to Cloud Run'
        run: |
          gcloud builds submit \
            --config cloudbuild.yaml \
            --project $PROJECT_ID

  deploy-functions:
    runs-on: ubuntu-latest
    needs: deploy-frontend
    steps:
      - uses: actions/checkout@v3
      
      - id: 'auth'
        uses: 'google-github-actions/auth@v1'
        with:
          credentials_json: '${{ secrets.GCP_SA_KEY }}'
      
      - name: 'Set up Cloud SDK'
        uses: 'google-github-actions/setup-gcloud@v1'
      
      - name: 'Deploy Cloud Functions'
        run: |
          gcloud builds submit \
            --config cloudbuild-functions.yaml \
            --project $PROJECT_ID
```

## モニタリング設定

```yaml
# monitoring/alerts.yaml
displayName: "PixelForge Alerts"
conditions:
  - displayName: "High Error Rate"
    conditionThreshold:
      filter: |
        resource.type="cloud_run_revision"
        resource.labels.service_name="pixelforge"
        metric.type="run.googleapis.com/request_count"
        metric.labels.response_code_class!="2xx"
      comparison: COMPARISON_GT
      thresholdValue: 10
      duration: 60s

  - displayName: "High Latency"
    conditionThreshold:
      filter: |
        resource.type="cloud_run_revision"
        resource.labels.service_name="pixelforge"
        metric.type="run.googleapis.com/request_latencies"
      comparison: COMPARISON_GT
      thresholdValue: 2000  # 2秒
      duration: 300s
```

## 開発フロー

### ブランチ戦略
- `main`: 本番環境用ブランチ
- `develop`: 開発統合ブランチ
- `feature/phase-X-[phase-name]`: 各Phase用のフィーチャーブランチ

### プルリクエストルール
1. 各Phaseごとに専用のブランチを作成
2. Phase完了時にプルリクエストを作成
3. PRタイトル: `[Phase X] Phase名`
4. PR説明には完了したタスクのチェックリストを含める
5. レビュー後にdevelopブランチにマージ

## 開発ロードマップ

### Phase 0: プロジェクトセットアップ
**ブランチ名**: `feature/phase-0-setup`

- [ ] developブランチの作成
- [ ] feature/phase-0-setupブランチの作成とチェックアウト
- [ ] Next.jsプロジェクトの作成（App Router、TypeScript、Tailwind CSS）
- [ ] yarnでMUIと関連パッケージのインストール
- [ ] プロジェクト構造の作成（ディレクトリ構成）
- [ ] ESLintとPrettierの設定
- [ ] images/ディレクトリのアセットを適切に配置
- [ ] 基本的なレイアウトコンポーネントの作成
- [ ] READMEの更新（プロジェクト概要、セットアップ手順）
- [ ] PR作成: `[Phase 0] プロジェクトセットアップ`

### Phase 1: MVP（2週間）
**ブランチ名**: `feature/phase-1-mvp`

#### ブランディング実装
- [ ] feature/phase-1-mvpブランチの作成とチェックアウト
- [ ] ロゴコンポーネント（Logo.tsx）の作成
- [ ] MUIテーマ設定（theme.ts）の作成
- [ ] カラーパレット定数の定義
- [ ] ヘッダーコンポーネントの実装
- [ ] フッターコンポーネントの実装
- [ ] ファビコンとメタデータの設定
- [ ] OGP画像の作成と設定

#### 画像アップロード機能
- [ ] ImageUploaderコンポーネントの作成
- [ ] DropZoneコンポーネントの実装（react-dropzone使用）
- [ ] ファイル選択ボタンの実装
- [ ] クリップボードハンドラーの実装（Ctrl+V対応）
- [ ] ファイルバリデーション（形式、サイズ）の実装
- [ ] アップロード時のプレビュー表示
- [ ] エラーハンドリングとユーザーフィードバック

#### ピクセル化処理
- [ ] PixelConverterコンポーネントの作成
- [ ] Canvas要素の設定と管理
- [ ] 基本的なピクセル化アルゴリズムの実装
- [ ] ピクセルサイズスライダー（2-64px）の実装
- [ ] リアルタイムプレビューの実装（debounce付き）
- [ ] 処理前後の比較表示機能
- [ ] useImageProcessingフックの作成

#### ダウンロード機能
- [ ] ダウンロードボタンコンポーネントの作成
- [ ] PNG形式でのエクスポート機能
- [ ] ファイル名の自動生成（pixelated_[元のファイル名]）
- [ ] ダウンロード成功時のフィードバック
- [ ] PR作成: `[Phase 1] MVP - 基本的な画像変換機能`

### Phase 2: 拡張機能（2週間）
**ブランチ名**: `feature/phase-2-advanced-features`

#### プリセット機能
- [ ] feature/phase-2-advanced-featuresブランチの作成とチェックアウト
- [ ] PresetSelectorコンポーネントの作成
- [ ] PresetCardコンポーネントの実装
- [ ] ゲーム機プリセットの定義（ファミコン、ゲームボーイ、スーファミ）
- [ ] アートスタイルプリセットの定義（8bit、16bit、モダン）
- [ ] プリセット適用ロジックの実装
- [ ] CustomPresetDialogの作成
- [ ] LocalStorageへのカスタムプリセット保存機能
- [ ] usePresetsフックの作成

#### カラーパレット機能
- [ ] ColorPaletteコンポーネントの作成
- [ ] 色数制限スライダーの実装（2-256色）
- [ ] k-means法による自動色抽出の実装
- [ ] プリセットパレット（レトロゲーム機）の追加
- [ ] PaletteEditorコンポーネントの実装
- [ ] カスタムパレット作成機能
- [ ] ColorPickerコンポーネントの実装
- [ ] ディザリングオプション（Floyd-Steinberg）の追加

#### エフェクト機能
- [ ] エフェクト選択UIの作成
- [ ] モノクロ変換の実装
- [ ] セピア調変換の実装
- [ ] ネガティブ変換の実装
- [ ] グリッドライン表示オプション
- [ ] スキャンライン効果の実装
- [ ] CRTモニター風湾曲効果の実装
- [ ] effects.tsにエフェクトロジックを集約
- [ ] PR作成: `[Phase 2] 拡張機能 - プリセット、パレット、エフェクト`

### Phase 3: 共有機能（1週間）
**ブランチ名**: `feature/phase-3-sharing`

#### バックエンド実装
- [ ] feature/phase-3-sharingブランチの作成とチェックアウト
- [ ] Google Cloud Functionsプロジェクトの作成
- [ ] requirements.txtの作成（Python依存関係）
- [ ] main.pyの実装（HTTPエンドポイント）
- [ ] share_handler.pyの実装（共有ロジック）
- [ ] storage_manager.pyの実装（GCS操作）
- [ ] Firestoreスキーマの設計と実装
- [ ] CORS設定の追加
- [ ] エラーハンドリングの実装

#### フロントエンド共有機能
- [ ] ShareDialogコンポーネントの作成
- [ ] 共有ボタンの実装
- [ ] 共有URL生成の実装
- [ ] useShareフックの作成
- [ ] 共有ページ（/p/[id]）の作成
- [ ] 共有画像の表示機能
- [ ] 共有パラメータによる再編集機能
- [ ] 有効期限表示

#### SNS連携
- [ ] SocialButtonsコンポーネントの作成
- [ ] Twitter共有ボタンの実装
- [ ] Facebook共有ボタンの実装
- [ ] LINE共有ボタンの実装
- [ ] OGPメタタグの動的生成
- [ ] 共有時のプレビュー画像生成
- [ ] PR作成: `[Phase 3] 共有機能 - バックエンドとSNS連携`

### Phase 4: 最適化とポリッシュ（1週間）
**ブランチ名**: `feature/phase-4-optimization`

#### パフォーマンス最適化
- [ ] feature/phase-4-optimizationブランチの作成とチェックアウト
- [ ] Web Workerの実装（大画像処理用）
- [ ] 画像処理のチャンク化
- [ ] メモリリーク対策の実装
- [ ] Lighthouse スコアの改善
- [ ] バンドルサイズの最適化
- [ ] 画像の遅延読み込み
- [ ] キャッシュ戦略の実装

#### UI/UX改善
- [ ] ローディング状態の改善（スケルトン、プログレスバー）
- [ ] エラー状態のデザイン改善
- [ ] アニメーションの追加（ロゴパルス、トランジション）
- [ ] レスポンシブデザインの完全対応
- [ ] キーボードショートカットの実装
- [ ] ツールチップとヘルプテキストの追加
- [ ] 初回訪問時のチュートリアル

#### PWA対応
- [ ] manifest.jsonの作成
- [ ] Service Workerの実装
- [ ] オフライン対応
- [ ] インストールプロンプトの実装
- [ ] プッシュ通知の準備（将来の機能用）

#### テストとデプロイ
- [ ] ユニットテストの作成（主要コンポーネント）
- [ ] E2Eテストの作成（Playwright）
- [ ] デプロイスクリプトの作成
- [ ] Cloud Runへのデプロイ
- [ ] Cloud Functionsへのデプロイ
- [ ] 本番環境での動作確認
- [ ] パフォーマンスモニタリングの設定
- [ ] PR作成: `[Phase 4] 最適化とポリッシュ - パフォーマンスとPWA`

### Phase 5: 本番リリース
**ブランチ名**: `release/v1.0.0`

- [ ] developブランチから release/v1.0.0 ブランチを作成
- [ ] 最終的なバグ修正
- [ ] ドキュメントの最終確認
- [ ] mainブランチへのPR作成: `[Release] v1.0.0 - PixelForge 初回リリース`
- [ ] タグの作成（v1.0.0）

## 開発開始手順

前提：リポジトリは既にクローンされており、images/ディレクトリにロゴアセットが配置されている状態

### 0. Git初期設定
```bash
# developブランチの作成
git checkout -b develop
git push -u origin develop

# Phase 0のブランチ作成
git checkout -b feature/phase-0-setup
```

### 1. Next.jsプロジェクトの作成
```bash
# 現在のディレクトリにNext.jsプロジェクトを作成
yarn create next-app . --typescript --app --tailwind --src-dir=false --import-alias "@/*"

# 既存のファイルがある場合は上書きの確認が出るので、適切に選択
```

### 2. 必要なパッケージのインストール
```bash
# MUIと関連パッケージ
yarn add @mui/material @emotion/react @emotion/styled @mui/icons-material

# 画像処理関連
yarn add react-dropzone

# ユーティリティ
yarn add lodash @types/lodash

# 開発用パッケージ
yarn add -D @types/node
```

### 3. プロジェクト構造の作成
```bash
# ディレクトリ構造を作成
mkdir -p app/p/\[id\]
mkdir -p app/api/share
mkdir -p app/api/shared/\[id\]
mkdir -p components/{ImageUploader,PixelConverter,PresetSelector,ColorPalette,ShareDialog,layout,common}
mkdir -p lib/{pixelate,presets,share,utils}
mkdir -p hooks
mkdir -p types
mkdir -p styles
mkdir -p functions
```

### 4. 開発サーバーの起動
```bash
yarn dev
```

### 5. Git設定
```bash
# .gitignoreに追加
echo ".env.local" >> .gitignore
echo ".env.production" >> .gitignore
echo "*.log" >> .gitignore
```

### 6. Phase 0完了時のコミットとPR
```bash
# すべての変更をコミット
git add .
git commit -m "feat: プロジェクトの初期セットアップ完了"

# プッシュしてPRを作成
git push -u origin feature/phase-0-setup
# GitHubでPRを作成: feature/phase-0-setup → develop
```

## プルリクエストテンプレート

```markdown
## 概要
[Phase X] の実装が完了しました。

## 完了したタスク
- [x] タスク1
- [x] タスク2
...（該当するPhaseのチェックリストをコピー）

## 主な変更点
- 変更点1
- 変更点2

## スクリーンショット
（UIに関する変更がある場合）

## テスト方法
1. yarn dev でサーバーを起動
2. http://localhost:3000 にアクセス
3. 機能をテスト

## 今後の課題
- 課題があれば記載
```

この開発フローに従って、各Phaseごとにブランチを作成し、完了後にプルリクエストを送信してください。
