# 問題追加ガイド

**対象**: 開発者・問題作成者  
**目的**: 新しい問題パターンを追加する手順を標準化

---

## 📋 問題追加の流れ

```
1. 問題パターンの設計
   ↓
2. 変数の決定
   ↓
3. 計算ロジックの実装
   ↓
4. 図の描画実装
   ↓
5. テスト・検証
   ↓
6. ドキュメント更新
```

---

## ステップ1: 問題パターンの設計

### 1-1. 問題の選定基準
以下の条件を満たす問題を選ぶ：
- [ ] 中学受験で頻出の単元
- [ ] 数値を変えても問題として成立する
- [ ] 計算式が明確（解答が一意に定まる）
- [ ] 図で表現可能

参考サイト
- https://rikanojugyou.com/?cat=12
- https://www.jes-jissen.com/E_rika.htm

### 1-2. 問題パターンの整理
```
【例：てこの問題】

問題タイプ: 支点・力点・作用点の位置関係
変数候補:
  - 支点の位置
  - 力点にかかる力
  - 作用点にかかる力
  - 棒の長さ
  
小問構成:
  (1) つり合う条件を求める
  (2) 必要な力を計算する
  (3) 別の条件での計算
```

---

## ステップ2: 変数の決定

### 2-1. 変数の抽出
問題から可変にできる数値を洗い出す：

```javascript
// 変数リスト（例：てこ）
const variables = {
    barLength: 100,        // 棒の長さ (cm)
    fulcrumPosition: 40,   // 支点の位置（左端から）(cm)
    leftWeight: 50,        // 左側のおもり (g)
    rightWeight: null,     // 右側のおもり (g) ※計算で求める
    leftDistance: null,    // 左腕の長さ (cm) ※計算で求める
    rightDistance: null    // 右腕の長さ (cm) ※計算で求める
}
```

### 2-2. 変数の制約設定
各変数に適切な範囲を設定：

```javascript
const variableRanges = {
    barLength: { min: 50, max: 200, default: 100 },
    fulcrumPosition: { min: 20, max: 80, default: 40 },
    leftWeight: { min: 10, max: 200, default: 50 }
}
```

### 2-3. 妥当性チェック
不適切な値の組み合わせを排除：
- 割り切れない値
- 現実的でない大きな値
- 物理的に矛盾する値

---

## ステップ3: 計算ロジックの実装

### 3-1. 解答計算関数の作成

```javascript
function calculateLeverAnswers(vars) {
    // 左腕と右腕の長さを計算
    const leftDistance = vars.fulcrumPosition;
    const rightDistance = vars.barLength - vars.fulcrumPosition;
    
    // てこの原理：左側の力 × 左腕 = 右側の力 × 右腕
    const rightWeight = (vars.leftWeight * leftDistance) / rightDistance;
    
    return {
        leftDistance: leftDistance,
        rightDistance: rightDistance,
        rightWeight: rightWeight,
        // 追加の計算結果があればここに
    };
}
```

### 3-2. 検証テストケース

```javascript
// テストケース1
const test1 = calculateLeverAnswers({
    barLength: 100,
    fulcrumPosition: 40,
    leftWeight: 50
});
console.assert(test1.rightWeight === 33.33..., "計算エラー");

// テストケース2（境界値）
const test2 = calculateLeverAnswers({
    barLength: 100,
    fulcrumPosition: 50,  // 中央
    leftWeight: 50
});
console.assert(test2.rightWeight === 50, "中央支点のチェック");
```

---

## ステップ4: 図の描画実装

### 4-1. Canvas描画関数の作成

```javascript
function drawLeverDiagram(canvasId, vars, answers) {
    const canvas = document.getElementById(canvasId);
    const ctx = canvas.getContext('2d');
    const width = canvas.width = 600;
    const height = canvas.height = 400;
    
    // 背景
    ctx.fillStyle = '#fafafa';
    ctx.fillRect(0, 0, width, height);
    
    // スケール計算
    const scale = (width - 100) / vars.barLength;
    const barY = height / 2;
    const barStartX = 50;
    
    // 棒の描画
    ctx.strokeStyle = '#8B4513';
    ctx.lineWidth = 8;
    ctx.beginPath();
    ctx.moveTo(barStartX, barY);
    ctx.lineTo(barStartX + vars.barLength * scale, barY);
    ctx.stroke();
    
    // 支点の描画（三角形）
    const fulcrumX = barStartX + vars.fulcrumPosition * scale;
    ctx.fillStyle = '#333';
    ctx.beginPath();
    ctx.moveTo(fulcrumX, barY);
    ctx.lineTo(fulcrumX - 15, barY + 30);
    ctx.lineTo(fulcrumX + 15, barY + 30);
    ctx.closePath();
    ctx.fill();
    
    // おもりの描画（左側）
    drawWeight(ctx, barStartX, barY - 80, vars.leftWeight + 'g');
    
    // おもりの描画（右側）
    drawWeight(ctx, barStartX + vars.barLength * scale, barY - 80, 
               answers.rightWeight + 'g');
    
    // ラベル
    ctx.fillStyle = '#333';
    ctx.font = '14px Arial';
    ctx.textAlign = 'center';
    ctx.fillText('支点', fulcrumX, barY + 50);
}

// おもり描画のヘルパー関数
function drawWeight(ctx, x, y, label) {
    ctx.fillStyle = '#FFD700';
    ctx.fillRect(x - 25, y, 50, 50);
    ctx.strokeStyle = '#B8860B';
    ctx.lineWidth = 2;
    ctx.strokeRect(x - 25, y, 50, 50);
    
    ctx.fillStyle = '#333';
    ctx.font = 'bold 12px Arial';
    ctx.textAlign = 'center';
    ctx.fillText(label, x, y + 30);
}
```

### 4-2. 図の要素チェックリスト
- [ ] 問題の条件が視覚的に分かる
- [ ] 寸法が明記されている
- [ ] おもりや力が明示されている
- [ ] 支点・力点・作用点が分かる
- [ ] ラベルが読みやすい

---

## ステップ5: HTML統合

### 5-1. HTML構造の追加

```html
<!-- 新しい問題タイプのタブを追加 -->
<div class="tab-content" id="tab-lever">
    <div class="card">
        <div class="settings-section">
            <h3>📝 問題の設定</h3>
            <div class="input-group">
                <div class="input-field">
                    <label>棒の長さ (cm)</label>
                    <input type="number" id="lever-bar-length" value="100" min="50" max="200">
                </div>
                <div class="input-field">
                    <label>支点の位置（左端から） (cm)</label>
                    <input type="number" id="lever-fulcrum" value="40" min="10" max="90">
                </div>
                <div class="input-field">
                    <label>左側のおもり (g)</label>
                    <input type="number" id="lever-left-weight" value="50" min="10" max="200">
                </div>
            </div>
            <button class="generate-btn" onclick="generateLeverProblem()">問題を生成</button>
        </div>
        
        <div class="output-section" id="lever-output">
            <div class="problem-box">
                <h3>📋 問題</h3>
                <div class="problem-text" id="lever-problem-text"></div>
                <div class="conditions" id="lever-conditions"></div>
                <canvas class="diagram-canvas" id="lever-canvas"></canvas>
                <div class="questions" id="lever-questions"></div>
            </div>
            
            <button class="toggle-answer-btn" onclick="toggleAnswer('lever')">解答を表示</button>
            <div class="answer-section" id="lever-answer">
                <div class="answer-box" id="lever-answer-content"></div>
            </div>
        </div>
    </div>
</div>
```

### 5-2. JavaScript生成関数の実装

```javascript
function generateLeverProblem() {
    // 入力値の取得
    const barLength = parseFloat(document.getElementById('lever-bar-length').value);
    const fulcrumPosition = parseFloat(document.getElementById('lever-fulcrum').value);
    const leftWeight = parseFloat(document.getElementById('lever-left-weight').value);
    
    // 計算
    const answers = calculateLeverAnswers({
        barLength: barLength,
        fulcrumPosition: fulcrumPosition,
        leftWeight: leftWeight
    });
    
    // 問題文の生成
    document.getElementById('lever-problem-text').innerHTML = 
        '図のように、長さ' + barLength + 'cmの棒を使って、てこの実験を行いました。次の問いに答えなさい。';
    
    // 条件の表示
    document.getElementById('lever-conditions').innerHTML = `
        <ul>
            <li>棒の長さ：${barLength}cm</li>
            <li>支点の位置：左端から${fulcrumPosition}cm</li>
            <li>左側のおもり：${leftWeight}g</li>
        </ul>
    `;
    
    // 問いの表示
    document.getElementById('lever-questions').innerHTML = `
        <p><strong>(1)</strong> 左腕の長さは何cmですか。</p>
        <p><strong>(2)</strong> 右腕の長さは何cmですか。</p>
        <p><strong>(3)</strong> 棒を水平に保つには、右側に何gのおもりをつるせばよいですか。</p>
    `;
    
    // 図の描画
    drawLeverDiagram('lever-canvas', {
        barLength: barLength,
        fulcrumPosition: fulcrumPosition,
        leftWeight: leftWeight
    }, answers);
    
    // 解答の生成
    document.getElementById('lever-answer-content').innerHTML = `
        <h4>解答と解説</h4>
        
        <div class="answer-item">
            <h5>(1) 左腕の長さ</h5>
            <p><strong>考え方：</strong> 支点から左端までの距離</p>
            <div class="formula">
                左腕の長さ = ${fulcrumPosition}cm
            </div>
            <div class="answer-value">答え: ${answers.leftDistance}cm</div>
        </div>
        
        <div class="answer-item">
            <h5>(2) 右腕の長さ</h5>
            <p><strong>考え方：</strong> 支点から右端までの距離</p>
            <div class="formula">
                右腕の長さ = ${barLength}cm - ${fulcrumPosition}cm = ${answers.rightDistance}cm
            </div>
            <div class="answer-value">答え: ${answers.rightDistance}cm</div>
        </div>
        
        <div class="answer-item">
            <h5>(3) 右側のおもり</h5>
            <p><strong>考え方：</strong> てこの原理を使用</p>
            <div class="formula">
                左側の力 × 左腕 = 右側の力 × 右腕<br>
                ${leftWeight}g × ${answers.leftDistance}cm = 右側の力 × ${answers.rightDistance}cm<br>
                右側の力 = (${leftWeight} × ${answers.leftDistance}) ÷ ${answers.rightDistance}<br>
                = ${answers.rightWeight}g
            </div>
            <div class="answer-value">答え: ${answers.rightWeight}g</div>
        </div>
    `;
    
    // 出力セクションを表示
    document.getElementById('lever-output').classList.add('show');
}
```

---

## ステップ6: テスト・検証

### 6-1. 動作確認チェックリスト
- [ ] 問題が正しく表示される
- [ ] 図が正確に描画される
- [ ] 解答が正しく計算される
- [ ] 様々な数値パターンで動作確認
- [ ] エラー処理が適切

### 6-2. 境界値テスト
```javascript
// テストケース一覧
const testCases = [
    // 通常パターン
    { barLength: 100, fulcrumPosition: 40, leftWeight: 50 },
    
    // 支点が中央
    { barLength: 100, fulcrumPosition: 50, leftWeight: 50 },
    
    // 支点が左寄り
    { barLength: 100, fulcrumPosition: 20, leftWeight: 50 },
    
    // 支点が右寄り
    { barLength: 100, fulcrumPosition: 80, leftWeight: 50 },
    
    // 最小値
    { barLength: 50, fulcrumPosition: 10, leftWeight: 10 },
    
    // 最大値
    { barLength: 200, fulcrumPosition: 90, leftWeight: 200 }
];

// 各テストケースを実行
testCases.forEach((testCase, index) => {
    console.log(`テスト ${index + 1}:`, testCase);
    const result = calculateLeverAnswers(testCase);
    console.log('結果:', result);
});
```

### 6-3. 実際の教師によるレビュー
- [ ] 問題文が自然か
- [ ] 中学受験レベルとして適切か
- [ ] 解説が分かりやすいか
- [ ] 図が見やすいか

---

## ステップ7: ドキュメント更新

### 7-1. README.mdの更新
```markdown
## 追加された問題

### てこの問題
- **種類**: 支点・力点・作用点
- **変数**: 棒の長さ、支点の位置、おもりの重さ
- **難易度**: ★★☆☆☆
- **追加日**: 2026年2月15日
```

### 7-2. CHANGELOG.mdの作成
```markdown
# 変更履歴

## [1.1.0] - 2026-02-15
### 追加
- てこの問題パターンを追加
  - 支点・力点・作用点の基本問題
  - 変数：棒の長さ、支点位置、おもりの重さ
```

---

## 📚 参考：よくある問題パターン

### 物理分野

#### ばね
- ✅ 単一のばね（実装済み）
- ✅ 直列つなぎ（実装済み）
- ✅ 並列つなぎ（実装済み）
- [ ] ばねとおもりの運動

#### てこ
- [ ] 基本のてこ
- [ ] 棒の重さを考慮
- [ ] 複数のおもり

#### 浮力
- [ ] 物体の浮き沈み
- [ ] 水に浮かぶ物体
- [ ] 異なる液体での浮力

#### 滑車
- [ ] 定滑車
- [ ] 動滑車
- [ ] 組み合わせ滑車

---

## 🔧 トラブルシューティング

### 問題1: 計算結果が割り切れない
**対策**: 入力値の制約を追加
```javascript
// 例：10の倍数のみ許可
<input type="number" step="10" min="10" max="200">
```

### 問題2: 図が崩れる
**対策**: Canvas要素のサイズ固定
```javascript
const canvas = document.getElementById('canvas-id');
canvas.width = 600;  // 固定値
canvas.height = 400; // 固定値
```

### 問題3: 問題文が不自然
**対策**: テンプレートリテラルを活用
```javascript
const problemText = `
    図のように、長さ${barLength}cmの棒があります。
    支点は左端から${fulcrumPosition}cmの位置にあります。
`;
```

---

## ✅ 問題追加完了チェックリスト

- [ ] 問題パターンを設計した
- [ ] 変数を決定し、範囲を設定した
- [ ] 計算ロジックを実装した
- [ ] テストケースで検証した
- [ ] 図の描画を実装した
- [ ] HTMLに統合した
- [ ] 複数パターンで動作確認した
- [ ] 境界値テストを実施した
- [ ] 教師によるレビューを受けた
- [ ] ドキュメントを更新した

---

**この手順に従うことで、品質の高い問題パターンを効率的に追加できます。**
