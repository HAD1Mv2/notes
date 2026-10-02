- [Python repo naming convention](#python-repo-naming-convention)
  - [Key Naming Conventions](#key-naming-conventions)
  - [Comparison Overview](#comparison-overview)
  - [Best Practices](#best-practices)
- [Python folder and files naming convention](#python-folder-and-files-naming-convention)
  - [Key Naming Conventions](#key-naming-conventions-1)
  - [Standard Project Structure Example](#standard-project-structure-example)
  - [Critical Rules](#critical-rules)
- [Python Class Naming Convention](#python-class-naming-convention)
  - [💡 Core Class Naming Rules](#-core-class-naming-rules)
  - [⚠️ Special Class Conventions](#️-special-class-conventions)
  - [🔍 Class Components vs. Class Names](#-class-components-vs-class-names)
- [Python Function and Variable Naming Convention](#python-function-and-variable-naming-convention)
  - [💡 Core Rules for Functions and Variables](#-core-rules-for-functions-and-variables)
  - [⚠️ Special Naming Conventions](#️-special-naming-conventions)
  - [🔍 Quick Summary Table](#-quick-summary-table)
    - [Another coding example](#another-coding-example)

# Python repo naming convention
The standard convention for naming a Python project repository is **lowercase with underscores** (`lowercase_with_underscores`) or **lowercase with hyphens** (`lowercase-with-hyphens`). [1, 2, 3] 
While Git repositories widely accept various formats, aligning your repository name with your Python package name prevents confusion. [4] 
## Key Naming Conventions

* ***Repository Name (GitHub/GitLab)**: Use `lowercase-with-hyphens` (e.g., `requests-stream`) or `lowercase_with_underscores` (e.g., `requests_stream`). Hyphens are generally preferred for URL readability.
* **Package/Module Name (Import name)**: Use lowercase_with_underscores strictly ([PEP 8](https://peps.python.org/pep-0008/)). Hyphens are illegal in Python import statements.
* **Distribution Name (PyPI)**: Use `lowercase-with-hyphens` or `lowercase_with_underscores`. PyPI treats both as equivalent during installation (p`ip install my-package` matches `my_package`). [5, 6, 7, 8, 9] 

## Comparison Overview

| Target [10, 11, 12, 13, 14] | Convention | Example | Allowed Characters |
|---|---|---|---|
| Repository | Hyphens or Underscores | `data-processor` or `data_processor` | Alphanumeric, `-`, `_` |
| PyPI Package | Hyphens or Underscores | `data-processor` | Alphanumeric, `-`, `_`, `.` |
| Python Import | Underscores Only | `import data_processor` | Alphanumeric, `_` (No hyphens) |

## Best Practices

* Match Repo to Import: Name your repository with underscores if you want an exact 1:1 match with your local Python code imports.
* Keep it Short: Choose concise, mnemonic names.
* Avoid Prefixes: Do not prefix names with "python-" or "-py" unless necessary, as the ecosystem context is already implied. [15, 16] 


[1] [https://techversantinfotech.com](https://techversantinfotech.com/python-naming-conventions-points-you-should-know/)
[2] [https://forum.inductiveautomation.com](https://forum.inductiveautomation.com/t/ignition-project-development-style-guide/10425/8)
[3] [https://medium.com](https://medium.com/@nur26691/repository-naming-conventions-1065467de776)
[4] [https://unstop.com](https://unstop.com/blog/python-namespace)
[5] [https://retailtechinnovationhub.com](https://retailtechinnovationhub.com/home/2024/2/29/the-ultimate-guide-to-structuring-a-python-package)
[6] [https://dev.to](https://dev.to/thumbone/publishing-a-python-package-what-i-wish-the-maze-of-tutorials-covered-1838)
[7] [https://commandlinux.com](https://commandlinux.com/how-to/python-54axhg5/)
[8] [https://github.com](https://github.com/scottclowe/python-template-repo)
[9] [https://dev.to](https://dev.to/thumbone/publishing-a-python-package-what-i-wish-the-maze-of-tutorials-covered-1838)
[10] [https://python.plainenglish.io](https://python.plainenglish.io/how-to-organize-your-python-project-the-right-way-in-2025-c8671d873332)
[11] [https://learn.microsoft.com](https://learn.microsoft.com/en-us/azure/devops/organizations/settings/naming-restrictions?view=azure-devops)
[12] [https://packaging.python.org](https://packaging.python.org/tutorials/packaging-projects/)
[13] [https://reza-ta.medium.com](https://reza-ta.medium.com/complete-guide-from-developing-a-python-package-to-registering-on-pypi-75063786c14e)
[14] [https://techversantinfotech.com](https://techversantinfotech.com/python-naming-conventions-points-you-should-know/)
[15] [https://labex.io](https://labex.io/tutorials/git-changing-the-name-of-your-github-repository-step-by-step-398328)
[16] [https://medium.com](https://medium.com/@ashley.e.shultz/naming-company-python-packages-d1295fb9008d)

# Python folder and files naming convention
Python folder and file naming conventions strictly follow [PEP 8](https://peps.python.org/pep-0008/) guidelines. [1] 
## Key Naming Conventions

* **Packages (Folders with `__init__.py`)**: Short, all-lowercase names. Underscores are discouraged unless necessary for readability (e.g., `analytics` or `data_parser`).
* **Modules (Files)**: All-lowercase names. Underscores can be used freely to improve readability (e.g., `file_reader.py`).
* **Scripts (Executable Files)**: All-lowercase names with underscores if needed (e.g., `run_pipeline.py`).
* **Tests**: Must start with `test_` for test discovery frameworks like `pytest` (e.g., `test_auth.py`). [2, 3, 4, 5, 6] 

## Standard Project Structure Example
Here is how these naming conventions look in a standard layout:

```
my-project-repo/               <-- Repo name (hyphens or underscores)
├── .gitignore
├── README.md
├── pyproject.toml
├── src/                       <-- Source directory (lowercase)
│   └── my_package/            <-- Package folder (lowercase, underscores if needed)
│       ├── __init__.py
│       ├── core_logic.py      <-- Module file (lowercase_with_underscores)
│       └── data_loader.py     <-- Module file
└── tests/                     <-- Test folder (lowercase)
    ├── test_core.py           <-- Test file (starts with test_)
    └── test_loader.py         <-- Test file
```

## Critical Rules

* **No Hyphens in Files/Folders**: Hyphens are illegal characters in Python imports. A file named `my-script.py` cannot be imported into another file.
* **Avoid Mixed Case**: Do not use `CamelCase` for files or folders (e.g., avoid `DataLoader.py`).
* **Don't Conflict with Built-ins**: Never name a file `sys.py`, `os.py`, `json.py`, or `math.py`, as this breaks standard library imports. [7, 8] 


[1] [https://stackoverflow.com](https://stackoverflow.com/questions/52188866/python-project-structure-folder-naming-convention)
[2] [https://softwareengineering.stackexchange.com](https://softwareengineering.stackexchange.com/questions/308972/python-file-naming-convention)
[3] [https://www.reddit.com](https://www.reddit.com/r/learnpython/comments/ungwcm/the_standard_way_to_name_py_fileprojects/)
[4] [https://www.reddit.com](https://www.reddit.com/r/learnpython/comments/5oxkfv/python_file_naming_standard/)
[5] [https://softwareengineering.stackexchange.com](https://softwareengineering.stackexchange.com/questions/308972/python-file-naming-convention)
[6] [https://www.reddit.com](https://www.reddit.com/r/learnpython/comments/5oxkfv/python_file_naming_standard/)
[7] [https://dev.to](https://dev.to/cicirello/comment/1j0bl)
[8] [https://discuss.ocaml.org](https://discuss.ocaml.org/t/module-files-module-names-and-capitalization/1025)


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


# Python Function and Variable Naming Convention

According to PEP 8, both functions and variables must follow the **snake_case convention**. This means they should be entirely lowercase, with words separated by underscores. [1, 2, 3]

## 💡 Core Rules for Functions and Variables

* Use **only lowercase letters**.
* Separate words with a **single underscore** to improve readability.
* Choose names that describe the **purpose or value**, not the data type. [4, 5, 6, 7, 8] 
  
``` python
# 🟢 Correct (snake_case)

user_age = 25
def calculate_total_price(price, tax):
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

### Another coding example

``` python
# ConstantMAX_ITEMS = 100
class ShoppingCart:  # PascalCase for Classes
    def __init__(self):
        self._items = []  # Single underscore for internal/private use
        self.total_price = 0.0  # snake_case for instance variables

    def add_item(self, item_name, price):  # snake_case for functions/methods
        if len(self._items) < MAX_ITEMS:
            self._items.append(item_name)
            self.total_price += price
```

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
