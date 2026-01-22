## Print name 5 times


**Tail:** 
``` cpp
#include <iostream>
using namespace std;

void printName(int i, int n) {
    // Base case: stop when i > n
    if (i > n) return;

    // Work
    cout << "name" << endl;

    // Recursive call
    printName(i + 1, n);
}

int main() {
    printName(1, 5);  // Print "ChatGPT" 5 times
    return 0;
}
```
**Head**
``` cpp
#include <iostream>
using namespace std;

void printNameBacktrack(int i, int n) {
    // Base case: stop when i > n
    if (i > n) return;

    // Recursive call FIRST (head recursion)
    printNameBacktrack(i + 1, n);

    // Work AFTER returning (on the way back - backtracking)
    cout << "ChatGPT" << endl;
}

int main() {
    printNameBacktrack(1, 5);  // Print "ChatGPT" 5 times
    return 0;
}
```

---

## Print Linearly from 1 to N

### Approach 1: Normal Recursion (Print Before Recursive Call)
**Tail**
```cpp
#include <iostream>
using namespace std;

void printLinear(int i, int n) {
    // Base case
    if (i > n) return;
    
    // Print first
    cout << i << " ";
    
    // Then recurse
    printLinear(i + 1, n);
}

int main() {
    printLinear(1, 5);
    return 0;
}
```

### Approach 2: Backtracking (Print After Recursive Call)
**Head**
``` cpp
#include <iostream>
using namespace std;

void printLinearBacktrack(int i, int n) {
    // Base case
    if (i < 1) return;
    
    // Recurse first
    printLinearBacktrack(i - 1, n);
    
    // Print after (on the way back)
    cout << i << " ";
}

int main() {
    printLinearBacktrack(5, 5);  // Start from n
    return 0;
}
```
**Note:** Both produce the same output but traverse differently!

---

## Print Linearly from N to 1:

### Approach 1: Normal Recursion
**Tail**
```cpp
#include <iostream>
using namespace std;

void printReverse(int i, int n) {
    // Base case
    if (i < 1) return;
    
    // Print first
    cout << i << " ";
    
    // Then recurse
    printReverse(i - 1, n);
}

int main() {
    printReverse(5, 5);
    return 0;
}
```

### Approach 2: Backtracking (Print After Recursive Call)
**Head:**
``` cpp
#include <iostream>
using namespace std;

void printReverseBacktrack(int i, int n) {
    // Base case
    if (i > n) return;
    
    // Recurse first
    printReverseBacktrack(i + 1, n);
    
    // Print after (on the way back)
    cout << i << " ";
}

int main() {
    printReverseBacktrack(1, 5);  // Start from 1
    return 0;
}
```
### Key Takeaway:
**Same output, different execution:**
**Normal**: Print while going down the recursion
**Backtrack**: Print while coming back up the recursion
Both are valid, but backtracking demonstrates a head recursion pattern!


