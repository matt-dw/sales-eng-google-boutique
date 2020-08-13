# frontend

## Running locally

Install `dep`:

    brew install dep

Run the following command to restore dependencies to `vendor/` directory:

    dep ensure --vendor-only

Run from the command line:

    go run .
    
  By default, HTTP traffic will be served on localhost:8080