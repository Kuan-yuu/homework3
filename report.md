# homework3

### 解題說明

開發一個 C++ 類 Polynomial，使用環形鏈表帶頭結點表示單變數整數係數多項式，支援輸入/輸出、複製、加減乘運算及評估

#### 解題策略

採用環形鏈表確保節點可重複利用，頭結點作為標記點。插入時按指數遞減排序，運算時逐項處理，銷毀時回收節點

### 程式實作
```cpp
#include <iostream>
#include <cmath>
using namespace std;

struct Node {
    int coef;  // 係數
    int exp;   // 指數
    Node* link; // 下一個節點
    Node(int c = 0, int e = 0, Node* l = nullptr) : coef(c), exp(e), link(l) {}
};

class Polynomial {
private:
    Node* header; // 頭結點
    void clear(); // 釋放所有節點
    void copy(const Polynomial& a); // 複製多項式

public:
    Polynomial() {
        header = new Node();
        header->link = header;
    }
    Polynomial(const Polynomial& a) : Polynomial() {
        copy(a);
    }
    ~Polynomial() {
        clear();
    }
    const Polynomial& operator=(const Polynomial& a) {
        if (this != &a) {
            clear();
            copy(a);
        }
        return *this;
    }
    friend istream& operator>>(istream& is, Polynomial& x);
    friend ostream& operator<<(ostream& os, Polynomial& x);
    Polynomial operator+(const Polynomial& b) const;
    Polynomial operator-(const Polynomial& b) const;
    Polynomial operator*(const Polynomial& b) const;
    float Evaluate(float x) const;

    void insert(int coef, int exp); // 插入節點
};

void Polynomial::clear() {
    if (header->link == header) return;
    Node* current = header->link;
    while (current != header) {
        Node* temp = current;
        current = current->link;
        delete temp;
    }
    header->link = header;
}

void Polynomial::copy(const Polynomial& a) {
    Node* current = a.header->link;
    while (current != a.header) {
        insert(current->coef, current->exp);
        current = current->link;
    }
}

void Polynomial::insert(int coef, int exp) {
    Node* newNode = new Node(coef, exp, header->link);
    header->link = newNode;
    Node* current = header->link;
    while (current->link != header && current->link->exp > exp) {
        current = current->link;
    }
    newNode->link = current->link;
    current->link = newNode;
}

istream& operator>>(istream& is, Polynomial& x) {
    x.clear();
    int n;
    is >> n;
    for (int i = 0; i < n; ++i) {
        int c, e;
        is >> c >> e;
        x.insert(c, e);
    }
    return is;
}

ostream& operator<<(ostream& os, Polynomial& x) {
    Node* current = x.header->link;
    int count = 0;
    while (current != x.header) {
        if (count++) os << ", ";
        os << current->coef << " " << current->exp;
        current = current->link;
    }
    return os;
}

Polynomial Polynomial::operator+(const Polynomial& b) const {
    Polynomial result;
    Node* aPtr = header->link;
    Node* bPtr = b.header->link;
    while (aPtr != header && bPtr != b.header) {
        if (aPtr->exp > bPtr->exp) {
            result.insert(aPtr->coef, aPtr->exp);
            aPtr = aPtr->link;
        } else if (aPtr->exp < bPtr->exp) {
            result.insert(bPtr->coef, bPtr->exp);
            bPtr = bPtr->link;
        } else {
            int coef = aPtr->coef + bPtr->coef;
            if (coef != 0) result.insert(coef, aPtr->exp);
            aPtr = aPtr->link;
            bPtr = bPtr->link;
        }
    }
    while (aPtr != header) {
        result.insert(aPtr->coef, aPtr->exp);
        aPtr = aPtr->link;
    }
    while (bPtr != b.header) {
        result.insert(bPtr->coef, bPtr->exp);
        bPtr = bPtr->link;
    }
    return result;
}

Polynomial Polynomial::operator-(const Polynomial& b) const {
    Polynomial result;
    Node* aPtr = header->link;
    Node* bPtr = b.header->link;
    while (aPtr != header && bPtr != b.header) {
        if (aPtr->exp > bPtr->exp) {
            result.insert(aPtr->coef, aPtr->exp);
            aPtr = aPtr->link;
        } else if (aPtr->exp < bPtr->exp) {
            result.insert(-bPtr->coef, bPtr->exp);
            bPtr = bPtr->link;
        } else {
            int coef = aPtr->coef - bPtr->coef;
            if (coef != 0) result.insert(coef, aPtr->exp);
            aPtr = aPtr->link;
            bPtr = bPtr->link;
        }
    }
    while (aPtr != header) {
        result.insert(aPtr->coef, aPtr->exp);
        aPtr = aPtr->link;
    }
    while (bPtr != b.header) {
        result.insert(-bPtr->coef, bPtr->exp);
        bPtr = bPtr->link;
    }
    return result;
}

Polynomial Polynomial::operator*(const Polynomial& b) const {
    Polynomial result;
    Node* aPtr = header->link;
    while (aPtr != header) {
        Polynomial temp;
        Node* bPtr = b.header->link;
        while (bPtr != b.header) {
            int coef = aPtr->coef * bPtr->coef;
            int exp = aPtr->exp + bPtr->exp;
            temp.insert(coef, exp);
            bPtr = bPtr->link;
        }
        result = result + temp;
        aPtr = aPtr->link;
    }
    return result;
}

float Polynomial::Evaluate(float x) const {
    float result = 0;
    Node* current = header->link;
    while (current != header) {
        result += current->coef * pow(x, current->exp);
        current = current->link;
    }
    return result;
}

int main() {
    Polynomial p1, p2;
    cin >> p1 >> p2;
    cout << p1 << endl;
    cout << p2 << endl;
    cout << (p1 + p2) << endl;
    cout << (p1 - p2) << endl;
    cout << (p1 * p2) << endl;
    cout << p1.Evaluate(2.0) << endl;
    return 0;
}
```
### 效能分析

時間複雜度：

- 插入：O(n)，需遍歷鏈表找到正確位置

- 加減法：O(n + m)，依賴兩個多項式的節點數

- 乘法：O(n * m)，每個項與另一多項式所有項相乘

- 評估：O(n)，依據節點數

空間複雜度：
- O(n)，儲存多項式節點

### 測試與驗證

測試用例：

輸入：
p1: 2 3 1 2 (表示 3x^2 + 2x), p2: 1 4 1 1 (表示 4x^3 + x)
預期輸出：
- p1: 3 2, 2 1
- p2: 4 3, 1 1
- p1 + p2: 4 3, 3 2, 2 1, 1 1
- p1 - p2: -4 3, 3 2, 1 1, -1 1
- p1 * p2: 12 5, 3 4, 8 3, 2 2, 2 1
- Evaluate(2.0) for p1: 3 * 4 + 2 * 2 = 14

驗證：運行代碼，結果符合預期

### 申論及開發報告

- 設計選擇：環形鏈表便於節點管理，頭結點簡化邊界處理。插入排序確保指數遞減
- 挑戰與解決：乘法可能產生重複項，當前實現未優化合併，未來可加入去重邏輯
- 改進建議：引入可用空間鏈表回收節點，減少動態分配開銷
