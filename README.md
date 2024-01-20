# cpp-token-scanner
A dead simple library for generating a scanner based on a set of user defined tokens.

# Usage

Simply define a list tokens.

There are three different tokens which you can add.
- `SYMBOL_TOKEN`
- `KEYWORD_TOKEN`
- `IGNORE_TOKEN`

> [!NOTE]  
> - There are some tokens which are added by default: `Error`, `EndOfFile`, `Number` and `Identifier`.
> - `IGNORE_TOKEN` can only be used to ignore characters.
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
KEYWORD_TOKEN(If,     "if")
KEYWORD_TOKEN(Else,   "else")

/**
 * Characters to ignore.
 */
IGNORE_TOKEN(" ")
IGNORE_TOKEN("\n")
IGNORE_TOKEN("\t")
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

# Scanner API
The following fundamental APIs are provided by the Scanner class.
- `set_source(string source) -> void`
- `read_source(string path) -> void`
- `scan_number() -> Token`
- `scan_identifier() -> Token`
- `scan_string() -> Token`
- `scan_until_character(char character) -> Token`
- `scan_until_token(TokenType token) -> Token`
- `scan_body(TokenType head, TokenType tail) -> Token`
- `scan_token() -> Token`
- `is_at_end() -> bool`
# Parser
A `Basic Parser` type is provided for convenience. The following showcases how a paresr class may be created.
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
};
```
The following methods are provided.
- `consume` - Consume the current token under the assumption that it matches a certain token. An error message is to be provided in the case that the match fails.
- `advance` - Move to the next token.
- `match` - Give a token type, match the current token with the type, if it matches, advance.
- `report_error` - Report error specified error message.

Here is how one might write a parsing loop.

```cpp
// Inside the parser class

using Token = TestTokenType;

auto parse() -> bool 
{
 advance();

 while (!match(Token::EndOfFile))
 {
    declaration();
 }

 return !this->has_error;
}

auto declaration() -> void
{
  if (match(Token::If)
  {
     parse_if();
  }
}

auto parse_if() -> void
{
   // At this point, the previous match would've already consumed the `If`, so we can now parse the expression.
   auto expression = parse_expression();

   // Consume the beginning of the if statement body.
   consume(Token::LBrace, "Expected '{' at the beginning of if body.");

   while (!match(Token::EndOfFile) && !match(Token::RBrace))
   {
      // parse each statements in the body...
   }

   if (previous.type == Token::EndOfFile)
   {
      report_error("Expected '}' at the end of if body.");
   }

   if (match(Token::Else))
   {
      // parse else...
   }
}
```