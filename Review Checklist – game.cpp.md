# Review Checklist – game.cpp

SVV Lab Task · Static code verification of `game.cpp` · Result: **ISSUE** / **OK**

## 1. Review Checklist

| Category | Result | Defect |
| --- | --- | --- |
| 1 – Initialization | ISSUE | Line 77: `double sum` is never initialized before `sum +=` in `averageAttack()`, so the result is garbage. |
| 2 – Pointers | ISSUE | Lines 56 and 72: `findMonster()` can return `nullptr` (e.g. id 99 on line 123) and the result is dereferenced without a check. Lines 85–86: `m` is used after `delete m` (use-after-free) and the dangling pointer stays in the array (double delete in the destructor). Line 121: `bonus` is never freed (memory leak). |
| 3 – Arrays / buffers | ISSUE | Line 41: no capacity check, so the 6th monster spawned on line 119 writes past `monsters[5]`. Lines 102–105: `cin >> name` into a `char[10]` has no length limit (buffer overflow). |
| 4 – Loops | ISSUE | Line 64: `i <= monsterCount` is off by one and reads `monsters[monsterCount]`, an uninitialized pointer. |
| 5 – Arithmetic | ISSUE | Lines 55–56: `unsigned dmg` makes `health - dmg` unsigned, so `< 0` can never be true (wraparound). Line 80: division by zero when `monsterCount` is 0, and the `double` result is truncated to `int`. Line 91: integer division truncates before conversion to `double`. |
| 6 – Control flow | ISSUE | Line 47: `=` used instead of `==` inside the `if`. Lines 94–99: `playerRank()` has no return statement when `score <= 100`. |
| 7 – Function calls | ISSUE | Line 73: returns a reference to local variable `label`. Line 109: `delete` used on memory from `new[]` (needs `delete[]`). Line 123: `attackMonster(99, 20)` is called with an id that does not exist and its null result is never handled. Line 107: input from `cin` is not validated. |
| 8 – Classes | ISSUE | Line 133: `Game copy = game;` uses the default copy, which shares the raw pointers and causes a double delete (Rule of Three violated; `Game` owns raw pointers, lines 27 and 35–38). Lines 17 and 40: strings passed by value (minor inefficiency). |

**Summary:** 8 of 8 categories contain at least one issue.
