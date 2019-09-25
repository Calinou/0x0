# 0x0

**Upload files from local or remote locations to [0x0.st](https://0x0.st/).**

## Installation

The only dependency is [curl](https://curl.haxx.se/), which can be installed
from your distribution's repositories.

### Using [basher](https://github.com/basherpm/basher)

```bash
basher install Calinou/0x0
```

Using this installation method, `0x0` will be immediately available in your PATH.

### Manual installation

```bash
# Download the script
curl -LO https://raw.githubusercontent.com/Calinou/0x0/master/bin/0x0

# Make the script executable
chmod +x 0x0

# Move the script to a location in your PATH
sudo mv 0x0 /usr/local/bin
```

## Usage

```bash
# Upload a local file
0x0 some_file.png

# Upload from an URL (the file won't be fetched locally).
# The URL must start with `http://` or `https://`.
0x0 http://example.com

# Upload from standard input.
# Example usage with a pipe: `tail some_file | 0x0 -`
0x0 -
```

If the upload is successful, the file URL will be printed to standard output.

## License

Copyright © 2018-2019 Hugo Locurcio and contributors

Unless otherwise specified, files in this repository are licensed under
the MIT license; see [LICENSE.md](LICENSE.md) for more information.
