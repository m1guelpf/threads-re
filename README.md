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

## Mobile Apps

### Authentication

> **Warning**
> This endpoint currently only works for accounts without 2FA enabled.

The mobile apps use Meta's Bloks framework ([originally built for Instagram Lite](https://thenewstack.io/instagram-lite-is-no-longer-a-progressive-web-app-now-a-native-app-built-with-bloks/)) for authentication.

The bloks versioning ID for threads is `00ba6fa565c3c707243ad976fa30a071a625f2a3d158d9412091176fe35027d8`. Bloks also requires you to provide a device id (of shape `ios-RANDOM` | `android-RANDOM`, `RANDOM` being a random set of 13 chars).

```bash
curl --request POST \
  --url 'https://i.instagram.com/api/v1/bloks/apps/com.bloks.www.bloks.caa.login.async.send_login_request/' \
  --header 'user-agent: Barcelona 289.0.0.77.109 Android' \
  --header 'sec-fetch-site: same-origin' \
  --header 'content-type: application/x-www-form-urlencoded; charset=UTF-8' \
  --data 'params={"client_input_params":{"password":"$PASSWORD","contact_point":"$USERNAME","device_id":"$DEVICE_ID"},"server_params":{"credential_type":"password","device_id":"$DEVICE_ID"}}' \
  --data 'bloks_versioning_id=00ba6fa565c3c707243ad976fa30a071a625f2a3d158d9412091176fe35027d8'
```

This request returns a big JSON payload. Your token will be immediately after the string `Bearer IGT:2:`, and should be 160 characters long.

### Creating a text post

```bash
curl --request POST \
  --url 'https://i.instagram.com/api/v1/media/configure_text_only_post/' \
  --header 'content-type: application/x-www-form-urlencoded; charset=UTF-8' \
  --header 'user-agent: Barcelona 289.0.0.77.109 Android' \
  --header 'authorization: Bearer IGT:2:$TOKEN' \
  --header 'sec-fetch-site: same-origin' \
  --data 'signed_body=SIGNATURE.{"publish_mode":"text_post","text_post_app_info":"{\"reply_control\":0}","timezone_offset":"0","source_type":"4","_uid":"$USER_ID","device_id":"$DEVICE_ID","caption":"$POST_TEXT","device":{"manufacturer":"OnePlus","model":"ONEPLUS+A3003","android_version":26,"android_release":"8.1.0"}}
```

## Misc

### How to get a profile's id from their username?

Threads uses the same ID system used by Instagram. The best approach to convert from username to id seems to be requesting the user's instagram page (`instagram.com/:username`) and manually parsing the response HTML. For other methods, see [this StackOverflow question](https://stackoverflow.com/questions/11796349/instagram-how-to-get-my-user-id-from-username).
