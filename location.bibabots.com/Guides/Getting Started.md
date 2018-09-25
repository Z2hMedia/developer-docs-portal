Getting Started
===

## Security

To make a request, users must be authenticated via one of the following:

* Biba JWT
* Biba Service Runner JWT


### Biba JWT

To get a Biba JWT, check the docs for the `Authorize` method in the Users
service.

### Service Runner JWT

To get a service runner JWT, you'll first need a JSON key file.

1. Go to the **service account list** page in [Google Cloud Console](https://console.cloud.google.com/iam-admin/serviceaccounts?project=biba-backbot)
2. Find the service account with name 'service-runner'
3. Click the actions menu
4. Select 'Create key'
5. Leave the key type as 'JSON'
6. Click 'Create', and save the file

Now that you've got a key file, you can get the JWT.

Below is an example of how to get the JWT in Go:

```go
d, err := ioutil.ReadFile("path/to/json/key.json")
if err != nil {
	log.Panic(err)
}

src, err := google.JWTAccessTokenSourceFromJSON(d, "biba-devops")
if err != nil {
	log.Panic(err)
}

tkn, err := src.Token()
if err != nil {
	log.Panic(err)
}

```

To use the token in an outgoing GRPC request, you'll have to create a new
outgoing context like so:

```go
ctx = metadata.NewOutgoingContext(
	parentCtx,
	metadata.Pairs(
			"authorization", fmt.Sprintf("Bearer %v", tkn.AccessToken),
		),
	)
```

You can then use `ctx` as the first parameter when calling GRPC methods and you
should be properly authenticated.
