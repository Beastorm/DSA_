## Expression Add Operators

**Problem Statement:**  

Given a string `num` that contains only digits and an integer `target`, return all possible expressions by inserting binary operators `'+'`, `'-'`, and `'*'` between the digits so that the evaluated result equals the `target`.

- No leading zeros in operands (e.g., `"05"` is invalid).
- Numbers can have multiple digits.
- Consider operator precedence: `'*'` has higher precedence than `'+'` or `'-'`.

---

>### Examples:

### Example 1:
**Input:** `num = "123"`, `target = 6`    
**Output:** `["1+2+3", "1*2*3"]`    
**Explanation:** Both expressions evaluate to `6`.    

---

### Example 2:
**Input:** `num = "232"`, `target = 8`    
**Output:** `["2+3*2", "2*3+2"]`    
**Explanation:** Both expressions evaluate to `8`.   

---
## Approach

### Intuition:

Need to insert operators (`+`, `-`, `*`) between digits to reach the target value. Challenge: handle operator precedence (`*` before `+` or `-`) without evaluating the full expression each time.

**Key Insight:** Track two values:
- eval: current expression value
- prev: last operand (needed to "undo" it when '*' appears)

For multiplication, we undo the last operation and redo it with multiplication:
eval - prev + (prev * curr)

**Example:** "2+3*4" when we reach '*4', undo the '+3', then add '3*4'

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

class ExpressionBuilder {
public:
    vector<string> validExpressions;

    void findExpressionsRecursive(string& digitString, int targetValue, int currentIndex, 
                                   long currentResult, long previousOperand, string currentExpression) {
        // Base case: processed all digits
        if (currentIndex == digitString.size()) {
            if (currentResult == targetValue) {
                validExpressions.push_back(currentExpression);
            }
            return;
        }

        // Try all possible operand lengths starting from current position
        for (int endIndex = currentIndex; endIndex < digitString.size(); endIndex++) {
            // Skip numbers with leading zeros (except single '0')
            if (endIndex != currentIndex && digitString[currentIndex] == '0') break;

            string operandString = digitString.substr(currentIndex, endIndex - currentIndex + 1);
            long operandValue = stol(operandString);

            if (currentIndex == 0) {
                // First operand: no operator preceding it
                findExpressionsRecursive(digitString, targetValue, endIndex + 1, 
                                         operandValue, operandValue, operandString);
            } else {
                // Try all 3 operators: +, -, *
                
                // Addition: simply add to the current result
                findExpressionsRecursive(digitString, targetValue, endIndex + 1, 
                                         currentResult + operandValue, 
                                         operandValue, 
                                         currentExpression + "+" + operandString);
                
                // Subtraction: subtract from current result (store negative for potential multiplication)
                findExpressionsRecursive(digitString, targetValue, endIndex + 1, 
                                         currentResult - operandValue, 
                                         -operandValue, 
                                         currentExpression + "-" + operandString);
                
                // Multiplication: undo previous operation, apply multiplication with higher precedence
                findExpressionsRecursive(digitString, targetValue, endIndex + 1, 
                                         currentResult - previousOperand + previousOperand * operandValue, 
                                         previousOperand * operandValue, 
                                         currentExpression + "*" + operandString);
            }
        }
    }

    vector<string> findAllValidExpressions(string digitString, int targetValue) {
        validExpressions.clear();
        if (digitString.empty()) return {};
        findExpressionsRecursive(digitString, targetValue, 0, 0, 0, "");
        return validExpressions;
    }
};

int main() {
    ExpressionBuilder builder;

    // Test case 1
    string inputDigits1 = "123";
    int targetValue1 = 6;
    vector<string> expressions1 = builder.findAllValidExpressions(inputDigits1, targetValue1);
    cout << "Input: digits = \"" << inputDigits1 << "\", target = " << targetValue1 << "\nOutput:\n";
    for (const string& expression : expressions1) {
        cout << expression << endl;
    }
    cout << endl;

    // Test case 2
    string inputDigits2 = "232";
    int targetValue2 = 8;
    vector<string> expressions2 = builder.findAllValidExpressions(inputDigits2, targetValue2);
    cout << "Input: digits = \"" << inputDigits2 << "\", target = " << targetValue2 << "\nOutput:\n";
    for (const string& expression : expressions2) {
        cout << expression << endl;
    }
    cout << endl;

    // Test case 3
    string inputDigits3 = "105";
    int targetValue3 = 5;
    vector<string> expressions3 = builder.findAllValidExpressions(inputDigits3, targetValue3);
    cout << "Input: digits = \"" << inputDigits3 << "\", target = " << targetValue3 << "\nOutput:\n";
    for (const string& expression : expressions3) {
        cout << expression << endl;
    }
    cout << endl;

    // Test case 4: Leading zeros
    string inputDigits4 = "00";
    int targetValue4 = 0;
    vector<string> expressions4 = builder.findAllValidExpressions(inputDigits4, targetValue4);
    cout << "Input: digits = \"" << inputDigits4 << "\", target = " << targetValue4 << "\nOutput:\n";
    for (const string& expression : expressions4) {
        cout << expression << endl;
    }
    cout << endl;

    return 0;
}
```

### Complexity Analysis:

- **Time Complexity:** O(4^n)
   - At each digit, branch into approximately 4 choices (3 operators + varying number lengths)
   - n = length of input string
   - Actual complexity closer to O(3^n * n) due to pruning

- **Space Complexity:** O(n)
   - Recursion depth: O(n)
   - String path: O(n)
   - Result storage is not counted in the space complexity analysis
