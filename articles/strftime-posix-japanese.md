---
title: "strftime, strftime_l - convert date and time to a string"
emoji: "📅" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "strftime", "posix" ]
layout: default
published: false
---

https://pubs.opengroup.org/onlinepubs/9699919799/functions/strftime.html の後半の日本語訳です。厳密に読み込まなければならないことがあったので、解釈しながら (一部注釈を加えながら) 訳を作ったものを、もののついでに公開しておくものです。訳の正しさについて保証するものではありません。

----

## 例

### ローカライズした日付文字列を取得する

> The following example first sets the locale to the user's default. The locale information will be used in the nl_langinfo() and strftime() functions. The nl_langinfo() function returns the localized date string which specifies how the date is laid out. The strftime() function takes this information and, using the tm structure for values, places the date and time information into datestring.

以下の例は、最初にロケール (locale) をユーザーのデフォルトに設定しています。ロケールの情報は `nl_langinfo()` 関数と `strftime()` 関数で使われます。 `nl_langinfo()` 関数は、日付をレイアウトを指定するためのローカライズした日付文字列を返します。 `strftime()` 関数はその情報を受け取って、日付と時刻の情報を `tm` 構造体の値から日付文字列 (datestring) として出力します。

```c
#include <time.h>
#include <locale.h>
#include <langinfo.h>
...
struct tm *tm;
char datestring[256];
...
setlocale (LC_ALL, "");
...
strftime (datestring, sizeof(datestring), nl_langinfo (D_T_FMT), tm);
...
```

## APPLICATION USAGE

> The range of values for %S is [00,60] rather than [00,59] to allow for the occasional leap second.

`%S` の値の範囲は `[00,59]` ではなく、ときどきあるうるう秒を受け付けるため `[00,60]` です。

> Some of the conversion specifications are duplicates of others. They are included for compatibility with nl_cxtime() and nl_ascxtime(), which were published in Issue 2.

変換指定子 (conversion specification) のうちいくつかは、他のものと重複しています。それらは Issue 2 [^issue-2] で公開された `nl_cxtime()` 関数や `nl_ascxtime()` 関数との互換性のために (今でも) 含まれています。

[^issue-2]: おそらく XPG2: X/Open Portability Guide, Issue 2 のことと考えられる。

> The %C, %F, %G, and %Y format specifiers in strftime() always print full values, but the strptime() %C, %F, and %Y format specifiers only scan two digits (assumed to be the first two digits of a four-digit year) for %C and four digits (assumed to be the entire (four-digit) year) for %F and %Y. This mimics the behavior of printf() and scanf(); that is:

`strftime()` のフォーマット指定子 (format specifier) `%C`, `%F`, `%G`, `%Y` は常に値の全体を印字しますが、 `strptime()` のフォーマット指定子 `%C` は二桁のみ (四桁の年の最初の二桁と考えられる) をスキャンし,  `%F` と `%Y` は四桁 ((四桁の) 年の全体と考えられる) をスキャンします。これは、以下の `printf()` と `scanf()` の挙動を真似たものです。

```c
printf("%2d", x = 1000);
```

> prints "1000", but:

上のコードは `"1000"` を印字しますが、

```c
scanf("%2d", &x);
```

> when given "1000" as input will only store 10 in x). Applications using extended ranges of years must be sure that the number of digits specified for scanning years with strptime() matches the number of digits that will actually be present in the input stream. Historic implementations of the %Y conversion specification (with no flags and no minimum field width) produced different output formats. Some always produced at least four digits (with 0 fill for years from 0 through 999) while others only produced the number of digits present in the year (with no fill and no padding). These two forms can be produced with the '0' flag and a minimum field width options using the conversions specifications %04Y and %01Y, respectively.

上のコードに入力として `"1000"` が与えられると `x` には `10` だけが格納されます。年の範囲を拡大して使うアプリケーションは、年をスキャンするために指定した桁数が、実際に入力ストリームに現れる桁数と一致することを確認しなければなりません。 (フラグも最小フィールド幅の指定もない) `%Y` 変換指定子に対して、歴史的な実装では、実装によって異なるフォーマットを出力していました。一部の実装は常に最低でも四桁を (0 年から 999 年の間は 0 埋めして) 生成しましたが、その他の実装ではその年の桁数だけ (埋めなしに) 生成していました。これら二つの形式は `0` フラグと最小フィールド幅のオプションを用いて、それぞれ順に変換指定子 `%04Y` と `%01Y` で生成できます。

> In the past, the C and POSIX standards specified that %F produced an ISO 8601:2000 standard date format, but didn't specify which one. For years in the range [0001,9999], POSIX.1-2017 requires that the output produced match the ISO 8601:2000 standard complete representation extended format (YYYY-MM-DD) and for years outside of this range produce output that matches the ISO 8601:2000 standard expanded representation extended format (<+/-><Underline>Y</Underline>YYYY-MM-DD). To fully meet ISO 8601:2000 standard requirements, the producer and consumer must agree on a date format that has a specific number of bytes reserved to hold the characters used to represent the years that is sufficiently large to hold all values that will be shared. For example, the %+13F conversion specification will produce output matching the format "<+/->YYYYYY-MM-DD" (a leading '+' or '-' sign; a six-digit, 0-filled year; a '-'; a two-digit, leading 0-filled month; another '-'; and the two-digit, leading 0-filled day within the month).

過去の C や POSIX の標準では `%F` は ISO 8601:2000 標準の日付フォーマットを生成していましたが、どちらのものか [^which-iso-8601-2000] は指定していませんでした。 POSIX.1-2017 では、まず `[0001,9999]` の範囲の年に対しては ISO 8601:2000 標準 (ISO 8601:2000 standard) の完全表現 (complete representation) の拡張書式 (extended format) (`YYYY-MM-DD`) に一致することが必要です。そして、この範囲外の年に対しては ISO 8601:2000 標準 (ISO 8601:2000 standard) の展開表現 (expanded representation) の拡張書式 (extended format) (`<+/-><Underline>Y</Underline>YYYY-MM-DD`) に一致することを要求します。 [^iso8601-expression] To fully meet ISO 8601:2000 standard requirements, the producer and consumer must agree on a date format that has a specific number of bytes reserved to hold the characters used to represent the years that is sufficiently large to hold all values that will be shared. たとえば `%+13F` 変換指定子は `"<+/->YYYYYY-MM-DD"` (最初に `'+'` か `'-'` の符号; 六桁の 0 埋めした年; 文字 `'-'`; 二桁の 0 埋めした月; 再び文字 `'-'`; 二桁の 0 埋めした日) に一致するフォーマットを出力します。

[^which-iso-8601-2000]: 標準形式か拡張形式のどちらか、の意と考えられる。
[^iso8601-expression]: ISO 8601 中の各種名詞 (... representation とか ... format とか) については [https://wiki.suikawiki.org/n/ISO%208601](SuikaWiki の ISO 8601) が参考になります。

> Note that if the year being printed is greater than 9999, the resulting string from the unadorned %F conversion specifications will not conform to the ISO 8601:2000 standard extended format, complete representation for a date and will instead be an extended format, expanded representation (presumably without the required agreement between the date's producer and consumer).

> In the C or POSIX locale, the E and O modifiers are ignored and the replacement strings for the following specifiers are:

`C` と `POSIX` ロケールでは `E` と `O` 修飾子は無視され and the replacement strings for the following specifiers are:

* `%a`
    * The first three characters of %A.
* `%A`
    * One of Sunday, Monday, ..., Saturday.
* `%b`
    * The first three characters of %B.
* `%B`
    * One of January, February, ..., December.
* `%c`
    * Equivalent to %a %b %e %T %Y.
* `%p`
    * One of AM or PM.
* `%r`
    * Equivalent to %I : %M : %S %p.
* `%x`
    * Equivalent to %m / %d / %y.
* `%X`
    * Equivalent to %T.
* `%Z`
    * Implementation-defined.

## RATIONALE

> The %Y conversion specification to strftime() was frequently assumed to be a four-digit year, but the ISO C standard does not specify that %Y is restricted to any subset of allowed values from the tm_year field. Similarly, the %C conversion specification was assumed to be a two-digit field and the first part of the output from the %F conversion specification was assumed to be a four-digit field. With tm_year being a signed 32 or more-bit int and with many current implementations supporting 64-bit time_t types in one or more programming environments, these assumptions are clearly wrong.

`strftime()` への `%Y` 変換指定子は、多くの場合に四桁の年だと考えられます, but the ISO C standard does not specify that %Y is restricted to any subset of allowed values from the tm_year field. Similarly, the %C conversion specification was assumed to be a two-digit field and the first part of the output from the %F conversion specification was assumed to be a four-digit field. With tm_year being a signed 32 or more-bit int and with many current implementations supporting 64-bit time_t types in one or more programming environments, these assumptions are clearly wrong.

> POSIX.1-2017 now allows the format specifications %0xC, %0xF, %0xG, and %0xY (where 'x' is a string of decimal digits used to specify printing and scanning of a string of x decimal digits) with leading zero fill characters. Allowing applications to set the field width enables them to agree on the number of digits to be printed and scanned in the ISO 8601:2000 standard expanded representation of a year (for %F, %G, and %Y ) or all but the last two digits of the year (for %C ). This is based on a feature in some versions of GNU libc's strftime(). The GNU version allows specifying space, zero, or no-fill characters in strftime() format strings, but does not allow any flags to be specified in strptime() format strings. These implementations also allow these flags to be specified for any numeric field. POSIX.1-2017 only requires the zero fill flag ( '0' ) and only requires that it be recognized when processing %C, %F, %G, and %Y specifications when a minimum field width is also specified. The '0' flag is the only flag needed to produce and scan the ISO 8601:2000 standard year fields using the extended format forms. POSIX.1-2017 also allows applications to specify the same flag and field width specifiers to be used in both strftime() and strptime() format strings for symmetry. Systems may provide other flag characters and may accept flags in conjunction with conversion specifiers other than %C, %F, %G, and %Y; but portable applications cannot depend on such extensions.

> POSIX.1-2017 now also allows the format specifications %+xC, %+xF, %+xG, and %+xY (where 'x' is a string of decimal digits used to specify printing and scanning of a string of 'x' decimal digits) with leading zero fill characters and a leading '+' sign character if the year being converted is more than four digits or a minimum field width is specified that allows room for more than four digits for the year. This allows date providers and consumers to agree on a specific number of digits to represent a year as required by the ISO 8601:2000 standard expanded representation formats. The expanded representation formats all require the year to begin with a leading '+' or '-' sign. (All of these specifiers can also provide a leading '-' sign for negative years. Since negative years and the year 0 don't fit well with the Gregorian or Julian calendars, the normal ranges of dates start with year 1. The ISO C standard allows tm_year to assume values corresponding to years before year 1, but the use of such years provided unspecified results.)

> Some earlier version of this standard specified that applications wanting to use strptime() to scan dates and times printed by strftime() should provide non-digit characters between fields to separate years from months and days. It also supported %F to print and scan the ISO 8601:2000 standard extended format, complete representation date for years 1 through 9999 (i.e., YYYY-MM-DD). However, many applications were written to print (using strftime()) and scan (using strptime()) dates written using the basic format complete representation (four-digit years) and truncated representation (two-digit years) specified by the ISO 8601:2000 standard representation of dates and times which do not have any separation characters between fields. The ISO 8601:2000 standard also specifies basic format expanded representation where the creator and consumer of these fields agree beforehand to represent years as leading zero-filled strings of an agreed length of more than four digits to represent a year (again with no separation characters when year, month, and day are all displayed). Applications producing and consuming expanded representations are encouraged to use the '+' flag and an appropriate maximum field width to scan the year including the leading sign. Note that even without the '+' flag, years less than zero may be represented with a leading <hyphen-minus> for %F, %G, and %Y conversion specifications. Using negative years results in unspecified behavior.

> If a format specification %+xF with the field width x greater than 11 is specified and the width is large enough to display the full year, the output string produced will match the ISO 8601:2000 standard subclause 4.1.2.4 expanded representation, extended format date representation for a specific day. (For years in the range [1,99999], %+12F is sufficient for an agreed five-digit year with a leading sign using the ISO 8601:2000 standard expanded representation, extended format for a specific day "<+/->YYYYY-MM-DD".) Note also that years less than 0 may produce a leading <hyphen-minus> character ( '-' ) when using %Y or %C whether or not the '0' or '+' flags are used.

> The difference between the '0' flag and the '+' flag is whether the leading '+' character will be provided for years >9999 as required for the ISO 8601:2000 standard extended representation format containing a year. For example:

| Year   | Conversion Specification | `strftime()` Output | `strptime()` Scan Back |
| ------ | ------------------------ | ------------------- | ---------------------- |
| 1970   | `%Y`                     | `1970`              | `1970`                 |
| 1970   | `%+4Y`                   | `1970`              | `1970`                 |
| 27     | `%Y`                     | `27` or `0027`      | `27`                   |
| 270    | `%Y`                     | `270` or `0270`     | `270`                  |
| 270    | `%+4Y`                   | `0270`              | `270`                  |
| 17     | `%C%y`                   | `0017`              | `17`                   |
| 270    | `%C%y`                   | `0270`              | `270`                  |
| 12345  | `%Y`                     | `12345`             | `1234`*                |
| 12345  | `%+4Y`                   | `+12345`            | `123`*                 |
| 12345  | `%05Y`                   | `12345`             | `12345`                |
| 270    | `%+5Y` or `%+3C%y`       | `+0270`             | `270`                  |
| 12345  | `%+5Y` or `%+3C%y`       | `+12345`            | `1234`*                |
| 12345  | `%06Y` or `%04C%y`       | `012345`            | `12345`                |
| 12345  | `%+6Y` or `%+4C%y`       | `+12345`            | `12345`                |
| 123456 | `%08Y` or `%06C%y`       | `00123456`          | `123456`               |
| 123456 | `%+8Y` or `%+6C%y`       | `+0123456`          | `123456`               |

> In the cases above marked with a * in the strptime() scan back field, the implied or specified number of characters scanned by strptime() was less than the number of characters output by strftime() using the same format; so the remaining digits of the year were dropped when the output date produced by strftime() was scanned back in by strptime().
