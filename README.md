# ghlatest

This is a tool for downloading the latest release of a package from github and optionally extracting the contents.

## Installation

TBD This app is written in go and cross-compiled for various common platforms, probably downloading a binary release is the right strategy.

**Note:** Your system will need to have PKI trust roots of some kind in order to run ghlatest. One common package that provides these on unix systems is called `ca-certificates`.

## Usage


```
NAME:
   ghlatest - Release locator for software on github

USAGE:
   ghlatest [global options] command [command options] [arguments...]

VERSION:
   v0.1.1

COMMANDS:
     list, l, ls      list available releases
     download, d, dl  download the latest available release
     help, h          Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --help, -h     show help
   --version, -v  print the version
```

## Changelog

### v0.3

* argument change:

  * `chmod` -> `mode` - preferring mode to chmod to more accurately reflect that the file is being created with these permissions
  * `json` - add option
  * `source` - add option to include source zip file listing / downloading
  * `extract` - add download option to extract downloaded archives
  * `namefilter` -> `filter` - preferring shorter argument name
  * `outputfile` -> `outputpath` - preferring this because with extract in place the output could be a directory


### v0.2

* Fixes error handling bug in latestReleasedAssets - removing null pointer deref
* Adds README note about CA certificates requirement


## Type sniffing

### zipball

`Content-Type: application/zip`

see here

```
> GET /kolide/launcher/legacy.zip/0.5.0 HTTP/1.1
> Host: codeload.github.com
> User-Agent: curl/7.54.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Transfer-Encoding: chunked
< Access-Control-Allow-Origin: https://render.githubusercontent.com
< Content-Security-Policy: default-src 'none'; style-src 'unsafe-inline'; sandbox
< Strict-Transport-Security: max-age=31536000
< Vary: Authorization,Accept-Encoding
< X-Content-Type-Options: nosniff
< X-Frame-Options: deny
< X-XSS-Protection: 1; mode=block
< ETag: "6f697beacb9e530a97376c9eed45c9013564e946"
< Content-Type: application/zip
< Content-Disposition: attachment; filename=kolide-launcher-0.5.0-0-g6f697be.zip
< X-Geo-Block-List: 
< Date: Sun, 01 Jul 2018 06:47:12 GMT
< X-GitHub-Request-Id: C61E:0A11:113231:28BA33:5B3878F0

```


## Download Logic

### Filenames

Note that the content-disposition includes a filename!

for source:

`Content-Disposition: attachment; filename=kolide-launcher-0.5.0-0-g6f697be.zip`

for released files:

`Content-Disposition: attachment; filename=launcher_0.5.0.zip`

See https://golang.org/pkg/mime/multipart/#Part.FileName for how to parse that correctly

See https://tools.ietf.org/html/rfc6266 for the standard "Use of the Content-Disposition Header Field in the Hypertext Transfer Protocol (HTTP)"

### Extraction


 Download Contents                            | Action
----------------------------------------------|-----------
 single uncompressed file direct download     | writeFileWithNameAndMode()
 single file at root of archive               | writeFileWithNameAndMode(extract())
 single dir at root of archive                | writeDirWithNameAndMode(extract())
 multiple files/dirs at root of archive       | mkdirWithNameAndMode()

questions to answer at runtime

is the download actually compressed? 

is output path a file or a directory?

do we need to create the output directory?

does the zip contain one file or more than one file?



new idea for extraction logic:

* does the outputpath argument end in a slash?

    * if so, always write files into a directory
    * if the directory exists, write into it



## Repos with unusual behavior

* wikimedia/mediawiki - no releases, working with tags https://api.github.com/repos/wikimedia/mediawiki/tags
* ether/etherpad-lite - no assets released

## Github API Releases Endpoint

This app uses the github API's releases endpoint:

`$ curl https://api.github.com/repos/kolide/launcher/releases/latest`

The result looks like this:

```json
{
  "url": "https://api.github.com/repos/kolide/launcher/releases/9880525",
  "assets_url": "https://api.github.com/repos/kolide/launcher/releases/9880525/assets",
  "upload_url": "https://uploads.github.com/repos/kolide/launcher/releases/9880525/assets{?name,label}",
  "html_url": "https://github.com/kolide/launcher/releases/tag/0.5.0",
  "id": 9880525,
  "node_id": "MDc6UmVsZWFzZTk4ODA1MjU=",
  "tag_name": "0.5.0",
  "target_commitish": "master",
  "name": "0.5.0 ",
  "draft": false,
  "author": {
    "login": "groob",
    "id": 1526945,
    "node_id": "MDQ6VXNlcjE1MjY5NDU=",
    "avatar_url": "https://avatars3.githubusercontent.com/u/1526945?v=4",
    "gravatar_id": "",
    "url": "https://api.github.com/users/groob",
    "html_url": "https://github.com/groob",
    "followers_url": "https://api.github.com/users/groob/followers",
    "following_url": "https://api.github.com/users/groob/following{/other_user}",
    "gists_url": "https://api.github.com/users/groob/gists{/gist_id}",
    "starred_url": "https://api.github.com/users/groob/starred{/owner}{/repo}",
    "subscriptions_url": "https://api.github.com/users/groob/subscriptions",
    "organizations_url": "https://api.github.com/users/groob/orgs",
    "repos_url": "https://api.github.com/users/groob/repos",
    "events_url": "https://api.github.com/users/groob/events{/privacy}",
    "received_events_url": "https://api.github.com/users/groob/received_events",
    "type": "User",
    "site_admin": false
  },
  "prerelease": false,
  "created_at": "2018-02-28T19:16:36Z",
  "published_at": "2018-02-28T19:20:40Z",
  "assets": [
    {
      "url": "https://api.github.com/repos/kolide/launcher/releases/assets/6357751",
      "id": 6357751,
      "node_id": "MDEyOlJlbGVhc2VBc3NldDYzNTc3NTE=",
      "name": "launcher_0.5.0.zip",
      "label": null,
      "uploader": {
        "login": "groob",
        "id": 1526945,
        "node_id": "MDQ6VXNlcjE1MjY5NDU=",
        "avatar_url": "https://avatars3.githubusercontent.com/u/1526945?v=4",
        "gravatar_id": "",
        "url": "https://api.github.com/users/groob",
        "html_url": "https://github.com/groob",
        "followers_url": "https://api.github.com/users/groob/followers",
        "following_url": "https://api.github.com/users/groob/following{/other_user}",
        "gists_url": "https://api.github.com/users/groob/gists{/gist_id}",
        "starred_url": "https://api.github.com/users/groob/starred{/owner}{/repo}",
        "subscriptions_url": "https://api.github.com/users/groob/subscriptions",
        "organizations_url": "https://api.github.com/users/groob/orgs",
        "repos_url": "https://api.github.com/users/groob/repos",
        "events_url": "https://api.github.com/users/groob/events{/privacy}",
        "received_events_url": "https://api.github.com/users/groob/received_events",
        "type": "User",
        "site_admin": false
      },
      "content_type": "application/zip",
      "state": "uploaded",
      "size": 24866298,
      "download_count": 658,
      "created_at": "2018-02-28T19:21:42Z",
      "updated_at": "2018-02-28T19:21:50Z",
      "browser_download_url": "https://github.com/kolide/launcher/releases/download/0.5.0/launcher_0.5.0.zip"
    }
  ],
  "tarball_url": "https://api.github.com/repos/kolide/launcher/tarball/0.5.0",
  "zipball_url": "https://api.github.com/repos/kolide/launcher/zipball/0.5.0",
  "body": "* Add a local debug server. (#187)\r\n--TRUNCATED FOR README DOCS --"
}
```

## Resources

* <https://stackoverflow.com/a/50540591> - stackoverflow answer pointing to github api
* <https://developer.github.com/v3/repos/releases/#get-the-latest-release>