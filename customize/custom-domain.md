# Custom domain

Set up a custom domain to let your users to access the Authgear pages with your unique, brand-centric domain name. You can use a custom domain \(e.g. `auth.yourdomain.com`\) instead of the Authgear generated domain \(e.g. `<YOUR_APP>.authgearapps.com`\).

{% hint style="info" %}
A paid subscription is required for setting up a custom domain.
{% endhint %}

## 1. Add domain

* Go to **Custom Domain** in your project portal.
* Enter the custom domain name that you would like to connect to Authgear, and click **Add**.
* Your custom domain will appear on the list, click **Verify** to start the verification process.

## 2. Verify domain ownership

* Go to your domain provider's site, add DNS records based on the values shown on the portal page.  
* Click **Verify** after adding the DNS records, you may need to wait for the propagation of your updated DNS records.  

![Setup DNS records and verify](../.gitbook/assets/custom-domain-verification.png)

## 3. Activate your custom domain

* You will return to the custom domain list after verifying your custom domain. Click **Activate** to use your custom domain.
* Now you can access Authgear pages with your custom domain, your default Authgear generated domain \(e.g. `<YOUR_APP>.authgearapps.com`\) cannot be used anymore. **Update your SDK endpoint to use the new custom domain**.
* The certificate of your custom domain is managed by Authgear, you may need to wait for a while for certificate provisioning. 

