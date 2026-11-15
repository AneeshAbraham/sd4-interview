# Round 2: Technical Problem Solving / Coding Round

## **Overview**
- **Duration**: 60-90 minutes
- **Format**: Video call with shared editor (LeetCode, CoderPad, HackerRank, or similar)
- **Goal**: Evaluate problem-solving ability, coding skills, and communication
- **Interviewer**: Mid-level to senior engineer

---

## **What to Expect**

### **Problem Difficulty**
- **Easy to Medium LeetCode problems** (not hard)
- Typically 1-2 problems
- First problem: ~30-45 minutes
- Second problem (if time): ~20-30 minutes
- ~10-15 minutes for discussion/follow-up

### **Problem Categories**
1. **Array/String manipulation**
2. **Sorting & Searching**
3. **Hash Maps/Sets**
4. **Two pointers**
5. **Sliding window**
6. **Stacks & Queues**
7. **Basic linked lists**
8. **Simple trees/graphs** (less common)

---

## **Java-Specific Tips**

### **Important Methods to Know**
```java
// Arrays
Arrays.sort()
Arrays.binarySearch()
Arrays.asList()
Arrays.fill()

// Collections
HashMap<K,V>
HashSet<V>
TreeMap<K,V>
TreeSet<V>
ArrayList<T>
LinkedList<T>
Stack<T>, Queue<T>

// String
String.charAt()
String.substring()
String.split()
String.toCharArray()
String.indexOf()
String.replace()

// Integer/Math
Integer.MAX_VALUE
Integer.MIN_VALUE
Math.max()
Math.min()
Math.abs()
```

### **Common Patterns**

**Two Pointer:**
```java
int left = 0, right = arr.length - 1;
while (left < right) {
    if (condition) left++;
    else right--;
}
```

**Sliding Window:**
```java
int left = 0;
for (int right = 0; right < n; right++) {
    // Add arr[right] to window
    while (condition) {
        // Remove arr[left]
        left++;
    }
    // Update result
}
```

**Hash Map Count:**
```java
Map<Character, Integer> map = new HashMap<>();
for (char c : s.toCharArray()) {
    map.put(c, map.getOrDefault(c, 0) + 1);
}
```

---

## **Interview Approach (STAR-P for Coding)**

### **S - Situation / Understand**
1. Read the problem carefully
2. Ask clarifying questions:
   - Edge cases? (empty array, single element, duplicates?)
   - Input/output constraints? (size limits, negative numbers?)
   - Space/time complexity requirements?
   - What if there's no solution?

### **T - Think**
3. Think out loud:
   - "Let me start with a brute force approach..."
   - "I notice this pattern..."
   - "What if I use a hash map?"
4. Walk through an example:
   - Small test case first
   - Trace through your logic
   - Identify issues early

### **A - Approach / Code**
5. Write clean code:
   - Start with function signature
   - Add comments for complex logic
   - Use meaningful variable names
   - Don't write overly clever code

### **R - Run / Review**
6. Test your solution:
   - Run through your example again
   - Check edge cases:
     - Empty input
     - Single element
     - All same elements
     - Maximum constraints
   - Check for off-by-one errors

### **P - Optimize**
7. Optimize if asked or if you have time:
   - Can you improve time complexity?
   - Can you reduce space complexity?
   - Trade-offs involved?
   - Code readability vs. performance?

---

## **Common Mistakes to Avoid**

❌ Jumping into coding without understanding the problem
❌ Not testing with examples
❌ Ignoring edge cases
❌ Poor variable naming
❌ Writing unreadable code
❌ Getting stuck and not asking for hints
❌ Not discussing your thought process
❌ Ignoring time complexity

---

## **Example Problem & Solution**

### **Problem: Two Sum**
Given an array of integers `nums` and an integer `target`, return the indices of the two numbers that add up to the target. You may not use the same element twice.

**Good Approach:**
```java
public int[] twoSum(int[] nums, int target) {
    // Use hash map to store value -> index mapping
    Map<Integer, Integer> map = new HashMap<>();
    
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        
        // Check if complement exists
        if (map.containsKey(complement)) {
            return new int[] { map.get(complement), i };
        }
        
        // Add current number to map
        map.put(nums[i], i);
    }
    
    // No solution found
    return new int[] {};
}
```

**Why this is good:**
- Time: O(n) single pass
- Space: O(n) for HashMap
- Clear variable names
- Handles edge case (no solution)
- Comments explain the approach

---

## **Communication Tips**

✅ **Talk as you code**: "Now I'm checking if the complement exists..."
✅ **Ask questions**: "Should I assume the input is always valid?"
✅ **Show your thinking**: "I could do this recursively, but iterative is cleaner"
✅ **Acknowledge tradeoffs**: "This uses more space but is faster"
✅ **Be honest about stuck**: "I'm not immediately seeing the optimal solution. Let me start with brute force..."

---

## **What They're Evaluating**

✅ Can you understand a problem?
✅ Do you think about edge cases?
✅ Can you write clean, working code?
✅ How do you approach optimization?
✅ Can you communicate your thinking?
✅ Do you test your work?
✅ How do you respond to feedback?

---

## **Time Management**

| Time | Activity |
|------|----------|
| 0-5 min | Understand problem, ask questions |
| 5-10 min | Discuss approach, get agreement |
| 10-40 min | Code solution |
| 40-50 min | Test & fix bugs |
| 50-60 min | Discuss complexity, optimizations, extensions |

---

## **Study Resources**

- **LeetCode**: Problems tagged "Easy" and "Medium"
- **HackerRank**: Java preparation kit
- **GeeksforGeeks**: Algorithm explanations
- **Interview Bit**: Targeted interview prep

**Focus areas for Java:**
- Arrays and strings (30%)
- Hash maps (25%)
- Sorting (15%)
- Two pointers (15%)
- Stacks/Queues (15%)

---

## **After the Interview**

- Ask what they're looking for in an ideal solution
- Get feedback on your approach
- Clarify if there were any misunderstandings
