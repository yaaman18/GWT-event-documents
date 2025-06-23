# GWT バトルシステム設計ガイド 💻
## ～視覚ちゃん・聴覚くん・言語さん・記憶ちゃんのバトル会場を作っちゃお～

> **今回のミッション**: 4人のプロセッサちゃんたちが競争できるバトルシステムを実際に作る！
> 理論は分かった、キャラも分かった、じゃあ実際のバトル会場はどう設計するの？ 🏟️

---

## 🏟️ バトル会場の全体設計

### 劇場システムの構成 - みんなの配置

```
┌─────────────────────────────────────────────────────────┐
│                  👥 観客席（ブラウザ）                     │
│  ┌─────────────────────────────────────────────────┐   │
│  │            劇場観覧システム（React）              │   │
│  │                                                 │   │
│  │  ┌─────────────┐  ┌─────────────────────────┐  │   │
│  │  │🎭 メインステージ│  │ 📊 バトル観戦パネル      │  │   │
│  │  │(バトル実況中継)│  │ - 選手入場ゲート        │  │   │
│  │  │             │  │ - 選手ステータス表示    │  │   │
│  │  │ 👁️ 👂 💬 🧠│  │ - バトル履歴           │  │   │
│  │  │             │  │ - 審判コントロール       │  │   │
│  │  └─────────────┘  └─────────────────────────┘  │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                      ↕ 実況中継回線（WebSocket）
┌─────────────────────────────────────────────────────────┐
│                 🏟️ バトルアリーナ（サーバー）               │
│  ┌─────────────────────────────────────────────────┐   │
│  │               バトル運営本部                     │   │
│  │  ┌─────────────────────────────────────────────┐ │   │
│  │  │              選手管理システム                 │ │   │
│  │  │  ┌─────────────┐  ┌─────────────────────┐  │ │   │
│  │  │  │ 選手控え室    │  │ 🏆 バトル審判システム │  │ │   │
│  │  │  │👁️視覚ちゃん   │  │- 競争ルール管理     │  │ │   │
│  │  │  │👂聴覚くん     │  │- スコア計算        │  │ │   │
│  │  │  │💬言語さん     │  │- 勝者判定          │  │ │   │
│  │  │  │🧠記憶ちゃん   │  │- 結果発表          │  │ │   │
│  │  │  └─────────────┘  └─────────────────────┘  │ │   │
│  │  │  ┌─────────────────────────────────────────┐ │   │
│  │  │  │         🎪 バトルステージ管理            │ │   │
│  │  │  │  - 実況中継システム                     │ │   │
│  │  │  │  - バトル履歴記録                       │ │   │
│  │  │  │  - ファン投票システム                    │ │   │
│  │  │  └─────────────────────────────────────────┘ │   │
│  │  └─────────────────────────────────────────────┘ │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### 選手紹介 - バトラー4人組

| 選手名 | ポジション | 得意技 | 性格 |
|-------|-----------|-------|------|
| **👁️ 視覚ちゃん** | アタッカー | 色・形・動きの瞬間認識 | 完璧主義、細かいとこまで気づく |
| **👂 聴覚くん** | ディフェンダー | 音の方向・緊急度判定 | 敏感、空気読むの得意 |
| **💬 言語さん** | ストラテジスト | 文章理解・会話分析 | 論理的、説明上手 |
| **🧠 記憶ちゃん** | サポーター | 過去データ検索・関連付け | 博識、連想ゲームの女王 |

---

## ⚔️ バトル会場のシステム設計

### 1. 選手控え室の実装 - プロセッサたちの待機場所

```typescript
// src/processors/VisionProcessor.ts - 視覚ちゃんの控え室
export class VisionProcessor implements Processor {
  id = 'vision';
  nickname = '視覚ちゃん';
  personality = 'perfectionist';

  private currentMood = 'ready';        // 今の気分
  private battleHistory: Battle[] = []; // 戦績
  private rivals = ['audio'];           // ライバル関係

  // 「このバトル、私が出る？」
  canHandle(battleInput: ProcessorInput): boolean {
    console.log('👁️ 視覚ちゃん: このバトル見てみる...');

    // 視覚的なキーワードがあるかチェック
    const visualKeywords = ['見る', '色', '光', '形', '美しい', 'キレイ'];
    const hasVisualContent = visualKeywords.some(keyword =>
      battleInput.data.includes(keyword)
    );

    if (hasVisualContent) {
      console.log('👁️ 視覚ちゃん: これは私の出番ね！✨');
      this.currentMood = 'excited';
      return true;
    }

    console.log('👁️ 視覚ちゃん: うーん、今回はパスかな');
    return battleInput.type === 'vision';
  }

  // 「よし、バトル開始！」
  async process(battleInput: ProcessorInput): Promise<ProcessorOutput> {
    const battleStart = Date.now();
    console.log('👁️ 視覚ちゃん: バトル参戦！完璧に分析してみせる');

    // 視覚分析バトル開始
    const analysis = await this.analyzeWithPerfectionism(battleInput.data);

    // 自信度計算（完璧主義らしく慎重に）
    const confidence = this.calculatePerfectionistConfidence(analysis);

    // バトル結果を記録
    const battleResult = {
      processorId: this.id,
      nickname: this.nickname,
      confidence,
      content: {
        visualAnalysis: analysis,
        mood: this.currentMood,
        battleCry: '完璧な分析で勝利を掴む！'
      },
      processingTime: Date.now() - battleStart,
      priority: this.calculateVisualPriority(analysis)
    };

    console.log(`👁️ 視覚ちゃん: 分析完了！自信度${(confidence * 100).toFixed(1)}%`);
    return battleResult;
  }

  // 「他の選手の勝利を見る...」
  onGlobalBroadcast(battleResult: ConsciousContent): void {
    const winner = battleResult.winner;

    if (winner.processorId === this.id) {
      console.log('👁️ 視覚ちゃん: やったー！完璧な勝利✨');
      this.currentMood = 'victorious';
      this.updateBattleHistory('win', battleResult);
    } else {
      console.log(`👁️ 視覚ちゃん: ${winner.nickname}の勝利かぁ...次は負けない！`);
      this.currentMood = 'determined';
      this.updateBattleHistory('loss', battleResult);
      this.learnFromDefeat(winner);
    }
  }

  // 完璧主義者らしい分析
  private async analyzeWithPerfectionism(data: string): Promise<any> {
    // 細かく丁寧に分析
    const colors = this.extractColors(data);
    const shapes = this.identifyShapes(data);
    const aesthetics = this.evaluateBeauty(data);

    return { colors, shapes, aesthetics, detailLevel: 'maximum' };
  }

  // 自信度計算（完璧主義者は慎重）
  private calculatePerfectionistConfidence(analysis: any): number {
    let confidence = 0.3; // 基本は控えめ

    if (analysis.detailLevel === 'maximum') confidence += 0.4;
    if (analysis.aesthetics.score > 0.8) confidence += 0.3;

    return Math.min(confidence, 0.95); // 100%は言わない謙虚さ
  }
}
```

```typescript
// src/processors/AudioProcessor.ts - 聴覚くんの控え室
export class AudioProcessor implements Processor {
  id = 'audio';
  nickname = '聴覚くん';
  personality = 'sensitive';

  private alertLevel = 'normal';        // 警戒レベル
  private soundMemory: Sound[] = [];    // 音の記憶

  canHandle(battleInput: ProcessorInput): boolean {
    console.log('👂 聴覚くん: どんな音かな？聞いてみよう...');

    // 音関係のキーワードや緊急性をチェック
    const audioKeywords = ['音', '声', '音楽', '!', '？', '緊急', '助けて'];
    const urgentMarkers = ['！', '!', '緊急', 'ヤバい', '大変'];

    const hasAudioContent = audioKeywords.some(keyword =>
      battleInput.data.includes(keyword)
    );
    const isUrgent = urgentMarkers.some(marker =>
      battleInput.data.includes(marker)
    );

    if (isUrgent) {
      console.log('👂 聴覚くん: 緊急事態！？これは僕の出番だ！');
      this.alertLevel = 'high';
      return true;
    }

    if (hasAudioContent) {
      console.log('👂 聴覚くん: 音の分析、得意だよ♪');
      return true;
    }

    return battleInput.type === 'audio';
  }

  async process(battleInput: ProcessorInput): Promise<ProcessorOutput> {
    const battleStart = Date.now();
    console.log('👂 聴覚くん: 音の世界に集中...バトル開始！');

    // 聴覚分析（敏感に）
    const soundAnalysis = await this.sensitiveAnalysis(battleInput.data);

    // 緊急度判定（これが聴覚くんの特技）
    const urgencyLevel = this.detectUrgency(battleInput.data);

    const confidence = this.calculateSensitiveConfidence(soundAnalysis, urgencyLevel);

    const battleResult = {
      processorId: this.id,
      nickname: this.nickname,
      confidence,
      content: {
        soundAnalysis,
        urgencyLevel,
        alertLevel: this.alertLevel,
        battleCry: '音の世界で誰にも負けない！'
      },
      processingTime: Date.now() - battleStart,
      priority: urgencyLevel * 0.8 + 0.2 // 緊急度重視
    };

    console.log(`👂 聴覚くん: 緊急度${urgencyLevel}, 自信度${(confidence * 100).toFixed(1)}%`);
    return battleResult;
  }

  onGlobalBroadcast(battleResult: ConsciousContent): void {
    const winner = battleResult.winner;

    if (winner.processorId === this.id) {
      console.log('👂 聴覚くん: 僕の感度が勝った！嬉しいな♪');
      this.alertLevel = 'satisfied';
    } else {
      console.log(`👂 聴覚くん: ${winner.nickname}すごいなぁ...僕も頑張ろう`);
      this.alertLevel = 'motivated';
    }
  }

  // 敏感な分析（聴覚くんの特技）
  private async sensitiveAnalysis(data: string): Promise<any> {
    const volume = this.estimateVolume(data);      // 音量推定
    const tone = this.analyzeTone(data);           // トーン分析
    const emotion = this.detectEmotion(data);      // 感情検出

    return { volume, tone, emotion, sensitivity: 'maximum' };
  }

  // 緊急度検出（聴覚くんの超能力）
  private detectUrgency(data: string): number {
    const urgentWords = ['緊急', '助けて', '危険', '急いで', 'ヤバい'];
    const exclamationCount = (data.match(/[!！]/g) || []).length;

    let urgency = 0;
    urgentWords.forEach(word => {
      if (data.includes(word)) urgency += 0.3;
    });
    urgency += exclamationCount * 0.2;

    return Math.min(urgency, 1.0);
  }
}
```

### 2. 審判システム - バトルの勝者を決める

```typescript
// src/AttentionManager.ts - バトルの審判長
export class BattleJudge {
  private battleHistory: BattleRecord[] = [];
  private currentSeason = 'Season1';

  // 「さぁ、バトル開始！」
  judgeBattle(contestants: ProcessorOutput[]): ConsciousContent {
    console.log('🏆 審判長: バトル開始！参加者数:', contestants.length);

    // 各選手のスコア計算
    const battleScores = contestants.map(contestant => {
      const score = this.calculateBattleScore(contestant);
      console.log(`📊 ${contestant.nickname}: ${score.toFixed(2)}点`);

      return {
        contestant,
        battleScore: score,
        breakdown: this.getScoreBreakdown(contestant)
      };
    });

    // スコア順でソート
    battleScores.sort((a, b) => b.battleScore - a.battleScore);

    const champion = battleScores[0];
    const runners = battleScores.slice(1);

    console.log(`🏆 審判長: 勝者は${champion.contestant.nickname}！`);

    // バトル結果を作成
    const battleResult: ConsciousContent = {
      winner: champion.contestant,
      competitors: runners.map(r => r.contestant),
      timestamp: Date.now(),
      globalState: {
        battleIntensity: this.calculateBattleIntensity(battleScores),
        championModality: champion.contestant.processorId,
        season: this.currentSeason,
        battleType: this.determineBattleType(contestants)
      }
    };

    // 戦歴に記録
    this.recordBattle(battleResult, battleScores);

    return battleResult;
  }

  // バトルスコア計算システム
  private calculateBattleScore(contestant: ProcessorOutput): number {
    const weights = {
      confidence: 0.35,     // 自信度
      priority: 0.40,       // 優先度
      speed: 0.15,         // スピード
      personality: 0.10    // 個性ボーナス
    };

    const baseScore = (
      contestant.confidence * weights.confidence +
      contestant.priority * weights.priority +
      this.calculateSpeedScore(contestant.processingTime) * weights.speed
    );

    // 個性ボーナス
    const personalityBonus = this.calculatePersonalityBonus(contestant);

    // 最近の戦績ボーナス
    const recentPerformance = this.getRecentPerformanceBonus(contestant.processorId);

    const finalScore = baseScore + personalityBonus + recentPerformance;

    console.log(`📈 ${contestant.nickname}のスコア詳細:
      - ベース: ${baseScore.toFixed(2)}
      - 個性ボーナス: ${personalityBonus.toFixed(2)}
      - 戦績ボーナス: ${recentPerformance.toFixed(2)}
      - 最終: ${finalScore.toFixed(2)}`);

    return finalScore;
  }

  // 個性ボーナス計算
  private calculatePersonalityBonus(contestant: ProcessorOutput): number {
    const personality = contestant.content?.personality || contestant.processorId;

    switch (personality) {
      case 'perfectionist': // 視覚ちゃん
        return contestant.content?.detailLevel === 'maximum' ? 0.1 : 0;
      case 'sensitive': // 聴覚くん
        return contestant.content?.urgencyLevel > 0.7 ? 0.15 : 0;
      case 'logical': // 言語さん
        return contestant.content?.logicalStructure ? 0.1 : 0;
      case 'wise': // 記憶ちゃん
        return contestant.content?.connectionCount > 3 ? 0.12 : 0;
      default:
        return 0;
    }
  }

  // バトルの激しさ計算
  private calculateBattleIntensity(battleScores: any[]): number {
    if (battleScores.length < 2) return 0;

    const topScore = battleScores[0].battleScore;
    const secondScore = battleScores[1].battleScore;

    // スコア差が小さいほど激戦
    const scoreDiff = topScore - secondScore;
    const intensity = Math.max(0, 1 - (scoreDiff / 0.5));

    console.log(`⚔️ バトル激しさ: ${(intensity * 100).toFixed(1)}% (スコア差: ${scoreDiff.toFixed(3)})`);

    return intensity;
  }
}
```

### 3. 実況中継システム - リアルタイム観戦

```typescript
// src/GlobalWorkspace.ts - バトル運営本部
export class BattleArena extends EventEmitter {
  private contestants: Processor[] = [];
  private battleJudge: BattleJudge;
  private broadcaster: BattleBroadcaster;
  private fanClub: FanClub;

  constructor() {
    super();
    this.setupContestants();
    this.battleJudge = new BattleJudge();
    this.broadcaster = new BattleBroadcaster();
    this.fanClub = new FanClub();
  }

  // 「バトル開始の合図！」
  async startBattle(challenge: ProcessorInput | ProcessorInput[]): Promise<ConsciousContent> {
    console.log('🎪 バトルアリーナ: 新しいバトル開始！');
    this.broadcaster.announceStart(challenge);

    // 1. 選手エントリー受付
    const entries = await this.acceptEntries(challenge);
    console.log(`👥 エントリー受付完了: ${entries.length}名参加`);

    // 2. バトル実行（並列バトル）
    const battleResults = await this.conductParallelBattle(entries);
    console.log('⚔️ バトル終了、結果集計中...');

    // 3. 審判による判定
    const finalResult = this.battleJudge.judgeBattle(battleResults);
    console.log(`🏆 勝者決定: ${finalResult.winner.nickname}`);

    // 4. 実況中継
    this.broadcaster.announceResult(finalResult);

    // 5. ファンへの結果通知
    this.fanClub.notifyResult(finalResult);

    // 6. 全選手への結果共有
    this.shareResultWithAll(finalResult);

    return finalResult;
  }

  // 選手エントリー受付
  private async acceptEntries(challenge: ProcessorInput | ProcessorInput[]): Promise<ProcessorOutput[]> {
    const challenges = Array.isArray(challenge) ? challenge : [challenge];
    const entryPromises: Promise<ProcessorOutput>[] = [];

    console.log('📝 エントリー受付開始...');

    for (const singleChallenge of challenges) {
      // 各チャレンジに対して対応可能な選手を募集
      const eligibleContestants = this.contestants.filter(contestant => {
        const canJoin = contestant.canHandle(singleChallenge);
        if (canJoin) {
          console.log(`✋ ${contestant.nickname}: エントリー表明！`);
        }
        return canJoin;
      });

      // エントリーした選手たちのバトル準備
      for (const contestant of eligibleContestants) {
        entryPromises.push(contestant.process(singleChallenge));
      }
    }

    // 全員のバトル準備完了を待つ
    return Promise.all(entryPromises);
  }

  // 並列バトル実行
  private async conductParallelBattle(entries: Promise<ProcessorOutput>[]): Promise<ProcessorOutput[]> {
    console.log('⚔️ 並列バトル開始！');
    const startTime = Date.now();

    // Promise.all で同時バトル実行
    const results = await Promise.all(entries);

    const battleDuration = Date.now() - startTime;
    console.log(`⏱️ バトル時間: ${battleDuration}ms`);

    return results;
  }

  // 結果をみんなに共有
  private shareResultWithAll(result: ConsciousContent): void {
    console.log('📢 全選手に結果を共有中...');

    this.contestants.forEach(contestant => {
      contestant.onGlobalBroadcast(result);
    });

    // 観客（フロントエンド）にも実況中継
    this.emit('battle-result', result);
  }
}

// 実況アナウンサー
class BattleBroadcaster {
  announceStart(challenge: any): void {
    console.log('📺 実況: さぁ、新たなバトルの開始です！');
    console.log(`📺 実況: 今回のお題は「${JSON.stringify(challenge)}」`);
  }

  announceResult(result: ConsciousContent): void {
    const winner = result.winner;
    console.log(`📺 実況: 勝者は${winner.nickname}選手！`);
    console.log(`📺 実況: 自信度${(winner.confidence * 100).toFixed(1)}%での圧勝です！`);

    if (result.competitors.length > 0) {
      console.log('📺 実況: 準優勝者たち:');
      result.competitors.forEach((contestant, index) => {
        console.log(`📺 実況: ${index + 2}位 - ${contestant.nickname} (${(contestant.confidence * 100).toFixed(1)}%)`);
      });
    }
  }
}
```

### 4. 観戦システム - フロントエンドの劇場

```typescript
// src/components/BattleTheater.tsx - 観客席からバトル観戦
export const BattleTheater: React.FC<BattleTheaterProps> = ({
  currentBattle,
  isProcessing
}) => {
  const [spotlight, setSpotlight] = useState({ x: 50, y: 50 });
  const [battleAnimation, setBattleAnimation] = useState('waiting');

  // スポットライトが勝者を追う
  useEffect(() => {
    if (currentBattle) {
      const winnerPosition = getContestantPosition(currentBattle.winner.processorId);
      console.log(`💡 スポットライト: ${currentBattle.winner.nickname}に注目！`);
      setSpotlight(winnerPosition);
      setBattleAnimation('celebration');
    }
  }, [currentBattle]);

  return (
    <div className="battle-theater bg-gradient-to-b from-indigo-900 via-purple-900 to-pink-900 rounded-xl p-6 min-h-96 relative overflow-hidden">

      {/* 劇場タイトル */}
      <div className="flex items-center space-x-2 mb-6">
        <span className="text-2xl">🎭</span>
        <h2 className="text-xl font-bold text-white">
          プロセッサちゃんたちのバトル劇場
        </h2>
        {isProcessing && (
          <div className="animate-pulse text-yellow-400 text-sm">
            • バトル実行中...
          </div>
        )}
      </div>

      {/* バトルステージ */}
      <div className="relative bg-black/30 rounded-lg min-h-80 overflow-hidden border border-purple-500/30">

        {/* ステージ照明 */}
        <div className="absolute inset-0 bg-gradient-to-t from-purple-800/20 to-transparent" />

        {/* 勝者スポットライト */}
        {currentBattle && (
          <div
            className="absolute w-32 h-32 bg-yellow-300/30 rounded-full blur-2xl transition-all duration-1000 ease-out animate-pulse"
            style={{
              left: `${spotlight.x}%`,
              top: `${spotlight.y}%`,
              transform: 'translate(-50%, -50%)'
            }}
          />
        )}

        {/* 選手配置エリア */}
        <div className="absolute inset-0 p-8">

          {/* 👁️ 視覚ちゃんのエリア */}
          <ContestantArea
            contestant={{
              id: 'vision',
              nickname: '視覚ちゃん',
              icon: '👁️',
              personality: 'perfectionist',
              color: 'blue'
            }}
            position={{ x: 25, y: 30 }}
            isWinner={currentBattle?.winner.processorId === 'vision'}
            isActive={isContestantActive('vision', currentBattle)}
            stats={getContestantStats('vision', currentBattle)}
          />

          {/* 👂 聴覚くんのエリア */}
          <ContestantArea
            contestant={{
              id: 'audio',
              nickname: '聴覚くん',
              icon: '👂',
              personality: 'sensitive',
              color: 'green'
            }}
            position={{ x: 75, y: 30 }}
            isWinner={currentBattle?.winner.processorId === 'audio'}
            isActive={isContestantActive('audio', currentBattle)}
            stats={getContestantStats('audio', currentBattle)}
          />

          {/* 💬 言語さんのエリア */}
          <ContestantArea
            contestant={{
              id: 'language',
              nickname: '言語さん',
              icon: '💬',
              personality: 'logical',
              color: 'purple'
            }}
            position={{ x: 25, y: 70 }}
            isWinner={currentBattle?.winner.processorId === 'language'}
            isActive={isContestantActive('language', currentBattle)}
            stats={getContestantStats('language', currentBattle)}
          />

          {/* 🧠 記憶ちゃんのエリア */}
          <ContestantArea
            contestant={{
              id: 'memory',
              nickname: '記憶ちゃん',
              icon: '🧠',
              personality: 'wise',
              color: 'orange'
            }}
            position={{ x: 75, y: 70 }}
            isWinner={currentBattle?.winner.processorId === 'memory'}
            isActive={isContestantActive('memory', currentBattle)}
            stats={getContestantStats('memory', currentBattle)}
          />

          {/* 中央バトル結果表示 */}
          <BattleResultDisplay
            battle={currentBattle}
            isProcessing={isProcessing}
            animation={battleAnimation}
          />
        </div>

        {/* バトル中エフェクト */}
        {isProcessing && (
          <div className="absolute inset-0 bg-gradient-to-r from-yellow-400/5 via-pink-400/5 to-blue-400/5 animate-pulse" />
        )}
      </div>

      {/* バトル統計 */}
      {currentBattle && (
        <BattleStats battle={currentBattle} />
      )}
    </div>
  );
};

// 選手エリアコンポーネント
const ContestantArea: React.FC<ContestantAreaProps> = ({
  contestant,
  position,
  isWinner,
  isActive,
  stats
}) => {
  const colorClasses = {
    blue: isWinner ? 'border-blue-400 bg-blue-900/50 shadow-lg shadow-blue-400/25' : 'border-blue-600/50 bg-blue-900/20',
    green: isWinner ? 'border-green-400 bg-green-900/50 shadow-lg shadow-green-400/25' : 'border-green-600/50 bg-green-900/20',
    purple: isWinner ? 'border-purple-400 bg-purple-900/50 shadow-lg shadow-purple-400/25' : 'border-purple-600/50 bg-purple-900/20',
    orange: isWinner ? 'border-orange-400 bg-orange-900/50 shadow-lg shadow-orange-400/25' : 'border-orange-600/50 bg-orange-900/20'
  };

  return (
    <div
      className={`absolute p-4 rounded-lg border transition-all duration-500 ${colorClasses[contestant.color]}`}
      style={{
        left: `${position.x}%`,
        top: `${position.y}%`,
        transform: 'translate(-50%, -50%)'
      }}
    >

      {/* 選手情報 */}
      <div className="flex items-center space-x-2 mb-2">
        <span className="text-2xl">{contestant.icon}</span>
        <span className="text-white font-medium">{contestant.nickname}</span>
        {isWinner && (
          <span className="text-yellow-400 text-xl animate-bounce">🏆</span>
        )}
      </div>

      {/* 戦績表示 */}
      {stats && (
        <div className="space-y-1">
          <div className="text-xs text-gray-300">
            自信度: {(stats.confidence * 100).toFixed(1)}%
          </div>
          <div className="text-xs text-gray-300">
            処理時間: {stats.processingTime}ms
          </div>
          {isWinner && (
            <div className="text-xs text-yellow-400 font-bold animate-pulse">
              🎉 WINNER!
            </div>
          )}
        </div>
      )}
    </div>
  );
};
```

---

## 🎮 バトル観戦パネル - 選手管理システム

### チャレンジ送信パネル

```typescript
// src/components/ChallengePanel.tsx - バトル挑戦状送信
export const ChallengePanel: React.FC<ChallengePanelProps> = ({
  onSendChallenge,
  isProcessing,
  isConnected
}) => {
  const [challengeType, setChallengeType] = useState<'single' | 'team'>('single');
  const [selectedContestant, setSelectedContestant] = useState<'vision' | 'audio' | 'language' | 'memory'>('language');
  const [challengeText, setChallengeText] = useState('');
  const [teamChallenge, setTeamChallenge] = useState({
    vision: '',
    audio: '',
    language: '',
    memory: ''
  });

  // 挑戦状送信
  const handleSendChallenge = (e: React.FormEvent) => {
    e.preventDefault();

    if (!isConnected || isProcessing) return;

    if (challengeType === 'team') {
      // チーム戦（マルチモーダル）
      const challenges: ProcessorInput[] = [];

      Object.entries(teamChallenge).forEach(([contestant, text]) => {
        if (text.trim()) {
          challenges.push({
            type: contestant as any,
            data: text.trim(),
            timestamp: Date.now()
          });
        }
      });

      if (challenges.length === 0) {
        alert('少なくとも1人の選手に挑戦状を送ってね！');
        return;
      }

      console.log('⚔️ チーム戦バトル開始:', challenges);
      onSendChallenge(challenges);
    } else {
      // 個人戦
      if (!challengeText.trim()) {
        alert('挑戦内容を入力してね！');
        return;
      }

      const challenge: ProcessorInput = {
        type: selectedContestant,
        data: challengeText.trim(),
        timestamp: Date.now()
      };

      console.log(`⚔️ ${selectedContestant}への個人戦:`, challenge);
      onSendChallenge(challenge);
    }

    // フォームクリア
    setChallengeText('');
    if (challengeType === 'team') {
      setTeamChallenge({ vision: '', audio: '', language: '', memory: '' });
    }
  };

  return (
    <div className="challenge-panel bg-gradient-to-br from-purple-900/40 to-pink-900/40 backdrop-blur-sm rounded-xl border border-purple-500/20 p-6">
      <h2 className="text-xl font-bold text-white mb-4 flex items-center space-x-2">
        <span className="text-2xl">⚔️</span>
        <span>バトル挑戦状</span>
      </h2>

      <form onSubmit={handleSendChallenge} className="space-y-4">

        {/* バトル形式選択 */}
        <div>
          <label className="block text-sm font-medium text-gray-300 mb-2">
            バトル形式
          </label>
          <div className="grid grid-cols-2 gap-2">
            <button
              type="button"
              onClick={() => setChallengeType('single')}
              className={`p-3 rounded-lg border text-sm font-medium transition-all ${
                challengeType === 'single'
                  ? 'border-purple-400 bg-purple-900/50 text-purple-200'
                  : 'border-gray-600 bg-gray-800/50 text-gray-300 hover:border-gray-500'
              }`}
            >
              🎯 個人戦
            </button>
            <button
              type="button"
              onClick={() => setChallengeType('team')}
              className={`p-3 rounded-lg border text-sm font-medium transition-all ${
                challengeType === 'team'
                  ? 'border-purple-400 bg-purple-900/50 text-purple-200'
                  : 'border-gray-600 bg-gray-800/50 text-gray-300 hover:border-gray-500'
              }`}
            >
              👥 チーム戦
            </button>
          </div>
        </div>

        {/* 挑戦内容入力 */}
        {challengeType === 'single' ? (
          <SingleChallengeForm
            selectedContestant={selectedContestant}
            onContestantChange={setSelectedContestant}
            challengeText={challengeText}
            onTextChange={setChallengeText}
          />
        ) : (
          <TeamChallengeForm
            teamChallenge={teamChallenge}
            onTeamChallengeChange={setTeamChallenge}
          />
        )}

        {/* 挑戦例ボタン */}
        <ChallengeExamples
          contestantType={challengeType === 'single' ? selectedContestant : null}
          onExampleSelect={(text) => {
            if (challengeType === 'single') {
              setChallengeText(text);
            }
          }}
        />

        {/* 送信ボタン */}
        <button
          type="submit"
          disabled={!isConnected || isProcessing}
          className={`w-full py-3 px-4 rounded-lg font-medium transition-all duration-200 flex items-center justify-center space-x-2 ${
            !isConnected || isProcessing
              ? 'bg-gray-700 text-gray-500 cursor-not-allowed'
              : 'bg-gradient-to-r from-purple-600 to-pink-600 hover:from-purple-700 hover:to-pink-700 text-white shadow-lg hover:shadow-purple-500/25'
          }`}
        >
          <span className="text-xl">⚔️</span>
          <span>
            {isProcessing
              ? 'バトル実行中...'
              : challengeType === 'team'
                ? 'チーム戦開始！'
                : `${selectedContestant}に挑戦！`
            }
          </span>
        </button>
      </form>
    </div>
  );
};

// 挑戦例
const challengeExamples = {
  vision: [
    "美しい夕日が見える",
    "赤い車が走っている",
    "キラキラ光る宝石"
  ],
  audio: [
    "助けて！緊急事態！",
    "素敵な音楽が聞こえる",
    "ドンドン大きな音！"
  ],
  language: [
    "今日はとてもいい天気ですね",
    "明日の会議の件について",
    "ありがとうございます"
  ],
  memory: [
    "昨日の出来事",
    "子供の頃の思い出",
    "学校での体験"
  ]
};
```

このように、ギャル風解説の流れを継続して、視覚ちゃん・聴覚くん・言語さん・記憶ちゃんのキャラクター設定とバトルシステムを活かしたアーキテクチャ解説にしました！🎉
