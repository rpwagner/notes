---
 toc: true
 layout: post
 description: A script to do HTTPS uploads to a Globus collection.
 title: Globus HTTPS PUT
---
 
# Globus HTTPS `PUT`

The script
[`globuscollectionput.py`](https://github.com/rpwagner/serverless-data/blob/main/bin/globuscollectionput.py) 
uses [`requests`](https://requests.readthedocs.io/en/latest/) to do a
PUT over HTTPS to a [Globus
collection](https://docs.globus.org/globus-connect-server/v5.4/https-access-collections/). To
make that useful, it can act either as a [Native
App](https://globus-sdk-python.readthedocs.io/en/stable/examples/native_app.html)
or [Confidential
Client](https://globus-sdk-python.readthedocs.io/en/stable/examples/client_credentials.html)
to get the necessary access tokens from [Globus
Auth](https://docs.globus.org/globus-connect-server/v5.4/https-access-collections/#accessing_data).

## Why?

Sometimes, it just isn't possible to get a Globus endpoint working where you
need it. And for sending the occassional modest-sized file, a Python
script with minimal dependencies is more likely to "just work". 

And not needing to register even a personal endpoint opens up some new
use cases. For example, this could be used with a CI process to push build
 artifacts to a collection. And keeping a client secret in the build
 secrets is well supported.
 
The caveat is that this doesn't have the reliability mechanisms of
Globus Transfer. At some point, a file will take too long, or someone
will close their laptop and kill the upload.

## Dependencies

Outside of the Python standard library, you'll need the
[Click](https://click.palletsprojects.com/), [Globus Python
SDK](https://globus-sdk-python.readthedocs.io/), and [Fair Research
Login ](https://github.com/fair-research/native-login) packages.

```shell
python -m pip install click globus_sdk fair_research_login
```

This main reason Fair Research Login is used is for the convenient
browser interaction when logging in.

## Basic Usage

`globuscollectionput.py` has three required positional arguments:
- The name of the file to upload
- The destination (a path or URL)
- The UUID of the collection

```shell
globuscollectionput.py [OPTIONS] FILENAME DESTINATION COLLECTION_ID
```

For example
```shell
./globuscollectionput.py science.json /data/ 6528bad5-bc02-497d-8a4f-a38547d0e72a
```
will try to PUT the file `science.json` into the folder `/data/` of
the collection `6528bad5-bc02-497d-8a4f-a38547d0e72a`. In Globus CLI
syntax, the result would be a file at

```
6528bad5-bc02-497d-8a4f-a38547d0e72a:/data/science.json
```

## Destination: URL or Path

If the destination is a path (e.g, `/data`), it will be appended to
the base URL of the collection to make the destination URL. The base
URL will be found by calling [`TransfeClient.get_endpoint`](https://docs.globus.org/globus-connect-server/v5.4/https-access-collections/#using_the_python_sdk)
and checking the key `'https_server'`
```python
collection_info = tc.get_endpoint(collection_id)
base_url = collection_info['https_server']
```
In this case a slash `/` will be prepended to the path if needed.

```shell
$ ./globuscollectionput.py -v science.json https://example.edu/data/ \
	6528bad5-bc02-497d-8a4f-a38547d0e72a
Filename: science.json
Destination: https://example.edu/data/science.json
PUT to https://example.edu/data/science.json status 200
```

## Destination: File or Folder

The destination can be given as a file or folder. The script will
check the end of the destination for a trailing slash to see which was
provided. This allows for renaming files when they're PUT.

If the destination ends with a slash, e.g. `/data/` or
  `https://example.edu/data/`, the file will be uploaded with its base
  name, like `https://example.edu/data/science.json`. Otherwise, the
  destination will be path the file on the collection after upload.

Of course, this is about what the script tries to do. If the
resource on the destination is a folder but is given as `/folder`,
without the trailing slash, the PUT will fail.

## Destination Matrix

Suppose the collection has the base URL of `https://example.edu`, the
top-level folder `/data/`, and a user wants to upload the file
`science.json`. Here are the results from various inputs to destination:

| Filename | Destination | Result |
| `science.json` | `/data/`                                                                     | File at `https://example.edu/data/science.json`      |
| `science.json` | `/data/isawesome.json`                                         | File at `https://example.edu/data/isawesome.json`  |
| `science.json` | `data/`                                                                       | File at `https://example.edu/data/science.json`      |
| `science.json` | `data/isawesome.json`                                           | File at `https://example.edu/data/isawesome.json`  |
| `science.json` | `https://example.edu/data/`                               | File at `https://example.edu/data/science.json`      |
| `science.json` |  `https://example.edu/data/isawesome.json`  | File at `https://example.edu/data/science.json`      |
| `science.json` |  `data`                                                                        | Failed.  Tried to PUT file where a folder exists.                      |
| `science.json` | `https://example.edu/data`                                 | Failed.  Tried to PUT file where a folder exists.                      |

## Authentication

### Native App (Default)

By default, this is a Native App, and will prompt the user to login
via Globus and then store their access and refresh  tokens for
reuse. The tokens will be stored in the file
`~/.globus-native-apps.cfg`. As you interact with different
collections, the tokens for each will be add to the file. Removing
this file is effectively logging out.

When doing Native App authentication, the web-browser and local HTTP
server flow can be disabled with the flags `-n` or `no-browser`. This is
necessary when running it on remote systems (i.e., over SSH).

### Confidential Client

The option `-c`, `--client-config`, accepts the name of a JSON file
containing the  client ID and secret of a Confidential Client
specified as `"client_id"` and `"client_secret"`
  
```json
{
	"client_id": "82d3a...",
    "client_secret": "QmUvb..."
}
```

Follow [these
instructions](https://docs.globus.org/api/auth/developer-guide/#register-app)
to register a Confidential Client on the [Globus developer site](https://developers.globus.org)

The client will need permissions to access the collection. You can do
this by [mapping a client identity to a local
account](https://docs.globus.org/globus-connect-server/v5/use-client-credentials/)
or by giving the client access to a folder in a guest collection. In
either case, the client's identity will be
```
<client_id>@clients.auth.globus.org
```
Where `<client_id>` is the UUID of the client.

Example
```shell
$ ./globuscollectionput.py -v -c /etc/secret-sauce.json \
	science.json https://example.edu/data/ \
	6528bad5-bc02-497d-8a4f-a38547d0e72a
Using Confidential Client
Filename: science.json
Destination: https://example.edu/data/science.json
PUT to https://example.edu/data/science.json status 200
```

## Great, What About `GET`, `DELETE`, `OPTIONS`, and `HEAD`?

This is intended to show HTTPS can be used with Globus collections,
beyond the important but simple case of enabling public access. If
people find it useful or there's a project that needs it, adding the
other methods is a possibility.

And if it helps, this might make its way to PyPI.

Pull requests welcome.

## Getting Help

Help is accessed using `--help`.

```
$ ./globuscollectionput.py --help
Usage: globuscollectionput.py [OPTIONS] FILENAME DESTINATION COLLECTION_ID

  Use HTTPS to PUT a file to a Globus Collection

  FILENAME is the local file to be uploaded, data.json

  DESTINATION path (/foo) or URL (https://example.org/foo)

  COLLECTION_ID UUID of the Globus Collection

  By default, this is a Native App, and will prompt the user to login via
  Globus and then store their access and refresh tokens for reuse. You may
  also provide the path to a JSON file containing the client ID and secret
  of a Confidential Client specified as 'client_id' and 'client_secret':
  
  {
     "client_id": "82d3a...",
     "client_secret": "QmUvb..."
  }

  Follow these instructions to create a Confidential Client:
  https://docs.globus.org/api/auth/developer-guide/#register-app

  If DESTINATION is a path, it will be appended to the base URL of the
  collection to make the destination URL.

  If the destination ends with a slash, e.g. /foo/ or https://example.edu/foo/
  the file will be uploaded with its base name, like
  https://example.edu/foo/data.json. Otherwise, the DESTINATION will be the
  path of the file on the collection after upload.

Options:
  -n, --no-browser          Do not use the local server and do not try to open
                            browser. Use this when running remote (e.g., over
                            SSH).
  -c, --client-config TEXT  Confidential Client configuration file.
  -v, --verbose             Print more information.
  --help                    Show this message and exit.```
```
