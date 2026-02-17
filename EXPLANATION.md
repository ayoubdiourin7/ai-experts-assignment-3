## What was the bug?
`Client.request(..., api=True)` failed to recover from invalid token state.  
When `oauth2_token` was a `dict` (not an `OAuth2Token`), the client skipped refresh and sent the request without an `Authorization` header.

## Why did it happen?
The refresh condition only ran when:
- token was missing (`None`), or
- token was an `OAuth2Token` and expired.

So a non-`OAuth2Token` value like a `dict` did not trigger refresh.  
Header assignment also only happened for `OAuth2Token`, which produced an unauthenticated API request.

## Why does my fix solve it?
The condition now refreshes whenever the token is not an `OAuth2Token` or when it is expired.  
That guarantees invalid token shapes are replaced with a valid fresh token before sending the request, and the `Authorization` header is then attached correctly.

Tests now verify:
- valid token keeps auth header,
- missing token refreshes,
- dict token refreshes,
- expired token refreshes.

## One realistic case / edge case my tests still don’t cover
If `refresh_oauth2()` fails (network/auth provider error), there is no retry or explicit error handling test for that failure path.
