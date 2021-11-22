# Softduplex

[![Linting and Tests](https://github.com/frzb/softduplex/actions/workflows/checks.yaml/badge.svg?branch=main)](https://github.com/frzb/softduplex/actions/workflows/checks.yaml)

Softduplex is a service that merges two one-sided PDF documents into a into two-sided PDF document.  
It was made to provide the ability for software based duplex document scanning for document scanner devices which not support duplex scan.

## Input

It provides two input directories

- One for the odd pages - the PDF document with the front pages: `./input/odd`
- One for the even pages - the PDF documents with back pages: `./input/even`

Softduplex waits until a PDF document with the front pages is stored in `./input/odd` and it waits until a PDF documents with the back pages is stored in  `./input/even` than it merges both documents to a combined two-sided PDF document.

Both input PDF documents are deleted after the merging into a two-sided PDF document.

## Output

The merged two-sided PDF document is stored under: `./output`

## Requirements

- Docker ≥ 20.10.2

## Recommendations 

- Docker Compose ≥ 1.25.0

## Installation

Clone this Git repository

```
$ git clone git@github.com:frzb/softduplex.git
```

### Configruation

Adapt the settings in the `.env` according to your needs.

#### `UID`

The `UID` has to to align with user that starts the Softduplex Docker container to have the proper file permission for the `./input` and `./output` directories inside the container.

### `REVERSE`

When using ADF and flipping the stack of paper for scanning after scanning the front pages the back pages are scanned in reverse order. 
This has to be taken in consideration when merging the front and back page PDF documents and be configured by setting `REVERSE="true"`.

### Volumes for input and output

You might also adjust the Docker volumes for the input and output directories for your setup.  
Refer to the example setup below for the integration in  setup with Paperless-ng and Samba.

## Start Softduplex

```
$ docker-compose up -d
```

## Example integration 

This compose file reflects the integration of Softduplex with a Paperless-ng document management system for ingesting the merged one-sided documents and a Samba services providing the two SMB drive shares that are configured at the scanner as storage destination for scanning front and back side pages.

## `docker-compose.yml`

```
version: "3.4"
services:
  broker:
    image: redis:6.0
    restart: unless-stopped

  webserver:
    image: jonaswinkler/paperless-ng:latest
    restart: unless-stopped
    depends_on:
      - broker
    ports:
      - 80:8000
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000"]
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - data:/usr/src/paperless/data
      - media:/usr/src/paperless/media
      - /srv/scanner/paperless:/usr/src/paperless/consume
    env_file: docker-compose.env
    environment:
      PAPERLESS_REDIS: redis://broker:6379

  samba:
    image: elswork/samba
    environment:
      TZ: 'Europe/Berlin'
    ports:
      - "139:139"
      - "445:445"
    restart: unless-stopped
    volumes:
      - /srv/scanner/paperless:/share/scanner/paperless
      - /srv/scanner/softduplex_odd:/share/scanner/softduplex_odd
      - /srv/scanner/softduplex_even:/share/scanner/softduplex_even
    command: '-u "1000:1000:scanner:scanner:scanner" -s "Paperless:/share/scanner/paperless:rw:scanner" -s "Paperless_Frontpages:/share/scanner/softduplex_odd:rw:scanner" -s "Paperless_Backpages:/share/scanner/softduplex_even:rw:scanner"'

volumes:
  data:
  media:
```
