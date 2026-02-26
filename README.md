# Zustand(ズースタンド)

## サンプルコード

```javascript
import { create } from 'zustand';

// ストアの作成
const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

// コンポーネントでの使用
function Counter() {
  const count = useStore((state) => state.count);
  const increment = useStore((state) => state.increment);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+1</button>
    </div>
  );
}
```

## ドキュメント

**公式サイト**

https://zustand.docs.pmnd.rs/learn/getting-started/introduction

**Github**

https://github.com/pmndrs/zustand

## Zustandとは

## 特徴

- 軽量(~1kbyte未満)
- フックベースでシンプルな使用方法
- Context APIのような再レンダリングがない

## Zestandが解決すること

1. Context APIの再レンダリング問題

```javascript
// Context APIの例
const AppContext = createContext()

function Provider({ children }) {
  const [state, setState] = useState({
    user: { name: 'John' },
    count: 0
  })

  return (
    <AppContext.Provider value={state}>
      {children}
    </AppContext.Provider>
  )
}

// このコンポーネントは count だけ使いたい
function Counter() {
  const { count } = useContext(AppContext)
  return <div>{count}</div>
}

// 問題: user が変更されても Counter が再レンダリングされる！

問題点: Context の値のどこか一部が変更されると、そのContextを使用しているすべてのコンポーネントが再レンダリングされます。

Zustandの解決方法

// Zustandの例
const useStore = create((set) => ({
  user: { name: 'John' },
  count: 0,
  setUser: (user) => set({ user }),
  increment: () => set((state) => ({ count: state.count + 1 }))
}))

// このコンポーネントは count だけ監視
function Counter() {
  const count = useStore((state) => state.count)
  return <div>{count}</div>
}

// 利点: user が変更されても Counter は再レンダリングされない！
// count が変更された時だけ再レンダリングされる
```

Zustandの仕組み:

- セレクター関数 (state) => state.count で必要な部分だけを監視
- 監視している値が変更された時だけ再レンダリング
- パフォーマンスが大幅に向上

## ミドルウェアのサポート

Zustandには様々な便利なミドルウェアがあります。

### 1. persist（永続化）

```javascript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

const useStore = create(
  persist(
    (set) => ({
      user: null,
      token: null,
      login: (user, token) => set({ user, token }),
      logout: () => set({ user: null, token: null }),
    }),
    {
      name: 'auth-storage', // localStorageのキー名
    }
  )
);

// 利点: ページをリロードしても状態が保持される
```

### 2. immer（イミュータブル更新を簡単に）

```javascript
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

const useStore = create(
  immer((set) => ({
    user: { name: 'John', age: 30 },
    // 通常の書き方（複雑）
    updateNameNormal: (name) =>
      set((state) => ({
        user: { ...state.user, name },
      })),
    // immerを使った書き方（シンプル）
    updateNameWithImmer: (name) =>
      set((state) => {
        state.user.name = name; // 直接変更できる！
      }),
  }))
);
```

### 3. devtools（Redux DevToolsとの統合）

```javascript
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';

const useStore = create(
  devtools(
    (set) => ({
      count: 0,
      increment: () => set((state) => ({ count: state.count + 1 }), false, 'increment'),
    }),
    { name: 'MyStore' }
  )
);

// 利点: ブラウザのRedux DevToolsで状態変化を確認できる
```

### 4. カスタムミドルウェア

```javascript
// ログ出力ミドルウェアの例
const log = (config) => (set, get, api) =>
  config(
    (...args) => {
      console.log('変更前:', get());
      set(...args);
      console.log('変更後:', get());
    },
    get,
    api
  );

const useStore = create(
  log((set) => ({
    count: 0,
    increment: () => set((state) => ({ count: state.count + 1 })),
  }))
);

// すべての状態変更時にログが出力される
```

### 5. ミドルウェアの組み合わせ

```javascript
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

const useStore = create(
  devtools(
    persist(
      immer((set) => ({
        todos: [],
        addTodo: (todo) =>
          set((state) => {
            state.todos.push(todo);
          }),
      })),
      { name: 'todo-storage' }
    ),
    { name: 'TodoStore' }
  )
);

// 複数のミドルウェアを組み合わせて使える
```

## まとめ

再レンダリングの最適化

- Context API: 全体が再レンダリング → 遅い
- Zustand: 必要な部分だけ再レンダリング → 速い

ミドルウェアの利点

- 永続化（persist）: ページリロード後も状態を保持
- イミュータブル更新（immer）: 複雑なネスト構造を簡単に更新
- デバッグ（devtools）: 開発時の状態確認が簡単
- カスタム: 独自の機能を追加可能
