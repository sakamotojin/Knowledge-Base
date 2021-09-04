# Read SQLite File

```javascript
const sqlite3 = require('sqlite3');

let db = new sqlite3.Database('./History', (err) => {
    if (err) {
        console.error(err.message);
    }
    console.log('Connected to the my database.');
});

db.serialize(function () {
    db.each('SELECT * FROM urls ORDER BY last_visit_time DESC', function (err, row) {
    });
});

db.close();
```

