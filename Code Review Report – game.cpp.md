# Code Review Report – game.cpp

SVV Lab Task · Static code verification of `game.cpp` · Severity scale: Critical, High, Medium, Low

## 2. Code Review Report

| Line Number | Defect Type | Why It Is a Problem | Proposed Fix | Severity |
| --- | --- | --- | --- | --- |
| 17, 40 | Class design: pass-by-value | `std::string` parameters are copied on every call; the constructor assigns members instead of using an initializer list. | Take `const std::string&` and use an initializer list. | Low |
| 41 | Array out of bounds | `monsters` holds 5 pointers, but `main()` spawns 6 (line 119), so the write goes past the array (undefined behavior, memory corruption). | Check `monsterCount < MAX_MONSTERS` before inserting and return `false` if full, or use `std::vector`. | High |
| 47 | Assignment instead of comparison | `monsters[i]->id = id` overwrites the id and is true for any non-zero id, so the first monster is always returned and ids are corrupted. | Use `==`: `if (monsters[i]->id == id)`. | Critical |
| 55–56 | Signed/unsigned arithmetic | `dmg` is `unsigned`, so `m->health - dmg` is unsigned and `< 0` is never true; health wraps to a huge value. The function also returns 0 without updating health. | Use `int`; if `damage >= m->health` set `health = 0`, else subtract. | High |
| 56 | Null pointer dereference | `findMonster(99)` (line 123) returns `nullptr`, and `m->health` is then dereferenced, causing a crash. | Add `if (m == nullptr) return -1;` before using `m`. | Critical |
| 64–65 | Loop off-by-one | `i <= monsterCount` reads `monsters[monsterCount]`, an uninitialized pointer, so `->health` is undefined behavior. | Change the condition to `i < monsterCount`. | High |
| 72 | Null pointer dereference | `getMonsterLabel` dereferences `m->id` and `m->name` without checking for `nullptr`. | Check `m` first and return an empty or "unknown" label. | High |
| 73 | Reference to local variable | The function returns `std::string&` to local `label`, which is destroyed on return, so the caller uses a dangling reference. | Return `std::string` by value. | Critical |
| 77 | Uninitialized variable | `double sum;` has an indeterminate value, so `sum +=` produces garbage. | Initialize: `double sum = 0.0;`. | High |
| 80 | Division by zero / narrowing | If `monsterCount` is 0 the division is by zero, and the `double` average is silently truncated to `int`. | Guard `monsterCount == 0` and return `double`. | Medium |
| 85–86 | Use after free / dangling pointer | `m->name` is read after `delete m`. The freed pointer also stays in `monsters[]`, so the destructor deletes it again (double free). `m` may also be `nullptr`. | Null-check `m`, print before deleting, then remove it from the array (shift the rest, set the last slot to `nullptr`, decrement `monsterCount`). | Critical |
| 91 | Integer division truncation | `attack * armorPercent / 100` is integer math, so the fraction is lost before conversion to `double` (e.g. 5 × 25 / 100 gives 1, not 1.25); large inputs can overflow. | Use `attack * armorPercent / 100.0`. | Medium |
| 94–99 | Missing return statement | `playerRank()` has no return when `score <= 100` (undefined behavior; the call on line 128 uses 50). | Add a final `else return 3;` (or a default return). | High |
| 102–105 | Buffer overflow | `std::cin >> name` into a `char[10]` has no length limit; a name over 9 characters overwrites the heap. | Use `std::string name;` with `std::cin >> name`. | Critical |
| 107 | Unvalidated input | If the user enters non-numeric text, `health` is not set properly and the program continues with bad data. | Check `if (!(std::cin >> health))` and handle the error. | Low |
| 109 | Mismatched delete | Memory allocated with `new[]` is freed with `delete`, which is undefined behavior. | Use `delete[] name;`, or remove the allocation by using `std::string`. | High |
| 121 | Memory leak | `bonus` is allocated with `new` and never deleted. | `delete bonus;`, or use a stack object or `std::unique_ptr`. | Medium |
| 133 | Shallow copy (Rule of Three) | `Game copy = game;` copies the raw `Monster*` pointers, so both objects delete the same monsters on destruction (double free). | Delete the copy constructor and assignment operator, or implement a deep copy; better, use `std::vector<std::unique_ptr<Monster>>`. | Critical |

**Totals:** 18 defects — 6 Critical, 7 High, 3 Medium, 2 Low.
