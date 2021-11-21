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

The `UID` has to to align with user that start the Softduplex Docker container to have the proper file permission for the `./input` and `./output` directories inside the container.

### `REVERSE`

When using ADF and flipping the stack of paper for scanning after scanning the front pages the back pages are scanned in reverse order. 
This has to be taken in consideration when merging the front and back page PDF documents and be configured by setting `REVERSE="true"`.

## Start Softduplex

```
$ docker-compose up -d
```
