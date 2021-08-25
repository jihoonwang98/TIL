# Thymeleaf utility methods



Thymeleaf 엔진은 아래와 같은 utility class들을 가지고 있다.

- `String`
- `Calendar`
- `Boolean`
- `List`
- `Map`
- `Array`
- `Number`
- `Date`





## 1. URI/URL (`#uris`)

| Method                                                       | Purpose                                                      | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `${#uris.escapePath(uri)}` `${#uris.escapePath(uri, encoding)}` `${#uris.unescapePath(uri)}` `${#uris.unescapePath(uri, encoding)}` | Escape/Unescape as a URI/URL path                            | Allowed characters in URI path are: `A-Z a-z 0-9`, `- . _ ~`, `! $ & ' ( ) * + , ; =`, `@`, `/`. Those chars will not be escaped. All other chars converted to the sequence of bytes that represents them in the specified `encoding` and then representing each byte in the hexadecimal representation of that byte. |
| `${#uris.escapePathSegment(uri)}` `${#uris.escapePathSegment(uri, encoding)}` `${#uris.unescapePathSegment(uri)}` `${#uris.unescapePathSegment(uri, encoding)}` | Escape/Unescape as a URI/URL path segment (between '/' symbols) | Escapes the given string for use as a URL path segment       |
| `${#uris.escapeFragmentId(uri)}` `${#uris.escapeFragmentId(uri, encoding)}` `${#uris.unescapeFragmentId(uri)}` `${#uris.unescapeFragmentId(uri, encoding)}` | Escape/Unescape as a Fragment Identifier (#frag)             | Escapes the given string for use as a URL path segment       |
| `${#uris.escapeQueryParam(uri)}` `${#uris.escapeQueryParam(uri, encoding)}` `${#uris.unescapeQueryParam(uri)}` `${#uris.unescapeQueryParam(uri, encoding)}` | Escape/Unescape as a Query Parameter (?var=value)            | Escapes the given string for use as a URL query param        |



### 예제

```html
<a th:href="${#uris.escapePath('https://frontbackend.com/©!$%&()*+,-./:;<=>?@[\]^_`{|}~')}" th:text="${#uris.escapePath('https://frontbackend.com/©!$%&()*+,-./:;<=>?@[\]^_`{|}~')}"></a>

<br/>
<a th:href="${'https://frontbackend.com/' + #uris.escapePathSegment('©!$%&()*+,-./:;<=>?@[\]^_`{|}~')}" th:text="${'https://frontbackend.com/' + #uris.escapePathSegment('©!$%&()*+,-./:;<=>?@[\]^_`{|}~')}"></a>

<br/>
<a th:href="${'https://frontbackend.com#' + #uris.escapeFragmentId('©!$%&()*+,-./:;<=>?@[\]^_`{|}~')}" th:text="${'https://frontbackend.com#' + #uris.escapeFragmentId('©!$%&()*+,-./:;<=>?@[\]^_`{|}~')}"></a>

<br/>
<a th:href="${'https://frontbackend.com?param=' + #uris.escapeQueryParam('©!$%&()*+,-./:;<=>?@[\]^_`{|}~')}" th:text="${'https://frontbackend.com?param=' + #uris.escapeQueryParam('©!$%&()*+,-./:;<=>?@[\]^_`{|}~')}"></a>
<br/>

<a th:href="${'https://frontbackend.com/' + #uris.escapePathSegment('https://example.com')}" th:text="${'https://frontbackend.com/' + #uris.escapePathSegment('https://example.com')}"></a>
<br/>

<a th:href="${'https://frontbackend.com?param=' + #uris.escapeQueryParam('https://example.com')}" th:text="${'https://frontbackend.com?param=' + #uris.escapeQueryParam('https://example.com')}"></a><br/>
```



**결과**

![](https://frontbackend.com/storage/thymeleaf/thymeleaf-utility-methods-uri-urls.png)







## 2. Arrays (`#arrays`)

| Method                                                       | Purpose                                                  | Description                                                  |
| :----------------------------------------------------------- | :------------------------------------------------------- | :----------------------------------------------------------- |
| `${#arrays.toArray(object)}`                                 | Converts to array, trying to infer array component class | If resulting array is empty, or if the elements of the target object are not all of the same class, this method will return `Object[]` |
| `${#arrays.toStringArray(object)}` `${#arrays.toIntegerArray(object)}` `${#arrays.toLongArray(object)}` `${#arrays.toDoubleArray(object)}` `${#arrays.toFloatArray(object)}` `${#arrays.toBooleanArray(object)}` | Convert to arrays of the specified component class       |                                                              |
| `${#arrays.length(array)}`                                   | Get array length                                         |                                                              |
| `${#arrays.isEmpty(array)}`                                  | Check whether array is empty                             |                                                              |
| `${#arrays.contains(array, element)}` `${#arrays.containsAll(array, elements)}` | Check if array contains given element or all elements    |                                                              |





## 3. Booleans (`#bools`)

| Method                                                       | Purpose                                                      | Description                           |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :------------------------------------ |
| `${#bools.isTrue(obj)}` `${#bools.arrayIsTrue(objArray)}` `${#bools.listIsTrue(objList)}` `${#bools.setIsTrue(objSet)}` | Evaluate a condition in the same way that it would be evaluated in a [th:if](https://frontbackend.com/thymeleaf/using-conditions-in-thymeleaf) tag | Also works with arrays, lists or sets |
| `${#bools.isFalse(cond)}` `${#bools.arrayIsFalse(condArray)}` `${#bools.listIsFalse(condList)}` `${#bools.setIsFalse(condSet)}` | Evaluate with negation                                       | Also works with arrays, lists or sets |
| `${#bools.arrayAnd(condArray)}` `${#bools.listAnd(condList)}` `${#bools.setAnd(condSet)}` | Evaluate and apply AND operator                              |                                       |
| `${#bools.arrayOr(condArray)}` `${#bools.listOr(condList)}` `${#bools.setOr(condSet)}` | Evaluate and apply OR operator                               |                                       |





## 4. Objects

Thymeleaf provides several methods to check if the given object is null.

| Method                                                       | Purpose                                                      | Description                           |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :------------------------------------ |
| `${#objects.nullSafe(obj,default)}` `${#objects.arrayNullSafe(objArray,default)}` `${#objects.listNullSafe(objList,default)}` `${#objects.setNullSafe(objSet,default)}` | If given `obj` is null then `default` value will be returned otherwise (object is not null) `obj` will be returned | Also works with arrays, lists or sets |





## 5. Strings

| Method                                                       | Purpose                                                      | Description                                              |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :------------------------------------------------------- |
| `${#strings.toString(obj)}`                                  | This is a Null-safe toString() method                        | You will not get NullPointerException when `obj` is null |
| `${#strings.isEmpty(name)}` `${#strings.arrayIsEmpty(nameArr)}` `${#strings.listIsEmpty(nameList)}` `${#strings.setIsEmpty(nameSet)}` | Check whether a `String` is empty (or null).                 | Also works with arrays, lists or sets                    |
| `${#strings.defaultString(text,default)}` `${#strings.arrayDefaultString(textArr,default)}` `${#strings.listDefaultString(textList,default)}` `${#strings.setDefaultString(textSet,default)}` | This method perform an 'isEmpty()' check on a `String` and return it if false, when `String` is empty the default value will be returned. | Also works with arrays, lists or sets                    |
| `${#strings.contains(name,'ez')}` `${#strings.containsIgnoreCase(name,'ez')}` | Check if `String` contains another `String`                  | Also works with arrays, lists or sets                    |
| `${#strings.startsWith(name,'Don')}` `${#strings.endsWith(name,endingFragment)}` | Check whether a String starts or ends with a fragment        | Also works with arrays, lists or sets                    |
| `${#strings.indexOf(name,frag)}` `${#strings.substring(name,3,5)}` `${#strings.substringAfter(name,prefix)}` `${#strings.substringBefore(name,suffix)}` `${#strings.replace(name,'las','ler')}` | Substring-related operations                                 | Also works with arrays, lists or sets                    |
| `${#strings.prepend(str,prefix)}` `${#strings.append(str,suffix)}` | Add prefix or suffix to the `String`                         | Also works with arrays, lists or sets                    |
| `${#strings.toUpperCase(name)}` `${#strings.toLowerCase(name)}` | Print `String` uppercased or lowercased                      | Also works with arrays, lists or sets                    |
| `${#strings.arrayJoin(namesArray,',')}` `${#strings.listJoin(namesList,',')}` `${#strings.setJoin(namesSet,',')}` `${#strings.arraySplit(namesStr,',')}` `${#strings.listSplit(namesStr,',')}` `${#strings.setSplit(namesStr,',')}` | Split `String` to Array or join Array items to the `String` using specified separator |                                                          |
| `${#strings.trim(str)}`                                      | Trim `String` - remove white speces from the beginning and end of a `String` |                                                          |
| `${#strings.length(str)}`                                    | Compute `String` length                                      |                                                          |
| `${#strings.abbreviate(str,10)}`                             | Abbreviate text making it have a maximum size of n. If text is bigger, it will be clipped and finished in "..." |                                                          |
| `${#strings.capitalize(str)}` `${#strings.unCapitalize(str)}` | Convert the first character to upper-case (and vice-versa)   |                                                          |
| `${#strings.capitalizeWords(str)}` `${#strings.capitalizeWords(str,delimiters)}` | Convert the first character of every word to upper-case      |                                                          |
| `${#strings.escapeXml(str)}` `${#strings.escapeJava(str)}` `${#strings.escapeJavaScript(str)}` `${#strings.unescapeJava(str)}` `${#strings.unescapeJavaScript(str)}` | Escape and unescape the `String`                             | Also works with arrays, lists or sets                    |
| `${#strings.equals(first, second)}` `${#strings.equalsIgnoreCase(first, second)}` `${#strings.concat(values...)}` `${#strings.concatReplaceNulls(nullValue, values...)}` | Null-safe comparison and concatenation                       |                                                          |
| `${#strings.randomAlphanumeric(count)}`                      | Generate random `String` with given length                   |                                                          |







## 6. Numbers

| Method                                                       | Purpose                                                      | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `${#numbers.formatInteger(num,3)}` `${#numbers.arrayFormatInteger(numArray,3)}` `${#numbers.listFormatInteger(numList,3)}` `${#numbers.setFormatInteger(numSet,3)}` | Set minimum integer digits                                   | Also works with arrays, lists or sets                        |
| `${#numbers.formatInteger(num,3,'POINT')}` `${#numbers.arrayFormatInteger(numArray,3,'POINT')}` `${#numbers.listFormatInteger(numList,3,'POINT')}` `${#numbers.setFormatInteger(numSet,3,'POINT')}` | Set minimum integer digits and thousands separator           | Available separators: 'POINT', 'COMMA', 'WHITESPACE', 'NONE' or 'DEFAULT' (by locale). |
| `${#numbers.formatDecimal(num,3,2)}` `${#numbers.arrayFormatDecimal(numArray,3,2)}` `${#numbers.listFormatDecimal(numList,3,2)}` `${#numbers.setFormatDecimal(numSet,3,2)}` | Set minimum integer digits and (exact) decimal digits        | Also works with arrays, lists or sets                        |
| `${#numbers.formatDecimal(num,3,2,'COMMA')}` `${#numbers.arrayFormatDecimal(numArray,3,2,'COMMA')}` `${#numbers.listFormatDecimal(numList,3,2,'COMMA')}` `${#numbers.setFormatDecimal(numSet,3,2,'COMMA')}` | Set minimum integer digits and (exact) decimal digits, and also decimal separator. | Available separators: 'POINT', 'COMMA', 'WHITESPACE', 'NONE' or 'DEFAULT' (by locale). |
| `${#numbers.formatDecimal(num,3,'POINT',2,'COMMA')}` `${#numbers.arrayFormatDecimal(numArray,3,'POINT',2,'COMMA')}` `${#numbers.listFormatDecimal(numList,3,'POINT',2,'COMMA')}` `${#numbers.setFormatDecimal(numSet,3,'POINT',2,'COMMA')}` | Set minimum integer digits and (exact) decimal digits, and also thousands and decimal separator. |                                                              |
| `${#numbers.formatCurrency(num)}` `${#numbers.arrayFormatCurrency(numArray)}` `${#numbers.listFormatCurrency(numList)}` `${#numbers.setFormatCurrency(numSet)}` | Formatting currencies                                        |                                                              |
| `${#numbers.formatPercent(num)}` `${#numbers.arrayFormatPercent(numArray)}` `${#numbers.listFormatPercent(numList)}` `${#numbers.setFormatPercent(numSet)}` | Formatting percentages                                       |                                                              |
| `${#numbers.formatPercent(num, 3, 2)}` `${#numbers.arrayFormatPercent(numArray, 3, 2)}` `${#numbers.listFormatPercent(numList, 3, 2)}` `${#numbers.setFormatPercent(numSet, 3, 2)}` | Set minimum integer digits and (exact) decimal digits.       |                                                              |
| `${#numbers.sequence(from,to)}` `${#numbers.sequence(from,to,step)}` |                                                              |                                                              |



## 7. Calendars

지금부터 소개하는 utility method들은 `java.util.Calendar` 객체들을 위한 메서드들이다.

| Method                                                       | Purpose                                                      |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `${#calendars.format(cal)}` `${#calendars.arrayFormat(calArray)}` `${#calendars.listFormat(calList)}` `${#calendars.setFormat(calSet)}` | Format calendar with the standard locale format              |
| `${#calendars.formatISO(cal)}` `${#calendars.arrayFormatISO(calArray)}` `${#calendars.listFormatISO(calList)}` `${#calendars.setFormatISO(calSet)}` | Format calendar with the ISO8601 format                      |
| `${#calendars.format(cal, 'dd/MMM/yyyy HH:mm')}` `${#calendars.arrayFormat(calArray, 'dd/MMM/yyyy HH:mm')}` `${#calendars.listFormat(calList, 'dd/MMM/yyyy HH:mm')}` `${#calendars.setFormat(calSet, 'dd/MMM/yyyy HH:mm')}` | Format calendar with the specified pattern                   |
| `${#calendars.day(date)}` `${#calendars.month(date)}` `${#calendars.monthName(date)}` `${#calendars.monthNameShort(date)}` `${#calendars.year(date)}` `${#calendars.dayOfWeek(date)}` `${#calendars.dayOfWeekName(date)}` `${#calendars.dayOfWeekNameShort(date)}` `${#calendars.hour(date)}` `${#calendars.minute(date)}` `${#calendars.second(date)}` `${#calendars.millisecond(date)}` | Obtain calendar properties                                   |
| `${#calendars.create(year,month,day)}` `${#calendars.create(year,month,day,hour,minute)}` `${#calendars.create(year,month,day,hour,minute,second)}` `${#calendars.create(year,month,day,hour,minute,second,millisecond)}` | Create calendar (java.util.calendar) objects from its components |
| `${#calendars.createNow()}` `${#calendars.createNowForTimeZone()}` | Create a calendar (java.util.calendar) object for the current date and time |
| `${#calendars.createToday()}` `${#calendars.createTodayForTimeZone()}` | Create a calendar (java.util.Calendar) object for the current date (time set to 00:00) |





## 8. Dates

지금부터 소개하는 utility method들은 `java.util.Date` 객체들을 위한 메서드들이다.



| Method                                                       | Purpose                                                      |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `${#dates.format(date)}` `${#dates.arrayFormat(datesArray)}` `${#dates.listFormat(datesList)}` `${#dates.setFormat(datesSet)}` | Format date with the standard locale format                  |
| `${#dates.formatISO(date)}` `${#dates.arrayFormatISO(datesArray)}` `${#dates.listFormatISO(datesList)}` `${#dates.setFormatISO(datesSet)}` | Format date with the ISO8601 format                          |
| `${#dates.format(date, 'dd/MMM/yyyy HH:mm')}` `${#dates.arrayFormat(datesArray, 'dd/MMM/yyyy HH:mm')}` `${#dates.listFormat(datesList, 'dd/MMM/yyyy HH:mm')}` `${#dates.setFormat(datesSet, 'dd/MMM/yyyy HH:mm')}` | Format date with the specified pattern                       |
| `${#dates.day(date)}` `${#dates.month(date)}` `${#dates.monthName(date)}` `${#dates.monthNameShort(date)}` `${#dates.year(date)}` `${#dates.dayOfWeek(date)}` `${#dates.dayOfWeekName(date)}` `${#dates.dayOfWeekNameShort(date)}` `${#dates.hour(date)}` `${#dates.minute(date)}` `${#dates.second(date)}` `${#dates.millisecond(date)}` | Obtain date properties                                       |
| `${#dates.create(year,month,day)}` `${#dates.create(year,month,day,hour,minute)}` `${#dates.create(year,month,day,hour,minute,second)}` `${#dates.create(year,month,day,hour,minute,second,millisecond)}` | Create date (java.util.Date) objects from its components     |
| `${#dates.createNow()}` `${#dates.createNowForTimeZone()}`   | Create a date (java.util.Date) object for the current date and time |
| `${#dates.createToday()}` `${#dates.createTodayForTimeZone()}` | Create a date (java.util.Date) object for the current date (time set to 00:00) |



## 9. Lists

| Method                                  | Purpose                                       |
| :-------------------------------------- | :-------------------------------------------- |
| `${#lists.toList(object)}`              | Converts to list                              |
| `${#lists.size(list)}`                  | Compute size                                  |
| `${#lists.isEmpty(list)}`               | Check whether list is empty                   |
| `${#lists.contains(list, element)}`     | Check if element is contained in list         |
| `${#lists.containsAll(list, elements)}` | Check if elements are contained in list       |
| `${#lists.sort(list)}`                  | Sort a copy of the given list                 |
| `${#lists.sort(list, comparator)}`      | Sort a copy of the given list with comparator |







## 10. Messages

| Method                                                       | Purpose                                     |
| :----------------------------------------------------------- | :------------------------------------------ |
| `${#messages.msg('msgKey')}` `${#messages.msg('msgKey', param1)}` `${#messages.msg('msgKey', param1, param2)}` `${#messages.msg('msgKey', param1, param2, param3)}` `${#messages.msgWithParams('msgKey', new Object[] {param1, param2})}` | Obtain externalized message for one key     |
| `${#messages.arrayMsg(messageKeyArray)}` `${#messages.listMsg(messageKeyList)}` `${#messages.setMsg(messageKeySet)}` | Retrieve externalized messages for key list |
| `${#messages.msgOrNull('msgKey')}` `${#messages.msgOrNull('msgKey', param1)}` `${#messages.msgOrNull('msgKey', param1, param2)}` `${#messages.msgOrNull('msgKey', param1, param2, param3)}` `${#messages.msgOrNullWithParams('msgKey', new Object[] {param1, param2})}` `${#messages.arrayMsgOrNull(messageKeyArray)}` `${#messages.listMsgOrNull(messageKeyList)}` `${#messages.setMsgOrNull(messageKeySet)}` | Obtain externalized messages or null        |





