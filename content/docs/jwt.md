---
title: jwt
type: docs
directive: true
plugin: true
link: https://github.com/BTBurke/caddy-jwt
---

This middleware implements an authorization layer based on JSON Web Tokens (JWT). You can learn more about using JWT in your application at <a href="https://jwt.io">jwt.io</a>.

This directive can be specified more than once.

### Basic Syntax

<code class="block"><span class="hl-directive">jwt</span> <span class="hl-arg"><i>path</i></span></code>

*   **path** is the file or directory to protect.

By default every resource under path will be secured using JWT validation.

<mark class="block">**Important:** You must set the secret used to construct your token in an environment variable named `JWT_SECRET`. Otherwise, your tokens will always silently fail validation. Caddy will start without this value set, but it must be present at the time of the request for the signature to be validated.</mark>

### Advanced Syntax

You can optionally use claim information to further control access to your routes. In a `jwt` block you can specify rules to allow or deny access based on the value of a claim.

<code class="block"><span class="hl-directive">jwt</span> {
	<span class="hl-subdirective">path</span>  <i>resource</i>
	<span class="hl-subdirective">allow</span> <i>claim</i> <i>value</i>
	<span class="hl-subdirective">deny</span>  <i>claim</i> <i>value</i>
}</code>

*   **resource** is a path to protect; one per block
*   **claim** is a claim to either allow or deny
*   **value** is the value of the claim to match

The **allow** and **deny** lines can be specified multiple times in the same block.

To authorize access based on a claim, use the `allow` syntax. To deny access, use the `deny` keyword. You can use multiple keywords to achieve complex access rules. If any `allow` access rule returns true, access will be allowed. If a `deny` rule is true, access will be denied. Deny rules will allow any other value for that claim.

### Examples

For example, suppose you have a token with `user: someone` and `role: member`. If you have the following access block:

<code class="block"><span class="hl-directive">jwt</span> {
	<span class="hl-subdirective">path</span>  /protected
	<span class="hl-subdirective">deny</span>  role member
	<span class="hl-subdirective">allow</span> user someone
}</code>


The middleware will deny everyone with `role: member` but will allow the specific user named `someone`. A different user with a `role: admin` or `role: foo` would be allowed because the deny rule will allow anyone that doesn't have role member.

### Ways of passing a token for validation

There are three ways to pass the token for validation: (1) in the Authorization header, (2) as a cookie, and (3) as a URL query parameter.  The middleware will look in those places in the order listed and return 401 if it can't find any token.

<table style="border-spacing: 30px 10px;">
<thead>
	<tr>
		<th>Method</th>
		<th>Format</th>
	</tr>
</thead>
<tbody>
	<tr>
		<td>Authorization Header</td>
		<td style="font-family: monospace;">Authorization: Bearer <i>token</i></td>
	 </tr>
	 <tr>
		 <td>Cookie</td>
		 <td style="font-family: monospace;">"jwt_token": <i>token</i></td>
	 </tr>
	 <tr>
		 <td>URL Query Parameter</td>
		 <td style="font-family: monospace;">/protected?token=<i>token</i></td>
	 </tr>
</tbody>
</table>

### Constructing a valid token

JWTs consist of three parts: header, claims, and signature. To properly construct a JWT, it's recommended that you use a JWT library appropriate for your language. At a minimum, this authorization middleware expects the following fields to be present:

##### Header

```json
{
	"typ": "JWT",
	"alg": any supported algorithm except none
}
```

##### Claims

```json
{
	"exp": any supported algorithm except none
}
```

### Acting on claims in the token

You can of course add extra claims in the claim section. Once the token is validated, the claims you include will be passed as headers to a downstream resource. Since the token has been validated by Caddy, you can be assured that these headers represent valid claims from your token. For example, if you include the following claims in your token:


```json
{
	"user": "test",
	"role": "admin",
	"logins": 10
}
```

The following headers will be added to the request that is proxied to your application:

```html
Token-Claim-User: test
Token-Claim-Role: admin
Token-Claim-Logins: 10
Token: full token string
```

Token claims will always be converted to a string. If you expect your claim to be another type, remember to convert it back before you use it. The full token string is passed as <code>Token</code> so you don't have to extract it from the header, cookie, or URL parameter.

### Caveats

JWT validation depends only on validating the correct signature and that the token is unexpired. You can also set the nbf field to prevent validation before a certain timestamp. Other fields in the specification such as aud, iss, sub, iat, and jti will not affect the validation step.
