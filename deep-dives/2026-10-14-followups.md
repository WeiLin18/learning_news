# 深入追問筆記 2026-10-14

這份文件整理使用者針對 LogRocket 「AI-Ready Frontend Architecture Guide」 的追問，補上比日報摘要更完整的架構思維與實務範例。

## 目錄

1. [為什麼架構對 AI 編碼助手特別重要](#1-為什麼架構對-ai-編碼助手特別重要)（追問對象：https://blog.logrocket.com/ai-ready-frontend-architecture-guide/）
2. [三個 AI 會放大的架構反模式（含 before/after）](#2-三個-ai-會放大的架構反模式)
3. [跟 2026-10-06 日報第 4 則的互補關係](#3-跟日報第-4-則的互補關係)

---

## 1. 為什麼架構對 AI 編碼助手特別重要

日報第 4 則講的是「使用者遇上 AI 編碼助手的三個血淚教訓」——競態條件、邊界案例、可存取性漏洞，都是用上 AI 以後才發現的問題。那這篇文章的角度不一樣：它問的是「怎麼**預防** AI 産出那些問題」。

核心洞察是：**AI 不會憑空發明壞習慣，它會學你的程式碼模式，然後把它放大 10 倍。**

類比：像是一個很會模仿的學生，他看到老師隨便在黑板寫草稿，就認定「原來老師就是這樣寫，我以後也這樣寫」。你不能怪學生，要怪的是你自己沒設立明確的標準。

---

## 2. 三個 AI 會放大的架構反模式

### 問題 1：模式不一致 → AI 產出多套「方言」

**現象**：同一個業務概念，卻在不同檔案裡有不同的命名/結構方式。

❌ **Before（混亂的例子）**：

```ts
// user-service.ts 的命名
async function getUser(id: string) { }
async function createNewUser(data: UserData) { }

// product-service.ts 的命名  
async function fetchProduct(id: string) { }
async function addProduct(data: ProductData) { }

// order-service.ts 的命名
async function loadOrder(id: string) { }
async function insertOrder(data: OrderData) { }
```

AI 工具看到這段，會學到「get/create/fetch/add/load/insert 好像都是同一回事」，下次生成新的 service 時，就會隨機選一個，造成整個 codebase 充滿別名。團隊要花時間到處改。

✅ **After（一致的例子）**：

```ts
// 統一定義 entity repository 的命名規範（寫進 ARCHITECTURE.md）
interface Repository<T> {
  findById(id: string): Promise<T>;
  create(data: CreateDTO<T>): Promise<T>;
  update(id: string, data: UpdateDTO<T>): Promise<T>;
  delete(id: string): Promise<void>;
}

// 所有 service 都實作相同介面
class UserRepository implements Repository<User> {
  async findById(id: string) { /* ... */ }
  async create(data: CreateUserDTO) { /* ... */ }
}

class ProductRepository implements Repository<Product> {
  async findById(id: string) { /* ... */ }
  async create(data: CreateProductDTO) { /* ... */ }
}
```

AI 看到這套統一的介面模式後，下次生成新的 repository 時，就會自動套用相同命名。

**實務建議**：把 repository 層、validator 層、hook 層的**簽名和命名**明確寫成 TypeScript interface，讓 AI 有「標準範本」可複製。

---

### 問題 2：業務邏輯散落在 UI 層 → AI 把驗證、權限檢查全寫進元件

**現象**：沒有清晰的邊界，權限檢查、資料驗證、API 邏輯全混在 component 或 event handler 裡。

❌ **Before（業務邏輯洩漏進 component）**：

```jsx
function PaymentForm() {
  const [amount, setAmount] = useState(0);
  
  async function handleSubmit() {
    // ❌ 驗證邏輯直接寫在 handler 裡
    if (amount < 10 || amount > 10000) {
      setError('Amount must be between 10-10000');
      return;
    }
    
    // ❌ 權限檢查也在這裡
    const user = await fetchUser();
    if (!user.verified) {
      setError('User must be verified');
      return;
    }
    
    // ❌ API 呼叫邏輯也在這裡
    const response = await fetch('/api/payment', {
      method: 'POST',
      body: JSON.stringify({ amount, userId: user.id })
    });
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={amount} onChange={e => setAmount(Number(e.target.value))} />
      {error && <p>{error}</p>}
      <button>Pay</button>
    </form>
  );
}
```

AI 看到這個例子，會複製同樣的反模式到其他表單，導致邏輯散落各地，後來想改驗證規則時要改 10 個檔案。

✅ **After（清晰的邊界層）**：

```ts
// 1. 驗證層（domain logic，獨立檔案，可單獨測試）
export const PaymentValidator = {
  validateAmount(amount: number): Result<number, string> {
    if (amount < 10 || amount > 10000) {
      return { error: 'Amount must be between 10-10000' };
    }
    return { data: amount };
  },
  
  validateUserVerification(user: User): Result<void, string> {
    if (!user.verified) {
      return { error: 'User must be verified' };
    }
    return { data: undefined };
  }
};

// 2. 業務邏輯層（獨立的 use case）
export async function processPayment(
  amount: number,
  userId: string
): Promise<Result<PaymentResult, string>> {
  const user = await userRepository.findById(userId);
  
  // 引用驗證器，不是重寫
  const amountValidation = PaymentValidator.validateAmount(amount);
  if (amountValidation.error) return amountValidation;
  
  const verificationCheck = PaymentValidator.validateUserVerification(user);
  if (verificationCheck.error) return verificationCheck;
  
  return await paymentService.process(amount, userId);
}

// 3. React 元件（只負責 UI，邏輯呼叫外部函式）
function PaymentForm() {
  const [amount, setAmount] = useState(0);
  const [error, setError] = useState('');
  
  async function handleSubmit() {
    // 元件只知道「呼叫業務邏輯」，不知道邏輯怎麼做
    const result = await processPayment(amount, userId);
    
    if (result.error) {
      setError(result.error);
    } else {
      // 成功邏輯
    }
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={amount} onChange={e => setAmount(Number(e.target.value))} />
      {error && <p>{error}</p>}
      <button>Pay</button>
    </form>
  );
}
```

AI 看到這個結構，下次生成類似表單時，就會自動：
1. 先找是否有既有的 validator
2. 把邏輯寫進 use case 層，不是 component 裡
3. component 只負責 UI 和呼叫

**實務建議**：建立明確的層（validator → use case → UI），並用 TypeScript 介面強制這個結構。這樣 AI 就有「藍圖」可循。

---

### 問題 3：AI 複製並強化既有的反模式

**現象**：如果 codebase 裡已經有「某個功能這樣寫」的範例，AI 會認定「這是標準做法」，然後在每個新功能裡重複這個做法，即使它本來就是反模式。

**例子**：假設你的 codebase 裡有一個老元件這樣寫：

```jsx
// ❌ 既有反模式：直接在 component 裡管理複雜狀態
function UserDashboard({ userId }) {
  const [userData, setUserData] = useState(null);
  const [posts, setPosts] = useState(null);
  const [comments, setComments] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    Promise.all([
      fetchUser(userId),
      fetchUserPosts(userId),
      fetchUserComments(userId)
    ])
    .then(([u, p, c]) => {
      setUserData(u);
      setPosts(p);
      setComments(c);
      setIsLoading(false);
    })
    .catch(err => setError(err));
  }, [userId]);
  
  // ... 100 行 render 邏輯
}
```

AI 會複製這個模式到所有新的複雜元件。但如果團隊決定改用 TanStack Query 或 SWR，這個決定就得套用到 100 個地方，而不是一個集中的層。

✅ **Fix（提供「更好的範本」給 AI 抄）**：

1. **把最佳實踐寫成可重複使用的 hook**：

```ts
// hooks/useUserDashboard.ts
export function useUserDashboard(userId: string) {
  const { data: userData, isLoading: userLoading } = useQuery(
    ['user', userId],
    () => fetchUser(userId)
  );
  
  const { data: posts, isLoading: postsLoading } = useQuery(
    ['posts', userId],
    () => fetchUserPosts(userId)
  );
  
  const { data: comments, isLoading: commentsLoading } = useQuery(
    ['comments', userId],
    () => fetchUserComments(userId)
  );
  
  return {
    userData,
    posts,
    comments,
    isLoading: userLoading || postsLoading || commentsLoading
  };
}
```

2. **在 component 裡只引用這個 hook**：

```jsx
function UserDashboard({ userId }) {
  const { userData, posts, comments, isLoading } = useUserDashboard(userId);
  
  if (isLoading) return <Spinner />;
  
  return (
    // ... render 邏輯
  );
}
```

3. **把這個 hook 當成「標準做法」文檔化**：

```markdown
## Data Fetching Pattern

所有複雜資料查詢都應該抽象成 custom hook，
不要在 component 裡直接用 useState + useEffect。

使用 TanStack Query（useQuery/useMutation）
或 SWR，這樣中央可以統一切換策略。

範例見 `hooks/useUserDashboard.ts`。
```

現在 AI 有了「更好的範本」後，會優先複製這個，而不是老的反模式。

**實務建議**：主動為團隊「豎起好的標竿」，並保證新生成的程式碼看得到這些標竿。

---

## 3. 跟日報第 4 則的互補關係

**日報第 4 則**（「AI 編碼助手在實務上的三個血淚教訓」）講的是：

- 競態條件：舊請求比新請求晚到
- 邊界案例：AI 只寫快樂路徑，漏掉異常
- 可存取性：鍵盤導航、ARIA 標籤常遺漏

**這篇文章的角度**是：這些問題在架構層面可以**預防**。

| 問題 | 日報第 4 則的解法 | 本篇的預防 |
|---|---|---|
| **競態條件** | Review 時加上 AbortController | 把「資料載入」抽象成統一的 custom hook，hook 層面統一處理 abort 邏輯 |
| **邊界案例** | 寫測試先，AI 照著測試做 | 把驗證邏輯提出來寫成 pure function，分離 「UI 呼叫邏輯」和「邏輯實作」，這樣漏掉的邊界案例會更難寫進去 |
| **可存取性** | 事後手動檢查 ARIA / 鍵盤導航 | 提供**可存取的 UI component library** 給 AI 抄，讓 AI 下次生成表單時自動套用 accessible 的基礎元件 |

**兩篇一起讀的重點**：

- **日報第 4 則**教的是「事後怎麼救」(reactive)
- **這篇**教的是「事前怎麼架構讓 AI 不容易出問題」(proactive)

現實是：兩個都需要。架構做得好，能減少 AI 犯的錯誤，但不是零；所以還是要有測試和 review 的防線，跟日報第 4 則講的一樣。

---

**後續行動**：如果你的團隊已經在用 AI 編碼助手，可以嘗試這個優先度：

1. **優先等級 1**：把最常用的驗證/業務邏輯提出來寫成獨立 function 或 hook（問題 2 的解法）
2. **優先等級 2**：定義 repository/validator 層的 interface 標準（問題 1 的解法）
3. **優先等級 3**：提供 accessible component 範本和 prompt 檔案給 AI（問題 3 的解法）

其他都是延伸，這三個做好了，AI 產出程式碼的品質會明顯上升。
