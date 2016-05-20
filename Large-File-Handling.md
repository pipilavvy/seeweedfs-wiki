In order to achieve high concurrency, SeaweedFS tries to read and write the whole file into memory. But this would not work for large files. 

The following is implemented in "weed upload" command. For 3rd party clients, here is the spec.

To support large files, SeaweedFS supports these two kinds of files:
* Chunk File. Each chunk file is actually just normal files to SeaweedFS.
* Chunk Manifest. A simple json file with the list of all the chunks. 

This piece of code shows the json file structure:

https://github.com/chrislusf/seaweedfs/blob/master/go/operation/chunked_file.go#L24

```
type ChunkInfo struct {
	Fid    string `json:"fid"`
	Offset int64  `json:"offset"`
	Size   int64  `json:"size"`
}

type ChunkList []*ChunkInfo

type ChunkManifest struct {
	Name   string    `json:"name,omitempty"`
	Mime   string    `json:"mime,omitempty"`
	Size   int64     `json:"size,omitempty"`
	Chunks ChunkList `json:"chunks,omitempty"`
}
```

When reading Chunk Manifest files, the SeaweedFS will find and send the data file based on the list of ChunkInfo.

## Create new large file

SeaweedFS delegates the effort to the client side. The steps are:

1. split large files into chunks
1. upload each file chunks as usual, with mime type "application/octet-stream". Save the related info into ChunkInfo struct.
1. upload the manifest file with mime type "application/json", and add url parameter "cm=true".


## Update large file

Usually we just append large files. Updating a specific chunk of file is almost the same.

The steps to append a large file:

1. upload the new file chunks as usual, with mime type "application/octet-stream". Save the related info into ChunkInfo struct.
1. update the updated manifest file with mime type "application/json", and add url parameter "cm=true".