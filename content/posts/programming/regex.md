---
date: 2025-05-10T19:04:01+02:00
# description: ""
# image: ""
lastmod: 2025-05-10
showTableOfContents: true
tags: ["python", "regex", "notes"]
title: "Regular expressions (regex)"
type: "post"
weight: 3
---

## Introduction

Regular expressions are a powerful tool used for searching, matching and
manipulating strings based on patterns. To use it the following steps can be
followed:

1. Import the **regex** module with `import re`.
2. Create a *Regex object* with the `re.compile()` function.
(Remember to use a raw string.)
3. Pass the string to search into the Regex object’s `search()` method.
This returns a *Match object*.
4. Call the Match object’s `group()` method to return the first found string of
the actual matched text.

```py
import re

>>> phone_regex = re.compile(r'\d{3}-\d{3}-\d{4}')  # check for a phone number
>>> mo = phone_regex.search('My number is 312-156-8940.')  # get match objects
>>> print(mo.group())  # print match objects
'312-156-8940'
```

## Pattern matching capabilities

### Grouping with parenthesis

Adding parenthesis to the regular expression will create *groups*. The different
groups can be returned with the `group()`method, specifying the group number
(1-indexed) to return.

```py
>>> phone_regex = re.compile(r'(\d{3})-(\d{3}-\d{4})')  # added parenthesis
>>> mo = phone_regex.search('My number is 312-156-8940.')
>>> mo.group(1)
'312'
>>> mo.group(2)
'156-8940'
>>> mo.group()  # returns all as a string
'312-156-8940'
>>> mo.groups()  # returns tuple
('312', '156-8940')
```

### Other functionalities

| Regex | Returns | Functionality |
| ----- | ------- | :------------ |
| `r'Batman\|Robin'` | `There's Batman and Robin` -> `Batman` | Looks for either of the given substrings |
| `r'Bat(man\|mobile\|copter)'` | `Batmobile lost a wheel` -> `Batmobile` | The first string + any of the substrings in parenthesis |
| `r'Bat(wo)?man'` | `Batman and Batwoman` -> `Batman` | The substring before `?` can be zero or once |
| `r'Bat(wo)*man'` | `Batwowoman and Batwoman` -> `Batwowoman` | The substring before `*` can be zero or more times |
| `r'Bat(wo)+man'` | `Batman and Batwoman` -> `batwoman` | The substring before `+` has to be once or more |
| `r'(ha){3}'` | `hahahaha` -> `hahaha` | Match the pattern `{n}` times |
| `r'(ha){2,4}'` | `hahahahaha` -> `hahahaha` | Match pattern within range `{n,m}`, always takes higher range by default (greedy)  |
| `r'(ha){2,4}?'` | `hahahahaha` -> `haha` | Match pattern within range `{n,m}`, takes lower range (non-greedy)  |

### search() vs findall()

While `search()` will return a *match object* of the *first* matched text in the
search string, the `findall()` method will return a *list* of every match in the
string. If there are groups in the regex, `findall()` will return a list of
tuples.

```py
>>> phoneNumRegex = re.compile(r'\d{3}-\d{3}-\d{4}')
>>> mo = phoneNumRegex.search('Cell: 415-555-9999 Work: 212-555-0000')
>>> mo.group()
'415-555-9999'
>>> phoneNumRegex.findall('Cell: 415-555-9999 Work: 212-555-0000')
['415-555-9999', '212-555-0000']
```

## Character Classes

### Shorthand codes for common character classes

[cheatsheet](https://pythex.org/)

| Shorthand character class | Represents |
| ------------------------- | ---------- |
| `\d` | Any numeric digit from 0-9 |
| `\D` | Any character that is *not* a numeric digit |
| `\w` | Any letter, numeric digit, or the underscore character. (like matching “word” characters) |
| `\W` | Any character that is not a letter, numeric digit, or the underscore character |
| `\s` | Any space, tab, or newline character ("space" characters) |
| `\S` | Any character that is not a space, tab, or newline |

### Creating own character classes

To create character classes, the characters or character ranges can be specified
inside square brackets, for example:

* `r'[aeiouAEIOU]'` for vowels
* `r'[a-zA-Z]'` for lower case and upper case letters
* `r'[0-5.]'` for characters from 0 to 5 and period. 

A negative character class can also be created, i.e. a class to exclude the
speficied characters. To do this, the *caret character* (`^`)  is used at the
beginning of the class. For example:

* `r'[^aeiouAEIOU]'` for any character that is not a vowel

#### The caret and dollar signs

The caret `^` is also used to indicate that the match must occur at the
*beginning* of the searched text. Likewise, the dollar sign `$` character
indicates that the string *must end* with the regex pattern.

```py
# starts with
>>> begginsw = re.compile(r'^Hello')
>>> begginsw.search('Hello world')
<re.Match object; span=(0, 5), match='Hello'>

# ends with
>>> endsWithNumber = re.compile(r'\d$')
>>> endsWithNumber.search('Your number is 42')
<re.Match object; span=(16, 17), match='2'>

# begins and ends with
>>> wholeStringIsNum = re.compile(r'^\d+$')
>>> wholeStringIsNum.search('1234567890')
<re.Match object; span=(0, 10), match='1234567890'>
>>> wholeStringIsNum.search('12345xyz67890') == None
True
```

#### Wildcard character

* The dot `.` is a *wildcard* and will match anything except for a new line.
* `.*` will match everything and anything except for a new line. Use case below.
* To match also the new line character, pass `re.DOTALL` as a second argument in
`re.compile()`

```py
# match everything and anything
>>> nameRegex = re.compile(r'First Name: (.*) Last Name: (.*)')
>>> mo = nameRegex.search('First Name: Al Last Name: Sweigart')
>>> mo.group(1)
'Al'
>>> mo.group(2)
'Sweigart'

# include new line
>>> newlineRegex = re.compile('.*', re.DOTALL)
>>> newlineRegex.search('Serve the public trust.\nProtect the innocent.').group()
'Serve the public trust.\nProtect the innocent.'
```

#### Case insensitive matching

To ignore casing pass the `re.IGNORECASE` or `re.I` as a second argument to
`re.compile()`

### Substituting strings

The `sub()` method for a *regex object* can be used to replace text patterns by
another one. It takes as arguments the string to replace any matches and the
string.

```py
# using another word
>>> namesRegex = re.compile(r'Agent \w+')
>>> namesRegex.sub('CENSORED', 'Agent Alice gave the secret documents to Agent Bob.')
'CENSORED gave the secret documents to CENSORED.'

# using the same word to replace
>>> agentNamesRegex = re.compile(r'Agent (\w)\w*')
>>> agentNamesRegex.sub(r'\1****', 'Agent Alice told Agent Carol that Agent
Eve knew Agent Bob was a double agent.')
A**** told C**** that E**** knew B**** was a double agent.'
```

### Multi-line regex

To improve readability and break complex regex into separate lines, the
`re.VERBOSE` argument can be passed to `re.compile()`. This would allow a regex
as follows:

```py
phoneRegex = re.compile(r'''(
    (\d{3}|\(\d{3}\))?            # area code
    (\s|-|\.)?                    # separator
    \d{3}                         # first 3 digits
    (\s|-|\.)                     # separator
    \d{4}                         # last 4 digits
    (\s*(ext|x|ext.)\s*\d{2,5})?  # extension
    )''', re.VERBOSE)
```

### Combining re.IGNORECASE, re.DOTALL, and re.VERBOSE

`re.compile()` only takes two arguments, so in order to use multiple
functionalities in the same object the pipe `|` character can be used:

```py
>>> someRegexValue = re.compile('foo', re.IGNORECASE | re.DOTALL)
>>> someRegexValue = re.compile('foo', re.IGNORECASE | re.DOTALL | re.VERBOSE)
```

## Cheat Sheet

| Regex | Functionality |
| ----: | ------------- |
| `?` | matches zero or one of the preceding group.|
| `*` | matches zero or more of the preceding group.|
| `+` | matches one or more of the preceding group.|
| `{n}` | matches exactly n of the preceding group.|
| `{n,}` | matches n or more of the preceding group.|
| `{,m}` | matches 0 to m of the preceding group.|
| `{n,m}` | matches at least n and at most m of the preceding group.|
| `{n,m}?`, `*?`, `+?` | performs a non-greedy match of the preceding group.|
| `^spam` | means the string must begin with spam.|
| `spam$` | means the string must end with spam.|
| `.` | matches any character, except newline characters.|
| `\d`, `\w`, `\s` | match a digit, word, or space character, respectively.|
| `\D`, `\W`, `\S` | match anything except a digit, word, or space character, respectively.|
| `[abc]` | matches any character between the brackets (such as a, b, or c).|
| `[^abc]` | matches any character that isn’t between the brackets.|