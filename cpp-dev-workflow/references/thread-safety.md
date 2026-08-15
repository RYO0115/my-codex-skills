# スレッドセーフ実装

マルチスレッドに関わるコードを書く・変更するときに読む。目標: **データ競合・メモリアクセス違反・デッドロックをゼロにする**。

## 基本方針(優先順)

1. **共有しない**: スレッド間でデータを共有せず、メッセージキューで値を渡す設計を最優先。
2. **不変にする**: 共有するなら `const`/immutable なデータのみ共有する。
3. **ロックする**: 可変データを共有する場合のみ mutex で保護する。

## 実装規則

- 共有可変データへのアクセスは**すべて**同じ mutex で保護する。「読むだけだから」と保護を省かない(read も data race)。
- ロックは RAII のみ: `std::lock_guard` / `std::scoped_lock` / `std::unique_lock`。生の `lock()`/`unlock()` 呼び出し禁止。
- 複数 mutex を同時に取るときは `std::scoped_lock(m1, m2)` を使う(取得順デッドロック防止)。
- ロック中にコールバック・仮想関数・未知のユーザーコードを呼ばない(デッドロックの典型)。
- カウンタ・フラグ程度は `std::atomic<T>` でよい。ただし複数変数の一貫性が必要なら mutex。
- スレッドの停止フラグは `std::atomic<bool>` + 終了時 `join()`。デタッチ (`detach()`) は原則禁止。
- 待ち合わせは busy-wait ではなく `std::condition_variable`(必ず述語付き `wait(lock, pred)` を使う。spurious wakeup 対策)。
- クラスを設計する際、スレッドセーフかどうかをクラスコメントに明記する(例: `// Thread-safe.` / `// Not thread-safe; guard externally.`)。

## クラス内での典型パターン

```cpp
class Counter {
 public:
  void Increment() {
    std::lock_guard<std::mutex> lock(mutex_);
    ++count_;
  }
  int Get() const {
    std::lock_guard<std::mutex> lock(mutex_);
    return count_;
  }

 private:
  mutable std::mutex mutex_;
  int count_ = 0;  // mutex_ で保護される
};
```

- 保護対象のメンバには「どの mutex で保護されるか」をコメントする。
- `const` メソッドでもロックが要るため mutex は `mutable` にする。

## ライフタイムの規則(メモリアクセス違反防止)

- スレッドに渡す参照・ポインタの生存期間がスレッドより長いことを設計で保証する。ラムダのキャプチャは原則値キャプチャ、参照キャプチャする場合は `join()` までの生存を確認する。
- オブジェクトのデストラクタで、そのオブジェクトを使うスレッドを必ず `join()` してからメンバを破棄する。

## 検証

テストは必ず ThreadSanitizer でも実行する:

```bash
cmake -S . -B build-tsan -DCMAKE_BUILD_TYPE=Debug \
  -DCMAKE_CXX_FLAGS="-fsanitize=thread -g"
cmake --build build-tsan -j
ctest --test-dir build-tsan --output-on-failure
```

TSan が警告を出したら、修正せずに無視・抑制することは禁止。
