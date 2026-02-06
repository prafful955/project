# 🧠 DSA Counterexample Cheat Sheet
### “Always Try These Cases Before Coding”

This file helps you **validate your logic early** by forcing counterexamples.
Use it **before writing full code**.

---

## 🔑 Golden Rule

> If a logic fails on a *small or extreme input*,  
> it will fail on large inputs too.

Always try to **break your idea first**.

---

## 1️⃣ Product / Multiplication Problems ⚠️
**Keywords:** product, multiply, max product, min product

### Always Try:
[-x]
[-x, -y]
[-x, +y, -z]
[0]
[x, 0, y]
---


### Why:
- Sign flips
- Negative × Negative = Positive
- Zero resets product

🚨 If your logic discards negative values → it’s wrong.

---

## 2️⃣ Sum / Kadane / Max Sum Subarray ➕
**Keywords:** sum, maximum sum, subarray sum

### Always Try:
[-x]
[-x, -y]
[x, -y, x]
---

### Why:
- Check if restarting helps
- Verify if negative sums should be dropped

Kadane works because negative sums are always harmful.

---

## 3️⃣ Sliding Window 🪟
**Keywords:** subarray, substring, at most, at least, window

### Always Try:
smallest window (size = 1)
window just breaking the condition
window with duplicates
---

### Ask:
> When window becomes invalid, does shrinking ALWAYS help?

If NO → sliding window is invalid.

---

## 4️⃣ Two Pointers 👉👈
**Keywords:** sorted array, pair sum, closest

### Always Try:
smallest + largest
all equal values
negative + positive
---

### Why:
- Pointer movement assumptions break at edges

---

## 5️⃣ Binary Search 🔍
**Keywords:** sorted, first, last, min, max

### Always Try:
1 element
2 elements
target not present
target at boundaries
---

🚨 Most bugs are off-by-one errors.

---

## 6️⃣ Dynamic Programming 📦
**Keywords:** maximum, minimum, ways, subarray, subsequence

### Always Try:
n = 1
n = 2
overlapping choices
bad state → good later
---

### Ask:
> Am I throwing away a state that could help later?

If YES → you need more DP states.

---

## 7️⃣ Greedy Algorithms 🎯
**Keywords:** minimum, maximum, optimal

### Always Try:
local best vs global best
early choice blocks later benefit

### Ask:
> Can an early greedy decision ruin the final answer?

If YES → greedy is wrong.

---

## 8️⃣ Recursion / Backtracking 🔁
**Keywords:** combinations, permutations, subsets

### Always Try:
empty input
single element
duplicate elements

---

## 9️⃣ Graph / BFS / DFS 🌐
**Keywords:** connected, traversal, path

### Always Try:
single node
cycle
disconnected graph


---

## 🔟 Universal Edge Cases (TRY EVERY TIME)

[]
[1]
[0]
[-1]
[1,1,1]
[-1,-1,-1]
[1,-1,1]


---

## ⏱️ 30-Second Pre-Code Ritual (MANDATORY)

Before coding, answer:

1. **Pattern I’m using:**  
2. **Why it should work:**  
3. **One case that could break it:**  

If logic survives → code  
If logic breaks → rethink pattern

---

## 🧠 Final Reminder

> Good problem solvers are not faster coders.  
> They are better at generating counterexamples.

Use this sheet **every day** until it becomes instinct.
