# Transcriptic Runner

The Transcriptic Runner is a command-line tool for interacting with the
Transcriptic API to submit and analyze protocols as well as upload them as packages to Transcriptic's website.

For more information on uploading and packaging releases, see the [Transcriptic Developer Hub](http://developers.transcriptic.com/v1.0/docs/package-creation-quickstart#packaging-and-uploading)


## Installation

```
$ pip install transcriptic
```

or

```
$ git clone https://github.com/transcriptic/runner.git
$ cd runner
$ pip install .
```

to upgrade to the latest version using pip or check whether you're already up to date:
```
$ pip install transcriptic --upgrade
```


## Usage

Access help by typing `$ transcriptic --help` or `$ transcriptic [COMMAND] --help`

## Mandatory first step:
**Log in to your Transcriptic account**
**\*Before using the runner, you'll need to log in to Transcriptic to fetch your
access key information. This will be saved in `~/.transcriptic` for future
commands.**\*

```
$ transcriptic login
Email: sanger@transcriptic.com
Password:
Logged in as sanger@transcriptic.com (cambridge)
```

## The Basics
**Preview Protocol Output**

Previewing a protocol supplies a script with parameters supplied in the "preview" section of a `manifest.json` file.  Read more about this below.
```
$ transcriptic preview MyProtocol
```

**Analyze a Protocol**

To check whether your Autoprotocol is valid using Transcriptic's server-side checker, pipe any script that prints Autoprotocol to STDOUT to `transcriptic analyze`:
```
$ python my_protocol.py | transcriptic analyze
✓ Protocol analyzed
  2 instructions
  1 container
```
alternatively:
```
$ transcriptic preview MyProtocol | transcriptic analyze
```

**Submit a Protocol to Transcriptic**
```
$ python my_protocol.py | transcriptic submit --project "sequencing" --title "Sequencing run"
Run created: https://secure.transcriptic.com/cambridge/sequencing/r1xa043277aekj
```

**Submit a Protocol to Transcriptic in Test Mode**
```
$ python my_protocol.py | transcriptic submit --project "sequencing" --title "Sequencing run" --test
```

**Translate a Protocol to English**

Pipe any valid autoprotocol to `transcriptic summarize` to get a summary of each step
```
$ transcriptic preview MyProtocol | transcriptic summarize
```

### Project Management
**List Existing Projects within Your Organization**
```
$ transcriptic projects
```

**Create a New Project**
```
$ transcriptic new-project "Genotype All The Things"
```

### Packaging and Releasing

**Create a New Empty Package**
```
$ trancriptic new-package "my_package" "This is a description for my package of protocols"
```

**List Existing Package Names and ids**
```
$ transcriptic packages
```

**Ititialize a Directory With an empty manifest template**

The init command creates an empty `manifest.json` file with the proper structure within the current directory.  Read below or [here](https://developers.transcriptic.com/v1.0/docs/the-manifest) to find out more about what a manifest does.   Command will prompt to overwrite if your folder already contains a file called `manifest.json`.
```
$ transcriptic init
```

**Compress All Files in Working Directory for Release**
```
$ transcriptic release
```
passing a --name argument allows you to name your release, otherwise it will be named `release_<version from manifest>` automatically

**Compress all files in working directory for release and upload to a specific package**
```
$ transcriptic release my_package
```

**Upload an existing compressed release to a package**
```
$ transcriptic upload release_v1.0.0.zip my_package
```


### Preview a Protocol

The [autoprotocol-python](https://github.com/autoprotocol/autoprotocol-python) library helps you generate Autoprotocol with easy to use functions. [autoprotocol.harness](https://github.com/autoprotocol/autoprotocol-python/blob/master/autoprotocol/harness.py) parses a set of typed input parameters contained in a `manifest.json` file and passes them back to the specified script when you run `transcriptic preview` (see above).  Input types also define protocol browser UI elements on transcriptic's website.

**Example
The example below assumes the following file structure:
```
protocols/
  manifest.json
  requirements.txt
  my_protocols/
    __init__.py
    sample_protocol.py
```

A manifest.json file contains metadata about protocols required when uploading a package to Transcriptic. A package can contain many protocols but for our example it will contain just one.  The `"inputs"` stanza defines expected parameter types which translate into the proper UI elements for that type when you upload the package to Transcriptic.  Read more about the manifest file [here](http://developers.transcriptic.com/v1.0/docs/the-manifest).  The preview section serves to provide your script with hard-coded parameters and refs for local testing:
```json
{
  "version": "1.0.0",
  "format": "python",
  "license": "MIT",
  "protocols": [
    {
      "name": "SampleProtocol",
      "command_string": "python -m my_protocols.sample_protocol",
      "description": "this is a sample protocol",
      "inputs": {
        "source_sample": {
          "type": "aliquot",
          "description": "A sample source aliquot",
        },
        "dest_sample": {
          "type": "aliquot",
          "description": "A sample destination aliquot"
        },
        "transfer_vol": {
          "type": "volume",
          "description": "Volume to transfer",
          "default": "12:microliter"
        }
      },
      "preview": {
        "refs": {
          "sample_plate": {
            "type": "96-pcr",
            "discard": true
          }
        },
        "parameters": {
          "source_sample": "sample_plate/A1",
          "dest_sample": "sample_plate/A2",
          "transfer_vol": "5:microliter"
        }
      },
      "dependencies": []
    }
  ]
}
```

The following is what your `sample_protocol.py` file would look like.  Note that there is no need to declare a Protocol object within the script or print the protocol to standard out, both of these things are taken care of by `autoprotocol.harness`.  **The `protocol_name` parameter in `autoprotocol.harness.run()` must match the name of that protocol within your manifest.json file**:
```python
def sample_protocol(protocol, params):
  protocol.transfer(params["source_sample"],
                    params["dest_sample"],
                    params["transfer_vol"])

if __name__ == "__main__":
  from autoprotocol.harness import run
  run(sample_protocol, protocol_name="SampleProtocol")
```

**Preview a Protocol's Output on the Command Line:**
```
$ transcriptic preview SampleProtocol
```

**Run a protocol and view its output on the command line by passing it an external .json file with parameters and refs (instead of using your `manifest.json`'s "preview" section)**:
```
$ transcriptic run SampleProtocol protocol_params.json
```

To submit the resulting protocol to transcriptic or analyze it, pipe that result to `transcriptic submit` or `transcriptic analyze` as above.
```
$ transcriptic preview SampleProtocol | transcriptic analyze
```

When you're ready to upload a package to Transcriptic, make sure to include the version of autoprotocol and any other packages you might have used in your `requirements.txt` file:
```
autoprotocol==2.1.0
```

A release consists of everything within the protocols_folder folder **(but do not zip the folder itself: the manifest.json file must be at the top level of the archive.)**

