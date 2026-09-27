# hodri.app

The public pages of Hodri: landing page, privacy policy and the account deletion page.
Static HTML, no build step. Published from the app's private repository; edits made here are overwritten.

## If the site will not load over HTTPS

`.app` is on the HSTS preload list, so a browser refuses the site outright when the certificate is
wrong. There is no falling back to plain HTTP, and `/privacy/` and `/delete-account/` are the two
URLs Google Play's forms point at.

Check what is actually served:

```bash
echo | openssl s_client -connect hodri.app:443 -servername hodri.app 2>/dev/null | openssl x509 -noout -subject
```

`CN=hodri.app` is healthy. `CN=*.github.io` means GitHub has no certificate for the domain.

On **2026-09-23**, hours after the DNS cutover, every Pages edge began answering with `*.github.io`
and the Pages API stopped reporting a certificate at all. DNS was correct, the `CNAME` file was
correct, the Pages setting still read `hodri.app`, and GitHub reported no incident.

Two things did **not** fix it: waiting, over four hours of it, and re-saving the same custom domain.

What fixed it was clearing the custom domain and adding it back. Open **Settings → Pages** in the
`hodri-site` repository, clear the custom domain, save, then type `hodri.app` back in and save again.
The certificate object appeared at once as `authorization_pending` and was issued within a minute.
The published `CNAME` file survives the round trip. **Enforce HTTPS** can only be ticked once issuing
finishes, and the whole state is visible in `gh api repos/burakkarabekir/hodri-site/pages`: no
`https_certificate` at all means nothing is being issued, and that is the signal to re-add.
