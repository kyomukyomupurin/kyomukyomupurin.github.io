---
layout: post
title:  "Codeforces Round #677 (Div.3) F"
date:   2020-10-29 19:19:18 +0900
categories: competitive-programming
---

# [F. Zero Remainder Sum](https://codeforces.com/contest/1433/problem/F)

少し前のバチャで DP をバグらせて解けなくて、そのあとも TLE で通せていなかった問題がやっと通った。非想定解を無理矢理高速化しただけだけど、通ったのでヨシ！

## 問題概要

$n\times m$ の行列 $a$ と整数 $k$ が与えられる。行列 $a$ の各行からは $\frac{m}{2}$ 個以下の要素を選択することができる。選択した要素の総和が $k$ で割り切れるようにするとき、要素の総和の最大値を求めよ。

### 制約

- $1\leq n,m,k\leq 70$
- $1\leq a_{i,j}\leq 70$
- time limit : 1 second

## 解法(嘘)

3 段階の DP で解いた。DP は次のように定義した。

- bool $dp1_{i,j,k}$ : $i$ 行目で $j$ 個使って $i$ 行目の和を $k$ にできるかどうか
- bool $dp2_{i,j}$ : $i$ 行目の和を $j$ にできるかどうか
- bool $dp3_{i,j}$ : $i$ 行目までで選択した要素の和を $j$ にできるかどうか

制約がそこそこ小さいので、まあこれで間に合うかなあと思い素直に実装した。DP 自体は単純なナップサックと同じような遷移である。

<details>
  <summary>最初のコード</summary>
  {% highlight cpp %}
#include <iostream>

using namespace std;

constexpr int N = 72;

int a[N][N];
bool dp1[N][N / 2][N * N / 2];
bool dp2[N][N * N / 2];
bool dp3[N][N * N * N / 2];

int main() {
  int n, m, K; cin >> n >> m >> K;
  for (int i = 0; i < n; ++i) {
    for (int j = 0; j < m; ++j) {
      cin >> a[i][j];
    }
  }

  for (int i = 0; i < n; ++i) dp1[i][0][0] = true;

  for (int i = 0; i < n; ++i) {
    for (int j = 0; j < m; ++j) {
      for (int k = m / 2; k >= 1; --k) {
        for (int l = 0; l < m / 2 * N; ++l) {
          if (l - a[i][j] < 0) continue;
          dp1[i][k][l] |= dp1[i][k - 1][l - a[i][j]];
        }
      }
    }
  }

  for (int i = 0; i < n; ++i) {
    for (int j = 0; j < m / 2 * N; ++j) {
      for (int k = 0; k <= m / 2; ++k) {
        dp2[i][j] |= dp1[i][k][j];
      }
    }
  }

  for (int i = 0; i <= n; ++i) dp3[i][0] = true;

  for (int i = 0; i < n; ++i) {
    for (int j = 0; j < m / 2 * N; ++j) {
      if (dp2[i][j]) {
        for (int k = 0; k < N * N * N / 2; ++k) {
          if (k - j < 0) continue;
          dp3[i + 1][k] |= dp3[i][k - j];
        }
      }
    }
  }

  for (int i = N * N * N / 2; i >= 0; --i) {
    if (dp3[n][i] && i % K == 0) {
      cout << i << endl;
      return 0;
    }
  }

  return 0;
}
  {% endhighlight %}
</details>

まあこんなもんか、という感じである。サンプルも合ったので意気揚々と提出すると **TLE**。$dp1$ の配列のサイズが約 $6\times 10^6$ と大きく、キャッシュミスが頻発しているのが遅さの原因かなあと思った。手元で最大ケースの入力を適当に作成し、```g++ -std=c++17 -O2``` でコンパイルし実行すると 31 秒くらいかかった。これはいけませんね...ということで高速化を試みることに。
まず最初のコードを眺めてわかることとして、$dp1$ と $dp3$ はどう見ても ```std::bitset``` で高速化してくださいと言わんばかりの遷移をしている。というわけで、この部分を書き直した。

<details>
  <summary>二番目のコード</summary>
  {% highlight cpp %}
#include <bitset>
#include <iostream>

using namespace std;

constexpr int N = 72;

int a[N][N];
bitset<N * N / 2> dp1[N][N / 2];
bool dp2[N][N * N / 2];
bitset<N * N * N / 2> dp3[N];

int main() {
  int n, m, K; cin >> n >> m >> K;
  for (int i = 0; i < n; ++i) {
    for (int j = 0; j < m; ++j) {
      cin >> a[i][j];
    }
  }

  for (int i = 0; i < n; ++i) dp1[i][0][0] = 1;

  for (int i = 0; i < n; ++i) {
    for (int j = 0; j < m; ++j) {
      for (int k = m / 2; k >= 1; --k) {
        dp1[i][k] |= (dp1[i][k - 1] << a[i][j]);
      }
    }
  }

  for (int i = 0; i < n; ++i) {
    for (int j = 0; j < m / 2 * N; ++j) {
      for (int k = 0; k <= m / 2; ++k) {
        dp2[i][j] |= dp1[i][k][j];
      }
    }
  }

  for (int i = 0; i <= n; ++i) dp3[i][0] = 1;

  for (int i = 0; i < n; ++i) {
    for (int j = 0; j < m / 2 * N; ++j) {
      if (dp2[i][j]) {
        dp3[i + 1] |= (dp3[i] << j);
      }
    }
  }

  for (int i = N * N * N / 2; i >= 0; --i) {
    if (dp3[n][i] && i % K == 0) {
      cout << i << endl;
      return 0;
    }
  }

  return 0;
}
  {% endhighlight %}
</details>

```std::bitset``` で(理論上は) 64 倍くらいの高速化が見込めるので結構速くなったでしょ、と思い試すと、ローカルで 約 1.6 秒、こどふぉのコードテストで走らせると 約 1.3 秒だった。うーん、惜しい。さすがにここまできて諦めるわけにはいかないので手元で ```g++ -std=c++17 -O3 -funroll-loops -mavx2``` でコンパイルし実行するとなんと約 0.7 秒で実行できた！というわけで以下の 3 行を加えて再びこどふぉのコードテストで走らせた。

```cpp
#pragma GCC target("avx2")
#pragma GCC optimize("unroll-loops")
#pragma GCC optimize("O3")
```

結果は 577 ms！これは勝ちましたね、ということで提出して無事 **AC**。なんかテストケースが 200 以上あってびびった。やっと通せたので意気揚々と解説を見たらもう少し賢い(というか普通の)解法が書かれていたのでへーとなった。

## まとめ

**力こそパワー**:muscle: