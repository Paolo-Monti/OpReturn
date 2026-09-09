# OP_RETURN Mailbox Reader

OP_RETURN Mailbox Reader is a Windows console application that finds and reads
public `OP_RETURN` data stored in Bitcoin transactions associated with an
address.

The application is read-only. It does not create transactions, send Bitcoin,
or require a wallet, seed phrase, or private key.

Copyright: Paolo Monti © 2026

## Main features

- Retrieves confirmed transactions and transactions still in the mempool.
- Uses Mempool by default and Blockstream as a fallback provider.
- Supports API-key authentication for custom compatible services.
- Supports HTTP and HTTPS proxies, including authenticated proxies.
- Reads all available transaction history, page by page.
- Finds every `OP_RETURN` output in the retrieved transactions.
- Interprets UTF-8 Unicode text, accented characters, and emoji.
- Converts `\n`, `\r\n`, and `\u000A` sequences into actual line breaks.
- Displays non-textual data in hexadecimal form.
- Shows the block height inside square brackets.
- Filters messages by one height, a comma-separated list, or height ranges.
- Classifies how each transaction is related to the requested address.
- Exports decoded records to JSON and CSV files.
- Verifies complete OpenPGP messages through GnuPG when requested.

## Requirements

- 64-bit Windows.
- An Internet connection for online queries.
- GnuPG is optional and is required only for `--verify-pgp`.
- An API key is not required for the default Mempool and Blockstream providers.

## Installation

Download `OpReturn.exe`, place it in a folder of your choice, and open Command
Prompt or PowerShell in that folder.

## Quick start

The Bitcoin address is mandatory and must always be the first argument:

```text
OpReturn.exe bc1q...
```

Display detailed information:

```text
OpReturn.exe bc1q... -v
```

Display hexadecimal payloads:

```text
OpReturn.exe bc1q... -x
```

Display messages from selected block heights:

```text
OpReturn.exe bc1q... -l 966087,966091-966100
```

Paginate long output with the Windows `more` command:

```text
OpReturn.exe bc1q... | more
```

Press `q` to stop displaying results. Closing the pager is treated as a normal
interruption and does not produce an I/O error.

Export decoded messages:

```text
OpReturn.exe bc1q... -j mailbox.json -c mailbox.csv
```

Export filenames must not already exist. The application does not overwrite
existing export files.

## Options

```text
-p, --provider auto|mempool|blockstream
                                  Select the Bitcoin data provider
-u, --api HTTPS_URL               Use a custom compatible API
-k, --api-key KEY                 Supply an API key directly
-e, --api-key-env NAME            Read the API key from an environment variable
-w, --api-key-header NAME         Set the authentication header name
-b, --api-key-prefix TEXT         Set the authentication value prefix
-q, --proxy URL                   Use an HTTP or HTTPS proxy
-a, --proxy-user USER             Set the proxy user name
-o, --proxy-password-env NAME     Read the proxy password from an environment variable
-t, --tx TXID                     Read one associated transaction
-f, --file transactions.json     Read transactions from an offline JSON file
-x, --hex                         Display the hexadecimal payload
-v, --verbose                     Display TXID, output index, and progress
-s, --verify-pgp                  Verify complete OpenPGP messages
-g, --gpg PATH                    Specify the path to gpg.exe
-j, --save-json FILE              Export decoded records to JSON
-c, --save-csv FILE               Export decoded records to CSV
-m, --max-pages N                 Limit history pages; 0 means all pages
-l, --height SPEC                 Filter heights, for example 100,105-110
-n, --no-color                    Disable console colors
-r, --self-test                   Run internal offline tests
-h, --help                        Display help
```

`-h` and `--help` display the same help. Every short alias has the same behavior
as its corresponding long option. Help and self-test are the only operations
that do not require a Bitcoin address.
Do not use `--address`; provide the address directly as the first argument.

## Custom API authentication

The default Mempool and Blockstream providers do not require an API key. API
authentication is available only with a custom HTTPS endpoint selected through
`--api` or `-u`.

Reading the key from an environment variable is recommended because a key
written directly on the command line may remain visible in command history or
in the process list:

```text
set OPRETURN_API_KEY=your-key
OpReturn.exe bc1q... -u https://api.example.com/api -e OPRETURN_API_KEY
```

The default request header is `Authorization: Bearer your-key`. Services that
expect a raw custom header can be selected by specifying its name; when the
header name is changed and no prefix is supplied, the key is sent without a
prefix:

```text
OpReturn.exe bc1q... -u https://api.example.com/api -e OPRETURN_API_KEY -w X-API-Key
```

For another authorization scheme, set the prefix explicitly:

```text
OpReturn.exe bc1q... -u https://api.example.com/api -e OPRETURN_API_KEY -b Token
```

The key is never written to normal or verbose application output.

## Proxy configuration

Use `--proxy` or `-q` with an HTTP or HTTPS proxy URL:

```text
OpReturn.exe bc1q... -q http://127.0.0.1:8080
```

For an authenticated proxy, pass the user name and read the password from an
environment variable:

```text
set OPRETURN_PROXY_PASSWORD=your-password
OpReturn.exe bc1q... -q http://proxy.example.com:8080 -a proxy-user -o OPRETURN_PROXY_PASSWORD
```

Credentials embedded in the proxy URL are rejected. The proxy password is
never written to application output.

## Understanding the output

Example:

```text
[966087]    [no clear-sign] [to-address] All messages will be in plaintext.
```

- `[966087]` is the height of the block containing the transaction.
- `[pending]` indicates a transaction that is still in the mempool.
- `to-address` means that the transaction sends funds to the address.
- `from-address` means that it spends funds associated with the address.
- `self/change` means that the address appears in both inputs and outputs.
- `associated` indicates a relationship without a more precise direction.
- `no clear-sign` means that no clear-signed OpenPGP message was recognized.

Verbose mode also displays the transaction ID and the output index. The `--hex`
option displays the original payload bytes in hexadecimal form.

## Filtering by block height

Use `--height` or `-l` to display and export only messages confirmed at selected
block heights. The option accepts:

- One height: `-l 966087`
- Separate heights: `-l 966087,966091,966100`
- An inclusive range: `-l 966087-966100`
- A combination: `-l 966087,966091-966100,966150`

When this filter is active, pending mempool messages are excluded because they
do not yet have a block height. Online address mode still reads the available
history before applying the filter. A `--max-pages` limit can therefore exclude
older matching blocks from the retrieved data.

## Offline JSON files

The file passed to `--file` must contain one Bitcoin transaction object or an
array of transaction objects in the format returned by a compatible Bitcoin
explorer API.

The JSON generated by `--save-json` contains decoded records and cannot be used
as the input of `--file`.

## OpenPGP verification

To request signature verification:

```text
OpReturn.exe bc1q... -s -g "C:\Program Files\GnuPG\bin\gpg.exe"
```

Verification works when one complete clear-signed OpenPGP message is stored in
the same payload and its public key is already available in the local GnuPG
keyring. The application does not download or import keys automatically.

`PGP OK` confirms that the signature is cryptographically valid. It does not,
by itself, establish the real-world identity of the person controlling the key.

## Attribution and data limitations

A message found in a transaction associated with an address was not necessarily
written by the owner of that address. Anyone can send a transaction to a public
address and include an `OP_RETURN` output.

The application obtains information from public Bitcoin explorer services.
Transaction history may change during a query because of new transactions or a
blockchain reorganization. Mempool results are also subject to the provider's
maximum response size.

## Exit codes

- `0`: operation completed successfully.
- `1`: argument, network, file, or processing error.
- `2`: incomplete results or decoding warnings.

## License

The application may be used free of charge for personal, educational, and
non-commercial research purposes.

Commercial use is prohibited. The executable files may not be modified.
Redistribution requires prior written authorization from Paolo Monti.

Read [LICENSE.txt](LICENSE.txt) before using or distributing the application.
