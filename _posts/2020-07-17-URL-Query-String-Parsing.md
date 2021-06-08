---
layout: default
title: "URL Query String Parsing"
tags: JavaScript
---

<pre><code class="javascript">const queryStringToJSON = (url) => {
    let pairs = [];
    if(url.search.slice !== undefined) {
        pairs = url.search.slice(1).split('&');
    } else {
        pairs = url.split('?').length > 1 ? url.split('?')[1].split('&') : [];
    }

    let result = {};
    pairs.forEach(pair => {
        pair = pair.split('=');
        result[pair[0]] = decodeURIComponent(pair[1] || '');
    });

    return JSON.parse(JSON.stringify(result));
};
console.log(queryStringToJSON(window.location.href));
</code></pre>

## 참고
<a href="https://docs.airbridge.io/ko/web/">https://docs.airbridge.io/ko/web/</a>
