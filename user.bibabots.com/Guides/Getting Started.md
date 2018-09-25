Getting Started
===

## Security

Most Biba APIs require a Biba JWT.

To get a Biba JWT, you'll need to be authenticate against one of the following
Firebase projects first:

* biba-data-serv
* biba-cloud
* biba-backend

Once you've got the JWT from Firebase, make a request to `Users.Authorize` with
the JWT in the `Authorization` header as `Bearer: <JWT here>`. If you are
succesfully authorized, the service will respond with a Biba JWT.

### Go

Here's an example of how to authenticate in Go:

```go
ctx = metadata.NewOutgoingContext(
	parentCtx,
	metadata.Pairs(
			"authorization", fmt.Sprintf("Bearer %v", jwtFromFirebase),
		),
	)

resp, err := usersClient.Authorize(ctx, &empty.Empty{})
if err != nil {
	log.Fatalf("unable to authorize user: %v", err)
}

// resp.Jwt has the Biba JWT
```

