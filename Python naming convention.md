- [Python Class Naming Convention](#python-class-naming-convention)
  - [💡 Core Class Naming Rules](#-core-class-naming-rules)
  - [⚠️ Special Class Conventions](#️-special-class-conventions)
  - [🔍 Class Components vs. Class Names](#-class-components-vs-class-names)
- [Python fFnction and Variable Naming Convention](#python-ffnction-and-variable-naming-convention)
  - [💡 Core Rules for Functions and Variables](#-core-rules-for-functions-and-variables)
  - [⚠️ Special Naming Conventions](#️-special-naming-conventions)
  - [🔍 Quick Summary Table](#-quick-summary-table)


# Python Class Naming Convention

According to the official Python style guide, [PEP 8](https://peps.python.org/pep-0008/), class names must follow the **CapWords convention**. This style is also widely known as **PascalCase** or **UpperCamelCase**. [1, 2, 3, 4] 

## 💡 Core Class Naming Rules

* Capitalize the first letter of every word.
* Do not use underscores to separate words.
* Keep names concise and descriptive.
* Capitalize all letters of an acronym if one is used. [1, 5, 6, 7, 8] 

``` Python
# 🟢 Correct (PascalCase / CapWords)
class UserProfile:
    pass
class HTTPServerError:
    pass


# 🔴 Incorrect
class user_profile:     # Uses snake_case
    pass
class HttpServerError: # Improper acronym capitalization
    pass
```

## ⚠️ Special Class Conventions

* **Exception Classes**: Custom error and exception classes must follow the CapWords convention and **always end with the suffix** `Error`.

``` python
class DatabaseConnectionError(Exception):
    pass
```
[9, 10, 11, 12] 

* **Internal / Private Classes**: If a class is designed purely for internal use within a specific module, prefix its name with a **single leading underscore**.


``` python
class _InternalCacheManager:
    pass
```
[13, 14]


* **Built-in Classes**: Python's native built-in classes deviate from this rule and are written in lowercase (e.g., `list`, `str`, `dict`, `int`). However, user-defined classes should always stick to CapWords. [4, 10] 

## 🔍 Class Components vs. Class Names
Do not confuse class names with the components written inside them. According to [Real Python guidelines](https://realpython.com/python-pep8/), internal components switch formatting: [15] 

| Type [7, 16, 17, 18] | Convention | Example |
|---|---|---|
| Class Name | CapWords / PascalCase | `class BankAccount: `|
| Methods | snake_case (lowercase with underscores) | `def deposit_funds(self):` |
| Attributes | snake_case (lowercase with underscores) | s`elf.account_balance = 0` |


[1] [https://peps.python.org](https://peps.python.org/pep-0008/)
[2] [https://softwareengineering.stackexchange.com](https://softwareengineering.stackexchange.com/questions/308972/python-file-naming-convention)
[3] [https://www.youtube.com](https://www.youtube.com/watch?v=uet8ZQpyJV8)
[4] [https://visualgit.readthedocs.io](https://visualgit.readthedocs.io/en/latest/pages/naming_convention.html)
[5] [https://www.geeksforgeeks.org](https://www.geeksforgeeks.org/python/python-naming-conventions/)
[6] [https://medium.com](https://medium.com/@rowainaabdelnasser/python-naming-conventions-10-essential-guidelines-for-clean-and-readable-code-fe80d2057bc9)
[7] [https://realpython.com](https://realpython.com/python-pep8/)
[8] [https://www.oracle.com](https://www.oracle.com/java/technologies/javase/codeconventions-namingconventions.html)
[9] [https://wiki.imindlabs.com.au](https://wiki.imindlabs.com.au/cs/lang/python/1_basics/0_naming_conventions/)
[10] [https://techversantinfotech.com](https://techversantinfotech.com/python-naming-conventions-points-you-should-know/)
[11] [https://www.cs.williams.edu](https://www.cs.williams.edu/~jannen/teaching/f16/cs135/lectures/lecture.27.pdf)
[12] [https://pwskills.com](https://pwskills.com/blog/python/exceptions-in-python-with-examples)
[13] [https://realpython.com](https://realpython.com/python-double-underscore/)
[14] [https://realpython.com](https://realpython.com/python-function-names/)
[15] [https://stackoverflow.com](https://stackoverflow.com/questions/42127593/should-python-class-filenames-also-be-camelcased)
[16] [https://profound.academy](https://profound.academy/python-mid/class-naming-conventions-bxS5gJ9tLnPvyZhd5giU)
[17] [https://derekarmstrong.dev](https://derekarmstrong.dev/blog/easy-to-understand-python-naming-conventions-and-coding-best-practices/)
[18] [https://inventwithpython.com](https://inventwithpython.com/beyond/chapter15.html)


# Python fFnction and Variable Naming Convention

According to PEP 8, both functions and variables must follow the **snake_case convention**. This means they should be entirely lowercase, with words separated by underscores. [1, 2, 3]

## 💡 Core Rules for Functions and Variables

* Use **only lowercase letters**.
* Separate words with a **single underscore** to improve readability.
* Choose names that describe the **purpose or value**, not the data type. [4, 5, 6, 7, 8] 
  
``` python
# 🟢 Correct (snake_case)

user_age = 25def calculate_total_price(price, tax):
    return price + tax

# 🔴 Incorrect

userAge = 25                 # Uses camelCase
def CalculateTotalPrice():   # Uses PascalCase
    pass
```

## ⚠️ Special Naming Conventions
Python uses specific prefix and suffix patterns to signal how a variable or function should be used:

* **Constants**: Use **ALL_CAPS** with underscores for values that never change during program execution.

```python
MAX_LOGIN_ATTEMPTS = 5
PI = 3.14159
```
[9, 10, 11, 12] 


* **Internal / Private Use**: Prefix a name with a **single leading underscore** to signal that a variable or function is intended for internal use within a class or module.

```python
_internal_counter = 0
def _verify_session():
    pass
```

[13] 
* **Name Clashes**: Append a **single trailing underscore** if your desired name conflicts with a built-in Python keyword.

```python
# Avoids conflict with the built-in 'class' keyword
def filter_by_class(class_="Premium"):
    pass
```

[14, 15] 
* **Dunder Methods (Avoid Custom Use)**: Names with **double leading and trailing underscores** (e.g., `__init__`, `__str__`) are reserved for Python's built-in magic methods. Never invent your own "dunder" names. [16, 17, 18, 19, 20] 

## 🔍 Quick Summary Table

| Type [21, 22, 23] | Convention | Example |
|---|---|---|
| **Variables **| snake_case | `total_score = 100` |
| **Functions** | snake_case | `def send_email():` |
| **Constants** | ALL_CAPS | `DATABASE_URL = "..."` |
| **Protected** | _leading_underscore | `_secret_key = "123" `|

[1] [https://discuss.python.org](https://discuss.python.org/t/why-in-pep-8-was-decided-both-vars-and-functions-to-be-snake-case/103376)
[2] [https://www.uniccm.com](https://www.uniccm.com/coding/variable)
[3] [https://wiki.imindlabs.com.au](https://wiki.imindlabs.com.au/cs/lang/python/1_basics/0_naming_conventions/)
[4] [https://kowainik.github.io](https://kowainik.github.io/posts/naming-conventions)
[5] [https://style.tidyverse.org](https://style.tidyverse.org/syntax.html)
[6] [https://wiki.imindlabs.com.au](https://wiki.imindlabs.com.au/cs/lang/python/1_basics/0_naming_conventions/)
[7] [https://www.janbasktraining.com](https://www.janbasktraining.com/community/python-python/what-is-the-naming-convention-in-python-for-variables-and-functions)
[8] [https://salesforcecodex.com](https://salesforcecodex.com/salesforce/the-art-of-naming-clean-code-for-salesforce-developers/)
[9] [https://www.rithmschool.com](https://www.rithmschool.com/good-ideas-for-better-variable-names/)
[10] [https://cse.iitkgp.ac.in](https://cse.iitkgp.ac.in/~pds/notes/datatype.html)
[11] [https://eng.libretexts.org](https://eng.libretexts.org/Bookshelves/Computer_Science/Programming_Languages/Introduction_to_Programming_using_Fortran_95_2003_2008_%28Jorgensen%29/04%3A_Fortran_95_2003_2008__Basic_Elements/4.03%3A_Declarations)
[12] [https://www.linkedin.com](https://www.linkedin.com/pulse/best-practices-code-naming-documentation-across-java-matikaynen)
[13] [https://algomaster.io](https://algomaster.io/learn/python/keywords-identifiers)
[14] [https://realpython.com](https://realpython.com/python-function-names/)
[15] [https://link.springer.com](https://link.springer.com/chapter/10.1007/979-8-8688-2133-2_2)
[16] [https://realpython.com](https://realpython.com/python-function-names/)
[17] [https://techvidvan.com](https://techvidvan.com/tutorials/python-identifiers/)
[18] [https://towardsdatascience.com](https://towardsdatascience.com/python-and-the-underscore-82d7ce8706d/)
[19] [https://medium.com](https://medium.com/data-science/5-different-meanings-of-underscore-in-python-3fafa6cd0379)
[20] [https://realpython.com](https://realpython.com/python-double-underscore/)
[21] [https://earthdatascience.org](https://earthdatascience.org/courses/intro-to-earth-data-science/write-efficient-python-code/intro-to-clean-code/python-pep-8-style-guide/)
[22] [https://infinum.com](https://infinum.com/handbook/android/building-quality-apps/naming)
[23] [https://www.koderhq.com](https://www.koderhq.com/tutorial/python/variables-constants/)
