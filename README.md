# cpp-token-scanner
A dead simple library for generating a scanner based on a set of user defined tokens.

# Usage

Simply define a list tokens.

There are two different tokens which you can add.
- `SYMBOL_TOKEN`
- `KEYWORD_TOKEN`

> [!NOTE]  
> There are some tokens which are added by default: `Error`, `EndOfFile`, `Number` and `Identifier`.
```c
// my_tokens.def

/**
 * Grouping.
 */
SYMBOL_TOKEN(LBrace,  "{")
SYMBOL_TOKEN(RBrace,  "}")
SYMBOL_TOKEN(LParen,  "(")
SYMBOL_TOKEN(RParen,  ")")
SYMBOL_TOKEN(LSqaure, "[")
SYMBOL_TOKEN(RSquare, "]")
SYMBOL_TOKEN(Quote,   "'")

SYMBOL_TOKEN(Comma,      ",")
SYMBOL_TOKEN(Semicolon,  ";")
SYMBOL_TOKEN(colon,      ":")
SYMBOL_TOKEN(Dot,        ".")
SYMBOL_TOKEN(Equal,      "=")

/**
 * Keywords.
 */
KEYWORD_TOKEN(Require, "REQUIRE")
KEYWORD_TOKEN(Var,     "VAR")
KEYWORD_TOKEN(Set,     "SET")
KEYWORD_TOKEN(Load,    "LOAD")
KEYWORD_TOKEN(Eval,    "EVAL")
KEYWORD_TOKEN(And,     "AND")
KEYWORD_TOKEN(Test,    "TEST")
KEYWORD_TOKEN(Is,      "IS")
KEYWORD_TOKEN(Not,     "NOT")
```

Now you can simply dedicate a header file for the declaration of the scanner.

> [!IMPORTANT]  
> You must define `TOKEN_CLASS_NAME` and `TOKEN_DESCRIPTOR_FILE`.

```cpp
// token.hpp
#pragma once
#define TOKEN_CLASS_NAME TestTokenType
#define TOKEN_DESCRIPTOR_FILE "my_tokens.def"
#include "scanner_gen.hpp"
```

Now we will be given `TestTokenType` and `TestTokenTypeScanner`.

Usage example:
```cpp
#include "token.hpp"

int main()
{
 // Print all keyword tokens.
 for (const auto v: TestTokenType::keyword_tokens)
 {
  std::cout << "Keyword: " << v.name() << '\n';
 }

 // Declare a new scanner.
 TestTokenTypeScanner scanner;

 // Read source.
 if (!scanner.read_source("example.txt"))
 {
  std::cerr << "Couldn't read file!\n";
  exit(1);
 }

 // Scan.
 while (!scanner.is_at_end())
 {
  const auto token = scanner.scan_token();
  std::cout << token.type.name() << " " << token.lexeme << '\n';
 }
} 
```

The generated token type also work with switch cases.

```cpp
const auto token = scanner.scan_token();

switch(token.type)
{
  case TestTokenType::Error:
  case TestTokenType::EndOfFile:
  case TestTokenType::Number:
  case TestTokenType::Identifier:
  // other cases...
}
```
Additionally, a `Basic Parser` type is provided. The following showcases how a paresr class may be created.
```cpp
class TestParser : public BaseParser<TestTokenTypeScanner, TestTokenType>
{
  public:
    /**
     * Constructor with file path of source.
     */
    [[nodiscard]] explicit TestParser(const std::string& file_path)
        : BaseParser<TestTokenTypeScanner, TestTokenType>(file_path)
    {
    }

    /**
     * Default Ctor prohibted.
     */
    constexpr TestParser() = delete;
```
The following methods are provided.
- `consume` - Consume the current token under the assumption that it matches a certain token. An error message is to be provided in the case that the match fails.
- `advance` - Move to the next token.
- `match` - Give a token type, match the current token with the type, if it matches, advance.
