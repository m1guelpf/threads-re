# How Threads Works

This repository contains my notes and discoveries while reverse-engineering Threads app. Feel free to PR if you've found something new, or to build clients with this info (with credit ofc 😉).

## Web (threads.net)

The web version of Threads is currently read-only, so not much can be learned about authentication or posting. It uses Meta's [Relay GraphQL Client](https://relay.dev) to talk to the backend (`threads.net/api/graphql`), which seems to be configured to disallow arbitrary queries. This leaves us limited to the existing queries found in the frontend's source:

> **Note**
> When querying the GraphQL backend, make sure to set an user-agent (seems like anything works here) and set the `x-ig-app-id` header to `238260118697367`.

### Get profile data

> Doc ID: `23996318473300828`
> 
> Variables: `userID` (the user's ID)

```bash
curl --request POST \
  --url https://www.threads.net/api/graphql \
  --header 'user-agent: threads-client' \
  --header 'x-ig-app-id: 238260118697367' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'variables={"userID":"314216"}' \
  --data doc_id=23996318473300828
```

### Get profile posts

> Doc ID: `6232751443445612`
> 
> Variables: `userID` (the user's ID)

```bash
curl --request POST \
  --url https://www.threads.net/api/graphql \
  --header 'user-agent: threads-client' \
  --header 'x-ig-app-id: 238260118697367' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'variables={"userID":"314216"}' \
  --data doc_id=6232751443445612
```

### Get profile replies

> Doc ID: `6307072669391286`
> 
> Variables: `userID` (the user's ID)

```bash
curl --request POST \
  --url https://www.threads.net/api/graphql \
  --header 'user-agent: threads-client' \
  --header 'x-ig-app-id: 238260118697367' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'variables={"userID":"314216"}' \
  --data doc_id=6307072669391286
```

### Get a post

> Doc ID: `5587632691339264`
> 
> Variables: `postID` (the post's ID)

```bash
curl --request POST \
  --url https://www.threads.net/api/graphql \
  --header 'user-agent: threads-client' \
  --header 'x-ig-app-id: 238260118697367' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'variables={"postID":"3138977881796614961"}' \
  --data doc_id=5587632691339264
```

### Get a list of users who liked a post

> Doc ID: `9360915773983802`
> 
> Variables: `mediaID` (the post's ID)

```bash
curl --request POST \
  --url https://www.threads.net/api/graphql \
  --header 'user-agent: threads-client' \
  --header 'x-ig-app-id: 238260118697367' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'variables={"mediaID":"3138977881796614961"}' \
  --data doc_id=9360915773983802
```

## Misc

### How to get a profile's id from their username?

Threads uses the same ID system used by Instagram. The best approach to convert from username to id seems to be requesting the user's instagram page (`instagram.com/:username`) and manually parsing the response HTML. For other methods, see [this StackOverflow question](https://stackoverflow.com/questions/11796349/instagram-how-to-get-my-user-id-from-username).
