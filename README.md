# Multer [![Build Status](https://travis-ci.org/expressjs/multer.svg?branch=master)](https://travis-ci.org/expressjs/multer) [![NPM version](https://badge.fury.io/js/multer.svg)](https://badge.fury.io/js/multer) [![js-standard-style](https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat)](https://github.com/feross/standard)

Multer is a node.js middleware for handling `multipart/form-data`, which is primarily used for uploading files. It is written
on top of [busboy](https://github.com/mscdex/busboy) for maximum efficiency.

**NOTE**: Multer will not process any form which is not multipart (`multipart/form-data`).
**NOTE** Modify for my project.


## Usage

Multer work with parse parameter.

Basic usage example:

```javascript
var express = require('express')
var multer  = require('multer')
var upload = multer({ dest: 'uploads/' })

var app = express()
var storage = multer.diskStorage({
  destination: function (req, file, value, callback) {
  
      //callback(null, pathFile);
  },
  filename: function (req, file, value, callback) {
    
    //callback(null, filename);
  }
});
var upload = multer({ 
  storage : storage,
  fileFilter: function(req, file, value, callback){
    
    // To reject this file pass `false`, like so: 
      // callback(null, false)
     
    // To accept the file pass `true`, like so: 
      callback(null, true)
     
    // You can always pass an error if something goes wrong: 
      // callback(new Error('I don\'t have a clue!'))
  }
}).fields([{name:'photos', maxCount: 1}]);

Example

app.post('/photos/upload', function (req, res, next) {
  // parse value if you want by id
  upload(req,res,id,function(err) {
    // req.files is array of `photos` files
    // req.body will contain the text fields, if there were any
  })
  
})


## API

### File information

Each file contains the following information:

Key | Description | Note
--- | --- | ---
`fieldname` | Field name specified in the form |
`originalname` | Name of the file on computer |
`encoding` | Encoding type of the file |
`mimetype` | Mime type of the file |
`size` | Size of the file in bytes |
`destination` | The folder to which the file has been saved | `DiskStorage`
`filename` | The name of the file within the `destination` | `DiskStorage`
`path` | The full path to the uploaded file | `DiskStorage`
`buffer` | A `Buffer` of the entire file | `MemoryStorage`

### `multer(opts)`

Multer accepts an options object, the most basic of which is the `dest`
property, which tells Multer where to upload the files. In case you omit the
options object, the files will be kept in memory and never written to disk.

By default, Multer will rename the files so as to avoid naming conflicts. The
renaming function can be customized according to your needs.

The following are the options that can be passed to Multer.

Key | Description
--- | ---
`dest` or `storage` | Where to store the files
`fileFilter` | Function to control which files are accepted
`limits` | Limits of the uploaded data

In an average web app, only `dest` might be required, and configured as shown in
the following example.

```javascript
var upload = multer({ dest: 'uploads/' })
```

If you want more control over your uploads, you'll want to use the `storage`
option instead of `dest`. Multer ships with storage engines `DiskStorage`
and `MemoryStorage`; More engines are available from third parties.

#### `.single(fieldname)`

Accept a single file with the name `fieldname`. The single file will be stored
in `req.file`.

#### `.array(fieldname[, maxCount])`

Accept an array of files, all with the name `fieldname`. Optionally error out if
more than `maxCount` files are uploaded. The array of files will be stored in
`req.files`.

#### `.fields(fields)`

Accept a mix of files, specified by `fields`. An object with arrays of files
will be stored in `req.files`.

`fields` should be an array of objects with `name` and optionally a `maxCount`.
Example:

```javascript
[
  { name: 'avatar', maxCount: 1 },
  { name: 'gallery', maxCount: 8 }
]
```

#### `.any()`

Accepts all files that comes over the wire. An array of files will be stored in
`req.files`.

**WARNING:** Make sure that you always handle the files that a user uploads.
Never add multer as a global middleware since a malicious user could upload
files to a route that you didn't anticipate. Only use this function on routes
where you are handling the uploaded files.

### `storage`

#### `DiskStorage`

The disk storage engine gives you full control on storing files to disk.

```javascript
var storage = multer.diskStorage({
  destination: function (req, file, value, cb) {
    cb(null, '/tmp/my-uploads')
  },
  filename: function (req, file, value, cb) {
    cb(null, file.fieldname + '-' + Date.now())
  }
})

var upload = multer({ storage: storage })
```

There are two options available, `destination` and `filename`. They are both
functions that determine where the file should be stored.

`destination` is used to determine within which folder the uploaded files should
be stored. This can also be given as a `string` (e.g. `'/tmp/uploads'`). If no
`destination` is given, the operating system's default directory for temporary
files is used.

**Note:** You are responsible for creating the directory when providing
`destination` as a function. When passing a string, multer will make sure that
the directory is created for you.

`filename` is used to determine what the file should be named inside the folder.
If no `filename` is given, each file will be given a random name that doesn't
include any file extension.

**Note:** Multer will not append any file extension for you, your function
should return a filename complete with an file extension.

Each function gets passed both the request (`req`) and some information about
the file (`file`) to aid with the decision.

Note that `req.body` might not have been fully populated yet. It depends on the
order that the client transmits fields and files to the server.

#### `MemoryStorage`

The memory storage engine stores the files in memory as `Buffer` objects. It
doesn't have any options.

```javascript
var storage = multer.memoryStorage()
var upload = multer({ storage: storage })
```

When using memory storage, the file info will contain a field called
`buffer` that contains the entire file.

**WARNING**: Uploading very large files, or relatively small files in large
numbers very quickly, can cause your application to run out of memory when
memory storage is used.

### `limits`

An object specifying the size limits of the following optional properties. Multer passes this object into busboy directly, and the details of the properties can be found on [busboy's page](https://github.com/mscdex/busboy#busboy-methods).

The following integer values are available:

Key | Description | Default
--- | --- | ---
`fieldNameSize` | Max field name size | 100 bytes
`fieldSize` | Max field value size | 1MB
`fields` | Max number of non-file fields | Infinity
`fileSize` | For multipart forms, the max file size (in bytes) | Infinity
`files` | For multipart forms, the max number of file fields | Infinity
`parts` | For multipart forms, the max number of parts (fields + files) | Infinity
`headerPairs` | For multipart forms, the max number of header key=>value pairs to parse | 2000

Specifying the limits can help protect your site against denial of service (DoS) attacks.

### `fileFilter`

Set this to a function to control which files should be uploaded and which
should be skipped. The function should look like this:

```javascript
function fileFilter (req, file, value, cb) {

  // The function should call `cb` with a boolean
  // to indicate if the file should be accepted

  // To reject this file pass `false`, like so:
  cb(null, false)

  // To accept the file pass `true`, like so:
  cb(null, true)

  // You can always pass an error if something goes wrong:
  cb(new Error('I don\'t have a clue!'))

}
```

## Error handling

When encountering an error, multer will delegate the error to express. You can
display a nice error page using [the standard express way](http://expressjs.com/guide/error-handling.html).

If you want to catch errors specifically from multer, you can call the
middleware function by yourself.

```javascript
var upload = multer().single('avatar')

app.post('/profile', function (req, res) {
  upload(req, res, function (err) {
    if (err) {
      // An error occurred when uploading
      return
    }

    // Everything went fine
  })
})
```

## Custom storage engine

See [the documentation here](/StorageEngine.md) if you want to build your own
storage engine.

## License

[MIT](LICENSE)
