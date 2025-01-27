**Public Report:** [https://hackerone.com/reports/1675516](https://hackerone.com/reports/1675516)

## Stored XSS in www.mercadolibre.com.ar

The general messaging functionality of www.mercadolibre.com.ar implemented an HTML sanitizer that allowed the use of a limited set of HTML tags while preventing XSS. While analyzing the functionality, @valent1ne discovered that sending multiple unclosed <p> tags (<p><p><p><p><p><p><p><p>) and appending an extra tag seemed to confuse the sanitizer parser, resulting in unexpected behavior.
This behavior allowed incluiding an extra arbitrary tag, bypassing the sanitizer. For instance, it enabled the appending of an <audio> HTML tag at the end of the following payload, which executed JavaScript code:

```html
<p><p><p><p><p><p><p><p><audio/src/onerror=alert(document.domain)>
```

Later, the reporter discovered that the number of unclosed <p> tags needed to be increased depending on the size of the appended HTML tag. This adjustment allowed the insertion of an <embed> tag that included external arbitrary HTML/JavaScript code.

After sending the payload, the parsed HTML would look like this:

```html
<p></p>
<p></p>
<p></p>
<p></p>
<p></p>
<p></p>
<p></p>
<p>
 <audio/src/onerror=alert(document.domain)>
</p>
```

Since this vulnerability affected the general messaging functionality of www.mercadolibre.com.ar, it could have been exploited as a wormable XSS, capable of spreading across multiple users.
