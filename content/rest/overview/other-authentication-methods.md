---
title: Other authentication methods
intro: You can use basic authentication for testing in a non-production environment.
redirect_from:
  - /v3/auth
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
topics:
  - API
shortTitle: Other authentication methods
---


{% ifversion fpt or ghes or ghec %}
While the API provides multiple methods for authentication, we strongly
recommend using [OAuth](/apps/building-integrations/setting-up-and-registering-oauth-apps/) for production applications. The other
methods provided are intended to be used for scripts or testing (i.e., cases
where full OAuth would be overkill). Third party applications that rely on
{% data variables.product.product_name %} for authentication should not ask for or collect {% data variables.product.product_name %} credentials.
Instead, they should use the [OAuth web flow](/apps/building-oauth-apps/authorizing-oauth-apps/).

{% endif %}

{% ifversion ghae %}

To authenticate we recommend using [OAuth](/apps/building-integrations/setting-up-and-registering-oauth-apps/) tokens, such a {% data variables.product.pat_generic %} through the [OAuth web flow](/apps/building-oauth-apps/authorizing-oauth-apps/).

{% endif %}

## Basic Authentication

The API supports Basic Authentication as defined in
[RFC2617](http://www.ietf.org/rfc/rfc2617.txt) with a few slight differences.
The main difference is that the RFC requires unauthenticated requests to be
answered with `401 Unauthorized` responses. In many places, this would disclose
the existence of user data. Instead, the {% ifversion fpt or ghec %}{% data variables.product.prodname_dotcom %}{% else %}{% data variables.product.product_name %}{% endif %} API responds with `404 Not Found`.
This may cause problems for HTTP libraries that assume a `401 Unauthorized`
response. The solution is to manually craft the `Authorization` header.

### Via {% data variables.product.pat_generic %}s

We recommend you use {% ifversion pat-v2%}{% data variables.product.pat_v2 %}s{% else %}{% data variables.product.pat_generic %}s{% endif %} to authenticate to the GitHub API.

```shell
$ curl -u USERNAME:TOKEN {% data variables.product.api_url_pre %}/user
```

This approach is useful if your tools only support Basic Authentication but you want to take advantage of {% data variables.product.pat_generic %} security features.

{% ifversion not ghae %}
### Via username and password

{% ifversion fpt or ghec %}
{% note %}

**Note:** {% data variables.product.prodname_dotcom %} has discontinued password authentication to the API starting on November 13, 2020 for all {% data variables.product.prodname_dotcom_the_website %} accounts, including those on a {% data variables.product.prodname_free_user %}, {% data variables.product.prodname_pro %}, {% data variables.product.prodname_team %}, or {% data variables.product.prodname_ghe_cloud %} plan. You must now authenticate to the {% ifversion fpt or ghec %}{% data variables.product.prodname_dotcom %}{% else %}{% data variables.product.product_name %}{% endif %} API with an API token, such as an OAuth access token, GitHub App installation access token, or {% data variables.product.pat_generic %}, depending on what you need to do with the token. For more information, see "[Troubleshooting](/rest/overview/troubleshooting#basic-authentication-errors)."
 
{% endnote %}
{% else %}

To use Basic Authentication with the {% data variables.product.product_name %} API, simply send the username and
password associated with the account.

For example, if you're accessing the API via [cURL][curl], the following command
would authenticate you if you replace `<username>` with your {% data variables.product.product_name %} username.
(cURL will prompt you to enter the password.)

```shell
$ curl -u USERNAME {% data variables.product.api_url_pre %}/user
```
If you have two-factor authentication enabled, make sure you understand how to [work with two-factor authentication](/rest/overview/other-authentication-methods#working-with-two-factor-authentication).
{% endif %}
{% endif %}

{% ifversion fpt or ghec %}
### Authenticating for SAML SSO

{% note %}

**Note:** Integrations and OAuth applications that generate tokens on behalf of others are automatically authorized.

{% endnote %}

{% note %}

**Note:** {% data reusables.getting-started.bearer-vs-token %}

{% endnote %}

If you're using the API to access an organization that enforces [SAML SSO][saml-sso] for authentication, you'll need to create a {% data variables.product.pat_generic %} and [authorize the token][allowlist] for that organization. Visit the URL specified in `X-GitHub-SSO` to authorize the token for the organization.

The generated URL is valid for one hour, and then expires. After one hour, you will need to generate another URL.

```shell
$ curl -v -H "Authorization: Bearer TOKEN" {% data variables.product.api_url_pre %}/repos/octodocs-test/test

> X-GitHub-SSO: required; url=https://github.com/orgs/octodocs-test/sso?authorization_request=AZSCKtL4U8yX1H3sCQIVnVgmjmon5fWxks5YrqhJgah0b2tlbl9pZM4EuMz4
{
  "message": "Resource protected by organization SAML enforcement. You must grant your personal token access to this organization.",
  "documentation_url": "https://docs.github.com"
}
```

When requesting data that could come from multiple organizations (for example, [requesting a list of issues created by the user][user-issues]), the `X-GitHub-SSO` header indicates which organizations require you to authorize your {% data variables.product.pat_generic %}:

```shell
$ curl -v -H "Authorization: Bearer TOKEN" {% data variables.product.api_url_pre %}/user/issues

> X-GitHub-SSO: partial-results; organizations=21955855,20582480
```

The value `organizations` is a comma-separated list of organization IDs for organizations require authorization of your {% data variables.product.pat_generic %}.
{% endif %}

{% ifversion fpt or ghes or ghec %}
## Working with two-factor authentication

When you have two-factor authentication enabled, [Basic Authentication](#basic-authentication) for _most_ endpoints in the REST API requires that you use a {% data variables.product.pat_generic %}{% ifversion ghes %} or OAuth token instead of your username and password{% endif %}.

You can generate a new {% data variables.product.pat_generic %} {% ifversion fpt or ghec %}using [{% data variables.product.product_name %} developer settings](https://github.com/settings/tokens/new){% endif %}{% ifversion ghes %} or with the "[Create a new authorization][/rest/reference/oauth-authorizations#create-a-new-authorization]" endpoint in the OAuth Authorizations API to generate a new OAuth token{% endif %}. For more information, see "[Creating a {% data variables.product.pat_generic %} for the command line](/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line)". Then you would use these tokens to [authenticate using OAuth token][oauth-auth] with the {% ifversion fpt or ghec %}{% data variables.product.prodname_dotcom %}{% else %}{% data variables.product.product_name %}{% endif %} API.{% ifversion ghes %} The only time you need to authenticate with your username and password is when you create your OAuth token or use the OAuth Authorizations API.{% endif %}

{% endif %}

{% ifversion ghes %}
### Using the OAuth Authorizations API with two-factor authentication

When you make calls to the OAuth Authorizations API, Basic Authentication requires that you use a one-time password (OTP) and your username and password instead of tokens. When you attempt to authenticate with the OAuth Authorizations API, the server will respond with a `401 Unauthorized` and one of these headers to let you know that you need a two-factor authentication code:

`X-GitHub-OTP: required; SMS` or `X-GitHub-OTP: required; app`.  

This header tells you how your account receives its two-factor authentication codes. Depending how you set up your account, you will either receive your OTP codes via SMS or you will use an application like Google Authenticator or 1Password. For more information, see "[Configuring two-factor authentication](/articles/configuring-two-factor-authentication)." Pass the OTP in the header:

```shell
$ curl --request POST \
  --url https://api.github.com/authorizations \
  --header 'authorization: Basic PASSWORD' \
  --header 'content-type: application/json' \
  --header 'x-github-otp: OTP' \
  --data '{"scopes": ["public_repo"], "note": "test"}'
```
{% endif %}

[curl]: http://curl.haxx.se/
[oauth-auth]: /rest/overview/resources-in-the-rest-api#authentication
[personal-access-tokens]: /articles/creating-a-personal-access-token-for-the-command-line
[saml-sso]: /articles/about-identity-and-access-management-with-saml-single-sign-on
[saml-sso-tokens]: https://github.com/settings/tokens
[allowlist]: /github/authenticating-to-github/authorizing-a-personal-access-token-for-use-with-saml-single-sign-on
[user-issues]: /rest/reference/issues#list-issues-assigned-to-the-authenticated-user
