# [Trang chủ](https://ppap-1264589.github.io/interesting-solution)

## Bài Toán
[Codeforces Round 375 - Problem D](https://codeforces.com/contest/375/problem/D)

# Hướng dẫn

## Khởi tạo: Kĩ thuật "duỗi cây" bằng DFS

Xét một ví dụ như sau:

![graph](https://user-images.githubusercontent.com/88699088/140309883-a7223fbe-211a-43bc-8e9d-9e5becf73979.png)

Giả sử ta xét đỉnh 1: Theo như bài toán, ta chỉ cần quan tâm đến những đỉnh nằm trong cây con gốc 1

Nhận xét này đưa ta đến một liên tưởng trong thuật chia căn: Nếu ta coi đỉnh 1 là con trỏ trái và các đỉnh trong cây con gốc 1 là con trỏ phải, ta sẽ nghĩ đến thuật "duỗi cây" DFS để có giá trị con trỏ của từng đỉnh một.

Trong ví dụ trên, giả sử đỉnh 1 có giá trị con trỏ trái là x, thì các đỉnh trong cây con gốc 1 có giá trị con trỏ phải là x+1, x+2, tương ứng với đỉnh 2,3,4,...

Thao tác "duỗi cây" tìm giá trị cho các con trỏ có thể được cài đặt như sau: Gọi In[u] là giá trị timer thăm đỉnh u, và Out[u] là giá trị timer thoát khỏi đỉnh u trong cây DFS

Ta có Out[u] = max(Out[u], Out[v]) với v là các đỉnh trong cây con gốc u. Hay nói cách khác, Out[u] là giá trị con trỏ phải lớn nhất đạt được khi thăm từ đỉnh u xuống cây DFS

Đồng thời lưu luôn một mảng node[], với ý nghĩa node[pos] là tên của đỉnh tại con trỏ pos
```c++
int tdfs;
int in[maxn], out[maxn];
int node[maxn] // lưu lại tên của đỉnh tại vị trí con trỏ
int DFS(int u, int parrent){
    in[u] = out[u] = ++tdfs;
    node[tdfs] = u;
    for (int v : a[u]){
        if (v != parrent) out[u] = max(out[u], DFS(v, u));
    }
    return out[u];
}
```

Tới đây, ta có thể thấy rằng với mỗi đỉnh u, ta có con trỏ trái là In[u], và con trỏ phải là Out[u]. 

## Truy vấn: Đếm số lượng màu xuất hiện ít nhất k lần trong cây con gốc v ?

Tiến hành ép các thông tin vào struct và sort các truy vấn theo quy tắc của thuật toán Mo

Sau đó xây dựng một số mảng quan trọng:

- Gọi pos là vị trí của con trỏ đang xét 
    -> tên của đỉnh đang xét là u = node[pos]

- Gọi cnt[C[u]] là số lần xuất hiện màu C[u]

- Gọi appear[k] là số lần cnt[C[u]] xuất hiện ít nhất k lần

Ta có mô hình cơ bản của bài toán chia căn như sau:

- Nếu duyệt thêm đỉnh u vào tập đang xét: tăng cnt[C[u]] lên 1, tăng appear[cnt[C[u]]] lên 1
 
- Nếu bỏ đỉnh u ra khỏi tập đang xét: giảm appear[cnt[C[u]]] đi 1, giảm cnt[C[u]] đi 1

Chú ý là các thao tác trên phải được thực hiện đúng thứ tự, theo quy tắc của thuật Mo

# Code
```c++
#include <bits/stdc++.h>
#define Task "A"
#define up(i,a,b) for (int i = a; i <= b; i++)
#define pii pair<int, int>
#define ep emplace_back
using namespace std;

const int maxn = 1e5 + 10;
const int BLOCK = 316;
int n,q;
vector<int> a[maxn];
int in[maxn], out[maxn], node[maxn];
int color[maxn], cnt[maxn], appear[maxn];
int res[maxn];
struct Query{
    int l,r,id,k;
    inline pii toPair() const{
        return make_pair((l/BLOCK), ((l/BLOCK) & 1 ? -r : r));
    }
} Q[maxn];
inline bool operator < (const Query& A, const Query& B){
    return A.toPair() < B.toPair();
}

int tdfs;
int DFS(int u, int parrent){
    in[u] = out[u] = ++tdfs;
    node[tdfs] = u;
    for (int v : a[u]){
        if (v != parrent) out[u] = max(out[u], DFS(v, u));
    }
    return out[u];
}

void add(int pos){
    int u = node[pos];
    int COLOR = color[u];
    ++cnt[COLOR];
    ++appear[cnt[COLOR]];
}

void sub(int pos){
    int u = node[pos];
    int COLOR = color[u];
    --appear[cnt[COLOR]];
    --cnt[COLOR];
}

signed main(){
    ios_base::sync_with_stdio(false);
    cin.tie(0);
    if (fopen(Task".inp", "r")){
        freopen(Task".inp", "r", stdin);
        freopen(Task".out", "w", stdout);
    }

    cin >> n >> q;
    up(i,1,n) cin >> color[i];
    up(i,1,n-1){
        int u,v;
        cin >> u >> v;
        a[u].ep(v);
        a[v].ep(u);
    }
    DFS(1, -1);

    up(i,1,q){
        int v, k;
        cin >> v >> k;
        Q[i].l = in[v];
        Q[i].r = out[v];
        Q[i].k = k;
        Q[i].id = i;
    }
    sort(Q+1, Q+q+1);

    int cl = 1, cr = 0;
    up(i,1,q){
        int l = Q[i].l;
        int r = Q[i].r;
        int id = Q[i].id;
        int k = Q[i].k;
        while (cl > l) add(--cl);
        while (cl < l) sub(cl++);
        while (cr > r) sub(cr--);
        while (cr < r) add(++cr);
        res[id] = appear[k];
    }
    up(i,1,q) cout << res[i] << "\n";
}
```
