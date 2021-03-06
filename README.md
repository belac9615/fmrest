# FMREST
[![Build Status](https://travis-ci.org/thomann061/fmrest.svg?branch=master)](https://travis-ci.org/thomann061/fmrest)
[![npm version](https://badge.fury.io/js/fmrest.svg)](https://badge.fury.io/js/fmrest)

A Node.js wrapper for Filemaker's Data API (REST API)

This API was tested with Filemaker 18 Data Api and using Filemaker ID login services.

Development is in progress.

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.

### Prerequisites

You will need Filemaker Server 18 and enable the Data API.  There is plenty of online documentation describing how to do this.

I have also supplied my testing file - db.fmp12.

## Running the tests

Run all tasks including tests:

```javascript
npm i gulp-cli -g
gulp
```

Run only tests:

```javascript
jasmine
```

Note: fmrestSpec.js is commented out; uncomment it to test in your own environment

## API Code Samples

Almost every function returns a Promise except createRequest, createGlobal and createSort.

### Configuration

```javascript
npm install fmrest


const Fmrest = require('fmrest');


// Set configuration
// Specify fmid for use with your Filemaker ID
const filemaker = new Fmrest({
    user: "user",
    password: "pass",
    host: "host",
    database: "db",
    auth: "fmid", // basic or fmid
//  layout: "db"  // optional at time of login
});

// Layout is not required for authentication anymore
filemaker.setLayout('db'); // need to supply layout eventually
```

### Authentication

```javascript
// Login
filemaker
    .login()
    .then(body => {
        console.log(JSON.stringify(body, null, 3));
    });


// Logout
filemaker
    .logout()
    .then(body => {
        console.log(JSON.stringify(body, null, 3));
    });
```

### Records

```javascript
// Create Record
const values = {
    "name": "Bill",
    "address": "102 park ave",
    "date": "6/21/2017"
}

filemaker
    .createRecord(values) // if empty, creates record w/ default values
    .then(body => {
        console.log(JSON.stringify(body, null, 3));
    });


// Delete Record
filemaker
    .deleteRecord(2) // supply with Filemaker's unique interal recordID
    .then(body => {
        console.log(JSON.stringify(body, null, 3));
    })


// Edit Record
const values2 = {
    "name": "James",
    "address": "105 lake dr",
    "date": "6/22/2017"
}

filemaker
    .editRecord(2, values2)
    .then(body => {
        console.log(JSON.stringify(body, null, 3));
    })


// Get Record
filemaker
    .getRecord(3)
    .then(body => {
        console.log(JSON.stringify(body, null, 3));
    })


// Get Record w/ Portal Data
let portal1 = filemaker
    .createPortal('portal1', 1, 2); // (portal name, offset, limit)
                                    // offset and limit are optional
let portal2 = filemaker
    .createPortal('portal2', 1, 2);

filemaker
    .getRecord(3, [portal1, portal2])
    .then(body => {
        console.log(JSON.stringify(body, null, 3));
    });


// Get All Records
filemaker
    .getAllRecords()
    .then(body => {
        console.log(JSON.stringify(body, null, 3));
    })


// Get All Records w/ Portal Data, sorting, offset and limit
let portal1 = filemaker
    .createPortal('portal1');

let sort1 = filemaker
    .createSort('name', 'ascend');

filemaker
    .getAllRecords({
        offset: 1,
        limit: 10,
        sorts: [sort1],
        portals: [portal1]
    })
    .then(body => {
        console.log(JSON.stringify(body, null, 3));
    });

// Upload Files to Container Fields
const containerFieldName = 'container';
const containerFieldRep = '1';
const recordId = '3';
const fs = require('fs');
const file = fs.createReadStream(`${__dirname}/filemaker_logo_vert.png`);

filemaker
    .uploadFile({ file, recordId, containerFieldName, containerFieldRep })
    .then(body => {
        console.log(JSON.stringify(body, null, 3));
    })
```

### Finds

```javascript
// Find Records
let request = filemaker
    .createRequest()
    .where('name').is('=bill');

filemaker
    .find({requests: [request]})
    .then(body => {
        console.log(JSON.stringify(body, null, 3));
    })


// Find w/ multiple requests
let request = filemaker
    .createRequest()
    .where('name').is('=bill')
    .where('address').is('102*');

let request2 = filemaker
    .createRequest()
    .where('name').is('=james');

filemaker               // Append .omit() to make the request omit records
    .find({requests: [request.omit(), request2]})
    .then(body => {
        console.log(JSON.stringify(body, null, 3));
    })


// Find w/ multiple requests and sorting
let request = filemaker
    .createRequest()
    .where('name').is('*');

let request2 = filemaker
    .createRequest()
    .where('address')
    .is('102*');

let sort = filemaker
    .createSort('name', 'ascend');

let sort2 = filemaker
    .createSort('address', 'descend');

filemaker
    .find({
        requests: [request, request2],
        sorts: [sort, sort2]
    })
    .then(body => {
        console.log(JSON.stringify(body, null, 3));
    })


// Find w/ all optional parameters
let request = filemaker.createRequest().where('name').is('*');

let sort = filemaker.createSort('name', 'ascend');

let portal1 = filemaker.createPortal('portal1', 1, 1);

filemaker
    .find({
        requests: [request],
        sorts: [sort],      // optional
        offset: 2,          // optional
        limit: 10,          // optional
        portals: [portal1]  // optional
    })
    .then(body => {
        console.log(JSON.stringify(body, null, 3));
    })
```

### Set Global Fields

```javascript
// Set Global Fields
let global1 = filemaker
    .createGlobal('db::global1', 'aValue');

let global2 = filemaker
    .createGlobal('db::global2', 'anotherValue');

filemaker
    .setGlobals([global1, global2])
    .then(body => {
        console.log(JSON.stringify(body, null, 3));
    })
```

## TODO

- [x] Layout is optional at login
- [ ] Add optional fm data source to login body
- [ ] Add support for OAuth
- [x] Add support for filemaker id login
- [x] Add support for Container Data
- [ ] Global fields are still not working (help :)
- [ ] Add script query parameters to Records

## Resources

[Filemaker Data 18 API Guide](https://fmhelp.filemaker.com/docs/18/en/dataapi/)

## Contributing / Code of Conduct

Please read [CONTRIBUTING.md](https://github.com/thomann061/fmrest/blob/master/CONTRIBUTING.md) and [CODE_OF_CONDUCT.md](https://github.com/thomann061/fmrest/blob/master/CODE_OF_CONDUCT.md) for details on our code of conduct, and the process for submitting pull requests to us.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details