In scenarios where your Next.js site is public and doesn't have user login functionality, CSRF protection is still applicable for forms and state-changing operations to ensure that the submissions or actions are intentionally made from your site.

Here's how you can implement CSRF protection in such a scenario:

### 1. Install Necessary Packages:

```bash
npm install cookie uuid
```

### 2. Generate and Set CSRF Token for Site Visitors:

When a user visits a particular page or endpoint where you'd like CSRF protection (e.g., a form page), generate a CSRF token and set it as a cookie.

For example, in a server-side function of your page:

```javascript
import { v4 as uuidv4 } from 'uuid';
import cookie from 'cookie';

export async function getServerSideProps(context) {
  const csrfToken = uuidv4();  // Generate a new CSRF token.

  // Set the CSRF token as a cookie.
  context.res.setHeader('Set-Cookie', cookie.serialize('csrfToken', csrfToken, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 3600,
    path: '/',
  }));

  return { props: { csrfToken } };
}
```

### 3. Embed CSRF Token in Forms:

Embed the generated CSRF token in your form as a hidden input. When the form is submitted, this token will be sent along with other form data:

```javascript
function FormPage({ csrfToken }) {
  return (
    <form action="/api/submit" method="post">
      {/* ... other form fields ... */}
      <input type="hidden" name="csrfToken" value={csrfToken} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### 4. Verify CSRF Token on the Server:

When the form is submitted, verify that the CSRF token provided in the request matches the one stored in the user's cookies:

```javascript
import cookie from 'cookie';

export default function handler(req, res) {
  const cookies = cookie.parse(req.headers.cookie || '');
  const clientCsrfToken = req.body.csrfToken;
  
  if (clientCsrfToken !== cookies.csrfToken) {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }

  // Handle the form submission...

  res.status(200).json({ success: true });
}
```

### Considerations:

- If you're concerned about the overhead of generating a CSRF token for every visitor, you might decide to generate one only when a user interacts with a certain feature (e.g., starts filling out a form).

- Always use HTTPS for your site. Even without user login, ensuring communication is encrypted is a best practice, especially when implementing security measures like CSRF protection.

- Remember, CSRF protection is meant to protect against a specific type of attack where malicious sites trick users into making unwanted requests to your site. It's just one part of a broader web security strategy.
