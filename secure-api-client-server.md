To verify that an API call to your Next.js backend originated from your website, you can use a combination of the following techniques:

1. **Check the Referer Header**: The HTTP `Referer` header indicates the address of the webpage that linked to the resource being requested. While this can be a hint, note that headers can be spoofed, so do not solely rely on this.

   ```javascript
   export default function handler(req, res) {
     const referer = req.headers.referer;
     if (!referer || !referer.startsWith('https://yourwebsite.com')) {
       return res.status(403).json({ error: 'Invalid referer' });
     }
     // ... handle the API call
   }
   ```

2. **Use CSRF Tokens**: Cross-Site Request Forgery (CSRF) tokens are a robust method to ensure that a request to your API came from a legitimate session on your website.

   - When a user visits your website, generate a unique CSRF token for them and store it in their session.
   - Include this token as a hidden field in forms or as a header in AJAX requests.
   - On the server-side, verify the token attached to incoming requests against the token stored in the user's session.

3. **SameSite Cookies**: If you're using cookies for session management, you can use the `SameSite` attribute, which ensures that the cookie is only sent in requests originating from the same site. This helps in preventing CSRF attacks.

4. **Content Security Policy (CSP)**: You can use the CSP header to restrict where your site's content can be loaded from. For API calls, a strict CSP would not directly prevent calls from another domain but would prevent malicious sites from loading your pages or scripts that make these API calls.

5. **Origin Header Check**: Similar to the Referer header, the `Origin` header can be used to determine where a request came from. Most modern browsers set this header during cross-site requests.

   ```javascript
   export default function handler(req, res) {
     const origin = req.headers.origin;
     if (origin !== 'https://yourwebsite.com') {
       return res.status(403).json({ error: 'Invalid origin' });
     }
     // ... handle the API call
   }
   ```

6. **Rate Limiting**: Consider adding rate limiting to your API endpoints. While this doesn't guarantee a request is coming from your website, it can prevent abuse by limiting the number of requests an IP can make in a given time period.

7. **Secure and HttpOnly Cookies**: If you're using cookies for session management or tracking, ensure they're marked as `Secure` (so they're only transmitted over HTTPS) and `HttpOnly` (so they can't be accessed via JavaScript). This provides another layer of security against certain types of attacks, like cross-site scripting (XSS).

Remember that no single method is foolproof, and layering multiple methods of security is often the best approach.
