# CATL

CATL is a compact text notation format for **chord voicings** and **tab events** on guitar and other stringed instruments.

|  |  |
|--|--|
| Parser (node.js) | https://github.com/j6k4m8/catl-parser |
| Render SVGs | https://github.com/j6k4m8/catl-render |

---

# Short Spec (v0.1)

## Comments / Whitespace

* Line-oriented.
* Whitespace separates tokens.
* `#` starts a comment to end of line.

---

## Chords (voicing tokens)

### Voicing token

A per-string fingering string, left→right **high→low**.

* Frets `0–9`, open `0`
* Muted `X`/`x`
* Multi-digit frets: `(<digits>)`

Examples:

```catl
X554X5
```

<img width="120" alt="image" src="https://github.com/user-attachments/assets/c33f3928-5473-42e2-b246-c94c24595e01" />


```catl
X(10)9(12)XX
```

### Names and annotations

A chord may have:

* optional **prefix annotation**: `"annotation!":<voicing>`
* optional **suffix annotation**: `<voicing>:"annotation!"`

Examples:

```catl
"Gmin7":3x332x
"Gmin7":3x332x:"Nice chord!"
X554X5:"base chord"
```

---

## Event Mode (tab events)

### String header

Declares string identifiers in order **high→low**:

```catl
{eBGDAE}
```

You can omit the header if using `{eBGDAE}`.

Identifiers are **string labels**, not pitch classes.

### Events

Two equivalent forms:

* Named-string: `<fret><string>` (e.g., `5e`, `0A`)
* Indexed-string: `<fret>@<idx>` where `idx` is `1..N` under the header

Simultaneous notes use `+`:

```catl
{eBGDAE} 0A 0D 5e 7B 3e+0D
{eBGDAE} 0@5 0@4 5@1 7@2 3@1+0@4
```

<img width="400" alt="image" src="https://github.com/user-attachments/assets/0046c686-7dbc-4edb-9c1d-6349948265a1" />


Events (or `+` groups) may be annotated:

```catl
{eBGDAE} 0A:"When" 0D 5e:"you" 7B 3e+0D:"fall in love"
```

---

## Bars / Repeats

* Measure bar: `|`
* Repeat begin: `|:`
* Repeat end: `:|`

Example:

```catl
|: "Gmin7":3x332x | X554X5 :|
```

---

# EBNF (Draft v0.1)

```ebnf
file            = { line } ;

line            = [ ws ] ( comment | header | statement | ε ) [ ws ] newline ;
comment         = "#" { any-char-except-newline } ;

header          = "{" string-id { string-id } "}" ;

statement       = element { ws element } ;

element         = chord-spec | event-spec | bar | repeat-begin | repeat-end ;
bar             = "|" ;
repeat-begin    = "|:" ;
repeat-end      = ":|" ;

(* ----- Chords ----- *)
chord-spec      = [ qstring ":" ] chord-token [ label ] ;
chord-token     = chord-char { chord-char } ;
chord-char      = digit | mute | paren-fret ;
paren-fret      = "(" digit { digit } ")" ;
mute            = "X" | "x" ;

(* ----- Events ----- *)
event-spec      = event-group [ label ] ;
event-group     = event { "+" event } ;
event           = named-event | indexed-event ;

named-event     = fret string-id ;
indexed-event   = fret "@" string-index ;

fret            = digit { digit } ;
string-index    = digit { digit } ;      (* semantic: 1..N *)

label           = ":" qstring ;

(* ----- Strings / Lexemes ----- *)
string-id       = letter ;               (* v0.1: single-letter IDs *)

qstring         = '"' { qchar } '"' ;
qchar           = escape | any-char-except-quote-or-newline ;
escape          = "\\" ( "\\" | '"' ) ;

digit           = "0"|"1"|"2"|"3"|"4"|"5"|"6"|"7"|"8"|"9" ;
letter          = "A".."Z" | "a".."z" ;
ws              = " " | "\t" ;
newline         = "\n" | "\r\n" ;
ε               = ;
```


## Comments for Implementers

- Implementers should not support single quotes instead of double quotes for labels. (May choose to use `'` symbol later.)
- Implementers may choose to support 𝄆 and 𝄇 unicode symbols
- Annotations should not be used for indicating timing.
- The '/' character and `~` character are reserved.

## Comments for Renderers

- Prefix annotations should be rendered either before or above the content they are labeling. Suffix annotations should be rendered either after or below.
- Do not render barres. They are not explicitly described in the v0.1 but may be added later on.
