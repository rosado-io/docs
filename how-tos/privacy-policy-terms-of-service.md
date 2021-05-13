# Add Privacy Policy & Terms of Service Links

If you wish to add your privacy policy and terms of service links to the sign up page, you can include two special translation keys in translation.json.

The keys are `terms-of-service-link` and `privacy-policy-link`. The values will be used as the `href` of `<a>` HTML tag so they must be valid URL.

For example,

```javascript
{
  "terms-of-service-link": "https://mycompany.com/terms-of-service",
  "privacy-policy-link": "https://mycompany.com/privacy-policy"
}
```

If you wish to hide the whole paragraph, set BOTH to empty string.

