(Due to technical issues, the search service is temporarily unavailable.)

To implement token management (access token and refresh token) using Laravel Sanctum on the client side (e.g., a mobile app or frontend), you need to follow these steps:

---

### **1. Understand the Flow**
- **Access Token**: Short-lived token used to authenticate API requests.
- **Refresh Token**: Long-lived token used to obtain a new access token when it expires.
- When the access token expires, the client uses the refresh token to request a new access token without requiring the user to log in again.

---

### **2. Client-Side Implementation**

#### **Step 1: Login and Store Tokens**
When the user logs in, the server returns an access token and a refresh token. Store these securely (e.g., in secure storage like `AsyncStorage` for React Native or `localStorage` for web apps).

```javascript
// Example: Login request
const login = async (email, password) => {
  try {
    const response = await fetch('https://your-api.com/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ email, password }),
    });

    const data = await response.json();

    if (response.ok) {
      // Store tokens securely
      await SecureStorage.setItem('access_token', data.user.access_token);
      await SecureStorage.setItem('refresh_token', data.user.refresh_token);
    } else {
      throw new Error(data.message || 'Login failed');
    }
  } catch (error) {
    console.error('Login error:', error);
  }
};
```

---

#### **Step 2: Use the Access Token for Authenticated Requests**
Include the access token in the headers of authenticated API requests.

```javascript
const fetchData = async () => {
  try {
    const accessToken = await SecureStorage.getItem('access_token');

    const response = await fetch('https://your-api.com/protected-route', {
      method: 'GET',
      headers: {
        'Authorization': `Bearer ${accessToken}`,
      },
    });

    if (response.ok) {
      const data = await response.json();
      console.log('Data:', data);
    } else if (response.status === 401) {
      // Access token expired, try refreshing it
      await refreshToken();
      await fetchData(); // Retry the request
    } else {
      throw new Error('Request failed');
    }
  } catch (error) {
    console.error('Fetch error:', error);
  }
};
```

---

#### **Step 3: Refresh the Access Token**
When the access token expires, use the refresh token to get a new access token.

```javascript
const refreshToken = async () => {
  try {
    const refreshToken = await SecureStorage.getItem('refresh_token');

    const response = await fetch('https://your-api.com/refresh-token', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ refresh_token: refreshToken }),
    });

    const data = await response.json();

    if (response.ok) {
      // Store the new access token
      await SecureStorage.setItem('access_token', data.access_token);
      await SecureStorage.setItem('refresh_token', data.refresh_token);
    } else {
      // Refresh token is invalid or expired, log the user out
      await logout();
    }
  } catch (error) {
    console.error('Refresh token error:', error);
    await logout();
  }
};
```

---

#### **Step 4: Logout**
Clear the stored tokens when the user logs out.

```javascript
const logout = async () => {
  try {
    // Clear tokens
    await SecureStorage.removeItem('access_token');
    await SecureStorage.removeItem('refresh_token');

    // Optional: Notify the server to invalidate the refresh token
    await fetch('https://your-api.com/logout', {
      method: 'POST',
    });

    console.log('Logged out successfully');
  } catch (error) {
    console.error('Logout error:', error);
  }
};
```

---

### **3. Secure Storage**
- **Web Apps**: Use `localStorage` or `sessionStorage` (not recommended for sensitive data due to XSS vulnerabilities). For better security, use HTTP-only cookies.
- **Mobile Apps**: Use secure storage solutions like:
    - React Native: `@react-native-async-storage/async-storage` with encryption.
    - Flutter: `flutter_secure_storage`.
    - iOS: Keychain.
    - Android: EncryptedSharedPreferences.

---

### **4. Handle Token Expiry Gracefully**
- Check the expiry time of the access token (if provided by the server) and refresh it proactively before making API requests.
- If a request fails with a `401 Unauthorized` error, refresh the token and retry the request.

---

### **5. Example Workflow**
1. User logs in → Store access and refresh tokens.
2. Use the access token for API requests.
3. If the access token expires, use the refresh token to get a new one.
4. If the refresh token expires or is invalid, log the user out.

---

### **6. Backend Setup (Laravel Sanctum)**
Ensure your Laravel backend is configured correctly to issue and refresh tokens. Refer to the [Laravel Sanctum documentation](https://laravel.com/docs/sanctum) for details.

---

By following these steps, you can implement token management on the client side effectively. Let me know if you need further clarification!
