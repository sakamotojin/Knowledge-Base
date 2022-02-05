# Make HTTP Request

## Import Libraries

```javascript
const https = require('https');
```

## Make Request

```javascript
var data = "";

var req = https.get('https://www.imdb.com/chart/top/', function (res) {

    res.setEncoding('utf8');
    res.on('data', function (chunk) {
        data += chunk;
    });

    res.on('end', () => {
        console.log('end the connections');
        parse_html(data);
    });
});


req.on('error' , (err)=>{
    console.log(err.message);
});
```
