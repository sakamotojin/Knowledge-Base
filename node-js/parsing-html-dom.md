---
description: using JSDOM
---

# Parsing HTML DOM

* **Import Modules**

```javascript
const jsdom = require("jsdom");
const { JSDOM } = jsdom;
const https = require('https');
```

* **Get HTML Page**

```javascript
var data = "";

var req = https.get('https://www.imdb.com/chart/top/', function (res) {

    res.setEncoding('utf8');
    res.on('data', function (chunk) {
        data += chunk;
    });
});
```

{% hint style="info" %}
Wait For The Data To Load
{% endhint %}

* **Main Parsing Function**

```javascript
var movie_list = [];
function parse_html(data) {

    let rank = 0 ;
    let document = new JSDOM(String(data)).window.document;
    let movie_table = Array.from(document.getElementsByTagName('tr'));
   
    movie_table.forEach((ele,err) => {

        let movie_info = {};
        Array.from(ele.cells).forEach((cell, err) => {
            if(rank > 0){
                if (cell.matches('.titleColumn')){
                    movie_info["rank"] = rank;
                    movie_info["name"] = cell.children[0].textContent;
                    movie_info["year"] = cell.children[1].textContent;
                    let people = cell.children[0].getAttribute("title").split(',');
                    movie_info["people"] = people;
                
                }
                if (cell.matches('.imdbRating')) {
                    movie_info["rating"] = cell.children[0].textContent;
                }

            }
        });
        rank = rank +1;
        movie_list.push(movie_info);
    });

    movie_list.forEach((ele,err)=>{
        console.log(ele);
    });
};


```

