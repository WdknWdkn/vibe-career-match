# キャリアマッチAI ライト版 - 開発仕様書

## 1. プロジェクト概要

### 1.1 プロジェクト名
**キャリアマッチAI Lite** (Career Match AI Lite)

### 1.2 目的
AIのWeb検索機能を活用し、ユーザーの価値観分析に基づいて求人をマッチングする軽量Webアプリケーション

### 1.3 技術的特徴
- **フロントエンドのみで動作**（バックエンド不要）
- **データベース不要**（ローカルストレージ使用）
- **認証機能なし**（即座に利用可能）
- **AIのWeb検索API連携**による求人取得

### 1.4 開発期間
- MVP開発: 2週間
- 完成版: 1ヶ月

## 2. 機能要件

### 2.1 パーソナリティ分析機能

#### 2.1.1 MBTI診断
```typescript
interface MBTISelection {
  types: ['INTJ', 'INTP', 'ENTJ', 'ENTP', 'INFJ', 'INFP', 'ENFJ', 'ENFP',
          'ISTJ', 'ISFJ', 'ESTJ', 'ESFJ', 'ISTP', 'ISFP', 'ESTP', 'ESFP'];
  selectedType: string | null;
  saveToLocalStorage: () => void;
}
```

#### 2.1.2 価値観トレードオフ質問（30問）
```typescript
interface Question {
  id: string;
  content: string;
  category: ValueCategory;
  weight: 1 | 2 | 3;
}

type ValueCategory = 
  | 'salary'      // 給与・経済的報酬
  | 'growth'      // 成長・キャリア
  | 'worklife'    // ワークライフバランス
  | 'social'      // 社会貢献
  | 'stability'   // 安定性
  | 'challenge'   // チャレンジ・革新
  | 'skill'       // 専門性・スキル
  | 'relationship'// 人間関係
  | 'culture';    // 企業文化

interface Answer {
  questionId: string;
  value: 1 | 2 | 3 | 4 | 5; // 1:強く反対 ~ 5:強く同意
}
```

#### 2.1.3 価値観スコア計算
```javascript
function calculateValueScores(answers) {
  const scores = {};
  const maxPossible = {};
  
  questions.forEach(question => {
    const answer = answers[question.id] || 3;
    
    if (!scores[question.category]) {
      scores[question.category] = 0;
      maxPossible[question.category] = 0;
    }
    
    scores[question.category] += answer * question.weight;
    maxPossible[question.category] += 5 * question.weight;
  });
  
  // 正規化（0-100）
  Object.keys(scores).forEach(category => {
    scores[category] = Math.round((scores[category] / maxPossible[category]) * 100);
  });
  
  return scores;
}
```

### 2.2 AI求人検索機能

#### 2.2.1 検索クエリ生成
```javascript
class JobSearchQueryBuilder {
  /**
   * ユーザープロファイルから検索クエリを生成
   */
  static buildQuery(profile) {
    const { mbti, values, strengths, careerVision } = profile;
    
    // 価値観の上位3つを取得
    const topValues = Object.entries(values)
      .sort(([,a], [,b]) => b - a)
      .slice(0, 3)
      .map(([key]) => this.getValueKeywords(key));
    
    // 検索クエリの構築
    const queries = [
      this.buildIndustryQuery(topValues, careerVision),
      this.buildCompanyTypeQuery(values),
      this.buildSkillQuery(strengths),
      this.buildLocationQuery(profile.preferredLocation)
    ];
    
    return queries;
  }
  
  static getValueKeywords(valueCategory) {
    const keywords = {
      'salary': '高収入 年収 報酬',
      'growth': 'キャリアアップ 成長 スキルアップ',
      'worklife': 'ワークライフバランス フレックス リモート',
      'social': '社会貢献 SDGs ESG',
      'stability': '安定 大手企業 福利厚生',
      'challenge': 'スタートアップ ベンチャー 新規事業',
      'skill': '専門性 技術 研究開発',
      'relationship': 'チームワーク 社風 働きやすい',
      'culture': '企業理念 価値観 ビジョン'
    };
    return keywords[valueCategory] || '';
  }
}
```

#### 2.2.2 AI Web検索インターフェース
```javascript
class AIJobSearcher {
  constructor() {
    this.searchEndpoint = '/api/ai-search'; // AIのWeb検索エンドポイント
  }
  
  /**
   * AI Web検索を実行
   * 注: 実際の実装では、ChatGPT/Claude APIのWeb検索機能を呼び出す
   */
  async searchJobs(profile) {
    const queries = JobSearchQueryBuilder.buildQuery(profile);
    const searchPrompt = this.createSearchPrompt(profile, queries);
    
    // AI Web検索の実行をシミュレート
    const searchButton = document.getElementById('ai-search-button');
    searchButton.onclick = () => {
      this.displaySearchPrompt(searchPrompt);
      // ユーザーがAIに検索を依頼
    };
  }
  
  createSearchPrompt(profile, queries) {
    return `
以下の条件で日本の求人情報を検索してください：

【検索条件】
1. 価値観重視ポイント：
   ${this.formatTopValues(profile.values)}

2. MBTI性格タイプ：${profile.mbti}

3. 希望条件：
   - キャリアビジョン：${profile.careerVision}
   - 強み：${profile.strengths.join(', ')}
   - 希望勤務地：${profile.preferredLocation || '指定なし'}

【検索クエリ例】
${queries.map((q, i) => `${i+1}. ${q}`).join('\n')}

以下の求人サイトから最新の情報を検索してください：
- Indeed
- リクナビNEXT
- マイナビ転職
- ビズリーチ
- Wantedly
- Green（IT系）
- 各企業の採用ページ

検索結果は以下の形式でまとめてください：
1. 企業名
2. 職種
3. 給与レンジ
4. 勤務地
5. 必要スキル
6. 企業の特徴（価値観マッチの観点から）
7. 応募締切（わかれば）
8. 参照URL
    `;
  }
  
  formatTopValues(values) {
    return Object.entries(values)
      .sort(([,a], [,b]) => b - a)
      .slice(0, 3)
      .map(([key, score]) => `${this.getValueLabel(key)}: ${score}%`)
      .join('\n   ');
  }
}
```

### 2.3 データ管理（ローカルストレージ）

#### 2.3.1 ローカルストレージ管理
```javascript
class LocalStorageManager {
  static KEYS = {
    PROFILE: 'career_match_profile',
    ANSWERS: 'career_match_answers',
    VALUES: 'career_match_values',
    SEARCH_HISTORY: 'career_match_search_history',
    SAVED_JOBS: 'career_match_saved_jobs'
  };
  
  static saveProfile(profile) {
    localStorage.setItem(this.KEYS.PROFILE, JSON.stringify(profile));
  }
  
  static getProfile() {
    const data = localStorage.getItem(this.KEYS.PROFILE);
    return data ? JSON.parse(data) : null;
  }
  
  static saveAnswers(answers) {
    localStorage.setItem(this.KEYS.ANSWERS, JSON.stringify(answers));
  }
  
  static getAnswers() {
    const data = localStorage.getItem(this.KEYS.ANSWERS);
    return data ? JSON.parse(data) : {};
  }
  
  static saveSearchResult(result) {
    const history = this.getSearchHistory();
    history.push({
      ...result,
      timestamp: new Date().toISOString()
    });
    // 最新10件のみ保持
    const recent = history.slice(-10);
    localStorage.setItem(this.KEYS.SEARCH_HISTORY, JSON.stringify(recent));
  }
  
  static getSearchHistory() {
    const data = localStorage.getItem(this.KEYS.SEARCH_HISTORY);
    return data ? JSON.parse(data) : [];
  }
  
  static clearAll() {
    Object.values(this.KEYS).forEach(key => {
      localStorage.removeItem(key);
    });
  }
}
```

### 2.4 求人表示・管理機能

#### 2.4.1 求人カード表示
```javascript
class JobCard {
  constructor(jobData) {
    this.data = jobData;
  }
  
  render() {
    return `
      <div class="job-card">
        <div class="job-header">
          <h3>${this.data.title}</h3>
          <span class="company">${this.data.company}</span>
        </div>
        <div class="job-details">
          <span class="salary">💰 ${this.data.salary || '要相談'}</span>
          <span class="location">📍 ${this.data.location}</span>
        </div>
        <div class="job-match">
          <div class="match-score">${this.calculateMatch()}%</div>
          <div class="match-reasons">
            ${this.getMatchReasons().join(', ')}
          </div>
        </div>
        <div class="job-actions">
          <button onclick="saveJob('${this.data.id}')">保存</button>
          <button onclick="copyJobInfo('${this.data.id}')">コピー</button>
          <a href="${this.data.url}" target="_blank">詳細を見る</a>
        </div>
      </div>
    `;
  }
  
  calculateMatch() {
    // ユーザーの価値観と求人の特徴を比較
    const profile = LocalStorageManager.getProfile();
    // マッチング計算ロジック
    return Math.floor(Math.random() * 30 + 70); // 仮実装
  }
  
  getMatchReasons() {
    // マッチング理由の生成
    return ['成長機会', 'ワークライフバランス'];
  }
}
```

## 3. 技術スタック（シンプル構成）

### 3.1 フロントエンドのみ
```json
{
  "framework": "React (Create React App) または Vanilla JavaScript",
  "styling": "Tailwind CSS (CDN)",
  "storage": "LocalStorage API",
  "deployment": "Vercel / Netlify / GitHub Pages",
  "dependencies": {
    "react": "^18.2.0",
    "tailwindcss": "^3.3.0"
  }
}
```

### 3.2 単一HTMLファイル版（最もシンプル）
```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>キャリアマッチAI Lite</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body>
  <!-- アプリケーション全体を1ファイルに -->
  <script>
    // 全ロジックをここに実装
  </script>
</body>
</html>
```

## 4. UI/UX設計

### 4.1 画面構成
```
┌─────────────────────────────────────┐
│          キャリアマッチAI Lite       │
├─────────────────────────────────────┤
│  [STEP 1]  │  [STEP 2]  │  [STEP 3]  │
│  性格診断  │ 価値観診断 │ 求人検索   │
├─────────────────────────────────────┤
│                                     │
│         メインコンテンツエリア        │
│                                     │
└─────────────────────────────────────┘
```

### 4.2 ステップ別画面

#### STEP 1: MBTI診断
```javascript
const MBTISelector = () => {
  const types = ['INTJ', 'INTP', /* ... */];
  
  return (
    <div className="grid grid-cols-4 gap-4">
      {types.map(type => (
        <button 
          key={type}
          className="p-4 border rounded hover:bg-blue-100"
          onClick={() => selectMBTI(type)}
        >
          {type}
        </button>
      ))}
    </div>
  );
};
```

#### STEP 2: 価値観診断
```javascript
const ValueAssessment = () => {
  return (
    <div className="space-y-6">
      {questions.map((q, index) => (
        <QuestionCard 
          key={q.id}
          question={q}
          number={index + 1}
          onAnswer={(value) => saveAnswer(q.id, value)}
        />
      ))}
      <button 
        className="btn-primary"
        onClick={calculateAndSaveValues}
      >
        診断結果を見る
      </button>
    </div>
  );
};
```

#### STEP 3: AI求人検索
```javascript
const JobSearch = () => {
  const [searchPrompt, setSearchPrompt] = useState('');
  const [searchResults, setSearchResults] = useState([]);
  
  const generateSearchPrompt = () => {
    const profile = LocalStorageManager.getProfile();
    const prompt = AIJobSearcher.createSearchPrompt(profile);
    setSearchPrompt(prompt);
  };
  
  return (
    <div>
      <div className="bg-gray-100 p-6 rounded">
        <h3>AI検索プロンプト</h3>
        <pre className="text-sm">{searchPrompt}</pre>
        <button 
          className="btn-primary mt-4"
          onClick={generateSearchPrompt}
        >
          検索プロンプトを生成
        </button>
        <button 
          className="btn-secondary mt-4 ml-2"
          onClick={() => copyToClipboard(searchPrompt)}
        >
          プロンプトをコピー
        </button>
      </div>
      
      <div className="mt-6">
        <h3>検索結果入力エリア</h3>
        <textarea 
          className="w-full h-64 p-4 border rounded"
          placeholder="AIの検索結果をここに貼り付けてください"
          onChange={(e) => parseSearchResults(e.target.value)}
        />
      </div>
      
      <div className="mt-6 space-y-4">
        {searchResults.map(job => (
          <JobCard key={job.id} job={job} />
        ))}
      </div>
    </div>
  );
};
```

## 5. 実装コード例

### 5.1 完全なHTML実装例
```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>キャリアマッチAI Lite</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        .mbti-option.selected { background-color: #5e72e4; color: white; }
    </style>
</head>
<body class="bg-gray-50">
    <div class="container mx-auto max-w-6xl p-6">
        <!-- ヘッダー -->
        <header class="bg-gradient-to-r from-blue-600 to-purple-600 text-white p-6 rounded-lg mb-6">
            <h1 class="text-3xl font-bold">キャリアマッチAI Lite</h1>
            <p class="mt-2">AIを活用した価値観ベースの求人マッチング</p>
        </header>

        <!-- タブナビゲーション -->
        <div class="flex space-x-4 mb-6">
            <button onclick="switchTab('personality')" class="tab-btn px-6 py-3 bg-white rounded-lg shadow">
                STEP 1: 性格診断
            </button>
            <button onclick="switchTab('values')" class="tab-btn px-6 py-3 bg-white rounded-lg shadow">
                STEP 2: 価値観診断
            </button>
            <button onclick="switchTab('search')" class="tab-btn px-6 py-3 bg-white rounded-lg shadow">
                STEP 3: AI求人検索
            </button>
        </div>

        <!-- コンテンツエリア -->
        <div class="bg-white rounded-lg shadow-lg p-6">
            <!-- STEP 1: 性格診断 -->
            <div id="personality" class="tab-content active">
                <h2 class="text-2xl font-bold mb-4">MBTI性格診断</h2>
                <div id="mbti-grid" class="grid grid-cols-4 gap-4">
                    <!-- MBTIタイプが動的に生成される -->
                </div>
                <button onclick="savePersonality()" class="mt-6 px-6 py-3 bg-blue-600 text-white rounded-lg">
                    次へ進む
                </button>
            </div>

            <!-- STEP 2: 価値観診断 -->
            <div id="values" class="tab-content">
                <h2 class="text-2xl font-bold mb-4">価値観診断（30問）</h2>
                <div id="questions-container">
                    <!-- 質問が動的に生成される -->
                </div>
                <button onclick="calculateValues()" class="mt-6 px-6 py-3 bg-blue-600 text-white rounded-lg">
                    診断結果を見る
                </button>
            </div>

            <!-- STEP 3: AI求人検索 -->
            <div id="search" class="tab-content">
                <h2 class="text-2xl font-bold mb-4">AI求人検索</h2>
                
                <!-- 検索プロンプト生成 -->
                <div class="bg-gray-100 p-6 rounded-lg mb-6">
                    <h3 class="text-lg font-semibold mb-2">検索プロンプト</h3>
                    <pre id="search-prompt" class="text-sm whitespace-pre-wrap"></pre>
                    <div class="mt-4 space-x-2">
                        <button onclick="generatePrompt()" class="px-4 py-2 bg-green-600 text-white rounded">
                            プロンプト生成
                        </button>
                        <button onclick="copyPrompt()" class="px-4 py-2 bg-gray-600 text-white rounded">
                            コピー
                        </button>
                    </div>
                </div>

                <!-- 結果入力エリア -->
                <div class="mb-6">
                    <h3 class="text-lg font-semibold mb-2">AI検索結果を貼り付け</h3>
                    <textarea 
                        id="search-results-input" 
                        class="w-full h-48 p-4 border rounded-lg"
                        placeholder="ChatGPT/ClaudeのWeb検索結果をここに貼り付けてください"
                    ></textarea>
                    <button onclick="parseResults()" class="mt-2 px-4 py-2 bg-blue-600 text-white rounded">
                        結果を解析
                    </button>
                </div>

                <!-- 求人カード表示エリア -->
                <div id="job-cards" class="space-y-4">
                    <!-- 求人カードが動的に生成される -->
                </div>
            </div>
        </div>
    </div>

    <script>
        // アプリケーションのメインロジック
        const app = {
            profile: {
                mbti: null,
                values: {},
                answers: {},
                strengths: [],
                weaknesses: []
            },
            
            // 質問データ
            questions: [
                { id: '1', content: '高い給与のためなら週末も働くことは厭わない', category: 'salary', weight: 2 },
                { id: '2', content: '安定性を犠牲にしても、急成長できる環境を選びたい', category: 'growth', weight: 3 },
                // ... 30問すべて
            ],
            
            init() {
                this.loadFromStorage();
                this.renderMBTI();
                this.renderQuestions();
            },
            
            loadFromStorage() {
                const saved = localStorage.getItem('career_match_profile');
                if (saved) {
                    this.profile = JSON.parse(saved);
                }
            },
            
            saveToStorage() {
                localStorage.setItem('career_match_profile', JSON.stringify(this.profile));
            },
            
            renderMBTI() {
                const types = ['INTJ', 'INTP', 'ENTJ', 'ENTP', 'INFJ', 'INFP', 'ENFJ', 'ENFP',
                              'ISTJ', 'ISFJ', 'ESTJ', 'ESFJ', 'ISTP', 'ISFP', 'ESTP', 'ESFP'];
                const grid = document.getElementById('mbti-grid');
                
                grid.innerHTML = types.map(type => `
                    <button 
                        class="mbti-option p-4 border rounded hover:bg-blue-100"
                        onclick="selectMBTI('${type}')"
                    >
                        ${type}
                    </button>
                `).join('');
            },
            
            renderQuestions() {
                const container = document.getElementById('questions-container');
                container.innerHTML = this.questions.map((q, index) => `
                    <div class="mb-6 p-4 border rounded">
                        <p class="font-semibold mb-2">Q${index + 1}. ${q.content}</p>
                        <div class="flex space-x-2">
                            ${[1,2,3,4,5].map(value => `
                                <button 
                                    class="px-4 py-2 border rounded hover:bg-gray-100"
                                    onclick="answerQuestion('${q.id}', ${value})"
                                >
                                    ${value}
                                </button>
                            `).join('')}
                        </div>
                    </div>
                `).join('');
            },
            
            generateSearchPrompt() {
                const topValues = Object.entries(this.profile.values)
                    .sort(([,a], [,b]) => b - a)
                    .slice(0, 3);
                
                return `
以下の条件で日本の求人情報を検索してください：

【私のプロファイル】
- MBTI: ${this.profile.mbti || '未設定'}
- 重視する価値観:
${topValues.map(([cat, score]) => `  - ${this.getCategoryLabel(cat)}: ${score}%`).join('\n')}

【検索してほしい求人サイト】
- Indeed Japan
- リクナビNEXT
- マイナビ転職
- ビズリーチ
- Wantedly

各求人について以下の情報を含めてください：
1. 企業名・職種
2. 給与レンジ
3. 勤務地
4. 必要スキル
5. なぜ私の価値観にマッチするか
6. 応募URL

特に${topValues[0] ? this.getCategoryLabel(topValues[0][0]) : ''}を重視する企業を探してください。
                `;
            },
            
            getCategoryLabel(category) {
                const labels = {
                    salary: '給与・報酬',
                    growth: '成長機会',
                    worklife: 'ワークライフバランス',
                    social: '社会貢献',
                    stability: '安定性',
                    challenge: 'チャレンジ',
                    skill: '専門性',
                    relationship: '人間関係',
                    culture: '企業文化'
                };
                return labels[category] || category;
            }
        };
        
        // グローバル関数
        function switchTab(tabName) {
            document.querySelectorAll('.tab-content').forEach(tab => {
                tab.classList.remove('active');
            });
            document.getElementById(tabName).classList.add('active');
        }
        
        function selectMBTI(type) {
            app.profile.mbti = type;
            document.querySelectorAll('.mbti-option').forEach(btn => {
                btn.classList.remove('selected');
            });
            event.target.classList.add('selected');
            app.saveToStorage();
        }
        
        function answerQuestion(questionId, value) {
            app.profile.answers[questionId] = value;
            app.saveToStorage();
        }
        
        function calculateValues() {
            // 価値観スコア計算
            const scores = {};
            const maxPossible = {};
            
            app.questions.forEach(q => {
                const answer = app.profile.answers[q.id] || 3;
                if (!scores[q.category]) {
                    scores[q.category] = 0;
                    maxPossible[q.category] = 0;
                }
                scores[q.category] += answer * q.weight;
                maxPossible[q.category] += 5 * q.weight;
            });
            
            Object.keys(scores).forEach(cat => {
                scores[cat] = Math.round((scores[cat] / maxPossible[cat]) * 100);
            });
            
            app.profile.values = scores;
            app.saveToStorage();
            alert('価値観診断が完了しました！');
            switchTab('search');
        }
        
        function generatePrompt() {
            const prompt = app.generateSearchPrompt();
            document.getElementById('search-prompt').textContent = prompt;
        }
        
        function copyPrompt() {
            const prompt = document.getElementById('search-prompt').textContent;
            navigator.clipboard.writeText(prompt);
            alert('プロンプトをコピーしました！');
        }
        
        function parseResults() {
            const input = document.getElementById('search-results-input').value;
            // 簡易的な解析
            const jobs = input.split('\n\n').map((jobText, index) => ({
                id: index,
                raw: jobText,
                saved: false
            }));
            
            const container = document.getElementById('job-cards');
            container.innerHTML = jobs.map(job => `
                <div class="p-4 border rounded-lg">
                    <pre class="text-sm">${job.raw}</pre>
                    <button 
                        class="mt-2 px-4 py-2 bg-green-600 text-white rounded"
                        onclick="saveJob(${job.id})"
                    >
                        保存
                    </button>
                </div>
            `).join('');
        }
        
        function saveJob(jobId) {
            const saved = JSON.parse(localStorage.getItem('saved_jobs') || '[]');
            saved.push({ id: jobId, timestamp: new Date().toISOString() });
            localStorage.setItem('saved_jobs', JSON.stringify(saved));
            alert('求人を保存しました！');
        }
        
        // 初期化
        window.onload = () => app.init();
    </script>
</body>
</html>
```

## 6. デプロイメント（超簡単）

### 6.1 GitHub Pages
```bash
# リポジトリ作成
git init
git add index.html
git commit -m "Initial commit"
git remote add origin https://github.com/username/career-match-lite.git
git push -u origin main

# GitHub Pages設定
# Settings > Pages > Source: main branch
```

### 6.2 Vercel
```bash
# Vercelインストール
npm i -g vercel

# デプロイ
vercel

# 以降の更新
vercel --prod
```

### 6.3 Netlify
```bash
# ドラッグ&ドロップでデプロイ
# または
netlify deploy
netlify deploy --prod
```

## 7. 使用フロー

### 7.1 ユーザー操作フロー
```
1. サイトにアクセス
   ↓
2. MBTI性格タイプを選択
   ↓
3. 30問の価値観質問に回答
   ↓
4. 「プロンプト生成」ボタンをクリック
   ↓
5. 生成されたプロンプトをコピー
   ↓
6. ChatGPT/ClaudeでWeb検索を実行
   ↓
7. 検索結果をアプリに貼り付け
   ↓
8. マッチ度を確認して求人を保存
```

### 7.2 AI検索プロンプト例
```
ChatGPT/Claudeへの入力：
「以下のプロンプトでWeb検索してください」
[生成されたプロンプトを貼り付け]
```

## 8. 開発タスクリスト

### Week 1: 基本機能
- [ ] HTMLテンプレート作成
- [ ] MBTI選択機能
- [ ] 30問の質問実装
- [ ] 価値観スコア計算
- [ ] ローカルストレージ保存

### Week 2: AI連携機能
- [ ] プロンプト生成ロジック
- [ ] コピー機能実装
- [ ] 結果解析機能
- [ ] 求人カード表示
- [ ] 保存機能

### Week 3: UI/UX改善
- [ ] レスポンシブ対応
- [ ] アニメーション追加
- [ ] エラーハンドリング
- [ ] 使い方ガイド

### Week 4: 公開準備
- [ ] テスト実施
- [ ] デプロイ設定
- [ ] ドキュメント作成
- [ ] 公開

## 9. 注意事項

### 9.1 制限事項
- データベースなし（ローカルストレージのみ）
- 認証なし（誰でも使用可能）
- AI検索は手動（自動化なし）
- リアルタイムデータなし

### 9.2 将来の拡張可能性
- Firebase追加でデータ永続化
- OpenAI API統合で自動検索
- ユーザー認証追加
- 求人データベース構築

## 10. サポート情報

### 開発者向けリソース
- [MDN LocalStorage](https://developer.mozilla.org/ja/docs/Web/API/Window/localStorage)
- [Tailwind CSS Docs](https://tailwindcss.com/docs)
- [ChatGPT API](https://platform.openai.com/docs)
- [Claude API](https://docs.anthropic.com)

### トラブルシューティング
```javascript
// ローカルストレージ容量確認
function checkStorageUsage() {
  let total = 0;
  for(let key in localStorage) {
    if(localStorage.hasOwnProperty(key)) {
      total += localStorage[key].length + key.length;
    }
  }
  console.log('Storage used: ' + (total / 1024).toFixed(2) + ' KB');
}

// データリセット
function resetAllData() {
  if(confirm('すべてのデータを削除しますか？')) {
    localStorage.clear();
    location.reload();
  }
}
```