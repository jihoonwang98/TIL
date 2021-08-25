# Thymeleaf 기초



- **Simple expressions**
  - Variable Expressions: `${...}`
  - Selection Variable Expressions: `*{...}`
  - Message Expressions: `#{...}`
  - Link URL Expressions: `@{...}`
  - Fragment Expressions: `~{...}`



- **Literals**
  - Text literals: 'one text', 'Another one!', ...
  - Number literals: 0, 34, 3.0, 12.3, ...
  - Boolean literals: true, false
  - Null literal: null
  - Literal tokens: one, sometext, main, ...



- **Text operations**
  - String concatenation: +
  - Literal substitutions: |The name is ${name}|



- **Arithmetic operations**:

  - Binary operators: `+`, `-`, `*`, `/`, `%`
  - Minus sign (unary operator): `-`

  

- **Boolean operations**:

  - Binary operators: `and`, `or`
  - Boolean negation (unary operator): `!`, `not`

  

- **Comparisons and equality**:

  - Comparators: `>`, `<`, `>=`, `<=` (`gt`, `lt`, `ge`, `le`)

  - Equality operators: `==`, `!=` (`eq`, `ne`)

    

- **Conditional operators**:

  - If-then: `(if) ? (then)`
  - If-then-else: `(if) ? (then) : (else)`
  - Default: `(value) ?: (defaultvalue)`

  

- **Special tokens**:

  - No-Operation: `_`