---
title: DirectoryReaderSync
slug: Web/API/DirectoryReaderSync
page-type: web-api-interface
tags:
  - API
  - Reference
browser-compat: api.DirectoryReaderSync
---
{{APIRef("File and Directory Entries API")}}{{Non-standard_header}}

The `DirectoryReaderSync` interface lets you read the entries in a directory.

> **Warning:** This interface is deprecated and is no more on the standard track.
> _Do not use it anymore._ Use the [File and Directory Entries API](/en-US/docs/Web/API/File_and_Directory_Entries_API) instead.

## Basic concepts

Before you call the only method in this interface, [`readEntries()`](/en-US/docs/Web/API/DirectoryReaderSync#readentries "#readEntries()"), create the [`DirectoryEntrySync`](/en-US/docs/Web/API/DirectoryEntrySync) object. But DirectoryEntrySync (as well as [FileEntrySync](/en-US/docs/Web/API/FileEntrySync)) is not a data type that you can pass between a calling app and Web Worker thread. It's not a big deal, because you don't really need to have the main app and the worker thread see the same JavaScript object; you just need them to access the same files. You can do that by passing a list of  `filesystem:` URLs—which are just strings—instead of a list of entries. You can also use the `filesystem:` URL to look up the entry with `resolveLocalFileSystemURL()`). That gets you back to a DirectoryEntrySync (as well as FileEntrySync) object.

#### Example

In the following code snippet from [HTML5Rocks](https://web.dev/read-files/), we create Web Workers and pass data from it to the main app.

```js
// Taking care of the browser-specific prefixes.
  window.resolveLocalFileSystemURL = window.resolveLocalFileSystemURL ||
                                     window.webkitResolveLocalFileSystemURL;

// Create web workers
  var worker = new Worker('worker.js');
  worker.onmessage = function(e) {
    var urls = e.data.entries;
    urls.forEach(function(url, i) {
      window.resolveLocalFileSystemURL(url, function(fileEntry) {
        // Print out file's name.
        console.log(fileEntry.name);
      });
    });
  };

  worker.postMessage({'cmd': 'list'});
```

The following is Worker.js code that gets the contents of the directory.

```js
// Taking care of the browser-specific prefixes.
self.requestFileSystemSync = self.webkitRequestFileSystemSync ||
                             self.requestFileSystemSync;

// Global for holding the list of entry file system URLs.
var paths = [];

function getAllEntries(dirReader) {
  var entries = dirReader.readEntries();

  for (var i = 0, entry; entry = entries[i]; ++i) {
    // Stash this entry's filesystem in URL
    paths.push(entry.toURL());

    // If this is a directory, traverse.
    if (entry.isDirectory) {
      getAllEntries(entry.createReader());
    }
  }
}

// Forward the error to main app.
function onError(e) {
  postMessage('ERROR: ' + e.toString());
}

self.onmessage = function(e) {
  var data = e.data;

  // Ignore everything else except our 'list' command.
  if (!data.cmd || data.cmd != 'list') {
    return;
  }

  try {
    var fs = requestFileSystemSync(TEMPORARY, 1024*1024 /*1MB*/);

    getAllEntries(fs.root.createReader());

    self.postMessage({entries: paths});
  } catch (e) {
    onError(e);
  }
};
```

## Method overview

<table class="standard-table">
  <tbody>
    <tr>
      <td>
        <code>
          EntrySync <a href="#createreader" title="#readEntries">readEntries</a> ();
        </code>
      </td>
    </tr>
  </tbody>
</table>

## Method

### readEntries()

Returns a lost of entries from a specific directory. Call this method until an empty array is returned.

```
EntrySync readEntries ();
```

##### Returns

##### Parameter

None

##### Exceptions

This method can raise a [DOMException](/en-US/docs/Web/API/DOMException) with the following codes:

| Exception           | Description                                                                        |
| ------------------- | ---------------------------------------------------------------------------------- |
| `NOT_FOUND_ERR`     | The directory does not exist.                                                      |
| `INVALID_STATE_ERR` | The directory has been modified since the first call to readEntries was processed. |
| `SECURITY_ERR`      | The browser determined that it was not safe to look up the metadata.               |

## Specifications

This feature is not part of any specification anymore. It is no longer on track to become a standard.

## Browser compatibility

{{Compat}}

## See also

- [File and Directory Entries API](/en-US/docs/Web/API/File_and_Directory_Entries_API)
- [Introduction to the File and Directory Entries API](/en-US/docs/Web/API/File_and_Directory_Entries_API/Introduction)
