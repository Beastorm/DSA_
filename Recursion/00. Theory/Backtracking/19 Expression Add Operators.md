## Expression Add Operators

**Problem Statement:**  

Given a string `num` that contains only digits and an integer `target`, return all possible expressions by inserting binary operators `'+'`, `'-'`, and `'*'` between the digits so that the evaluated result equals the `target`.

- No leading zeros in operands (e.g., `"05"` is invalid).
- Numbers can have multiple digits.
- Consider operator precedence: `'*'` has higher precedence than `'+'` or `'-'`.

---

>### Examples:

### Example 1:
**Input:**  
`num = "123"`, `target = 6`  
**Output:**  
`["1+2+3", "1*2*3"]`  
**Explanation:**  
Both expressions evaluate to `6`.

---

### Example 2:
**Input:**  
`num = "232"`, `target = 8`  
**Output:**  
`["2+3*2", "2*3+2"]`  
**Explanation:**  
Both expressions evaluate to `8`.

---
## Approach

### Intuition:

Need to insert operators (`+`, `-`, `*`) between digits to reach the target value. Challenge: handle operator precedence (`*` before `+` or `-`) without evaluating the full expression each time.

Key Insight: Track two values:
- eval: current expression value
- prev: last operand (needed to "undo" it when '*' appears)

For multiplication, we undo the last operation and redo it with multiplication:
eval - prev + (prev * curr)

Example: "2+3*4" when we reach '*4', undo the '+3', then add '3*4'

### Algorithm:

1. DFS with backtracking through all positions in num

2. At each position:
   - Extract numbers of varying lengths (1 to remaining digits)
   - Skip numbers with leading zeros (except "0" itself)

3. For the first number:
   - No operator, just set eval = prev = number

4. For subsequent numbers, try 3 operators:
   - Addition: eval + curr, prev = curr
   - Subtraction: eval - curr, prev = -curr
   - Multiplication: eval - prev + prev*curr, prev = prev*curr

5. When the position reaches the end:
   - If eval == target, add path to results

6. Backtrack and try the next operator/number combination


### Key Points:

- eval: Running total of expression
- prev: Last operand value (positive for '+', negative for '-')
- Multiplication trick: eval - prev + prev * curr handles precedence
- Leading zeros: Skip except standalone "0"
- Multi-digit numbers: Loop extracts numbers of all valid lengths

### Edge Cases:

- Single digit: "1" returns ["1"] if target = 1
- Leading zeros: "105" allows "10-5" (valid), but "05" is invalid
- All zeros: "00" returns "0+0", "0-0", "0*0"
- Large numbers: Use long to avoid overflow


### Diagram:
![Expression Add Operators - Backtracking](https://github.com/Beastorm/DSA_/blob/10c79abf0fbb820ccd94f18c631e4cc39c5b8185/junk/expression_target.png?raw=true)  

### C++ Code:
``` cpp
#include <iostream>
#include <vector>
#include <string>
using namespace std;

class Solution {
public:
    vector<string> res;

    void dfs(string& num, int target, int pos, long eval, long prev, string path) {
        if (pos == num.size()) {
            if (eval == target) res.push_back(path);
            return;
        }

        for (int i = pos; i < num.size(); i++) {
            // Skip numbers with leading zeros
            if (i != pos && num[pos] == '0') break;

            string currStr = num.substr(pos, i - pos + 1);
            long curr = stol(currStr);

            if (pos == 0) {
                // First number, no operator preceding it
                dfs(num, target, i + 1, curr, curr, currStr);
            } else {
                // Try all 3 operators
                dfs(num, target, i + 1, eval + curr, curr, path + "+" + currStr);
                dfs(num, target, i + 1, eval - curr, -curr, path + "-" + currStr);
                dfs(num, target, i + 1, eval - prev + prev * curr, prev * curr, path + "*" + currStr);
            }
        }
    }

    vector<string> addOperators(string num, int target) {
        res.clear();
        if (num.empty()) return {};
        dfs(num, target, 0, 0, 0, "");
        return res;
    }
};

int main() {
    Solution sol;

    string num1 = "123";
    int target1 = 6;
    vector<string> result1 = sol.addOperators(num1, target1);
    cout << "Input: num = \"" << num1 << "\", target = " << target1 << "\nOutput:\n";
    for (const string& expr : result1) {
        cout << expr << endl;
    }
    cout << endl;

    string num2 = "232";
    int target2 = 8;
    vector<string> result2 = sol.addOperators(num2, target2);
    cout << "Input: num = \"" << num2 << "\", target = " << target2 << "\nOutput:\n";
    for (const string& expr : result2) {
        cout << expr << endl;
    }
    cout << endl;

    string num3 = "105";
    int target3 = 5;
    vector<string> result3 = sol.addOperators(num3, target3);
    cout << "Input: num = \"" << num3 << "\", target = " << target3 << "\nOutput:\n";
    for (const string& expr : result3) {
        cout << expr << endl;
    }
    cout << endl;

    string num4 = "00";
    int target4 = 0;
    vector<string> result4 = sol.addOperators(num4, target4);
    cout << "Input: num = \"" << num4 << "\", target = " << target4 << "\nOutput:\n";
    for (const string& expr : result4) {
        cout << expr << endl;
    }
    cout << endl;

    return 0;
}

```

### Complexity Analysis:

**Time Complexity:** O(4^n)
- At each digit, branch into approximately 4 choices (3 operators + varying number lengths)
- n = length of input string
- Actual complexity closer to O(3^n * n) due to pruning

**Space Complexity:** O(n)
- Recursion depth: O(n)
- String path: O(n)
- Result storage is not counted in the space complexity analysis
