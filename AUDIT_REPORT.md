# 社會科應用 (social-studies-app) - 完整審計報告

**應用URL**: https://github.com/blacKgreYcAt/social-studies-app  
**文件大小**: 1013 行代碼  
**審計日期**: 2026-06-10  
**審計狀態**: 🔍 進行中

---

## 📊 快速概覽

| 指標 | 結果 |
|------|------|
| 代碼行數 | 1013 |
| HTML 頁面 | 18 個（首頁 + 3 章節 + 8 遊戲 + 8 完成） |
| 題庫規模 | 64 道題（8 個任務 × 8 題） |
| 問題數 | 🔴 6 個 |
| 優先級分佈 | P1:2 個, P2:3 個, P3:1 個 |

---

## 🔴 發現的問題

### P1-1: XSS 漏洞 in renderQuestion() - innerHTML + 內聯 onclick
**位置**: index.html, 第 969-977 行  
**嚴重性**: 高 - 安全漏洞  
**當前代碼**:
```javascript
el.innerHTML = `
  <div class="quiz-icon">${['💧', '🌲', '🐟', '🚜', '🥬', '🏛️', '🎭', '🌟'][currentQuestion % 8]}</div>
  <div class="quiz-q">${q.q}</div>
  <div class="quiz-choices">
    ${q.choices.map((c, i) => `
      <button class="quiz-choice" onclick="selectAnswer(${i}, ${q.correct})">${c}</button>
    `).join('')}
  </div>
`;
```

**問題描述**:
- 使用 `innerHTML` 和內聯 `onclick` 動態生成內容
- 儘管當前數據相對可信（來自代碼定義的 quizData），但這是不安全的模式
- 如果題目或選項數據來自外部源，可能導致 XSS 漏洞
- 與 natural-science-app 中發現的同類問題

**修復方案**: 使用 DOM API + textContent + addEventListener

---

### P1-2: showScreen() 中重複添加 'screen' 類
**位置**: index.html, 第 927-928 行  
**嚴重性**: 中 - 代碼邏輯問題  
**當前代碼**:
```javascript
function showScreen(screenId) {
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  document.getElementById(screenId).classList.add('screen');  // 不必要
  document.getElementById(screenId).classList.add('active');
}
```

**問題描述**:
- 第 927 行嘗試添加 'screen' 類
- 所有屏幕 HTML 元素都已經有 'screen' 類（行 29）
- 這行代碼不會造成功能問題，但邏輯不清晰
- 應該只添加 'active' 類

**修復方案**: 刪除第 927 行

---

### P2-3: selectAnswer() 缺乏輸入驗證
**位置**: index.html, 第 980-1005 行  
**嚴重性**: 中 - 數據驗證缺陷  
**當前代碼**:
```javascript
function selectAnswer(selected, correct) {
  const quiz = quizData[currentGame];
  const q = quiz[currentQuestion];
  const buttons = document.querySelectorAll(`#quiz-box-${currentGame} .quiz-choice`);
  // ... 沒有驗證 selected, correct, currentQuestion
}
```

**問題描述**:
- 沒有驗證 `selected` 和 `correct` 是否為有效的數字
- 沒有驗證 `selected` 和 `correct` 是否在有效範圍內
- 沒有檢查 `quiz` 或 `q` 是否為 undefined
- 沒有驗證 `currentQuestion` 是否超出範圍

**修復方案**: 添加邊界檢查和數據驗證

---

### P2-4: startGame() 缺乏參數驗證
**位置**: index.html, 第 948-955 行  
**嚴重性**: 中 - 輸入驗證不足  
**當前代碼**:
```javascript
function startGame(chapter, task) {
  document.documentElement.setAttribute('data-chapter', chapter);
  currentChapter = chapter;
  currentGame = `${chapter}-${task}`;
  currentQuestion = 0;
  renderQuestion();  // 沒有檢查 quizData[currentGame] 是否存在
  showScreen(`screen-game-${chapter}-${task}`);
}
```

**問題描述**:
- 沒有驗證 `chapter` 參數（應該是 1-3）
- 沒有驗證 `task` 參數（應該根據章節而定）
- 沒有檢查 `quizData[currentGame]` 是否存在
- 如果傳入無效參數會導致運行時錯誤

**修復方案**: 添加參數驗證和邊界檢查

---

### P2-5: goHome() 缺少完整的狀態清理
**位置**: index.html, 第 931-936 行  
**嚴重性**: 中 - 狀態管理不完整  
**當前代碼**:
```javascript
function goHome() {
  document.documentElement.setAttribute('data-chapter', 'home');
  currentChapter = null;
  currentGame = null;
  showScreen('screen-home');
}
```

**問題描述**:
- 沒有重置 `currentQuestion` 變量
- 沒有清除任何可能存在的事件監聽器
- 與其他應用中已修復的返回函數邏輯不一致

**修復方案**: 添加 `currentQuestion = 0;` 和其他必要的清理邏輯

---

### P3-6: 事件處理器使用內聯 onclick
**位置**: HTML 整個文件  
**嚴重性**: 低 - 代碼風格和可維護性  
**示例**:
- 行 176: `<div class="chapter-card" data-chapter="1" onclick="gotoChapter(1)">`
- 行 208: `<div class="mission-card" onclick="startGame(1,1)">`
- 行 292: `<button class="back-btn show" onclick="goChapter(1)">← 返回</button>`

**問題描述**:
- 所有交互元素都使用內聯 `onclick`
- 這不符合現代 Web 開發最佳實踐
- 使得 HTML 和 JavaScript 高度耦合
- 難以進行事件委託和複雜的事件處理

**修復方案**: 遷移到 addEventListener 和事件委託

---

## 📈 問題分佈

```
安全性問題:    1 個 (P1-1)
功能性問題:    3 個 (P1-2, P2-3, P2-4)
狀態管理:      1 個 (P2-5)
代碼風格:      1 個 (P3-6)
```

---

## 📝 審計檢查清單

- [x] XSS 漏洞檢測
- [x] 輸入驗證檢查
- [x] 事件處理檢查
- [x] 狀態管理檢查
- [x] HTML 結構完整性
- [x] 返回按鈕功能
- [x] 題庫數據驗證
- [x] 響應式設計檢查

---

## 🔧 建議修復優先級

1. **立即修復** (P1):
   - P1-1: XSS 漏洞（安全問題）
   - P1-2: showScreen() 邏輯錯誤

2. **優先修復** (P2):
   - P2-3: selectAnswer() 驗證
   - P2-4: startGame() 驗證
   - P2-5: goHome() 狀態清理

3. **後續優化** (P3):
   - P3-6: 事件處理器遷移

---

## ✅ 應用優點

- ✅ 界面設計優秀（漸變色彩、動畫流暢）
- ✅ 響應式設計完整
- ✅ 題庫內容豐富（64 道題）
- ✅ 進度指示清晰
- ✅ 完成畫面反饋良好
- ✅ 數據結構清晰

---

**審計完成**: 待修復  
**下一步**: 實施修復並重新驗證
