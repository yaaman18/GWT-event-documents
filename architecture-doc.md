# System Architecture - GWT Interactive Demo

## 📚 目次
- [アーキテクチャ概要](#アーキテクチャ概要)  
- [バックエンド設計](#バックエンド設計)
- [フロントエンド設計](#フロントエンド設計)
- [データフロー](#データフロー)
- [パフォーマンス考慮事項](#パフォーマンス考慮事項)
- [拡張性・保守性](#拡張性保守性)
- [デプロイメント](#デプロイメント)

## 🏗️ アーキテクチャ概要

### システム全体構成

```
┌─────────────────────────────────────────────────────────┐
│                    Client Browser                       │
│  ┌─────────────────────────────────────────────────┐   │
│  │              React Frontend                     │   │
│  │  ┌─────────────┐  ┌─────────────────────────┐  │   │
│  │  │Theater View │  │ Control Panels         │  │   │
│  │  │(劇場の可視化)│  │ - Input Panel          │  │   │
│  │  │             │  │ - Processor Panel      │  │   │
│  │  │             │  │ - History Panel        │  │   │
│  │  │             │  │ - Control Panel        │  │   │
│  │  └─────────────┘  └─────────────────────────┘  │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                      ↕ WebSocket (Real-time)
┌─────────────────────────────────────────────────────────┐
│                   Node.js Server                        │
│  ┌─────────────────────────────────────────────────┐   │
│  │               Express + Socket.IO                │   │
│  │  ┌─────────────────────────────────────────────┐ │   │
│  │  │            GWT Engine                       │ │   │
│  │  │  ┌─────────────┐  ┌─────────────────────┐  │ │   │
│  │  │  │ Processors  │  │ Attention Manager    │  │ │   │
│  │  │  │- Vision     │  │- Competition Logic   │  │ │   │
│  │  │  │- Audio      │  │- Scoring System      │  │ │   │
│  │  │  │- Language   │  │- Winner Selection    │  │ │   │
│  │  │  │- Memory     │  └─────────────────────┘  │ │   │
│  │  │  └─────────────┘                          │ │   │
│  │  │  ┌─────────────────────────────────────────┐ │   │
│  │  │  │         Global Workspace                │ │   │
│  │  │  │  - Event Broadcasting                   │ │   │
│  │  │  │  - State Management                     │ │   │
│  │  │  │  - History Tracking                     │ │   │
│  │  │  └─────────────────────────────────────────┘ │   │
│  │  └─────────────────────────────────────────────┘ │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### 技術スタック

| レイヤー | 技術 | 目的 |
|---------|------|------|
| **Frontend** | React 18 + TypeScript | インタラクティブUI |
| **Styling** | Tailwind CSS | レスポンシブデザイン |
| **Icons** | Lucide React | 一貫したアイコン |
| **Charts** | Recharts | データ可視化 |
| **Communication** | Socket.IO Client | リアルタイム通信 |
| **Backend** | Node.js + Express | APIサーバー |
| **Real-time** | Socket.IO Server | WebSocket管理 |
| **Language** | TypeScript | 型安全性 |

## ⚙️ バックエンド設計

### 1. メインサーバー構成

```typescript
// server/server.ts
import express from 'express';
import { createServer } from 'http';
import { Server } from 'socket.io';
import { GlobalWorkspace } from '../src/GlobalWorkspace.js';

const app = express();
const server = createServer(app);
const io = new Server(server, {
  cors: { origin: "http://localhost:3000" }
});

// GWTエンジンのインスタンス
const workspace = new GlobalWorkspace();

// Socket.IO接続管理
io.on('connection', (socket) => {
  // リアルタイムイベント処理
  workspace.on('consciousness', (content) => {
    socket.emit('consciousness-event', content);
  });
  
  // クライアントからの処理要求
  socket.on('process-input', async (data) => {
    const result = await workspace.processInput(data);
    socket.emit('processing-completed', result);
  });
});
```

### 2. GWTエンジン アーキテクチャ

```typescript
// src/GlobalWorkspace.ts
export class GlobalWorkspace extends EventEmitter {
  private processors: Processor[];
  private attentionManager: AttentionManager;
  private consciousnessHistory: ConsciousContent[] = [];

  async processInput(input: ProcessorInput | ProcessorInput[]): Promise<ConsciousContent> {
    // 1. 並列処理フェーズ
    const results = await this.runParallelProcessing(input);
    
    // 2. 競争フェーズ  
    const consciousContent = this.attentionManager.compete(results);
    
    // 3. グローバル放送フェーズ
    this.globalBroadcast(consciousContent);
    
    // 4. 履歴管理
    this.updateHistory(consciousContent);
    
    return consciousContent;
  }
}
```

### 3. プロセッサ設計パターン

```typescript
// src/processors/BaseProcessor.ts
export interface Processor {
  id: string;
  canHandle(input: ProcessorInput): boolean;
  process(input: ProcessorInput): Promise<ProcessorOutput>;
  onGlobalBroadcast(content: ConsciousContent): void;
}

// 具体的な実装例
export class VisionProcessor implements Processor {
  id = 'vision';
  private currentContext: any = {};

  canHandle(input: ProcessorInput): boolean {
    return input.type === 'vision' || this.hasVisualContent(input.data);
  }

  async process(input: ProcessorInput): Promise<ProcessorOutput> {
    const startTime = Date.now();
    const content = await this.analyzeVisualInput(input.data);
    
    return {
      processorId: this.id,
      confidence: this.calculateConfidence(content),
      content,
      processingTime: Date.now() - startTime,
      priority: this.calculatePriority(content)
    };
  }

  onGlobalBroadcast(content: ConsciousContent): void {
    // 他のプロセッサの勝利結果を受信して状態更新
    this.currentContext.lastConsciousContent = content;
    this.updateInternalBias(content);
  }
}
```

### 4. 注意管理システム

```typescript
// src/AttentionManager.ts  
export class AttentionManager {
  compete(candidates: ProcessorOutput[]): ConsciousContent {
    // スコア計算
    const scored = candidates.map(candidate => ({
      candidate,
      score: this.calculateCompetitionScore(candidate)
    }));

    // 勝者決定
    const winner = scored.reduce((prev, current) => 
      prev.score > current.score ? prev : current
    );

    return {
      winner: winner.candidate,
      competitors: scored.filter(s => s !== winner).map(s => s.candidate),
      timestamp: Date.now(),
      globalState: this.calculateGlobalState(scored)
    };
  }

  private calculateCompetitionScore(candidate: ProcessorOutput): number {
    return (
      candidate.confidence * 0.4 +
      candidate.priority * 0.5 +
      this.calculateSpeedBonus(candidate.processingTime) * 0.1
    );
  }
}
```

## 🎨 フロントエンド設計

### 1. コンポーネント階層

```
App.tsx
├── TheaterView.tsx (メイン劇場)
│   ├── ProcessorNodes (プロセッサ配置)
│   ├── Spotlight (スポットライト効果)
│   ├── ConsciousnessDisplay (意識内容表示)
│   └── CompetitionVisualization (競争可視化)
├── InputPanel.tsx (入力コントロール)
│   ├── InputTypeSelector
│   ├── SingleInputForm
│   ├── MultimodalInputForm
│   └── ExampleButtons
├── ProcessorPanel.tsx (プロセッサ状態)
│   ├── ProcessorCard × 4
│   └── ProcessorMetrics
├── HistoryPanel.tsx (意識履歴)
│   ├── HistoryList
│   └── SessionStatistics
└── ControlPanel.tsx (システム制御)
    ├── ConnectionStatus
    ├── WorkspaceControls
    └── SystemInfo
```

### 2. 状態管理

```typescript
// src/App.tsx
interface AppState {
  isConnected: boolean;
  isProcessing: boolean;
  currentConsciousness: ConsciousContent | null;
  history: ConsciousContent[];
  workspaceState: any;
}

export default function App() {
  const [socket, setSocket] = useState<Socket | null>(null);
  const [state, setState] = useState<AppState>({
    isConnected: false,
    isProcessing: false,
    currentConsciousness: null,
    history: [],
    workspaceState: null
  });

  // WebSocket接続とイベント処理
  useEffect(() => {
    const newSocket = io('http://localhost:3001');
    
    newSocket.on('consciousness-event', (content: ConsciousContent) => {
      setState(prev => ({
        ...prev,
        currentConsciousness: content,
        isProcessing: false
      }));
    });

    setSocket(newSocket);
    return () => newSocket.close();
  }, []);
}
```

### 3. 劇場ビューの実装

```typescript
// src/components/TheaterView.tsx
export const TheaterView: React.FC<TheaterViewProps> = ({ 
  consciousness, 
  isProcessing 
}) => {
  const [spotlight, setSpotlight] = useState({ x: 50, y: 50 });

  // スポットライト位置のアニメーション
  useEffect(() => {
    if (consciousness) {
      const position = getProcessorPosition(consciousness.winner.processorId);
      setSpotlight(position);
    }
  }, [consciousness]);

  return (
    <div className="theater-container">
      {/* スポットライト効果 */}
      <div 
        className="spotlight"
        style={{
          left: `${spotlight.x}%`,
          top: `${spotlight.y}%`,
          transform: 'translate(-50%, -50%)'
        }}
      />
      
      {/* プロセッサ配置 */}
      {processors.map(processor => (
        <ProcessorNode
          key={processor.id}
          processor={processor}
          isWinner={consciousness?.winner.processorId === processor.id}
          position={getProcessorPosition(processor.id)}
        />
      ))}
      
      {/* 中央の意識内容表示 */}
      <ConsciousnessDisplay consciousness={consciousness} />
    </div>
  );
};
```

## 🔄 データフロー

### 1. 入力から出力までの流れ

```
1. User Input
   ↓
2. Frontend Validation
   ↓  
3. WebSocket Transmission
   ↓
4. Backend Processing
   ├── Parallel Processor Execution
   ├── Competition & Selection  
   ├── Global Broadcasting
   └── History Update
   ↓
5. WebSocket Response
   ↓
6. Frontend State Update
   ↓
7. UI Animation & Display
```

### 2. リアルタイム通信

```typescript
// クライアント → サーバー
socket.emit('process-input', {
  type: 'language',
  data: 'Hello world',
  timestamp: Date.now()
});

// サーバー → クライアント  
socket.emit('consciousness-event', {
  winner: { processorId: 'language', confidence: 0.85, ... },
  competitors: [...],
  timestamp: Date.now(),
  globalState: { competitionIntensity: 0.42, ... }
});
```

### 3. エラーハンドリング

```typescript
// 処理エラーの伝播
try {
  const result = await workspace.processInput(data);
  socket.emit('processing-completed', result);
} catch (error) {
  socket.emit('processing-error', { 
    message: error.message,
    timestamp: Date.now()
  });
}

// フロントエンドでのエラー処理
socket.on('processing-error', (error) => {
  console.error('Processing error:', error);
  setState(prev => ({ ...prev, isProcessing: false }));
  showErrorNotification(error.message);
});
```

## ⚡ パフォーマンス考慮事項

### 1. バックエンド最適化

```typescript
// 並列処理の最適化
async processInput(input: ProcessorInput[]): Promise<ConsciousContent> {
  // Promise.allで並列実行（直列ではない）
  const results = await Promise.all(
    input.flatMap(singleInput => 
      this.getCapableProcessors(singleInput).map(p => p.process(singleInput))
    )
  );
  
  // 競争は高速化（O(n)）
  const winner = this.attentionManager.compete(results);
  
  return winner;
}

// メモリ管理
private updateHistory(content: ConsciousContent): void {
  this.consciousnessHistory.push(content);
  
  // 履歴サイズ制限（メモリリーク防止）
  if (this.consciousnessHistory.length > 100) {
    this.consciousnessHistory = this.consciousnessHistory.slice(-100);
  }
}
```

### 2. フロントエンド最適化

```typescript
// 不要な再レンダリング防止
const ProcessorCard = React.memo<ProcessorCardProps>(({ processor, stats }) => {
  return (
    <div className={`processor-card ${stats ? 'active' : 'idle'}`}>
      {/* コンポーネント内容 */}
    </div>
  );
});

// 状態更新の最適化
const handleConsciousnessEvent = useCallback((content: ConsciousContent) => {
  setState(prev => ({
    ...prev,
    currentConsciousness: content,
    isProcessing: false
  }));
}, []);

// アニメーション最適化（CSS transforms使用）
.spotlight {
  transition: all 1000ms ease-out;
  transform: translate(-50%, -50%);
  will-change: left, top;
}
```

### 3. WebSocket最適化

```typescript
// 接続プール管理
const io = new Server(server, {
  cors: { origin: process.env.CLIENT_URL },
  transports: ['websocket', 'polling'],
  pingTimeout: 60000,
  pingInterval: 25000
});

// メッセージサイズ制限
socket.on('process-input', (data) => {
  if (JSON.stringify(data).length > 10000) {
    socket.emit('error', { message: 'Input too large' });
    return;
  }
  // 処理続行
});
```

## 🔧 拡張性・保守性

### 1. 新プロセッサの追加

```typescript
// 新しいプロセッサの実装
export class EmotionProcessor implements Processor {
  id = 'emotion';
  
  canHandle(input: ProcessorInput): boolean {
    return this.hasEmotionalContent(input.data);
  }
  
  async process(input: ProcessorInput): Promise<ProcessorOutput> {
    // 感情分析ロジック
    return {
      processorId: this.id,
      confidence: this.analyzeEmotionalTone(input.data),
      content: this.extractEmotions(input.data),
      processingTime: Date.now() - startTime,
      priority: this.calculateEmotionalPriority(input.data)
    };
  }
}

// GlobalWorkspaceに追加
const workspace = new GlobalWorkspace();
workspace.addProcessor(new EmotionProcessor());
```

### 2. 競争ルールのカスタマイズ

```typescript
// AttentionManagerの設定可能化
interface CompetitionConfig {
  confidenceWeight: number;
  priorityWeight: number;
  speedWeight: number;
  recencyBoost: number;
}

export class AttentionManager {
  constructor(private config: CompetitionConfig = DEFAULT_CONFIG) {}
  
  updateConfig(newConfig: Partial<CompetitionConfig>): void {
    this.config = { ...this.config, ...newConfig };
  }
}
```

### 3. プラグインシステム

```typescript
// プラグインインターフェース
interface GWTPlugin {
  name: string;
  version: string;
  install(workspace: GlobalWorkspace): void;
  uninstall(workspace: GlobalWorkspace): void;
}

// プラグイン管理
export class PluginManager {
  private plugins: Map<string, GWTPlugin> = new Map();
  
  register(plugin: GWTPlugin): void {
    this.plugins.set(plugin.name, plugin);
    plugin.install(this.workspace);
  }
}
```

## 🚀 デプロイメント

### 1. 開発環境セットアップ

```bash
# フルセットアップ
git clone https://github.com/your-org/gwt-interactive-demo.git
cd gwt-interactive-demo

# 依存関係インストール
npm install
cd server && npm install && cd ..

# 開発サーバー起動
npm run dev:full  # フロントエンド+バックエンド同時起動
```

### 2. 本番デプロイ構成

```yaml
# docker-compose.yml
version: '3.8'
services:
  backend:
    build: ./server
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=production
      - CORS_ORIGIN=https://gwt-demo.example.com
    
  frontend:
    build: .
    ports:
      - "80:80"
    depends_on:
      - backend
    environment:
      - REACT_APP_API_URL=https://api.gwt-demo.example.com
```

### 3. パフォーマンス監視

```typescript
// メトリクス収集
export class PerformanceMonitor {
  trackProcessingTime(duration: number, processorId: string): void {
    console.log(`${processorId}: ${duration}ms`);
    // メトリクス送信 (Prometheus, etc.)
  }
  
  trackCompetitionIntensity(intensity: number): void {
    console.log(`Competition intensity: ${intensity}`);
  }
}
```

この設計により、GWTの理論を正確に実装しつつ、拡張可能で保守しやすいシステムを構築できます。プログラマーは実際のコードを通じて意識のメカニズムを深く理解できるでしょう。