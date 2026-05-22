# Deluge Built-in Functions — Whitelist

**Only use functions listed in this file.** If a function is not here, it does not exist in Deluge. Do not invent functions based on JavaScript, Python, or any other language.

This file will grow as new confirmed examples are added by the user.

---

## Table of Contents

1. [String Functions](#1-string-functions)
2. [List Functions](#2-list-functions)
3. [Map Functions](#3-map-functions)
4. [Type Conversion](#4-type-conversion)
5. [Type Checking](#5-type-checking)
6. [Math Functions](#6-math-functions)
7. [Date and Time](#7-date-and-time)
8. [Control Flow](#8-control-flow)
9. [invokeurl Block](#9-invokeurl-block)
10. [sendmail Block](#10-sendmail-block)
11. [Common Traps — Do Not Use](#11-common-traps--do-not-use)

---

## 1. String Functions

```
str.contains("substring")           → boolean
str.startsWith("prefix")            → boolean
str.endsWith("suffix")              → boolean
str.equalsIgnoreCase("other")       → boolean
str.length()                        → number  [NOT .length property]
str.toLowerCase()                   → string
str.toUpperCase()                   → string
str.trim()                          → string
str.replaceAll("old","new")         → string
str.subString(startIndex, endIndex) → string
str.indexOf("substring")            → number (-1 if not found)
str.toList("delimiter")             → list    [NOT split()]
str.toNumber()                      → number  [use for integer-like strings]
str.toDecimal()                     → decimal
str.toLong()                        → long integer
str.toBoolean()                     → boolean
str.isNull()                        → boolean
str.isEmpty()                       → boolean  [true if "" or null]
```

---

## 2. List Functions

```
list.size()                         → number   [NOT .length]
list.get(index)                     → value
list.add(value)                     → void
list.addAll(otherList)              → void
list.remove(index)                  → void
list.removeAll(otherList)           → void
list.contains(value)                → boolean
list.isEmpty()                      → boolean
list.isNull()                       → boolean
list.toString()                     → string
list.toMap()                        → map      [only works if list is key-value pairs]
```

**Iteration — always use `for each`, never array indexing in a while loop:**
```
for each item in myList
{
    info item;
}
```

---

## 3. Map Functions

```
map.get("key")                      → value
map.put("key", value)               → void
map.remove("key")                   → void
map.containsKey("key")              → boolean
map.keys()                          → list of strings
map.values()                        → list of values
map.size()                          → number
map.isEmpty()                       → boolean
map.isNull()                        → boolean
map.toString()                      → string
```

**Map literal syntax:**
```
myMap = {"key1":"value1","key2":42,"key3":null};
```

---

## 4. Type Conversion

```
value.toString()                    → string
value.toNumber()                    → number
value.toDecimal()                   → decimal
value.toLong()                      → long
value.toBoolean()                   → boolean
value.toList()                      → list      [converts a string like "[a,b,c]"]
value.toMap()                       → map
```

There is no `parseInt()`, `parseFloat()`, `Number()`, `String()` — use the methods above.

---

## 5. Type Checking

```
value.isNull()                      → boolean
value.isEmpty()                     → boolean  [true if null OR empty string/list/map]
value.getDataType()                 → string   ["String","Map","List","Number","Boolean","Null"]
```

**Standard null/empty guard pattern:**
```
if(value.isEmpty() == true || value.isNull() == true)
{
    // handle missing value
}
```

---

## 6. Math Functions

```
math.abs(number)                    → number
math.ceil(decimal)                  → number
math.floor(decimal)                 → number
math.round(decimal)                 → number
math.pow(base, exponent)            → number
math.max(a, b)                      → number
math.min(a, b)                      → number
```

Standard arithmetic operators `+`, `-`, `*`, `/`, `%` work on numbers directly.

---

## 7. Date and Time

```
zoho.currentdate                    → date     [current date, no parentheses — it's a variable]
zoho.currenttime                    → time
zoho.currentdatetime                → datetime

date.toString("yyyy-MM-dd")         → string
date.toDate("yyyy-MM-dd")           → date     [parse a string into date]
date.getDay()                       → number   [1–31]
date.getMonth()                     → number   [1–12]
date.getYear()                      → number
date.getHours()                     → number
date.getMinutes()                   → number
dateA.before(dateB)                 → boolean
dateA.after(dateB)                  → boolean
dateA.equals(dateB)                 → boolean
```

---

## 8. Control Flow

**if/else:**
```
if(condition)
{
    ...
}
else if(otherCondition)
{
    ...
}
else
{
    ...
}
```

**for each (list iteration):**
```
for each item in myList
{
    ...
}
```

**for (index loop):**
```
for i = 0 to myList.size() - 1
{
    item = myList.get(i);
}
```

**while:**
```
while(condition)
{
    ...
}
```

**break / continue** work inside loops.

**return** exits the function and returns a value.

---

## 9. invokeurl Block

`invokeurl` is a **statement block**, not a function. The syntax is rigid.

```
/* GET */
response = invokeurl
[
    url :"https://www.zohoapis.com/crm/v8/Contacts/" + contactId
    type :GET
    connection:"zcrm"
];

/* POST with body */
response = invokeurl
[
    url :"https://www.zohoapis.com/crm/v8/Contacts"
    type :POST
    parameters:bodyMap.toString()
    connection:"zcrm"
];

/* POST with headers */
response = invokeurl
[
    url :targetUrl
    type :POST
    headers:headersMap
    parameters:bodyMap.toString()
    connection:"connectionName"
];
```

**Rules:**
- `url`, `type`, `connection` are the minimum required keys.
- `parameters` must be a **string** (call `.toString()` on the map before passing).
- `headers` takes a map.
- There is no `body`, `json`, `payload` key — use `parameters`.
- Valid types: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`.
- The response is always a **map**.

---

## 10. sendmail Block

```
sendmail
[
    from :zoho.adminuserid
    to :"recipient@example.com"
    subject :"Subject line"
    message :"<p>HTML or plain text body</p>"
];
```

---

## 11. Common Traps — Do Not Use

These do not exist in Deluge. Never generate them.

| Hallucinated | Real Deluge equivalent |
|---|---|
| `myList.map(fn)` | `for each` loop |
| `myList.filter(fn)` | `for each` loop with `if` |
| `myList.reduce(fn)` | `for each` loop with accumulator |
| `myList.forEach(fn)` | `for each item in myList` |
| `myList.length` | `myList.size()` |
| `myList.push(value)` | `myList.add(value)` |
| `myList.indexOf(value)` | Not available; loop manually |
| `parseInt(str)` | `str.toLong()` |
| `parseFloat(str)` | `str.toDecimal()` |
| `JSON.parse(str)` | Not needed — maps are native |
| `JSON.stringify(map)` | `map.toString()` |
| `str.split("delimiter")` | `str.toList("delimiter")` |
| `str.includes("sub")` | `str.contains("sub")` |
| `typeof value` | `value.getDataType()` |
| `null` keyword | Use `isNull()` checks; assign as `""` or omit |
| `console.log()` | `info value;` |
| `let x =` / `var x =` / `const x =` | `x =` (no declaration keyword) |
| `try/catch` | Not available; use response guards |
