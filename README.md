# pipelinewise-tap-s3-csv

[![PyPI version](https://badge.fury.io/py/pipelinewise-tap-s3-csv.svg)](https://badge.fury.io/py/pipelinewise-tap-s3-csv)
[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/pipelinewise-tap-s3-csv.svg)](https://pypi.org/project/pipelinewise-tap-s3-csv/)
[![License: MIT](https://img.shields.io/badge/License-GPLv3-yellow.svg)](https://opensource.org/licenses/GPL-3.0)

This is a [Singer](https://singer.io) tap that reads data from files located inside a given S3 bucket and produces JSON-formatted data following the [Singer spec](https://github.com/singer-io/getting-started/blob/master/SPEC.md).

This is a [PipelineWise](https://transferwise.github.io/pipelinewise) compatible tap connector.

## How to use it

The recommended method of running this tap is to use it from [PipelineWise](https://transferwise.github.io/pipelinewise). When running it from PipelineWise you don't need to configure this tap with JSON files and most of things are automated. Please check the related documentation at [Tap S3 CSV](https://transferwise.github.io/pipelinewise/connectors/taps/s3_csv.html)

If you want to run this [Singer Tap](https://singer.io) independently please read further.

### Install and Run

First, make sure Python 3 is installed on your system or follow these
installation instructions for [Mac](http://docs.python-guide.org/en/latest/starting/install3/osx/) or
[Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-python-3-and-set-up-a-local-programming-environment-on-ubuntu-16-04).

It's recommended to use a virtualenv:

```bash
  python3 -m venv venv
  pip install pipelinewise-tap-s3-csv
```

or

```bash
  python3 -m venv venv
  . venv/bin/activate
  pip install --upgrade pip
  pip install .
```

### Configuration

Here is an example of basic config, that's using the default Profile based authentication:

    ```json
    {
        "start_date": "2000-01-01T00:00:00Z",
        "bucket": "tradesignals-crawler",
        "warning_if_no_files": false,
        "table_suffix": "_extract1",
        "s3_proxies": {"http": "http://mickeymouse.com:3128","https": "http://mickeymouse.com:3128"},
        "set_empty_values_null": true,
        "tables": [{
            "search_prefix": "feeds",
            "search_pattern": ".csv",
            "table_name": "my_table",
            "key_properties": ["id"],
            "delimiter": ",",
            "string_overrides": ["field_1","field_2"],
            "remove_character": "\""
        }]
    }
    ```

### Profile based authentication

Profile based authentication used by default using the `default` profile. To use another profile set `aws_profile` parameter in `config.json` or set the `AWS_PROFILE` environment variable.

### Non-Profile based authentication

For non-profile based authentication set `aws_access_key_id` , `aws_secret_access_key` and optionally the `aws_session_token` parameter in the `config.json`. Alternatively you can define them out of `config.json` by setting `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` and `AWS_SESSION_TOKEN` environment variables.


 A bit of a run down on each of the properties:

- **aws_profile**: AWS Profile name for Profile based authentication. If not provided, `AWS_PROFILE` environment variable will be used.
- **aws_access_key_id**: AWS access key ID for Non-Profile based authentication. If not provided, `AWS_ACCESS_KEY_ID` environment variable will be used.
- **aws_secret_access_key**: AWS secret access key for Non-Profile based authentication. If not provided, `AWS_SECRET_ACCESS_KEY` environment variable will be used.
- **aws_session_token**: AWS session token for Non-Profile based authentication. If not provided, `AWS_SESSION_TOKEN` environment variable will be used.
- **aws_endpoint_url**: (Optional): The complete URL to use for the constructed client. Normally, botocore will automatically construct the appropriate URL to use when communicating with a service. You can specify a complete URL (including the "http/https" scheme) to override this behavior. For example https://nyc3.digitaloceanspaces.com
- **start_date**: This is the datetime that the tap will use to look for newly updated or created files, based on the modified timestamp of the file.
- **bucket**: The name of the bucket to search for files under.
- **warning_if_no_files**: (Optional): Will attempt to log a warning rather than error if there are no files found for the search criteria if the setting is set to `true`.
- **table_suffix**: (Optional): If set will append a suffix on each of the tables to provide some uniqueness e.g. a date or supplier identifier.
- **s3_proxies**: (Optional): A dict of proxies settings for use of a proxy server. Set to {} to avoid using a proxy server for s3 traffic.
- **set_empty_values_null**: (Optional): When set true will emit `null` (the JSON equivalent of None) instead of an empty string.
- **tables**: JSON object that the tap will use to search for files, and emit records as "tables" from those files.

The `table` field consists of one or more objects, that describe how to find files and emit records. A more detailed (and unescaped) example below:

```
[
    {
        "search_prefix": "exports"
        "search_pattern": "my_table\\/.*\\.csv",
        "table_name": "my_table",
        "key_properties": ["id"],
        "date_overrides": ["created_at"],
        "delimiter": ",",
        "string_overrides": ["field_1","field_2"],
        "remove_character": "\""
    },
    ...
]
```

- **search_prefix**: This is a prefix to apply after the bucket, but before the file search pattern, to allow you to find files in "directories" below the bucket.
- **search_pattern**: This is an escaped regular expression that the tap will use to find files in the bucket + prefix. It's a bit strange, since this is an escaped string inside of an escaped string, any backslashes in the RegEx will need to be double-escaped.
- **table_name**: This value is a string of your choosing, and will be used to name the stream that records are emitted under for files matching content.
- **key_properties**: These are the "primary keys" of the CSV files, to be used by the target for deduplication and primary key definitions downstream in the destination.
- **date_overrides**: Specifies field names in the files that are supposed to be parsed as a datetime. The tap doesn't attempt to automatically determine if a field is a datetime, so this will make it explicit in the discovered schema.
- **delimiter**: This allows you to specify a custom delimiter, such as `\t` or `|`, if that applies to your files.
- **string_overrides**: Specifies field names in the files that should be parsed as a string regardless of what was discovered.
- **guess_types**: (default `True`) By default, column data types will be determined via scanning the first file in a table_spec. Set this to `False` to disable this and set all columns to `string`.
- **remove_character**: Specifies a character which can be removed from each line in the the file e.g. `"\""` will remove all double-quotes.
- **encoding**: The encoding to use to read these files from [codecs -> Standard Encodings](https://docs.python.org/3/library/codecs.html#standard-encodings)

A sample configuration is available inside [config.sample.json](config.sample.json)

### To run tests:

1. Create tests within the `tests/` directory and
then run:

```
  make venv
```

Followed by a run of unit tests:
```
  make unit_tests
```

### Continuous Integration
2. Run through the full suite of tests and linters by running

```bash
poetry run tox
```

These must pass in order for PR's to be merged.

3. To run integration tests:

Integration tests require a valid S3 bucket and credentials should be passed as environment variables, this project uses Minio server.

First, start a Minio server docker container:
```shell
mkdir -p ./minio/data/awesome_bucket
UID=$(id -u) GID=$(id -g) docker-compose up -d
```

Run integration tests:
```shell
  make integration_tests
```

### To run pylint:

1. Install python dependencies and run python linter
```
  make venv pylint
```

---
